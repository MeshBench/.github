## MeshBench
An RF-accurate MeshCore network emulator/simulator.

It runs **real MeshCore firmware** — the actual C++ from `meshcore-dev/MeshCore`,
compiled for the host, driven through **its own radio driver over real
RadioLib** — against a **sample-accurate LoRa baseband channel** with real
noise.

The channel does not decide whether a packet arrives. It sums waveforms, applies
path loss over real terrain, adds thermal noise, and lets each receiver's
demodulator find out. Capture effect, partial collisions and sensitivity are
*emergent*, not rules someone wrote down.
