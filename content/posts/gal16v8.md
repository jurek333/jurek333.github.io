---
title: "GAL16V8 - logika"
date: 2020-05-30T13:05:13+02:00
draft: false
dziedziny:
  - Z80PC
zagadnienia:
  - Retro
  - FPGA
---

Procesor Z80 komunikując się z&nbsp;zewnętrznymi urządzeniami ustawia numer urządzenia na&nbsp;dolnym bajcie adresu, a&nbsp;zainteresowane urządzenie powinno na&nbsp;tej podstawie zacząć działać. Chcąc wprowadzić zewnętrzne dwa rejestry konfigurujące pracę komputera powinienem przypisać im&nbsp;odpowiednie 8&nbsp;bitowe adresy (np.&nbsp;0x00 i&nbsp;0x01) i&nbsp;w&nbsp;zależności, który będzie ustawiony podczas IOREQ, ten rejestr aktualizować wartościami z&nbsp;szyny danych.
Mógłbym ułożyć kilka bramek logicznych które sprawdziły by&nbsp;czy adres wynosi 0x00 czy 0x01 i&nbsp;z&nbsp;sygnałem IOREQ tworzyć puls odświeżający właściwy rejestr, ale to&nbsp;oznacza dużo miejsca i zasilania dla wielu bramek logicznych. Zamiast tego użyję jednego układu scalonego: Generic Array Logic.

<!--more-->

## Sprzęt
**GAL** ([Generic Array Logic<sup>1</sup>][1]) to układ scalony pozwalający zaprogramować na warstwie sprzętowej strukturę logiczną. Jest następcą **PAL** ([Programmable Array Logic<sup>2</sup>][2]) i poprzednikiem dzisiejszego **FPGA** ([Field-programmable Gate Array<sup>3</sup>][3]). Zasada działania nie jest skomplikowana, ale może pogubić czytelnika dokumentacji. Podstawą jest programowalna matryca, której wynik jest przetwarzany dalej przez - OLMC (Output Logic Macrocell) - programowalną makrokomórkę. Najtrafniej wg mnie pokazuje to fragment układu PALCE16V8 ([Rys. 1](#[palce16v8_partial])). Po lewej stronie widać matrycę. Do każdego wyjścia/OLMC mamy przypisane w tym przypadku 8 poziomych linii (np. 16-23). Do każdej z tych linii możemy podpiąć dowolne wejście lub jego negację oraz dowolne wyjście lub jego negację (lnie pionowe 0-31). Tak ustawione "poziome linie" są przetwarzane przez OLMC wg ustawionej konfiguracji. Zależnie od układu OLMC ma różne możliwości konfiguracji i pracy, programowalna matryca między modelami zazwyczaj różni się ilością wejść, możliwością dopięcia wyniku lub nie, itp.

Przy studiowaniu dokumentacji tego typu układów trzeba naprawdę dobrze przestudiować tryby pracy i operacje wykonywane na OLMC. Często niektóre wyjścia układu nie oferują takich samych możliwości jak pozostałe, OLMC może też pracować w kliku trybach, i zależnie od nich układ może mieć więcej lub mniej wejść/wyjść efektywnych. To naprawdę fascynujące układy. 

![palce16v8_partial](/Z80/Logic/PALCE16V8_partial.png "Rys. 1) Fragment PALCE16V8")

W dalszej części będę używał **GAL16V8A**, układu o 8 wejściach i 8 wyjściach do interpretacji 8b adresu by wskazać właściwy układ/urządzenie. A gdzie 9 bit IOREQ? No właśnie, gdy OLMC jest w trybie "prostym", pin 1-szy i 11-ty zazwyczaj zarezerwowane na CLK i OE, są dostępne jako wejścia ([Rys. 2](#gal16v8_partial)).

![gal16v8_partial](/Z80/Logic/GAL16V8_partial.png "Rys. 2) Fragment GAL16V8 w Simple Mode")
![gal16v8_olmc](/Z80/Logic/GAL16V8_OLMC_sipmle.png "Rys. 3) OLMC w Simple Mode")

Dokumentacja daje pełniejszy opis możliwość. Tutaj skupiłem się tylko na tym co mi potrzebne by wykonać prosty "chip selector". Chcę sprawdzić czy na adresie jest 0x00 czy 0x01 i jeśli jest IOREQ to na wyjściu 12 lub 13 dać wysoki sygnał (mój rejestr Flip-Flop zatrzaskuje się na przejściu z niskiego do wysokiego stanu CLK).

Logikę potrzebnego układu można zapisać jako:

```
Dev0 = !A0 & !A1 & !A2 & !A3 & !A4 & !A5 & !A6 & !A7 & !IOREQ
Dev1 = A0 & !A1 & !A2 & !A3 & !A4 & !A5 & !A6 & !A7 & !IOREQ
```

## Oprogramowanie
Zatem jak zaprogramować ten układ? Ja użyłem programu **WinCupl**, który można pobrać [tutaj][4]. Przy opisie programu znajduje się klucz do programu potrzebny po instalacji programu (program jest darmowy, nie trzeba się też rejestrować, trzeba tylko skopiować i użyć podany *serial number*). Minusem jest fakt, że program dostępny jest tylko na platformę Windows. Posłuży nam on do skompilowania kodu *CUPL* do pliku ***.jed**.

Kod dla scalaka:

```c
Name DEVSELECT ;
Partno B107A05 ;
Date 30/05/20 ;
Revision 01 ;
Designer JUREK ;
Company JUREK 333 ;
Assembly P8BC ;
Location U18 ;
Device g16v8a ;
Format j ;

/** Inputs **/
Pin 1 = IOREQ;
Pin [2..9] = [A0..7]; 

/** Outputs **/
Pin 12 = Dev0; /* Register 1 */
Pin 13 = Dev1; /* Register 2 */

/** Logic Equations **/
Dev0 = !IOREQ & !A0 & !A1 & !A2 & !A3 & !A4 & !A5 & !A6 & !A7;
Dev1 = !IOREQ & A0 & !A1 & !A2 & !A3 & !A4 & !A5 & !A6 & !A7;
```

Pierwsze 10 linijek to informacje nagłówkowe. Szczególną uwagę należy zwrócić na 2 z nich: *Device* i *Format*. Device określa urządzenie które programujemy, wprowadzenie tutaj złej wartości skończy się błędem kompilacji lub niewłaściwym plikiem jed (dla innego scalaka). Format określa plik wynikowy, *j* oznacza, że chcemy na wyjściu mieć plik jed.

W dalszej części definiujemy/nazywamy wejścia i wyjścia, których użyjemy w kodzie. I w końcu w liniach 21-22 logikę, którą chcemy zaimplementować. Podczas kompilacji nasze reguły zostaną wg specyfikacji urządzenia przetłumaczone na odpowiednią konfigurację programowalnej macierzy i makrokomórek. Dzięki temu nie musimy się martwić odcyfrowywaniem które linie łączyć. Dodatkowo kompilator może wprowadzić optymalizacje i minimalizacje kodu.

Po skompilowaniu plik devselect.jed wygląda tak (z dokładnością do przedstawienia znaków niewyświetlanych [STX]/[ETX]):
```
[STX]
CUPL(WM)        5.0a
Device          g16v8as  Library DLIB-h-40-2
Created         Sat May 30 05:44:33 2020
Name            DEVSELECT 
Partno          B107A05 
Revision        01 
Date            30/05/20 
Designer        JUREK 
Company         JUREK 333 
Assembly        P8BC 
Location        U18 
*QP20 
*QF2194 
*G0 
*F0 
*L01536 01101011101110111011101110111011
*L01792 10101011101110111011101110111011
*L02048 00000011010000100011000100110000
*L02080 00110111010000010011000000110101
*L02112 00100000111111001111111111111111
*L02144 11111111111111111111111111111111
*L02176 111111111111111110
*C12D5
*[ETX]8402
```

Używając programatora, który obsługuje GAL16V8 i takiego pliku jed można zaprogramować układ i od tego momentu realizuje on wgraną strukturę logiczną i to z szybkością "sprzętową".

W ten sposób otrzymałem układ który razem z dwoma 8 bitowymi rejestrami 74HC574 połączony z szyną danych i młodszym bajtem adresu stworzy mi dwa zewnętrzne rejestry do sterowania komputerem. Już wiem, że 2b zajmie banking pamięci, może 1b przeznaczę na kontrolowanie zegara systemowego. W sumie da mi to 16 bitów którymi system będzie mógł konfigurować/sterować pracą komputera.

## Linki
 
 - [[1]] [https://pl.wikipedia.org/wiki/GAL_(elektronika)][1]
 - [[2]] [https://pl.wikipedia.org/wiki/Programmable_Array_Logic][2]
 - [[3]] [https://pl.wikipedia.org/wiki/Bezpośrednio_programowalna_macierz_bramek][3]
 - [[4]] [https://www.microchip.com/design-centers/fpgas-and-plds/splds-cplds/pld-design-resources][4]


[1]: https://pl.wikipedia.org/wiki/GAL_(elektronika)
[2]: https://pl.wikipedia.org/wiki/Programmable_Array_Logic
[3]: https://pl.wikipedia.org/wiki/Bezpośrednio_programowalna_macierz_bramek
[4]: https://www.microchip.com/design-centers/fpgas-and-plds/splds-cplds/pld-design-resources