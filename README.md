# A remix of AndBu/YAIFM

His original can be found at: https://github.com/AndBu/YAIFM

The differences here:

1. No switches, we detect the motor stopping turning instead.
2. No switches means fewer parts need printing or assembling,
though we still use the motor housing and retention
clip untouched from AndBu.
3. Because we now rely on the blind 'jamming' to detect
when it is fully open, we need to use a metal cog as
the plastic isn't strong enough.
4. We have ESPHome figure out blind fully open after
first power on. It stores in flash what you tell it blind
fully closed is. After that, you can request '10% open'
or '50% closed' or whatever as usual within Home Assistant
and you'll get exactly that.

To tl;dr; the differences, we go for simpler hardware and
let the software do all the heavy lifting. I reckon total
BOM cost per blind is under €40 inc VAT delivered excluding
the 5v power supply (you can use any existing USB based
phone charger).

Thingiverse link: https://www.thingiverse.com/thing:6865302

Printables link: TODO

Detailed instructions: https://www.nedprod.com/Niall_stuff/vdiary/archives/1734888387.html

## Bill of Materials

1. A GA12-N20 motor with rotary encoder which runs at
50 RPM at 6v, and has around 1 kgf.cm of torque at normal
load. You probably will want one with as much torque
at stall as possible without consuming too much current,
this lets the motor 'push through' any badly furled blinds
or the 200 cm long model fully extended to 195 cm.
Aliexpress is probably your easiest choice here, there are
many GA12-N20 models to choose between. Mine cost about
€6 inc VAT delivered.

2. A DRV8833 dual H-Bridge motor driver. I used the one on
a breakout board called 'HW-627' which is very affordable,
also from Aliexpress for about €2 inc VAT delivered.

3. A ‘9 Teeth D Type’ metal cog which has an outer diameter
of 8 mm and is 7.4 mm long. The key measurement is between
the flat part of the D hole rising perpendicularly to the
topmost of the rounded part of the D hole – that needs to
be 2.4 mm if you want it loose, 2.3 mm if you want it tight.
For some reason, these can cost a lot or not at all
depending on the Aliexpress listing, so for reference mine
cost €0.82 inc VAT each delivered.

4. A 3D printer ideally capable of printing in bright white
ABS or ASA, but if not then one able to print in bright
white PETG. I wouldn't recommend PLA, the 3D printed parts
take a lot of repeated torsion stress and PLA doesn't do
well with that over time.

<img src="https://github.com/ned14/YAIFM/blob/d88eb432634e654cb08949fb2025a30c5706809c/Images/20241015_122448.jpg" width="3024" height="4032" style="width:33%;height:33%;">
 
5. Two steel washers to go between the cog/hub
and the motor to put as much horizontal load onto the motor
housing and not onto the motor shaft as possible. Most
steel washers are slightly curved, you can use that to
reduce the contact surface to minimum and reduce wear over
time. You can get these for pennies from anywhere.

6. Any microcontroller able to run ESPHome. The ESP32 Super
Mini costs about €1.50 inc VAT delivered and is perfectly
fine for this job. You will need a USB-C power supply for it.

7. Finally, your choice of IKEA Fridans blind. I tested the 180 cm
model from its full extension of 195 cm, and I would have
absolute confidence that there is enough motor power for the
200 cm blind. They cost between €18 and €28 inc VAT depending
on size.

https://github.com/user-attachments/assets/c323144f-e48f-4d71-8c1b-16eb3580f487


## Wiring

For the motor:

- Red: Motor forwards. Connect to OUT1 of the DRV8833 board.
- White: Motor backwards. Connect to OUT2 of the DRV8833 board.
- Blue: Common ground. Connect to the GND of your MCU.
- Black: 3.3v - 5v power supply for the encoder. Connect to
the 3.3v supply of your MCU.
- Green: Encoder A phase. Connect to an input of your MCU.
- Yellow: Encoder B phase. Connect to an input of your MCU.

For the DRV8833 'HW-627':

- IN1: Connect to an output of your MCU (forwards).
- IN2: Connect to an output of your MCU (backwards).
- VCC: Connect to the 5v supply of your MCU.
- EEP: Connect to the 5v supply of your MCU.
- OUT1: Connect to the Red Motor forwards of the motor.
- OUT2: Connect to the White Motor backwards of the motor.

## ESPHome

[You will find my firmware in this git repo](https://github.com/ned14/YAIFM/tree/main/ESPHome).
It is for an Olimex ESP32-PoE board. You **WILL** need to customise
it for your board.

What it does:

1. On boot from power off, slowly open the blind until
it stops moving. Store that as the new 'fully open'
state. Put the blind back to the last stored open/closed
position.

2. Publish to Home Assistant a Cover which can take
any open/close fraction between 0.0 and 1.0. We wind
the blind at full speed until we get close to the desired
position, then slow the blind to 25% speed until it is
exactly reached.

3. If during opening the blind stops, we currently
assume that this is the new fully open. A TODO FIXME is
to only do that if we are close to the currently known
'fully open' setting. To prevent motor burnout, we cut
the power if the motor stops turning for more than
160 milliseconds in either direction.

Note that it does still have a few bugs in it. Bug fixes
are welcome!
