---
title: "Automating my home's ventilation system"
url: automating-home-ventilation
date: 2022-04-07T00:00:00+02:00
draft: true
---

Currently, I am living in an apartment in Utrecht with a bathroom in the middle of the building, without any windows.
This means that it's necessary to have a proper ventilation system to avoid mold growing everywhere.
It's also a necessity to turn it on, but guess what I occassionally forgot...
Anyhow, there are ways to solve that problem: _home automation_!

I looked online on how to achieve this and luckily there were a lot of tutorials.
Most of these tutorials required you to solder a home automation device to the ventilation motor.
That was a problem for me, as it was located on the shared roof of the apartment building.
So, I had to think of a system that did not require me to access the ventilation motor itself.

I'm writing this, since I am going to move out to a new house soon! 
In that house we will have a bathroom with a window, so there I will not be needed this system and I would like to document how I did it.
And perhaps there are more people interested in building something similar.

## What did I use?

- A Raspberry PI running Home Assistant - I think I used a Raspberry Pi 2b;
- A [Philio 2-in-1 sensor](https://store.zwavecenter.com/index.php?route=product/product&product_id=380) for measuring the humidity in the bathroom;
- A [Fibaro Single/Double switch](https://manuals.fibaro.com/switch-2/) - to replace the old 3-way switch (similar to [this](https://schakelmateriaal-kopen.nl/product/centraalplaat-3-standen-schakelaar/));
- An [Aeotec Z-Wave Plus Z-Stick](https://www.robbshop.nl/aeotec-z-wave-plus-z-stick-gen5) - to connect everything together wirelessly;
- Some eletrical wires.
