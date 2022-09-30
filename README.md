# SH1107 driver

[![Crates.io](https://img.shields.io/crates/v/sh1107.svg)](https://crates.io/crates/sh1107)
[![Docs.rs](https://docs.rs/sh1107/badge.svg)](https://docs.rs/sh1107)

[![SH1107 display module showing the Rust logo](readme_banner.jpg?raw=true)](examples/image.rs)

I2C driver for the SH1107 OLED display written in 100% Rust

## [Documentation](https://docs.rs/sh1107)

## [Examples]

This crate uses [`probe-run`](https://crates.io/crates/probe-run) to run the examples. Once set up,
it should be as simple as `cargo run --example <example name> --release`. `--release` will be
required for some examples to reduce FLASH usage.

From [`examples/text.rs`](examples/text.rs):

```rust
#![no_std]
#![no_main]

use cortex_m_rt::{entry, exception, ExceptionFrame};
use embedded_graphics::{
    fonts::{Font6x8, Text},
    pixelcolor::BinaryColor,
    prelude::*,
    style::TextStyle,
};
use panic_semihosting as _;
use sh1107::{prelude::*, Builder};
use stm32f1xx_hal::{
    i2c::{BlockingI2c, DutyCycle, Mode},
    prelude::*,
    stm32,
};

#[entry]
fn main() -> ! {
    let dp = stm32::Peripherals::take().unwrap();

    let mut flash = dp.FLASH.constrain();
    let mut rcc = dp.RCC.constrain();

    let clocks = rcc.cfgr.freeze(&mut flash.acr);

    let mut afio = dp.AFIO.constrain(&mut rcc.apb2);

    let mut gpiob = dp.GPIOB.split(&mut rcc.apb2);

    let scl = gpiob.pb8.into_alternate_open_drain(&mut gpiob.crh);
    let sda = gpiob.pb9.into_alternate_open_drain(&mut gpiob.crh);

    let i2c = BlockingI2c::i2c1(
        dp.I2C1,
        (scl, sda),
        &mut afio.mapr,
        Mode::Fast {
            frequency: 100.khz().into(),
            duty_cycle: DutyCycle::Ratio2to1,
        },
        clocks,
        &mut rcc.apb1,
        1000,
        10,
        1000,
        1000,
    );

    let mut disp: GraphicsMode<_> = Builder::new().connect_i2c(i2c).into();

    disp.init().unwrap();
    disp.flush().unwrap();

    Text::new("Hello world!", Point::zero())
        .into_styled(TextStyle::new(Font6x8, BinaryColor::On))
        .draw(&mut display)
        .unwrap();

    Text::new("Hello Rust!", Point::new(0, 16))
        .into_styled(TextStyle::new(Font6x8, BinaryColor::On))
        .draw(&mut display)
        .unwrap();

    disp.flush().unwrap();

    loop {}
}

#[exception]
fn HardFault(ef: &ExceptionFrame) -> ! {
    panic!("{:#?}", ef);
}
```

## License

Licensed under either of

- Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE) or
  http://www.apache.org/licenses/LICENSE-2.0)
- MIT license ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

at your option.

### Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted for inclusion in the
work by you, as defined in the Apache-2.0 license, shall be dual licensed as above, without any
additional terms or conditions.
