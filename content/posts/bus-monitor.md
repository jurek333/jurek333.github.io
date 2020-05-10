---
title: "Bus Monitor"
date: 2020-05-10T09:54:19+02:00
draft: false
keywords: [arduino, mega2560, LCD]
dziedziny:
  - Z80PC
zagadnienia:
  - Arduino
---

Odpalając program na komputerze z Z80, ROM i RAM ciężko stwierdzić czy wszystko chodzi czy też nie. Przydałoby się mieć jakiś podgląd danych/adresów. Idealny byłby analizator stanów logicznych. Ale gdy takowego brak, można odgrzebać Arduino i wykonać sobie urządzenie zastępcze.

<!--more-->

Moje Z80 na ten moment może działać taktowane maksymalnie 4MHz-ami. Acz podczas debugowania będę działał na manualnym zegarze lub kliku hercowym (opartym na NE555). Do analizy stanów nie jest mi zatem na ten moment potrzebny demon szybkości. Użyję do tego Arduino Mega 2560 i LCD 16x2 znakowego. Taktowanie Arduino (16MHz) jest dla mnie wystarczające.

Wersję badającą 8 bitów szyny danych plus sygnał cyklu (M1), żądanie wejścia/wyjścia (IOREQ) oraz reset (RST) przedstawia poniższy schemat ([Rys. 1](#schemat)). Porty Arduino 22-29 połączone będą kolejno z wyjściami szyny danych komputera BUS0-BUS7, M1, IOREQ i RST zajmą porty 30, 31 i 33. Komunikacja i sterowanie LCD obędzie się poprzez wyjścia 48-53.

![schemat](/Z80/BusMonitor/bus_monitor_schemat.png "Rys. 1) Schemat układu")

Co do samego podłączenia LCD i sterowania nim to w sieci dostępnych jest wiele materiałów (np. na stronie [arduino.cc](https://www.arduino.cc/en/Tutorial/HelloWorld)). Mój model można sterować 8 bitami danych jak i 4-ma. Dla oszczędności miejsca wybrałem 4-bitową opcję. Dla Mega 2560 pozwala to na zmieszczenie całej szyny danych (8b) i szyny adresowej (16b) zostawiając 2 porty na 2 inne wejścia. Przy zastosowaniu oszczędniejszego protokołu komunikacji (np. I2C zyskalibyśmy dodatkowe 2 wejścia).

Teraz czas na kod który napędzi ten monitor. Jest on banalnie prosty. Podczas konfigurowania startowego ustawiam odpowiednio piny, inicjuję i czyszczę ekran LCD wypisując jednocześnie domyślny tekst. W pętli zaś odczytuję stan pinów szyny i sygnałów sterujących Z80, przeliczam ilość cykli i wypisuję dane.

```c
// bus_monitor.ino
#include <LiquidCrystal.h>
// LCD
#define COLS 16
#define ROWS 2
#define RS_PIN 52
#define ENABLE_PIN 53
#define D4_PIN 48
#define D5_PIN 49
#define D6_PIN 50
#define D7_PIN 51
// Probes
#define BUS0 22
#define BUS7 29
#define M1 30
#define IOREQ 31
#define RST 33

LiquidCrystal lcd(RS_PIN, ENABLE_PIN, 
    D4_PIN, D5_PIN, D6_PIN, D7_PIN);

int cycle = 0;
int m1 = HIGH;
char buf[8];

void setup()
{
  pinMode(RST, INPUT);
  pinMode(IOREQ, INPUT);
  pinMode(M1, INPUT);
  for(int b=BUS0; b<=BUS7; ++b)
    pinMode(b, INPUT);
    
  lcd.begin(COLS,ROWS);
  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print("bus: ");
}

byte getByte() 
{
  char buff[9];
  byte dataByte = 0;
  for(int b=BUS7; b>=BUS0; --b) {
    dataByte <<= 1;
    dataByte |= digitalRead(b);
  }
  return dataByte;
}

void print2ndLine(int ioreq) 
{
  lcd.setCursor(0, 1);
  if (m1 == HIGH)
    lcd.print("M1 ");
  else
    lcd.print("   ");
  if (ioreq == LOW)
    lcd.print("IO ");
  else
    lcd.print("   ");
  if(cycle == 0)
    lcd.print("        ");
  else
    lcd.print(cycle);
}

void loop()
{
  int rst = digitalRead(RST);
  int ioreq = digitalRead(IOREQ);
  int m1_state = digitalRead(M1);
  itoa(getByte(), buf, 16);

  if(m1_state == HIGH && m1 == LOW)
    ++cycle;
  m1 = m1_state;
  if(rst == HIGH)
    cycle = 0;

  lcd.setCursor(7, 0);
  lcd.print(buf);

  print2ndLine(ioreq);
}
```

Z80 realizuje instrukcje w kilku *cyklach zegarowych*. Pierwszy cykl zegarowy podczas którego CPU odczytuje instrukcję do wykonania jest zawsze sygnalizowany stanem niskim na wyjściu M1 procesora. Ja sobie go odwracam na SN74HC04 dlatego w kodzie w pętli warunkami wykrywam przejście z niskiego stanu na wysoki i tylko wtedy podbijam licznik cykli/instrukcji wykonanych (```cycle```). Jeśli dostanę sygnał resetu (RST) to zeruję ten licznik. IOREQ w procesorze też jest aktywne jeśli ma niski stan. Ten sygnał pozostawiłem nie odwrócony stąd wypisuję na ekran informację że występuje tylko kiedy jest w stanie niskim. 

Oczywiście dobrze jest sprawdzić urządzenie czy działa tak jak powinno. Można wpiąć "sondy" do płytki stykowej i podawać wybrane wartości (0,1) sprawdzając poprawność odczytu. Taki prosty "bus monitor" pozwolił mi podejrzeć zawartość szyny danych podczas wykonywania testowego kodu dla którego znałem wartości jakie spodziewałem się ujrzeć na szynie. I dzięki niemu znalazłem przypadkowo uziemioną nóżkę RAM-u (7-my bit danych miał zawsze wartość 0). Można dzięki niemu też debugować swój program. Podczas M1 CPU pobiera z ROM instrukcję do wykonania, to wtedy z szyny możemy odczytać wykonywaną instrukcję i porównać ze spodziewaną.

Co do przydatności...
Prosty 8 kanałowy 24MHz klon Saleae jest dostępny w okolicach 30-40zł. Zatem zamiast za tę kwotę kupować Arduino można od razu kupić taki klon. Do zabawy z cyfrową elektroniką na pewno się przyda, a oprogramowanie Saleae ma duże możliwości, choćby interpretowanie protokołów szeregowych. Zawsze jednak warto spróbować coś samemu zrobić by wiedzieć jak to działa albo dostosować rozwiązanie pod własne wymagania.