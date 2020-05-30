---
title: "Mini ekran dla Z80"
date: 2020-05-11T19:44:45+02:00
draft: false
dziedziny:
  - Z80PC
zagadnienia:
  - Retro
  - Arduino
---

Retro komputer odprawiający swoją magię w czeluściach scalaków bez wizualnej reprezentacji to nie najefektywniejsza rozrywka. Wykonam więc PoC z Arduino Micro i niewielkiego ekranu OLED *kartę graficzną* i ekran.

<!--more-->

Jak Z80 będzie wyświetlał dane na ekranie? Wymyśliłem sobie, że procesor będzie komunikował się z "kartą graficzną" poprzez port - przyjmijmy na ten moment - **0x80**. Instrukcjami ```out 0x80, <value>``` będę przekazywał wartość *\<value\>* do prowizorycznej karty graficznej, która zadba o wyświetlenie jej na OLEDzie. 

Od strony CPU wygląda to tak, że Z80 ustawia na dolnym bajcie pamięci adres portu - 0x80, na szynie danych podana wartość i ustawia wyjście IOREQ (żądanie operacji I/O) w stan niski. Na ten moment nie mam innych urządzeń zatem dla prostoty i przetestowania PoC pominę logikę sprawdzania czy adres naprawdę wynosi 0x80. Zakładam w tym PoC, że każde żądanie I/O dotyczy wyświetlenia znaku na ekranie. Zatem piny Arduino łączę z szyną danych, IOREQ i RST by czyścić ekran przy resecie komputera. Połączenia pokazane są na [Rys. 1](#oled-display).

![oled-display](/Z80/OledDisplay/oled_display.png "Rys. 1) Schemat połączenia wyświetlacza, arduino i komputera")

Komunikacja z oledem odbywa się po protokole I2C (wyprowadzenia SDA i SCL). Arduino posiada bibliotekę i przykład do obsługi takiego wyświetlacza. Należy zaimportować **Adafruit_SSD1306** i **Adafruit_GFX**. W przykładach Adafruit_SSD1306 znajdziemy kod dla wyświetlaczy 128x32 i 128x64 po I2C lub SPI. Przy I2C uwaga na adres urządzenia slave - u mnie 0x3C, niektóre urządzenia mogą mieć chyba 0x3D. Jest to wartość predefiniowana dla urządzenia. Arduino IDE w przykładach posiada *Wire/i2c_scanner.ino*, który wgrany i odpalony wypisze na terminal znalezione urządzenia i ich adresy.

Szyna zajmuje piny 4-6 i 8-12. Pin 7 specjalnie *omijam* i łączę z wyjściem IOREQ z Z80. Na stronach Arduino w dokumentacji funkcji [```attachInterrupt()```][1] możemy znaleźć informację, które piny w którym modelu arduino mogą zostać zastosowane do zaimplementowania zewnętrznego przerwania. Dzięki temu zamiast w pętli głównej sprawdzać cyklicznie IOREQ, mogę podać która funkcja ma się wykonać kiedy przerwanie (w tym wypadku stan niski na pinie 7) się pojawi. Arduino przerwie wtedy wykonanie głównej pętli i wykona kod podanej funkcji. Dzięki temu nie martwię się czy czas na wykonanie głównego kodu będzie na tyle długi bym przegapił jakieś żądanie I/O.

Przyszedł czas na kod dla mikrokontrolera. Obsługę można podzielić na 2 części. Wykonywanie zarejestrowanej obsługi przerwania zewnętrznego gdy na pinie 7-mym pojawi się zbocze opadające. Podczas tego zdarzenia odczytujemy dane z szyny i zapisuje je do globalnego bufora (zmienna ```data```). Jest nim tablica 8 znakowa dla której mamy index ostatniego elementu - przesuwany przez obsługę przerwania i index pierwszego niewyświetlonego elementu przesuwany przez pętlę główną. To pętla główna odpowiada za wyświetlanie znaków na wyświetlaczu. Chcemy by obsługa przerwania była jak najlżejsza, dodatkowo niektóre funkcje nie są dostępne podczas przerwania. Dlatego nadchodzące znaki zapisujemy w buforze i pozwalamy pętli głównej gonić "peleton" pomiędzy przerwaniami.

```c
// oled_display.ino
#include <SPI.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define SCREEN_WIDTH 128 // OLED display width, in pixels
#define SCREEN_HEIGHT 64 // OLED display height, in pixels

// Declaration for an SSD1306 display connected to I2C (SDA, SCL pins)
#define OLED_RESET     -1 // Reset pin # (or -1 if sharing Arduino reset pin)
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

#define D0 4
#define D1 5
#define D2 6
#define IOREQ 7
#define D3 8
#define D4 9
#define D5 10
#define D6 11
#define D7 12

volatile uint8_t data[8];
volatile uint8_t eInd = 0;
uint8_t bInd = 0;

void interruptHandler() {
  uint8_t val = 0;
    for(uint8_t pin = D0; pin <= D7; ++pin) {
      if(pin == IOREQ)
        continue;
      val = (val << 1)|digitalRead(pin);
    }
    data[eInd++] = val;
    if(eInd >= 8)
    eInd = 0;
}

void printData(uint8_t data) {
  display.write(data);
  display.display();
}

void setup() {
  pinMode(IOREQ, INPUT_PULLUP);

  for(uint8_t pin = D0; pin <= D7; ++pin) {
    if(pin == IOREQ)
      continue;
    pinMode(pin, INPUT);
  }
  // SSD1306_SWITCHCAPVCC = generate display voltage from 3.3V internally
  if(!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) { // Address 0x3C for 128x64
    for(;;); // Don't proceed, loop forever
  }
  display.clearDisplay();
  display.display();
  display.setTextSize(1);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0, 0);
  display.cp437(true);

  attachInterrupt(digitalPinToInterrupt(IOREQ), interruptHandler, FALLING);
  display.write("?> ");
  display.display();
}

void loop() {
  if(bInd == eInd) 
    return;
  
  printData(data[bInd++]);
  if(bInd >= 8)
    bInd = 0;
}
```

Układ pozwala mi wyświetlić 1 ekran (brak zaimplementowanego przewijania). Jest trochę zbyt mały bym chciał docelowo go zastosować w mojej retro maszynie, dlatego nie będę go rozwijał. Pozwolił mi zweryfikować, że komputer działa i potrafi wypisać "HELL" ([YT][4]). Docelowo nawet do pierwszej wersji szukam czegoś odrobinę większego: 320x280, może 640x480 i kolorowego, by móc zaimplementować również tryb graficzny.

A jeśli chodzi o konstrukcję "poważnej" karty graficznej obsługującej ekran VGA to polecam fajny filmik Bena Eatera [The world's worst video card?][2] i [World's worst video card? The exciting conclusion][3].


## Linki
 - [[1]] [Arduino.cc: attachInterrupt()][1]
 - [[2]] [Ben Eater: Najgorsza na świecie karta graficzna?][2]
 - [[3]] [Ben Eater: Najgorsza na świecie karta graficzna? Ekscytująca konkluzja.][3]
 - [[4]] [Z80, PoC karty graficznej i bus monitor w działaniu][4]

 [1]: https://www.arduino.cc/reference/en/language/functions/external-interrupts/attachinterrupt/
 [2]: https://www.youtube.com/watch?v=l7rce6IQDWs&t=8s
 [3]: https://www.youtube.com/watch?v=uqY3FMuMuRo
 [4]: https://www.youtube.com/watch?v=2SY2HrHUSgE