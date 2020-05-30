---
title: "Z80: Ekran 480x320"
date: 2020-05-17T13:25:31+02:00
draft: true
dziedziny:
  - Z80PC
zagadnienia:
  - Arduino
  - LCD
---

Ekran dla 8-bitowego komputera, jaki powinien być? W *swoich czasach* 8-bitowce pracowały na rozdzilczościach 256x192 (ZXSpectrum), 256x240 (NES), 320x200 (C64). Z tego powodu zdecydowałem się spróbować szczęścia z tanim kolorowym ekranem 480x320 pixeli.

<!--more-->

Wspomniane rozdzielczości dzisiaj charakteryzują raczej wyświetlacze zegarków niż monitory. W soich rozważaniach i poszukiwaniach zwracałem uwagę by wyświetlacz komunikował się równolegle 8-bitami (rozmiar magistrali danych), był w miarę duży przy niskiej rozdzielczości. Te założenia miały mi uprościć budowę *karty graicznej*. Wybrałem 3,5"TFT LCD Shield dla Arduino UNO z 8-bitową komunikacją równoległą. Płytki tego typu sterowane ILI9486L mają dobre wsparcie w bibliotekach Arduino, co się przydaje by stosując *reverse engeenering* odcyfrować jak sterować ekranem i przenieść to sterowanie na układ i kod dla Z80.

Dlaczego chcę wykonać osobny kawałek sprzętu pełniący rolę karty graficznej, zamiast użyć Arduino i shielda jak konstruktor przykazał? Projekt z OLED pozwolił mi zauważyć, że dla 4MHz Z80 potrafi wysyłać dane za szybko. Jeśli mogę mieć grafikę szybszą od procesora, który jej używa, tak by CPU nie musziało czekać na odebranie danych to spróbuję to osiągnąć.