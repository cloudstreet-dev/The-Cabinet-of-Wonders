# LED exfiltration

The little green LED on the front of a router can be controlled by software. So can the keyboard's caps-lock LED. So can the disk-activity LED on a server. If you have malware on a machine, you can blink any of these, very fast, to encode a data stream. A camera pointed at the LED — even from outside the building, even through a window, even through a transparent ventilation grille — recovers the data. The LED was not designed to be a transmitter. The CPU is using it as one.

This is the kind of side channel that exists not because of physics leaking past the abstraction (as with EM emanation) but because the abstraction includes a thing that emits visible light, and software can drive it. Air-gapped networks — disconnected from any external link, intended for high-security data — leak through their LEDs.

## How fast can an LED blink

An LED has a switching time of nanoseconds. The bottleneck is whatever software pipeline drives it. From user-mode software writing to a sysfs file controlling the LED, you can blink at a few hundred Hz. From kernel-mode driver or direct GPIO write, kilohertz. Some LEDs (network indicator, hard-disk activity) are driven directly by hardware events but can be turned on/off by software writes to a register.

A receiver — a high-frame-rate camera, a photodiode coupled with a small lens, even a CMOS sensor in a phone — can sample at hundreds of Hz to thousands of Hz. So the channel bandwidth is on the order of 100 to 10,000 bits per second per LED.

## The Mordechai Guri stack

Most public LED-exfiltration research comes from Mordechai Guri's group at Ben-Gurion University, who have systematically demonstrated channels through:

- **Hard-disk activity LED** ("LED-it-GO", 2017): malware on a server modulates LED via fast disk reads. Receiver: drone with camera at 1 km. Demonstrated bandwidth: 4 kbits/s.
- **Router LEDs** ("xLED", 2017): control router status LEDs from compromised firmware.
- **Keyboard caps-lock LED** ("CTRL-ALT-LED", 2019): keyboard LEDs are software-controllable on most OSes. Slow but reliable.
- **Power LEDs and printer LEDs**: any LED whose state is software-controllable.
- **Air-gap LEDs combined with optical reflections**: shine the modulated signal off shiny objects to bypass camera placement constraints.

In each case, the construction is the same: malware encodes data into a blink pattern; the camera or photodiode receives.

## Why this works against air gaps

An air-gap network is one with no electrical connection to the outside world. No Ethernet cable, no Wi-Fi, no cellular modem, no Bluetooth, ideally even no shared power supplies. Used in classified-information processing, critical infrastructure, certain financial systems.

The threat model is that attackers cannot get data out, even if they have malware running. But "no electrical connection" does not mean "no transmission medium." Light is a medium. Sound is a medium. Heat is a medium. Vibration is a medium. The malware can encode data into any physical quantity it can modulate.

LEDs are particularly attractive because:

- They are small and ubiquitous.
- They blink frequently anyway, so a slightly modified blink pattern is easy to overlook.
- They are software-controllable through standard interfaces.
- A camera observing them does not require any access inside the secure facility — line-of-sight from outside, through a window, suffices.

## Other physical channels

The same group and others have documented exfiltration through:

- **Heat (BitWhisper)**: modulate CPU load on the source; observe ambient temperature changes on a nearby thermometer-equipped machine. Slow (~bytes per hour) but works against thermal isolation.
- **Sound (AirHopper, Fansmitter, DiskFiltration, etc.)**: modulate fan speed, hard-drive seek patterns, or speaker output to encode data into ultrasonic or audible sound. A nearby phone records.
- **EM radiation from CPU bus, USB ports, GSM bands**: tune a CPU instruction loop to a frequency that radiates strongly through a leaky cable.
- **Magnetic fields (MAGNETO)**: modulate CPU operation to vary the magnetic field around the laptop. Magnetic field passes through Faraday cages that block electric fields. Receiver: magnetometer in a phone.
- **Power line (PowerHammer)**: modulate CPU activity to draw varying current; the variation propagates through the building's electrical wiring; a receiver tapped into the same circuit (even meters away) decodes.

The catalog grows yearly. Every "side channel" is just a physical quantity that the malware can modulate and a sensor outside can detect.

## The receiver side

For LED exfil specifically:

- **Phone cameras**: easy to deploy, ~30-60 fps, decent in low light.
- **High-speed industrial cameras**: 1000+ fps, expensive, heavy.
- **Photodiodes with optical filters**: cheap, fast, narrow field of view; the right tool for known-position LEDs.
- **Drones**: useful for getting line-of-sight to LEDs in otherwise inaccessible locations. The 2017 LED-it-GO paper used a hovering drone with a camera at the window of a target building.

The receiver does signal recovery, error correction (the channel is noisy), and decoding. Realistic bandwidths after error correction: kilobits per second from a hard-drive LED at moderate distance, hundreds of bits per second from a router LED at long distance.

## Defenses

Defenses against LED channels are mostly procedural and physical:

- **Cover or remove LEDs.** Tape over the disk-activity LED; remove the front panel that exposes it. Standard for high-security facilities.
- **Disable software LED control.** Patch kernels to make LED state non-writable from user space. Hard to do completely; many LEDs are tied to hardware events.
- **Window films and Faraday rooms.** Block line-of-sight from outside. Standard for SCIFs (Sensitive Compartmented Information Facilities).
- **Air gaps that are actually gaps.** Locate sensitive machines in interior rooms with no external windows or surfaces visible from outside.

Software defenses ("randomize the LED blink pattern") are insufficient because they cannot remove the channel; they only add noise that can often be filtered out.

## Why this is wonder, even though it is sad

The intuition before LED exfil research: an air-gapped machine cannot leak, except via human carriers (USB sticks, printed paper, the famous Stuxnet vector). LED exfil falsifies that. The machine carries information out continuously, encoded in its visible light, without any human carrier.

Defending against this requires expanding the concept of "the machine" to include every physical phenomenon it produces — heat, light, sound, vibration, EM emanation, current draw, magnetic field. Anything an attacker can detect is a potential channel. The set of such phenomena is essentially open: each new generation of research finds another one.

The wonder is in the breadth. Any time something physical depends on what software is doing — even an LED blinking when a disk is read — there is a possible exfiltration channel. The list of such physical dependencies is much larger than the list of network interfaces, and rooting out all of them turns out to be nearly impossible.

## Where to go deeper

- Mordechai Guri et al., *LED-it-GO*, ACSAC 2017. Hard-disk LED exfil.
- Loughry and Umphress, *Information Leakage from Optical Emanations*, ACM Transactions on Information and System Security, 2002. The first systematic study of LED exfiltration.
