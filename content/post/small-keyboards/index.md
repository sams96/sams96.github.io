---
title: "Why I Like Small Keyboards"
date: null
draft: true
url: /small-keyboards
build:
  list: never
  render: always
---

TODO intro

<!--more-->

I built my first keyboard in 2014, a 44 key Atreus[^atreus]. I used my school's
laser cutter to cut sheets of clear acrylic for the case, and I hand wired all
the keys and diodes to a Teensy microcontroller and made liberal use of a hot
glue gun. I spent a good while learning the layout, which was made more
difficult by the blank key caps I'd chosen (mostly because they were the
cheapest available). I kept using it for a while but would increasingly switch
back to my 60% board (for _DotA_) until the Atreus just stayed in the drawer.

{{< myfig src="corne.avif" caption="TODO" >}}

In 2020 I learned of the Corne keyboard[^corne], a 42 key split keyboard. I also
got my first software job at around the same time which felt like a good excuse
for a new keyboard, so I ordered some PCBs from China and got to work ripping
apart the old Atreus for its switches and key caps. Both the ergonomics of the
split layout and having less time to play _DotA_ meant the Corne stuck around. I
also find the 3x6 layout and 3 thumb keys much more comfortable than the Atreus’
4x5 layout with one thumb key.

## Okay, but why?

The argument I most commonly see against small keyboards is something along the
lines of “I’m a programmer I need access to all those symbols” or “I use the F
keys all the time”. To me this argument seems backwards, you want the keys you
use the most to be as close to the home row as possible, meaning that you don’t
need to move your entire hand to reach it.

For example, to press the opening bracket <kbd>(</kbd> key, something I’ve been
doing a lot of since I started writing Clojure professionally earlier this year,
on a standard keyboard you hold the shift key with your pinky and then reach two
rows up from the home row to press the <kbd>9</kbd> key. To do the same on my
Corne, I hold the <kbd>lower</kbd> key with my thumb to activate the lower
layer, and then press the <kbd>o</kbd> key just one row away from home.

Or as another example, instead of having to reach three rows up from the home
row, something which for me means moving my entire hand, with my corne I simply
hold the <kbd>raise</kbd> key to activate the upper layer and then I have access
to all of the function keys easily.

The big downside to using a keyboard like this is learning where all these keys
are, particularly the ones you don’t use very often. Having the symbols and
function keys you do use often available on another layer close to the home row
is great, but for the keys you don’t use often its nice to be able to look down
and just see that key, reach over and press it.

[^atreus]: https://github.com/ryantm/atreus
[^corne]: https://github.com/foostan/crkbd
