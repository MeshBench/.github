# MeshBench

An RF-accurate MeshCore network simulator: **real MeshCore firmware** against a
**sample-accurate LoRa baseband channel** with real noise. The question it
answers is not "would a packet get through" but "what arrived at the antenna,
and why".

Firmware runs one of two ways. **Native** compiles MeshCore for the host.
**Emulated** runs the image people actually flash, unmodified, inside an
emulator — and that is what the forks below are for.

## The forks, and what each one carries

Every one is upstream plus a patch we maintain, not a rewrite. They are here so
that setting MeshBench up is a download rather than an afternoon with a
toolchain.

| repository | branch | upstream | what it adds |
|---|---|---|---|
| [qemu](https://github.com/MeshBench/qemu) | `meshbench-sx1262` | Espressif's QEMU fork | An SX1262 SPI device, a working GPIO implementation, and machine properties for the radio wiring. Upstream's GPIO write handler is empty, and RadioLib drives chip select as an ordinary GPIO — without it the chip sees an unframed byte stream and the driver reports no chip present. |
| [tlib](https://github.com/MeshBench/tlib) | `sevonpend-any-pending` | [antmicro/tlib](https://github.com/antmicro/tlib) | SEVONPEND generates an event for *any* exception entering the pending state, not only ones the CPU would accept. ARM DDI0403E B1.5.17 does not qualify it by whether the exception is enabled. |
| [renode-infrastructure](https://github.com/MeshBench/renode-infrastructure) | `sevonpend-any-pending` | [renode/renode-infrastructure](https://github.com/renode/renode-infrastructure) | The C# half of that fix: the NVIC can answer the wider question, and setting the event flag now wakes a CPU already asleep. |
| [renode](https://github.com/MeshBench/renode) | `meshbench` | [renode/renode](https://github.com/renode/renode) | Ties the two together and builds a portable package in CI, runtime included. Also asserts both halves of the fix are in the tree it built. |

## The rest

| repository | what it is |
|---|---|
| [meshcore-native](https://github.com/MeshBench/meshcore-native) | Host and cross builds of MeshCore, the virtual SX1262, the bridge, and the radio model both emulators talk to |
| [meshbench-reports](https://github.com/MeshBench/meshbench-reports) | Published reports |

## Not ours

[MeshCore](https://github.com/meshcore-dev/MeshCore) is upstream and
**unmodified** — the build points at a checkout and compiles it as it stands,
which is the whole basis of the claim that this runs real firmware. Board images
come from its releases.

The Nordic SoftDevice is not ours and cannot be redistributed: anyone running a
published nRF52 image supplies their own copy.
