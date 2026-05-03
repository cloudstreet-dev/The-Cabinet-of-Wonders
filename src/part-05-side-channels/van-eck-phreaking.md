# Van Eck phreaking

A computer monitor — CRT, LCD, or modern flat panel — emits radio-frequency electromagnetic signals as it draws each pixel. With a sensitive enough receiver across the street, those emissions can be reconstructed into a real-time image of what is on the screen. The leak is unencrypted, broadband, continuous, and effectively invisible.

Wim van Eck demonstrated it for CRT monitors in 1985, when this was an early-Cold-War surprise. The method has been re-derived for every successive generation of display: VGA cables, DVI, LCD panels with internal serial buses, even HDMI. There is no known display technology that does not leak.

## Why displays radiate

Every digital signal in a computer is, electrically, a voltage that switches between two levels. Each switch is a step function. The Fourier transform of a step is a spectrum stretching to high frequencies. The wires carrying the signal — pins, traces, ribbon cables — are antennas. They radiate the spectrum.

A CRT scanned its electron beam across the screen line by line. Each pixel was a brief modulation of the beam current; brighter pixels meant more current. The beam current and the deflection voltages, both running at MHz rates, drove cables that radiated the entire pixel sequence as a complex RF waveform. Tune a receiver to the right frequency, demodulate, you have the screen contents back.

Modern displays use digital interfaces — VGA (analog R, G, B at high pixel rates), DVI, HDMI, eDP, MIPI-DSI. They are nominally "digital" but electrically they are still voltage signals, with rise times in the picosecond range, harmonics extending to GHz. The cables radiate at every harmonic.

## What a receiver actually does

The setup is conceptually simple:

1. A high-quality directional antenna (log-periodic or yagi) aimed at the target.
2. A wide-band software-defined radio (SDR) — USRP, HackRF, or BladeRF — sampling at hundreds of megahertz.
3. A computer doing real-time DSP: bandpass filter to a known emission frequency, demodulate (envelope detect, or for DVI, recover the differential), reshape into a 2D raster.

The hard part is *finding* the right frequency. CRTs emit broad spectra at the line rate and harmonics; modern interfaces emit narrow lines at the bit-clock harmonics. Once tuned to a strong harmonic, the rest is signal recovery.

Markus Kuhn published reconstruction software (Tempest for Eliza, then more sophisticated tools) that demodulates the signal and reassembles the image in real time. With a 30 MHz-bandwidth SDR and a directional antenna, you can read text on a target's screen from across a street, sometimes hundreds of meters.

## What changes for modern displays

CRT emissions are dominated by the low-frequency analog signals (line rate ~16 kHz, frame rate ~60 Hz, beam current at the dot clock ~25 MHz). Receiving them requires only modest equipment.

Digital displays use *transition-minimized differential signaling* (TMDS) on DVI/HDMI. Each pixel is encoded into a 10-bit balanced symbol; differential signaling means the two wires of each lane are 180° out of phase, and a perfectly balanced differential signal radiates very little (the two opposing radiators cancel). In theory.

In practice, the signaling is not perfectly balanced. Routing differences, common-mode currents, ground bounce, and ferrite imperfections all introduce single-ended residue. Kuhn's later work showed that TMDS emissions, while weaker, are still receivable, and the discrete pixel structure makes reconstruction *easier* than CRT (no analog blur). Modern reconstructions of HDMI traffic can read screen content from tens of meters with a good antenna.

LCD panels have internal LVDS or eDP serial links from the timing controller to the row drivers. These are even harder to suppress because the emitter is inside the screen housing, not on a long cable. Some LCDs leak distinctly.

## TEMPEST: the regulatory side

The US government has regulated emanations from sensitive computing equipment since the 1960s under the codename TEMPEST. The standards (NSTISSAM TEMPEST/1-92, since superseded) specify allowable emission levels for equipment processing classified information.

A "TEMPEST-rated" computer has reduced emissions — copper-mesh-shielded enclosure, ferrite chokes on every cable, filtered I/O, sometimes optical isolation. They are large, expensive, and rare outside government and finance.

For unrated equipment, mitigations include:

- **Fonts.** Kuhn showed that anti-aliased fonts radiate distinctly different signatures than crisp fonts. Choosing fonts with smooth edges reduces the high-frequency spectral content. Kuhn's *Soft Tempest* fonts (1998) are designed for low emission.
- **Background colors.** Lower-contrast backgrounds reduce the signal-to-noise of pixel emissions.
- **Distance.** The signal falls off with distance. Tens of meters of free air are typically enough; less if there are walls.
- **Compromising emanations from CPU and bus.** Less commonly attacked, but power buses and memory buses also radiate.

## Reading from far away: the wire and the wall

Kuhn's later work (2013, 2018) measured emanations from VGA, DVI, and HDMI displays at distances up to *60 meters* through office walls, with reconstruction quality high enough to read 12-point text. The cables and panel routing leak through standard drywall almost unattenuated; metal-stud walls help marginally; full Faraday shielding is required to genuinely block.

The takeaway: any standard office computer, doing nothing more than displaying a window of text, is broadcasting that text in the clear to anyone within RF range with the right equipment.

## Other displays

**Touch screen capacitive sensing.** A capacitive touchscreen has a grid of conductive traces driven by a controller; touches are detected by capacitance changes. The drive signals radiate, and Kuhn-style reconstruction can recover not just what is on the screen but what the user touches.

**E-ink displays.** Update slowly and radiate during page transitions only. Hard to attack continuously, but each redraw leaks content.

**Near-eye and HUD displays.** AR/VR headsets and heads-up displays emit similarly to laptop screens, with the added attack surface of being moved around in plain view.

## How this is different from acoustic cryptanalysis

Acoustic cryptanalysis (in the *Acoustic cryptanalysis* entry) targets cryptographic operations inside the CPU. It needs the target to be performing a specific kind of operation and exploits power-supply behavior to recover keys.

Van Eck phreaking targets the *display*. It does not require any specific operation; whatever is on the screen is what leaks. So passwords typed into a remote-desktop session, banking screens, classified documents, biometric capture interfaces — all visible to the receiver in real time.

The two are complementary. Acoustic gets you keys. Van Eck gets you everything else.

## The wonder, dark version

The intuition is that a display is a private object — only people in the room can see it. The reality is that every display is a transmitter. Every cable is an antenna. Every pixel transition radiates. The privacy is illusory; "looking at the screen" is a directional cone that humans see, but the screen is, at the same time, broadcasting omnidirectionally.

It is one of the cleanest demonstrations that an air gap is not really an air gap. The machine talks to its environment continuously, in all directions, in radio. Most of what it says is noise. Some of what it says — under the right reconstruction — is the screen's content, in real time, transmitted to anyone within the radius of the inverse-square law.

## Where to go deeper

- Wim van Eck, *Electromagnetic Radiation from Video Display Units: An Eavesdropping Risk?*, Computers & Security, 1985. The original.
- Markus Kuhn, *Compromising Emanations: Eavesdropping Risks of Computer Displays*, University of Cambridge Technical Report 577, 2003. The modern reference.
