---
title: "Jacob's Ladder Engineering"
author_profile: false
header:
  teaser: /assets/images/circuit-diagram.svg
classes: wide
use_math: true
tags:
  - math
---

In my <a href="{{site.url}}/Jacobs-ladder">post on my Jacob's ladder</a>, I wrote mainly about the process of building it and how it works. I also mentioned briefly the way the power supply works, but glossed over the majority of the math and principles behind how it works. Here, I want to go more in-depth into the mechanics of how it works. 

The first thing to note is that I'm basing my calculations on the circuit diagram I showed in the main post:

<img src="/assets/images/circuit-diagram.svg" alt="A circuit diagram of the power supply">

This diagram represents the circuit in a mostly faithful way, but there are some important ways it differs from reality. First, the inductance value for the transformer (which is the igntion coil) is based on observed behavior and online information about ignition coils, as I could not directly measure to inductance nor find that information from the manufacturer. Second, the TRIAC circuit (everything above the TRIAC itself) is a simplified version of the one I used, because the real one has smoothing inductors and capacitors which are not necessary in the idealized model. 

With those disclosures, the way the circuit operates can be discussed in more detail. At time $t=0$, we will assume no components have any stored energy. As the AC waveform starts to rise, the TRIAC is in its nonconducting state, so current cannot flow through that branch. Through the top branch, we have two things in the path of the current: the potentiometer, and the capacitor. For my application, I had the potentiometer set very low, likely in the range of 20 - 30 $k\Omega$. The capacitor's reactance would be $\frac{1}{i \omega C} \approx -35.4i k\Omega$ which along with the potentiometer have total impedance $z = 34.8 k\Omega$ (the impact from the inductor and larger capactiors is negligible, on the order of hundreds of ohms). This means, in the "off" state, the current permitted through has an RMS value of $\frac{120V}{34.8k\Omega} = 3.45mA$. 

In the "on" state, the the bottom branch (with the TRIAC in it) becomes effectively a short, and the current spikes to $\frac{120V}{135\Omega} = 0.889A$. The only final piece we need to discover the voltage that should be produced across the spark gap is how fast this change occurs, which is a property of the TRIAC. This could be found on the manufacturer's website, and was listed at $2\mu s$ (up to all currents this device can handle). Then, the equation for the voltage across the output is:

$$
\begin{align}
V_{out} &= L \frac{di}{dt} K \\
&= 0.0075 \frac{0.886}{0.000002} 115 \\
&= 382 kV
\end{align}
$$

This is, obviously, not what the oberved output voltage is.