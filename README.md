# rp2040

The RP2040's register map, as sysl constants — the chip on the original Raspberry Pi Pico.

*An independent sysl package for the RP2040. It is not affiliated with or endorsed by Raspberry Pi Ltd.*

Base addresses, register offsets, field masks and shifts, and the vendor's own names for the values a
field takes. One module per peripheral, 37 of them, 15,313 constants.

```
dependencies {
  rp2040 { git = "github.com/sysl-lang/rp2040", version = "0.1.0" }
}
```

```sysl
import sh.sysl.rp2040.io_bank0

// Point GPIO 25 -- the Pico's onboard LED -- at the single-cycle IO block, which is what a plain
// output pin is.
val ctrl = io_bank0.BASE + io_bank0.GPIO25_CTRL_OFFSET
val funcsel = (read(ctrl) & io_bank0.GPIO25_CTRL_FUNCSEL_MASK) >> io_bank0.GPIO25_CTRL_FUNCSEL_SHIFT

if funcsel == io_bank0.GPIO25_CTRL_FUNCSEL_SIO_25 then ...
```

## It is generated, and must not be edited

Every `.sysl` file here comes from Raspberry Pi's own `RP2040.svd` — the CMSIS description shipped
inside the pico-sdk — by way of [sysl-lang/svd](https://github.com/sysl-lang/svd):

```
svd constants RP2040.svd -o . -m sh.sysl.rp2040
```

A hand edit is lost the next time the chip is regenerated. A correction belongs in the generator, or
in a newer SVD.

**`sh/sysl/rp2040/tests.sysl` is the one exception** — the only hand-written file, and the guard on
regeneration. It checks a handful of addresses and field positions against the **datasheet** rather
than against the SVD, so that a generator change or a filter left switched on has something
independent to fail against. `sysl test .` runs it.

## This is not the RP2350's map with a name changed

`sh.sysl.rp2350` is the newer chip — the Pico 2 — and the two are far less alike than the naming
suggests. They share a vendor, a house style and the 0x40000000 APB base, and then diverge:

| | RP2040 | RP2350 |
|---|---|---|
| core | two Cortex-M0+ (Armv6-M) | two Cortex-M33, or two RV32 Hazard3 |
| sysl target | `thumbv6m-freestanding` | `thumb-freestanding-softfp`, `riscv32-freestanding` |
| `IO_BANK0` | `0x40014000` | `0x40028000` |
| PIO blocks | two | three |
| the timer | `TIMER` | `TIMER0` |
| peripherals | 37 | 54 |

A driver written against one compiles against the other and addresses the wrong silicon, which is why
a program names its map deliberately rather than reaching for whichever is installed.

## What it does not do

**It knows nothing about a board.** No pin assignments, no "the LED is on GPIO 25" — that line is in
the example above because it is *this* board's wiring, and the package does not say it. A board's
wiring is a board's business.

**It knows nothing about drivers.** It does not know what SPI *is*; it knows where `SPI0_BASE` is and
which bits of `SSPCR0` mean what. A driver is a different package.

**It costs the binary nothing.** Constants nobody names do not reach the executable, so a complete
map is free at run time. At compile time it is the whole package rather than the module imported —
see `package.hocon`, which has the measurement.

## Licence

ISC.
