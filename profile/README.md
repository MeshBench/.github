<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/MeshBench/.github/main/profile/banner.png">
  <img alt="MeshBench — an RF-accurate MeshCore network simulator" src="https://raw.githubusercontent.com/MeshBench/.github/main/profile/banner-light.png">
</picture>

An RF-accurate MeshCore network simulator: **real MeshCore firmware** against a
**sample-accurate LoRa baseband channel** with real noise. The question it
answers is not "would a packet get through" but "what arrived at the antenna,
and why".

The channel decides nothing. It sums waveforms, applies path loss over real
terrain, adds thermal noise, and lets each receiver's demodulator find out —
so capture effect, partial collisions and sensitivity are *emergent* rather than
rules somebody wrote down.

Firmware runs one of two ways. **Native** compiles MeshCore for the host.
**Emulated** runs the image people actually flash, unmodified, inside an
emulator — and that is what the forks below are for.

## The forks, and what each one carries

Every one is upstream plus a patch we maintain, not a rewrite. They are here so
that setting MeshBench up is a download rather than an afternoon with a
toolchain.

| repository | branch | upstream | what it adds |
|---|---|---|---|
| [qemu](https://github.com/MeshBench/qemu) | `meshbench-main` | Espressif's QEMU fork, 9.2.2 | An SX1262 on the SPI bus, the GPIO and interrupt behaviour MeshCore relies on, and enough of an ESP32-S3 for a published board image to boot and reach the air. Itemised below. |
| [tlib](https://github.com/MeshBench/tlib) | `meshbench-main` | [antmicro/tlib](https://github.com/antmicro/tlib) | SEVONPEND generates an event for *any* exception entering the pending state, not only ones the CPU would accept. ARM DDI0403E B1.5.17 does not qualify it by whether the exception is enabled, and MeshCore's published nRF52 builds sleep on `WFE` expecting the wider behaviour. |
| [renode-infrastructure](https://github.com/MeshBench/renode-infrastructure) | `meshbench-main` | [renode/renode-infrastructure](https://github.com/renode/renode-infrastructure) | The C# half of that fix: the NVIC can answer the wider question, and setting the event flag now wakes a CPU already asleep. |
| [renode](https://github.com/MeshBench/renode) | `meshbench-main` | [renode/renode](https://github.com/renode/renode) | Ties the two together and builds a portable package in CI, runtime included. Also asserts both halves of the fix are in the tree it built. |

### What the QEMU fork carries

`meshbench-main` is the integration branch, and the default; each item below landed on its own
branch first, and each one was a board that would not boot, would not transmit,
or would not relay until it went in.

- **An SX1262 SPI device**, with the front-end module's enable line brought in
  from the board, and a socket — path or `host:port` — to the radio model.
- **A working GPIO implementation, and interrupts from a pin.** Upstream's
  write handler is empty, and RadioLib drives chip select as an ordinary GPIO,
  so without it the chip sees an unframed byte stream and the driver reports no
  chip present. DIO1 needs the interrupt: a radio that cannot raise one is a
  radio that never finishes a transmission.
- **The ESP32-S3's general-purpose SPI controllers.** Arduino's default
  `SPIClass` is HSPI, which is controller 2 on an ESP32 and controller **3** on
  an S3 — and only the flash controller's register layout was modelled, where a
  transfer starts on a different bit and the data sits at a different offset.
- **GPIO0 coming up high, as its pull-up makes it.** Every input read low out of
  reset. GPIO0 is a strapping pin, and reading it low is the program button held
  down, so MeshCore powered the board off after two minutes — every time, before
  it had adverted once.
- **A GigaDevice part's quad-enable bit.** The flash model knew those parts by
  name and handled the bit nowhere, so it could be written and never took —
  which failed `esp_flash_init_default_chip()` on the QIO-built S3 images and
  left them restarting for ever, 360 times in one probe.
- **The rest of a board**: an I2C controller and the panel on it, two devices
  sharing one SPI bus with a colour display, the board's own buttons, a
  keyboard and touch panel, an ADC that answers, and a card slot that keeps
  quiet. An unmodelled input is not a zero — it is a spin.
- **Peripheral windows that say what they swallowed**, so the next one of these
  is an hour rather than a week.

## The rest

| repository | what it is |
|---|---|
| [meshcore-native](https://github.com/MeshBench/meshcore-native) | Host and cross builds of MeshCore, the virtual SX1262, the bridge, and the radio model both emulators talk to |
| [meshbench-reports](https://github.com/MeshBench/meshbench-reports) | Studies of how MeshCore actually behaves, run on real firmware against a simulated radio and channel |
| [gio](https://github.com/MeshBench/gio) | Mirror of [Gio](https://gioui.org), the toolkit the workbench is built in, carrying a branch that adds Wayland layer-shell windows. The build tracks upstream today. |

## Not ours

[MeshCore](https://github.com/meshcore-dev/MeshCore) is upstream and
**unmodified** — the build points at a checkout and compiles it as it stands,
which is the whole basis of the claim that this runs real firmware. Board images
come from its releases.

The Nordic SoftDevice is not ours and cannot be redistributed: anyone running a
published nRF52 image supplies their own copy.

## Later

Meshtastic is plausible once the hardware emulation is done, since both stacks
run on the same boards and the same radio. MeshCore comes first.
