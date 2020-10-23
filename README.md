# Rust bindings for the liblsl library

Safe high-level bindings for the `liblsl` library, the main network library of
the [lab streaming layer](https://github.com/sccn/labstreaminglayer) (LSL).
 
The lab streaming layer is a peer-to-peer pub/sub system on the local network that allows for 
real-time exchange of multi-channel time series (plus their meta-data) between applications and 
machines, with built-in cross-device time synchronization.

The most common use case is in lab spaces to make, e.g., instrument data from different pieces of
hardware (e.g., sensors) accessible in real time to client programs (e.g., experimentation scripts, 
recording programs, stream viewers, or live processing software). One of the main features of LSL 
is the uniform API that allows clients to read formatted multi-channel data from many device types 
(such as EEG, eye tracking, audio, human interface devices, events, etc.) with the same few lines 
of code.

## Examples

See the scripts in the [examples/](https://github.com/intheon/liblsl-rust/tree/main/examples) folder 
of the git repo for the fully-commented version of the below code. Examples for other use cases can 
be found there too.

#### Sending data to LSL
```rust
use lsl;
use lsl::Pushable;

fn main() -> Result<(), lsl::Error> {
    // declare a stream and create an outlet
    let info = lsl::StreamInfo::new(
        "BioSemi", "EEG", 8, 100.0,
        lsl::ChannelFormat::Float32, "myid234365")?;
    let outlet = lsl::StreamOutlet::new(&info, 0, 360)?;

    // stream some 8-channel data
    loop {
        let sample = vec![1, 2, 3, 4, 5, 6, 7, 8];
        outlet.push_sample(&sample)?;
        std::thread::sleep(std::time::Duration::from_millis(10));
    }
}
```

#### Receiving data from LSL
```rust
use lsl;
use lsl::Pullable;

fn main() -> Result<(), lsl::Error> {

    // resolve a data stream and create an inlet to read from it
    let res = lsl::resolve_bypred("name='BioSemi' and type='EEG'", 1, lsl::FOREVER)?;
    let inl = lsl::StreamInlet::new(&res[0], 360, 0, true)?;

    // read the streaming data and print the multi-channel samples 
    loop {
        let (sample, ts): (Vec<f32>, _) = inl.pull_sample(lsl::FOREVER)?;
        println!("got {:?} at time {}", sample, ts);
    }
}
```

## Adding as a dependency

Add a new entry to your dependencies in `Cargo.toml`:

```toml
[dependencies]
lsl = "0.1.0"
```

Add the following to your code:

```rust
use lsl;
```

## Getting the source code

```
git clone --recurse-submodules https://github.com/intheon/liblsl-rust
``` 

## Compiling

This crate currently links against the `liblsl` library statically (dynamic linking against the 
system library, if present, is planned for a future release). To compile the crate (and the native 
library), you need:

* [CMake](https://cmake.org/download/) 3.12 or higher. If you install that via your system's package 
  manager (e.g., `brew install cmake` on MacOS if you have [Homebrew](https://brew.sh/) or 
  `sudo apt-get install cmake` on Ubuntu), this might already pull in the second dependency (the 
  system compiler), so building may already work. 
* A system compiler; on Windows [MS Visual Studio](https://visualstudio.microsoft.com/) 2019 (older 
  versions might work too), on Linux `build-essentials`, and on MacOS XCode CLI). 

With these dependencies met, the package should compile for you. The following targets have been
tested with this crate so far:
```
* x86_64-pc-windows-msvc
* x86_64-unknown-linux-gnu
* x86_64-apple-darwin
```

## API Documentation

Doocumentation for the most recent release can be found at [docs.rs](https://docs.rs/lsl).

#### The [lsl-sys](https://crates.io/crates/lsl-sys) crate

The `lsl` crate depends on a lower-level `lsl-sys` crate that provides the raw (unsafe) native 
library bindings. Using that crate directly is not recommended (you might as well be coding in C). 
These bindings are autogenerated and can be updated to a newer upstream native library version by 
following the instructions in that crate's [readme](https://github.com/intheon/liblsl-rust/blob/main/lsl-sys/README.md).

## Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted for inclusion 
in the `lsl` crate by you, shall be licensed as MIT, without any additional terms or conditions.
