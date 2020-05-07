---
title: "Z80 Personal Computer"
date: 2020-05-06T16:30:27+02:00
draft: false
dziedziny:
  - Z80
zagadnienia:
  - Retro
  - CPU
---

Projekt 8 bitowego komputera osobistego opartego na procesorze Zilog Z80. Procesor ten trafił na rynek w 1976 roku. Jest on kompatybilny z Intelem 8080. Posiada 8-bitową szynę danych i 16-bitową szynę adresową. Najbardziej jest chyba znany z komputerów Sinclair ZX Spectrum i Elwro 800 Junior. Wciąż jest łatwo dostępny jako scalak DIP-40 i świetnie nadaje się do budowy domowego komputera.

<!--more-->

Posiadany przeze mnie model to STMicroelectronics Z8400AB1 Z80A CPU. Zatem mogę go taktować maksymalnie 4MHz (dostępne są modele pozwalające na 20MHz). By móc weryfikować działanie sprzętu ręcznie podając puls zegara i nie martwić się o odświeżanie RAM, zastosuję w projekcie pamięć SRAM. Program wykonywalny będzie wgrany ma EEPROM. Obie pamięci posiadam w wielkościach 8k x 8bit, zatem zacznę od konfiguracji 8k ROM + 16k RAM. Z czasem spróbuję wprowadzić pełne 64k i może "bankowanie pamięci".

W sieci dostępnych jest wiele treści dotyczących składania własnego komputera na Z80. Można też znaleźć książkę Steve Ciarcia "Build Your Own Z80 Computer". Rewelacyjne pozycja z 1981 roku, która może przeprowadzić nas przez cały projekt, albo podrzucić dużo wskazówek i rozwiązań do zastosowania. Jeśli chodzi o zrozumienie działania komputera na poziomie scalaków, świetnie wg mnie wyjaśnia to Ben Eater na swoim vlogu w 2-ch cyklach: [konstrukcji 8-bitowego komputera][1] i [budowaniu komputera opartego na 65C02][2].

## CPU

CPU posiada 40 wyprowadzeń. Poza 16 adresowymi i 8 dla danych jest oczywiście VCC i GND oraz: reset, clock, przerwania niemaskowane (NMI) i maskowane (INT), M1 (bit cyklu maszynowego), refresh (dla obsługi DRAM), wait oraz halt, read, write, rządanie pamięci (MREQ) i operacji wejścia/wyjścia (IOREQ), żądanie kontroli nad szyną i akceptacja oddania tej kontroli (BUSRQ i BUSACK). W wersji podstawowej poza zasilaniem musimy dostarczyć reset i taktowanie. Pozostałe wejścia (NMI, INT, WAIT, BUSRQ) ustawione będą w stan nieaktywny (w tym przypadku wysoki). Z wyjść użyjemy wszystkich wyjść adresowych i danych oraz RD, WR i MREQ.

![CPU_schemat](/Z80/MainBoard/cpu.png "Rys. 1) Schemat wyprowadzeń CPU")
  
Wyjścia danych D0-D7 należy połączyć z odpowiadającymi wyprowadzeniami w pamięciach RAM i ROM. Jako że są to tylko 8k kości to używają tylko bitów A0-A12. A13-A15 przekażę do demultipleksera 3b -> 8 linii, który posłuży mi do realizacji najprostszej logiki adresowania: 0x0000-0x1FFF ROM, 0x2000-0x3FFF - pierwsze 8k RAMu, 0x4000-0x5FFF - drugie 8k RAMu.

To dekodowanie załatwi wybór aktywnego scalaka i powinno się odbywać tylko jeśli CPU sięga do pamięci - zatem MREQ jest w stanie niskim. Dodatkowo do pamięci RAM powinniśmy dostarczyć bit mówiący czy będzie się odbywał odczyt (OE - output enabled) czy zapis (WE - write enabled). Zatem spinam te nóżki z odpowiednio RD i WR na CPU. W moim modelu pamięci RAM by układ był aktywny należy ustawić 2 bity CE1 i CE2. Na ten moment jedno z wejść (aktywowane wysokim stanem) na twardo podpinam poprzez rezystor do +5V (co widać na [Rys. 3](#main-board-shema)).

## Przycisk reset

By Z80 zaczęło wykonywać zadany program od początku (od adresu 0x0000) musi zostać zresetowane. W dokumentacji możemy znaleźć, że wymaga to by reset był w stanie niskim przynajmniej przez 3 cykle zegara. Warto też zadbać o debouncing przycisku reset. Z książki "Build Your Own Z80 Computer" zaczerpnąłem układ spełniający te wymagania:

![reset](/Z80/MainBoard/reset.png "Rys. 2) Schemat przycisku reset")

Podaje on nam reset w 2-ch stanach wysokim dla innych komponentów w przyszłości i niskim dla CPU. Kondensator C5 ładujący się po włączeniu poprzez opornik R5 służy nam do "odmierzenia czasu nadawania" sygnału RST. Dobierając te wartości możemy w razie potrzeby wydłużyć okienko czasowe, w którym po włączeniu zasilania lub puszczeniu przycisku reset układ będzie podawał stan niski - wstrzymywał reset CPU.

## Płyta główna

Całość tego skromnego PoCa zamieszczona jest na poniższym schemacie. Na razie możemy sprawdzić działanie programu tylko monitorując szyny danych i/lub adresu. Następnym krokiem będzie przygotowanie takiego "analizatora logicznego" w oparciu o np. Arduino Mega. Spróbuję wykonać takowy analizator oraz "testową kartę graficzną" przy pomocy której program mógłby wyświetlić np. powitanie na małym ekranie.

![main-board-shema](/Z80/MainBoard/MainBoard_v0.1.png "Rys. 3) Schemat podstawowego komputera")

Należy tutaj również zaznaczyć, że CPU nie ma nieskończonych możliwości jeśli chodzi o zasilenie innych układów poprzez swoje wyjścia. Dokumentacja wspomina o poborze prądu na poziomie 200uA. Na razie operujemy z niewieloma układami. Ale dla większej liczby może się okazać, że CPU nie będzie w stanie zapewnić natężenia. Stąd idąc za radą Stevena Ciarcii może być warto w przyszłości opatrzyć wyjścia w bufory zapewniające dodatkowy prąd w razie potrzeby. 

Warto już powoli myśleć o "wyglądzie" uniwersalnej magistrali dla komputera. Z80 obsługuje urządzenia I/O poprzez "dodatkową" adresację. Zamiast 0 na MREQ daje 0 na IOREQ i używa dolnych 8 bitów adresu do wybrania urządzenia/portu docelowego. Zatem dla peryferiów nie będziemy potrzebować pełnych 16 bitów adresu, a to już poważnie zmniejsza nam wielkość magistrali. Ta wiedza przyda się już podczas tworzenia testowej "karty graficznej".

Z80 jest na tyle wolny, że praktycznie nie musimy się przejmować przeliczaniem czasów obsługi dostępu do pamięci. Ale jeśli komunikowalibyśmy się z jakimś wolniejszym podzespołem, warto pamiętać o wsparciu jakie wtedy Z80 oferuje. Można wstrzymać wykonanie instrukcji przez procesor ustawiając WAIT na 0 do czasu kiedy dane nie będą gotowe/odczytane.

## Linki
 
 - [[1]] [Ben Eater: 8-bitowy procesor][1]
 - [[2]] [Ben Eater: Komputer oparty na 65C02][2]

 [1]: https://www.youtube.com/watch?v=HyznrdDSSGM&list=PLowKtXNTBypGqImE405J2565dvjafglHU
 [2]: https://www.youtube.com/watch?v=LnzuMJLZRdU&list=PLowKtXNTBypFbtuVMUVXNR0z1mu7dp7eH