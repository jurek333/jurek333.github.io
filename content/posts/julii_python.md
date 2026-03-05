---
title: "Zbiory Julii"
date: 2026-03-07T12:00:00+01:00
draft: false
tematy:
  - Math
znaczniki:
  - Python
  - Graphics
math: true
image: "/img/fractal-category.png"
---

Fraktale, sztuka w matematyce. Pojęcie to wprowadził Benoît Mandelbrot (ur. 1924-11-20 w Warszawie, zm. 2010-10-14 w Cambridge, MA). Ale takie zbiory były znane już wcześniej. Jednym z nich jest zbiór Julii badany w latach 1918-1920 przez Gastona Julii i Pierre'a Fatou. To zbiór punktów w przestrzeni liczb zespolonych $p \in C$ dla których ciąg wyrażony równaniem rekurencyjnym 

$$ 
\begin{align}
Z_{0} &= p \\\\
Z_{n+1} &= Z_{n}^2 + c 
\end{align}
$$

jest ciągiem ograniczonym. 

<!--more-->

Licba zespolona $c \in C$ jest parametrem zbioru, dyktuje kształt tego zbioru. Co ciekawe wszystkie $c \in C$ dla których zbiór Julli zawiera punkt $p=0$ stanowi zibór Mandelbrotha.

Z definicji zbioru wynika, że tutaj mamy do czynienia z 2-ma wynikami. Punkt należy do zbioru lub nie. Dwa kolory. To skąd te wielokolorowe obrazy?

Kolorujemy przestrzeń nienależącą do zbioru wg tego jak 'szybko' punkt przekracza założoną granicę / wychodzi z 'pasa' (w przykładowym kodzie [-4, 4]):

```python
    def julii(self, p: JuliiPoint):
        global N
        z = self.map_to_complex(p.x, p.y)
        c = self.c
        
        for n in range(N):
            z = z**2 + c
            if abs(z) > 4.:
                return self.pallete.to_rgb(n)
        return self.pallete.to_rgb(N)
```

W praktyce nie sprawdzamy zbieżności 'w nieskończoność', a na przykład w `N` krokach (maksymalna ilość kolorów), jeśli jednak z `N` kolorów w praktyce dostajemy ich znacznie mniej to może trzeba podnieść 'granice sprawdzania zbieżności' (`4.` w przykładzie).

## Linki
 
 - [[1]] [https://pl.wikipedia.org/wiki/Benoît_Mandelbrot][1]
 - [[2]] [https://pl.wikipedia.org/wiki/Zbi%C3%B3r_Julii][2]

[1]: https://pl.wikipedia.org/wiki/Benoît_Mandelbrot
[2]: https://pl.wikipedia.org/wiki/Zbi%C3%B3r_Julii
