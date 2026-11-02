# Gregorian dates, wall time, elapsed milliseconds

Import `temporalkit` from a sealed package. Requires existing dateparsekit and
timezonekit 0.1.0 packages. No host locale or native calendar parser is used.

Public API contracts:

## Install

From a Buoy consumer project with BPM installed:

```sh
bpm add temporalkit --git https://github.com/BuoyBay/temporalkit.git --rev refs/heads/main
```

BPM records the resolved commit in its lockfile. Keep that lockfile with the
consumer project for reproducible installs.
BPM also resolves `dateparsekit`, `timezonekit`.

## API entrypoints

- `New TemporalDateOnly(year, month, day)`
- `ToInvariant()`
- `New TemporalTimeOfDay(hour, minute, second = 0, millisecond = 0)`
- `New TemporalDurationMillis(milliseconds)`
- `AnalyzeTemporalDate(text, pattern = "yyyy-MM-dd")`

## Build from source

With the native `build-packages` driver from [devtools](https://github.com/BuoyBay/devtools)
on your `PATH`, run from this checkout:

```sh
build-packages --root "$PWD" --compile-consumer temporalkit
```

The driver checks source targets and compiles an import consumer. See the
[API guide](docs/API.md) for behavior, limits and any host prerequisites.

## Local Buoy setup

On your own machine, install Buoy and run:

```sh
buoy ai init --both
```

## Contributing

[Contributing](CONTRIBUTING.md) · [License](LICENSE)
