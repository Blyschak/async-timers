# Async Timers

[![Crates.io][crates-badge]][crates-url]
[![Unlicensed][licence-badge]][licence-url]
[![Documentation](https://docs.rs/async-timers/badge.svg)](https://docs.rs/async-timers)
[![Build Status][actions-badge]][actions-url]
[![codecov](https://codecov.io/github/Blyschak/async-timers/branch/main/graph/badge.svg?token=322R7ISIMY)](https://codecov.io/github/Blyschak/async-timers)
![LoC](https://raw.githubusercontent.com/Blyschak/async-timers/badges/badge.svg)

[crates-badge]: https://img.shields.io/crates/v/async-timers.svg
[crates-url]: https://crates.io/crates/async-timers
[licence-badge]: https://img.shields.io/badge/license-Unlicense-blue.svg
[licence-url]: https://github.com/Blyschak/async-timers/blob/master/LICENSE
[actions-badge]: https://github.com/Blyschak/async-timers/actions/workflows/build.yml/badge.svg
[actions-url]: https://github.com/Blyschak/async-timers/actions?query=branch%3Amain

This crate provides ```PeriodicTimer``` and ```OneshotTimer``` to be used in ```async``` context (tokio).

# Usage

[Example](examples/timers/main.rs)

```rust
use async_timers::{OneshotTimer, PeriodicTimer};
use tokio::time::Duration;

#[tokio::main]
async fn main() {
    let mut start_delay = OneshotTimer::expired();
    let mut stop_delay = OneshotTimer::expired();
    let mut periodic = PeriodicTimer::stopped();

    let mut exit_loop_delay = OneshotTimer::scheduled(Duration::from_secs(10));

    // The following call will block forever
    // start_delay.tick().await;

    // Useful in event loop in select! blocks

    start_delay.schedule(Duration::from_secs(2));
    println!("Periodic timer will start in 2 sec");

    loop {
        tokio::select! {
            _ = start_delay.tick() => {
                // Start periodic timer with period of 500 ms
                periodic.start(Duration::from_millis(500));

                stop_delay.schedule(Duration::from_secs(3));
                println!("Periodic timer will stop in 3 sec");
            }
            _ = stop_delay.tick() => {
                // Stop periodic timer
                periodic.stop();
                exit_loop_delay.schedule(Duration::from_secs(3));
                println!("Periodic timer stopped. Will exit in 3 sec");
            }
            _ = periodic.tick() => {
                println!("Periodic tick!");
            }
            _ = exit_loop_delay.tick() => {
                println!("Bye!");
                break;
            }
        }
    }
}
```
