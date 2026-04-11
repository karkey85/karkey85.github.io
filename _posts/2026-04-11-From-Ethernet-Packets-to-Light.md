
## From Ethernet Packets to Light: How Data Travels 12,000 km Across Oceans

Ever wondered how you’re able to make a voice or video call from your home to the farthest corner of the world?
Most of the time, the voice is crystal clear, and the video quality is surprisingly good. That’s the power of the internetwork of networks—the Internet.
We often say the Internet is the highway. But what really matters is who is driving on that highway and how efficiently the traffic is handled. That’s what decides latency, throughput, and overall experience.

Now, this is where it hits differently for a network engineer.

How does a packet—whether it’s a voice call, a WhatsApp message, or a LinkedIn post—travel from New York to Chennai (over 12,000 km away) in just a second or two?

Your guess is right.

Light.

The fastest known thing in the universe. A clear winner in any race.

But just saying “light” is not enough.
That doesn’t explain the engineering, design, and systems that make this possible at a global scale.

**Copper Has Limits, Optical Fiber Changes the Game**

For a network engineer, Ethernet over copper—CAT5(e)/CAT6(a)/CAT7/CAT8—is almost second nature. It’s the default medium for connecting routers, switches, and end devices in LAN environments.
But copper has very well-defined physical limits.

In standard Ethernet (1000BASE-T, 10GBASE-T), the maximum supported distance is ~100 meters. This is not arbitrary—it’s dictated by signal attenuation, noise, crosstalk (NEXT/FEXT), and timing constraints. As distance increases, the signal degrades, leading to higher bit error rates, frame corruption, and eventually link failure.

So from a design perspective, copper is optimized for short-reach, high-speed communication within a confined space—not for long-haul transport.

This is where optical fiber fundamentally changes the game.

Instead of electrical signaling, fiber operates in the optical domain—using light propagated through glass. The attenuation characteristics are orders of magnitude better than copper (typically ~0.2 dB/km in modern single-mode fiber).

Because of this:

* Signals can travel **80–100 km without regeneration** in standard long-haul systems
* With optical amplification (like EDFA), spans can be extended significantly
* In multi-span systems with amplification chains, signals routinely travel **thousands of kilometers (3000+ km)**
* Submarine cable systems, with carefully engineered amplification stages, easily extend beyond **10,000–12,000 km**

This is the physical foundation of the global internet—**optical fiber laid across continents and ocean floors, carrying massive volumes of data as light**.**

**Routers Don’t Speak “Light”, but Transponders do**

Core devices from companies like Cisco or Arista Networks deal with Ethernet/IP packets. Electrical domain. They don’t directly push signals across oceans. So where does the transition happen?

This is where optical vendors like Fujitsu and Ciena come in. They build transponders. 

Think of a transponder as a translator between two worlds:
Ethernet (electrical packets)
Optical (light signals)

What actually happens:

Ethernet traffic from a router enters the transponder
The signal is processed and mapped
It is converted into an optical signal
Assigned a specific wavelength
Sent into the optical line system

At the far end, the reverse happens.

The Core Idea: DWDM

The real power of optical networks comes from:

 ense Wavelength Division Multiplexing (DWDM)

Instead of sending just one signal per fiber, DWDM allows multiple signals to coexist.

Each signal is carried on a different wavelength (color of light).

So in a single fiber:

You don’t send one stream
You send dozens (sometimes 80+ wavelengths)

Each wavelength can carry:

100G
400G
800G (modern systems)

Which means a single fiber can carry multiple terabits per second.

That’s the real backbone of the internet.

Long Distance: Amplification Matters

Even with fiber, signals don’t stay perfect forever.

Over distance, they weaken.

So we use:

Optical amplifiers (like EDFA)
Raman amplification (used in long-haul systems to boost signal quality)

These are placed at intervals across the fiber—especially in submarine cables.

This is what allows signals to travel thousands of kilometers without being converted back to electrical form.

End-to-End Flow (Putting It Together)

If I simplify the entire journey:

Router → Transponder → Optical Line System → Fiber → Optical Line System → Transponder → Router

Or in a more real-world view:

Cisco router
→ transponder
→ DWDM / lambda system
→ long-haul fiber (even under ocean)
→ DWDM system
→ transponder
→ destination router

That’s your packet’s journey across continents.

**How Much Data Are We Talking?**

This part is crazy when you think about scale.

Single wavelength: up to 800G (and moving towards 1.6T)
Single fiber: 10–20 Tbps and beyond
With advanced systems: even higher using dense channel packing

This is what powers:

Cloud infrastructure
Video streaming
Global internet traffic
AI workloads

**Real-World Exposure**

I’ve worked on Fujistu 1finity products like Flashwave 9500, Transponder series (T-series), Lambda (optical line) systems and DCN environments
Seeing Ethernet traffic being converted into light and pushed across long distances—it’s something you don’t fully appreciate until you see it live.

**Future: With AI and large-scale compute**

Data center traffic is exploding. East-west traffic is massive. Bandwidth demand keeps increasing. There’s already movement toward tighter integration between Ethernet and optics. Companies like Arrcus working with optical platforms like Fujitsu’s 1FINITY is one such example.
Honestly, it wouldn’t be surprising if more of the data center fabric itself becomes optical over time.

Today, every packet you send, gets converted into light travelling across oceans, sharing fiber with dozens of other signals and finally reaching the destination in milliseconds.
And all of this happens silently, reliably, at massive scale.

That’s the internet we use every day and the power of networking.
