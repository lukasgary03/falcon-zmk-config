# Falcon ZMK Config

Firmware configuration for the Falcon — a custom wireless split keyboard built on the [nice!nano v2](https://nicekeyboards.com/nice-nano/) running [ZMK Firmware](https://zmk.dev/).

---

## Hardware

| Component | Details |
|---|---|
| Microcontroller | nice!nano v2 × 3 (left half, right half, dongle) |
| Sensor | PMW3610 optical trackball (right half) |
| Keys per half | 17 (3×5 main + 2 thumb keys) |
| Total keys | 34 |
| Battery | 3.7V 200mAh LiPo × 2 (one per half) |
| Communication | BLE split via dongle (dongle connects to computer over USB-C) |
| Diode direction | Row to column (row2col) |

---

## Repository Structure

```
zmk-config/
├── .github/
│   └── workflows/
│       └── build.yml          # GitHub Actions — builds firmware automatically on push
└── config/
    ├── west.yml               # ZMK + module dependencies
    └── boards/shields/
        ├── falcon_left/       # Left half shield
        │   ├── falcon_left.conf
        │   └── falcon_left.overlay
        ├── falcon_right/      # Right half shield (includes trackball)
        │   ├── falcon_right.conf
        │   └── falcon_right.overlay
        └── falcon_dongle/     # Dongle shield (central, USB to computer)
            ├── falcon_dongle.conf
            ├── falcon_dongle.keymap
            └── falcon_dongle.overlay
```

---

## Wiring Guide

Both halves are wired identically for the key matrix. The right half has the trackball wired in addition. The dongle has no wiring — it plugs straight into USB.

### Key Matrix (both halves)

The matrix uses **row-to-column** diodes — the cathode (stripe end) of each diode faces toward the column wire. One diode per key switch.

Each key switch is wired as follows:
- One leg of the switch connects to the **row wire** for that row
- The other leg connects to the **anode** (non-stripe end) of a diode
- The **cathode** (stripe end) of the diode connects to the **column wire** for that column

```
nice!nano pin    Role      Connected to
──────────────────────────────────────────────────────────
D8               row 0  →  one leg of every switch in row 0 (top row)
D9               row 1  →  one leg of every switch in row 1 (middle row)
D16              row 2  →  one leg of every switch in row 2 (bottom row)
D10              row 3  →  one leg of both thumb switches

D20              col 0  →  diode cathodes in column 0 (outermost)
D19              col 1  →  diode cathodes in column 1
D18              col 2  →  diode cathodes in column 2
D15              col 3  →  diode cathodes in column 3 (inner thumb col)
D14              col 4  →  diode cathodes in column 4 (inner thumb col)
```

Per-half wiring diagram:

```
         col0   col1   col2   col3   col4
          D20    D19    D18    D15    D14
           |      |      |      |      |
row0 D8 ---SW->--SW->--SW->--SW->--SW->--|
row1 D9 ---SW->--SW->--SW->--SW->--SW->--|
row2 D16---SW->--SW->--SW->--SW->--SW->--|
row3 D10--------------------SW->--SW->--|
                             thumb keys

SW = key switch, -> = diode (stripe/cathode toward column)
```

### Trackball Wiring (right half only)

The PMW3610 connects via SPI. Run 6 wires between the breakout board and the nice!nano:

```
PMW3610 pin    nice!nano pin    Notes
────────────────────────────────────────────────────────────
VDD         →  3.3V             Power — use the 3.3V pin, not VCC/RAW
GND         →  any GND pin
SDIO        →  D2               Bidirectional data (single wire, no separate MISO)
SCLK        →  D3               SPI clock
NCS         →  D4               Chip select (active low)
MOTION/MoT  →  D5               Motion interrupt
```

### Battery Wiring (both halves)

```
Battery wire    nice!nano pin
──────────────────────────────────────────────────────────
Red  (+)     →  BAT+  top-right pin  (solder direct — no socket or header)
Black (-)    →  GND   top-left pin   (solder direct — no socket or header)
```

---

## Pin Mapping

Both halves wire their nice!nano identically for the key matrix. The right half additionally has the trackball connected.

### Key Matrix (both halves)

| Role | Nice!nano Pin | nRF Port |
|---|---|---|
| Row 0 | D8 | P1.04 |
| Row 1 | D9 | P1.06 |
| Row 2 | D16 | P0.10 |
| Row 3 (thumb) | D10 | P0.09 |
| Col 0 (outer) | D20 | P0.29 |
| Col 1 | D19 | P0.02 |
| Col 2 | D18 | P1.15 |
| Col 3 (inner) | D15 | P1.13 |
| Col 4 (inner) | D14 | P1.11 |

### Trackball — Right Half Only (PMW3610)

| Signal | Nice!nano Pin | nRF Port |
|---|---|---|
| SDIO (MOSI/MISO) | D2 | P0.17 |
| SCLK | D3 | P0.20 |
| nCS | D4 | P0.22 |
| MoT (interrupt) | D5 | P0.24 |
| VIN | 3.3V | — |
| GND | GND | — |

### Battery (both halves)

| Wire | Nice!nano Pin |
|---|---|
| Positive (red) | BAT+ (top right pin) |
| Negative (black) | GND (top left pin) |

> ⚠️ Do not install sockets or headers on the BAT+ and GND battery pins.
> Solder the JST connector wires directly to these pins.

---

## Physical Layout

```
┌─────┬─────┬─────┬─────┬─────┐   ┌─────┬─────┬─────┬─────┬─────┐
│  Q  │  W  │  E  │  R  │  T  │   │  Y  │  U  │  I  │  O  │  P  │
├─────┼─────┼─────┼─────┼─────┤   ├─────┼─────┼─────┼─────┼─────┤
│  A  │  S  │  D  │  F  │  G  │   │  H  │  J  │  K  │  L  │  ;  │
├─────┼─────┼─────┼─────┼─────┤   ├─────┼─────┼─────┼─────┼─────┤
│  Z  │  X  │  C  │  V  │  B  │   │  N  │  M  │  ,  │  .  │  /  │
└─────┴─────┴─────┴─────┼─────┤   ├─────┼─────┴─────┴─────┴─────┘
                         │ NAV │   │ SPC │
                         ├─────┤   ├─────┤
                         │SHFT │   │ SYM │
                         └─────┘   └─────┘
```

### Layer 1 — Nav (hold left inner thumb)

```
┌─────┬─────┬─────┬─────┬─────┐   ┌─────┬─────┬─────┬─────┬─────┐
│  1  │  2  │  3  │  4  │  5  │   │  6  │  7  │  8  │  9  │  0  │
├─────┼─────┼─────┼─────┼─────┤   ├─────┼─────┼─────┼─────┼─────┤
│ TAB │ ALT │CTRL │ GUI │     │   │  ←  │  ↓  │  ↑  │  →  │BKSP │
├─────┼─────┼─────┼─────┼─────┤   ├─────┼─────┼─────┼─────┼─────┤
│CTRL │     │     │     │ (BT)│   │HOME │PGDN │PGUP │ END │ DEL │
└─────┴─────┴─────┴─────┼─────┤   ├─────┼─────┴─────┴─────┴─────┘
                         │▓▓▓▓▓│   │ ENT │
                         ├─────┤   ├─────┤
                         │▓▓▓▓▓│   │     │
                         └─────┘   └─────┘
```

### Layer 2 — Sym (hold right inner thumb)

```
┌─────┬─────┬─────┬─────┬─────┐   ┌─────┬─────┬─────┬─────┬─────┐
│  !  │  @  │  #  │  $  │  %  │   │  ^  │  &  │  *  │  (  │  )  │
├─────┼─────┼─────┼─────┼─────┤   ├─────┼─────┼─────┼─────┼─────┤
│  `  │  ~  │  -  │  =  │  |  │   │  [  │  ]  │  {  │  }  │  '  │
├─────┼─────┼─────┼─────┼─────┤   ├─────┼─────┼─────┼─────┼─────┤
│CAPS │     │  _  │  +  │  \  │   │     │     │  <  │  >  │  ?  │
└─────┴─────┴─────┴─────┼─────┤   ├─────┼─────┴─────┴─────┴─────┘
                         │     │   │▓▓▓▓▓│
                         ├─────┤   ├─────┤
                         │     │   │▓▓▓▓▓│
                         └─────┘   └─────┘
```

### Layer 3 — Bluetooth (hold NAV, then bottom-right key on left half)

```
┌──────┬──────┬──────┬──────┬──────┐   ┌─────┬─────┬─────┬─────┬───────┐
│ BT 0 │ BT 1 │ BT 2 │ BT 3 │ BT 4 │   │     │     │     │     │BT CLR │
├──────┼──────┼──────┼──────┼──────┤   ├─────┼─────┼─────┼─────┼───────┤
│      │      │      │      │      │   │     │     │     │     │       │
├──────┼──────┼──────┼──────┼──────┤   ├─────┼─────┼─────┼─────┼───────┤
│      │      │      │      │▓▓▓▓▓▓│   │     │     │     │     │RESET  │
└──────┴──────┴──────┴──────┼──────┤   ├─────┼─────┴─────┴─────┴───────┘
                             │▓▓▓▓▓▓│   │     │
                             ├──────┤   ├─────┤
                             │▓▓▓▓▓▓│   │     │
                             └──────┘   └─────┘
```

`BT 0–4` selects a Bluetooth profile slot. `BT CLR` clears the current profile's pairing.
`RESET` resets the right peripheral (useful for re-pairing).

---

## Flashing Firmware

### Entering Bootloader Mode

Double-tap the **reset button** on the nice!nano within 0.5 seconds.
The board will appear as a USB mass storage device called **`NICENANO`**.

> If your nice!nano does not have an exposed reset button (some handwired builds
> omit it), you can briefly short the **RST pin to GND** twice in quick succession
> using a wire or tweezers.

### Flashing on Linux / Arch

After building (see below), three `.uf2` files are produced — one per shield.
Flash each nice!nano one at a time:

```bash
# Put the nice!nano into bootloader (double-tap reset)
# It mounts automatically. Find the mount point:
lsblk -o NAME,LABEL,MOUNTPOINT | grep NICENANO

# Copy the correct .uf2 — e.g. for the left half:
cp falcon_left-nice_nano//zmk.uf2 /run/media/$USER/NICENANO/

# The board reboots automatically once the file is copied.
# Repeat for the right half and dongle with their respective .uf2 files.
```

Flash order does not matter, but it is easiest to do the dongle last so you can
confirm both halves connect to it after flashing.

---

## Building Firmware

### Via GitHub Actions (recommended)

1. Push any change to this repository.
2. Go to the **Actions** tab on GitHub.
3. Wait for the build to complete (usually 3–5 minutes).
4. Download the `firmware` artifact — it contains all three `.uf2` files.

### Locally (optional)

Follow the [ZMK Getting Started guide](https://zmk.dev/docs/development/setup/native) to set up a local build environment, then:

```bash
west build -b nice_nano//zmk -- -DSHIELD=falcon_left
west build -b nice_nano//zmk -- -DSHIELD=falcon_right
west build -b nice_nano//zmk -- -DSHIELD=falcon_dongle
```

---

## Pairing / First-Time Setup

1. Flash all three nice!nanos.
2. Power on both halves (connect batteries or USB).
3. Plug the dongle into your computer.
4. The two halves will automatically find and pair with the dongle over BLE —
   no manual pairing step is needed for the split connection.
5. The dongle presents itself to your computer as a standard USB HID keyboard
   and mouse — no drivers required.

### Switching Computers

The dongle handles the host connection. To use the keyboard with a different
computer, simply unplug the dongle and plug it into the new machine.
If you want wireless host connections instead, use the BT layer to switch profiles.

---

## Sleep / Power Management

ZMK automatically deep-sleeps both halves after inactivity. Any key press wakes
the keyboard. There is no manual power switch — sleep is the off state.

The trackball uses `force-awake` mode while ZMK is active for instant response,
and drops into the sensor's built-in low-power downshift mode once ZMK goes idle.

To adjust sleep timing, edit `falcon_dongle.conf`:

```conf
CONFIG_ZMK_IDLE_TIMEOUT=600000       # idle after 10 min (milliseconds)
CONFIG_ZMK_SLEEP=y
CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=1800000 # deep sleep after 30 min idle
```

---

## Modules Used

| Module | Purpose |
|---|---|
| [zmk-pmw3610-driver](https://github.com/badjeff/zmk-pmw3610-driver) | PMW3610 trackball sensor driver for ZMK |
| [zmk-split-peripheral-input-relay](https://github.com/badjeff/zmk-split-peripheral-input-relay) | Relays trackball input events from the right peripheral to the dongle over BLE |

---

## Troubleshooting

**Trackball not responding**
The most common cause on nice!nano v2 is VCC not being ready before the sensor
initialises. `CONFIG_PMW3610_ALT_INIT_POWER_UP_EXTRA_DELAY_MS=1000` is already
enabled in `falcon_right.conf` to handle this.
If you still see `Incorrect product id 0xFF (expecting 0x3E)` in logs, try
increasing the value to `2000`.

**One half not connecting to the dongle**
- Confirm both halves have been flashed with their correct `.uf2` files.
- Press reset on the unresponsive half once to re-trigger BLE advertising.
- If still stuck, use the BT layer `BT CLR` key on the dongle and re-pair.

**Keys in the wrong positions**
The matrix transform in the overlay files maps physical row/column positions to
logical key positions. If a key is triggering the wrong keycode, the most likely
cause is a wiring swap on a row or column. Check the pin mapping table above
against your physical wires.

**Dongle not appearing as a USB device**
Confirm the dongle `.uf2` was flashed (not a peripheral `.uf2`).
The dongle shield sets `CONFIG_ZMK_USB=y` and `CONFIG_ZMK_SPLIT_BLE_ROLE_CENTRAL=y`.
A peripheral flashed onto the dongle will not enumerate as USB HID.
