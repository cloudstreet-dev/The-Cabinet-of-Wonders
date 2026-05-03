# GPS and the relativistic clock correction

If GPS satellites used Newtonian time, your phone would tell you it was about ten kilometers away from where it actually is, and the error would grow by another ten kilometers every day. The system works because the firmware on every GPS satellite continuously corrects for two competing relativistic effects predicted by Einstein in 1905 and 1915, decades before anyone tried to navigate by satellite. The corrections are not optional. They are the only reason the system functions at all.

## What the satellites are actually doing

A GPS satellite is, at its core, a flying clock. It carries an atomic clock — a cesium or rubidium standard — and broadcasts a signal saying "this is my time, this is my position." Your receiver picks up signals from four or more satellites simultaneously, notes how long each took to arrive, multiplies by the speed of light to get distances, and trilateral-solves for its own position and clock offset.

The trilateration math is straightforward. The hard part is timing. Light travels about 30 centimeters per nanosecond. A GPS receiver wants meter-level accuracy, which means the satellite clocks have to agree with each other, and with the receiver's idea of time, to within a few nanoseconds.

A satellite at 20,000 km altitude moves at about 3.9 km/s in its orbit. That sounds slow compared to light, but precision-clock-wise, it is enormous.

## The two corrections, in opposite directions

Special relativity says a clock that is moving runs slow relative to a stationary one. The factor is approximately

\\[ \frac{\Delta t_{\text{moving}}}{\Delta t_{\text{stationary}}} \approx 1 - \frac{v^2}{2c^2} \\]

For \\(v \approx 3.87\ \text{km/s}\\) and \\(c = 3 \times 10^5\ \text{km/s}\\):

\\[ \frac{v^2}{2c^2} \approx \frac{(3.87)^2}{2 \cdot (3 \times 10^5)^2} \approx 8.3 \times 10^{-11} \\]

So the satellite's clock, by SR alone, runs slow by a factor of \\(8.3 \times 10^{-11}\\) — about 7 microseconds per day relative to a stationary observer.

General relativity says a clock deeper in a gravity well runs slower than a clock higher up. The fractional rate difference between two clocks at gravitational potentials \\(\Phi_1\\) and \\(\Phi_2\\) is approximately \\((\Phi_2 - \Phi_1)/c^2\\). The satellite is roughly 20,000 km up; the ground is at 6,371 km from Earth's center. Computing the potential difference \\(\Phi_{\text{sat}} - \Phi_{\text{ground}}\\) for Earth gives about \\(+5.3 \times 10^{-10}\\). Positive: the satellite is higher in the well, so its clock runs faster than ours. About 45 microseconds per day faster.

Net effect: \\(45 - 7 = 38\\) microseconds per day. The satellite clock runs fast by 38 μs every 24 hours.

In distance, 38 μs of clock error becomes \\(38 \times 10^{-6} \times 3 \times 10^8\ \text{m} \approx 11\ \text{km}\\) of position error per day. Without correction, GPS would diverge from reality at roughly the speed of a fast walk, day after day, until it was unusably wrong.

## What the engineers did

You could correct this in software on the receiver. The system designers chose differently. The atomic clocks in the satellites are physically tuned, before launch, to run slow on the ground by exactly the amount that will make them correct in orbit. The cesium standards are set to oscillate at 10.22999999543 MHz instead of the nominal 10.23 MHz. After the satellite reaches orbit and the relativistic effects kick in, the clock runs at the right rate as seen from Earth.

That is not the whole correction. The orbit is slightly elliptical, so the velocity and altitude change as the satellite goes around. Both relativistic effects therefore vary periodically. The receiver applies an additional eccentricity correction — a small term proportional to \\(e \sin E\\) where \\(e\\) is the orbital eccentricity and \\(E\\) is the eccentric anomaly — every position fix.

There are smaller effects too. The Sagnac effect, from Earth's rotation during the signal's transit time, contributes tens of nanoseconds. Tropospheric and ionospheric delays distort the speed of light in the atmosphere. Each is modeled and subtracted.

## What it would take to deny relativity

Engineers love to find ways to avoid being theoretical-physics-dependent. With GPS, you cannot. The relativistic corrections were not slipped in to flatter Einstein. They were tested empirically before the system was trusted operationally. The Block I prototype satellites in 1977 carried a cesium clock that could be switched between two oscillator settings: relativistically corrected, and uncorrected. They flew it uncorrected for the first 20 days of the Block I program. The frequency offset was measured. It matched the GR+SR prediction to within the precision of the clock. The relativistic correction was then enabled and has been running ever since.

This is one of the cleanest experimental confirmations of general relativity in human technology, and it is happening continuously, in roughly thirty satellites, right now, while you read this. Every position fix you have ever gotten from your phone has gone through it.

## The wonder

Two of the strangest predictions of twentieth-century physics — that motion slows time and that gravity slows time — are not curiosities for thought experiments. They are load-bearing components of an infrastructure the entire planet depends on for navigation, timing, agriculture, finance, and tracking shipping containers. If Einstein had been wrong, GPS would not work. He was not wrong. It does.

## Where to go deeper

- Neil Ashby, *Relativity in the Global Positioning System*, Living Reviews in Relativity 6 (2003). The canonical engineering-grade treatment. Open access.
- The IS-GPS-200 specification, the navigation message ICD, for the actual bit-level correction terms a real receiver applies.
