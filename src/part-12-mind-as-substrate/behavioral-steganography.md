# Behavioral steganography

You can hide a signal inside ordinary human behavior. Not in a watermark, not in a hidden file, not in a covert audio channel — but inside the *pattern* of unremarkable everyday activity. The timing of clicks. The choice of which photo to "like" first. The precise wording of an apparently mundane email. The pattern is a code; an external observer who does not know the code sees only the surface behavior.

This is behavioral steganography. It overlaps with classical digital steganography (hide data in the LSBs of an image) but the carrier is a human's actions rather than a media file. The carrier can be hard to detect because human behavior already has high natural variance; encoded variations are hard to distinguish from baseline noise.

This is a small, careful entry. The phenomenon is real (cognitive psychology, computer security, and ethnography all touch it), but practical applications are narrow and the literature is partly speculative.

## What it can carry

If a human's behavior has \\(N\\) bits per day of "natural variance" (e.g., the noise in their typing speed, the distribution of their app-launch times, the choice of words in their messages), you can in principle encode \\(N\\) bits of information per day into the choices the human makes. The receiver, observing the human's behavior, decodes.

To use this for actual communication, both parties need:
- A shared encoding scheme — a way to map bits onto behavior choices.
- A way to *coordinate*: the sender must know which choices count as "encoding" and which are noise.
- An observation channel: the receiver must be able to observe the relevant behavior.

Real implementations include:

**Tor entry/exit timing patterns.** A Tor user can encode bits in the timing of their requests; an observer who can correlate entry and exit traffic might decode the pattern. Defenses include Tor's deliberate timing randomization.

**Linguistic stylometry as steganography.** Choose between synonyms or sentence orderings to encode bits. Each "free choice" carries a bit. A 100-word email written by someone with linguistic flexibility might encode 20-50 bits without sounding wooden.

**Photo posting time as covert channel.** Post a photo at 09:13:42 vs. 09:13:43 to encode one bit. Over the course of normal social-media posting, dozens of bits per day are signalable.

**Finger-tapping as authentication.** A person taps a particular pattern; the system identifies them by stylistic features. Used in some experimental authentication schemes.

## What gets hard

Behavioral channels have low bandwidth (a few bits per minute, optimistically) and high error rates (humans are noisy carriers, and behavior is observed by others, not measured precisely). They are also fragile: any change in routine — a sick day, a different schedule — disrupts the encoded signal.

Detection is also hard. An observer needs to distinguish "this person's behavior contains an encoded message" from "this person's behavior is normally varied." Statistical tests on a few hundred bits' worth of data may not have enough power. Long, patient observation might detect a pattern; short observation usually will not.

For high-stakes adversarial settings (an authoritarian government tracking a dissident's online activity, an insider exfiltrating data through behavior), behavioral steganography offers a low-bandwidth but very-low-detection-probability channel.

## Connection to LED exfiltration and other side channels

Behavioral steganography is the human-substrate version of the side-channel exfiltration described in Part V. There, a compromised computer's hardware (a CPU, a fan, an LED) is the carrier of an embedded signal. Here, a compromised human (or a willing one) is the carrier. The ideas are parallel: any continuous physical or behavioral process can be modulated to encode bits, and the receiver demodulates.

The human version is harder than the computer version because:

- Human behavior is much noisier than CPU current draw.
- The "encoder" cannot easily achieve precise modulation (humans are bad at producing deterministic patterns).
- The bandwidth is much lower.

But it has compensating advantages:

- It does not require any physical access to the target's devices.
- It is usually *invisible to forensics* — there is no log file recording "this is a steganography signal."
- It is usually *legal*: no laws prohibit choosing one synonym over another.

## What this implies for surveillance

Modern surveillance — both state-level and commercial — collects vast amounts of behavioral data. App-launch logs, location histories, keystroke timing, scroll patterns. In principle, any of this is a possible carrier for steganographic encoding by a sophisticated actor. Whether anyone *is* using this in practice (as opposed to it being a theoretical channel) is mostly unknown — by definition, successful behavioral steganography would not be detected.

The reverse problem is more practical: surveillance systems often try to *identify individuals from behavioral patterns*. Stylometric writing analysis, gait recognition from CCTV, mouse-movement patterns for browser fingerprinting — all are forms of "extract a signature from behavior" rather than "encode a signal in behavior," but the underlying technical apparatus is similar. Adversarial users sometimes try to blunt these by introducing noise or mimicking other styles.

## Where this connects to the cabinet's broader theme

The wonders in this part are about minds, not silicon. Behavioral steganography is the wonder that *people themselves can be cryptographic carriers*. A human's everyday actions, with no special equipment, can transmit a covert signal that no observer can reliably extract. The signal exists; the carrier is a person; the bandwidth is small; the detectability is even smaller.

It is also a counterpoint to the side-channel work in Part V: there we saw computers leaking data through physical phenomena. Here we see *people* who can deliberately use the natural variability of their behavior as a channel. In both cases, the boundary between "the signal" and "the noise around it" is a soft, statistical one. Both are exploitable; both are hard to detect; and both rely on being below the observer's threshold of attention.

## A note on uncertainty

Most of this entry is structural and conceptual. There is comparatively little hard empirical literature, because behavioral steganography that works *would not* be widely visible (no one tells you they are doing it; no one is publicly successful at being detected doing it). Academic security work has formalized some of the channels (linguistic, timing-based, social-media-based), but real-world deployment is hard to characterize.

What can be said with confidence: the channel is real. Humans have enough natural variance in their behavior to carry quite a few bits per day, and adversarial encoders have demonstrated it in laboratory and limited-deployment settings. Whether sophisticated users employ it routinely is unknown; the absence of detection is, by the nature of the technique, evidence-incompatible with detection.

## Where to go deeper

- Brassil, Low, Maxemchuk, *Copyright protection for the electronic distribution of text documents*, Proc. IEEE 1999. The classical line-shifting steganography for printed text — closely related, document-based.
- Wayner, *Disappearing Cryptography* (3rd ed., 2009). Comprehensive overview of steganographic techniques, including behavioral/social ones.
