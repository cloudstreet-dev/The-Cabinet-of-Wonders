# Air-gap covert channels

A machine with no network connection still has a hundred ways to send a signal to a nearby observer: vibration through the desk, ultrasonic chirps from the speaker, heat fluctuations affecting a nearby sensor, fan-noise modulation, magnetic field changes that pass through walls, current modulation that propagates back into the building's wiring. Almost every physical quantity a computer affects can be modulated by software and detected by something outside.

The previous entry covered LED exfiltration as one example. This one is the broader catalogue. The phenomenon is that *air gap* is a network-layer concept; physics does not respect it.

## What "air gap" means

In high-security computing — military classified networks, SCADA controllers in critical infrastructure, financial-clearing core systems — the standard mitigation against external attack is to disconnect the machine from any external network. No Ethernet, no Wi-Fi, no Bluetooth, no cellular modem. Updates and operator interaction happen via a separate trusted procedure (USB media, console terminal, etc.).

The threat model: an attacker cannot reach the machine over the network because there is no network. If they get malware on the machine via insider action or USB-stick smuggling (Stuxnet's vector), the malware cannot phone home or exfiltrate data because there is no outbound channel.

The catalogue below shows that this last assumption is wrong. The malware can phone home or exfiltrate, just very slowly, through any of dozens of physical side channels.

## The catalogue

For each channel: a physical quantity the source can modulate, and a sensor that can detect the modulation.

### Acoustic channels

- **Speakers** ("AirHopper", 2014, "Fansmitter", 2016, "DiskFiltration", 2017): modulate fan, disk, or audio output into ultrasonic or audible signals. Nearby phone or microphone receives.
- **Hard-disk seek noise**: pattern access to make the actuator click. Detect the clicks acoustically.
- **CPU coil whine** (the inverse of acoustic cryptanalysis — modulate intentionally instead of accidentally): control CPU load to produce a deliberate audible signal.
- **POWER-SUPPLIES (POWER-SUPPLAY, 2020)**: SMPS power supplies under varying load emit different audible coil whine. The malware modulates CPU load to produce a controllable whine. A microphone meters away decodes.

### Optical channels

- **Status LEDs**: any software-driven indicator (LED-it-GO, xLED, etc.).
- **Display flicker**: modulate display brightness or pixel patterns to encode bits in optical patterns invisible to a human but detectable by a camera.
- **Reflected light**: modulate internal lighting and observe through reflective surfaces.

### Electromagnetic channels

- **GSM band**: control CPU instructions to emit specific patterns in the cellular bands. A nearby phone tuned to the right band picks up.
- **FM band**: similar, in radio frequencies. Demonstrated in *AirHopper* — emit signals receivable on a nearby standard FM radio.
- **USB-port radiated EM** (USBee, 2016): toggle USB lines to act as antennas at specific frequencies.
- **Memory bus radiation** (DDR3 frequencies in the AM band, etc.): modulate memory access patterns.
- **HDMI and display cables** (Van Eck phreaking, more carefully): modulate intentional patterns instead of receiving accidental ones.

### Magnetic channels

- **CPU magnetic field** (MAGNETO, 2018, ODINI, 2018): modulate CPU activity to produce a varying magnetic field. Magnetic fields pass through Faraday cages that block electric fields. Receiver: magnetometer in a phone, or a dedicated sensor placed against the chassis.
- **Hard-drive head positioning magnetic emanation**: modulate head movement to produce a varying field.

### Thermal channels

- **CPU temperature** (BitWhisper, 2015): modulate CPU load to vary the chassis surface temperature. A thermometer-equipped device nearby detects. Slow (bytes per hour) but works at moderate distance with no line-of-sight required.

### Power line channels

- **Current draw modulation** (PowerHammer, 2018): modulate CPU activity to draw varying current. The variation propagates through the building's electrical wiring. A receiver tapped into the same circuit, even on a different floor, decodes. Bandwidth: hundreds of bits per second, no line-of-sight, no nearby physical access required.

### Vibrational channels

- **CPU fans, hard-drive vibration**: modulate vibration patterns. A geophone or accelerometer on the desk or floor below detects.
- **Smartphone accelerometers**: a phone resting on the same desk picks up modulated vibrations.

## Bandwidth and range

The trade-off is consistent: stealthy = slow.

| Channel | Range | Bandwidth | Stealthy? |
|---|---|---|---|
| Speaker (ultrasonic) | ~10 m | 10s of bps | Yes (above hearing) |
| LED | line of sight, ~100 m | 100s of bps | Mostly |
| Magnetic field | ~1 m | 10 bps | Yes |
| Power line | building-wide | 100 bps | Yes |
| Heat | <1 m | bytes/hour | Yes |
| EM / radio | meters to km | 100s of bps | Detectable with sweep |

A few hundred bps is enough to leak a 4096-bit RSA key in seconds, an entire hard drive in months. Once the malware is in, sustaining a slow drip for a long time is feasible.

## How the malware encodes the data

The encoding choices are dictated by:

- **Channel bandwidth**: how many distinguishable states per second.
- **Receiver capability**: simple ASK (amplitude shift keying) for low-bandwidth channels, FSK or OOK for higher; OFDM-style for the more elaborate schemes.
- **Stealth**: must look like incidental noise, not a deliberate transmission. Spread spectrum, random gaps, mimicry of "normal" patterns.
- **Error correction**: convolutional codes, Reed-Solomon, LDPC — depending on bandwidth and noise.

A typical implementation: malware encodes a session key into Manchester-coded bits, AM-modulated onto a chosen carrier (CPU clock harmonic, fan speed, etc.). Receiver decodes with standard DSP.

## Defenses

Each channel needs its own defense:

- **Acoustic**: remove speakers, microphones; soundproofed rooms; jammers.
- **Optical**: cover LEDs, install opaque enclosures, control external sightlines.
- **EM**: Faraday cages (rare and expensive); cable shielding.
- **Magnetic**: mu-metal shielding (much harder to come by than copper).
- **Thermal**: HVAC isolation; sensors removed; not really mitigatable in a normal building.
- **Power line**: filtered power input; UPS; isolated power circuits.
- **Vibrational**: vibration-isolated mounting; remove sensors from the room.

Real-world high-security rooms (TEMPEST-rated SCIFs, classified-information facilities) implement many of these. They are expensive and operationally constrained. Most "air-gapped" production systems implement a fraction.

## Why the catalogue keeps growing

Every time computers add a new physical interface or sensor, a new covert channel becomes possible. Smartphone-class hardware is full of sensors (accelerometer, magnetometer, microphone, light sensor, barometer, gyroscope, thermometer, occasionally even radar) and any of them can become a receiver. Industrial equipment increasingly has IoT-style telemetry that creates similar receiver opportunities.

The set of physical phenomena a CPU can affect is much larger than the set of network protocols. So the work of cataloging covert channels is essentially endless. Each year, new ones are found.

## The wonder, sober version

The wonder here is partly that so many channels exist, and partly that they are so hard to notice. A magnetic field. The sound of a fan. The temperature of a chassis. None of these feel like they should be "data" — they feel like ambient phenomena the machine produces incidentally. Yet each can be modulated, each can carry kilobits per minute, and each is invisible to the standard threat model.

The intuition that an isolated machine is isolated breaks down when you ask the question: what physical state does this machine have, that another physical observer can detect? The answer is almost everything.

## Where to go deeper

- Mordechai Guri's group at Ben-Gurion University: dozens of papers on specific channels. Their preprints are the canonical literature.
- Guri, *A survey of air-gap covert channels for sensitive data exfiltration*, IEEE Communications Surveys & Tutorials 2020. The catalogue.
