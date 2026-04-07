---
title: "Jacob's Ladder"
tags:
  - maker
---

<div style="width:75%; margin:auto">
<center>
<img src="{{site.url}}/assets/images/top-arcs.jpg" align="center" style="border:none; border-radius: 7px; width:340px float:right;" alt="image">
</center>
<br>
</div>

A Jacob's ladder is a high-voltage electrical device that produces a rising arc between a pair of straight, nearly-vertical electrodes. It's often used in science demos to explain arcing, or in movies when a generic background electrical device is needed. The conditions this device needs to operate are:

 - A power supply which provides at least 10kV (in my case it was around 25kV)
 - Two identical, straight electrodes, which are angled ever-so-slightly away from each other such that the distance between them increases with the height from the base.
 - Relatively still air around the electrodes.
 
The principle of operation is somewhat surprisingly mainly thermodynamic: as the arc heats the air around it, the heated pocket will rise, and the heated air creates a more favorable environment for the arc to exist in, so the arc rises with it. The process continues until the arc either reaches the end of the electrodes, or they get far enough apart that there is no longer enough current to sustain an arc of that length. A new arc will then form at the bottom, where the electrodes are the closest and arcing is most favorable.

This process is highly volatile, and even slight disturbances in electrode spacing and straightness have large impacts in performance. In effect, this meant trying many different angles and spacing to achieve best results, which got the arcs to around 2-2.5 inches before reseting.

<b><u>The power supply</u>:</b>
<br>
<p>The power supply I built for this project was incredibly simple to assemble in practice, but has intriguing underlying theory. The components required were a car ignition coil, a 250v 20uF capacitor, and a TRIAC dimmer switch (which is the most common type of dimmer switch for lights and other appliances). As seen below, the circuit this represents looks quite complex; however, everything except the transformer (which is the ignition coil) and the 20 uF capacitor is part of the dimmer switch and comes prepackaged.

<div style="width:50%; margin:auto">
<img src="{{site.url}}/assets/images/circuit-diagram.svg" align="left" style="border:none; border-radius: 7px;" alt="image">
<img src="{{site.url}}/assets/images/circuit-structure.jpg" align="right" style="border:none; border-radius: 7px;" alt="image">
<br>
<br>
</div>


