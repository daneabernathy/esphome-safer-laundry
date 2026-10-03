# Changelog

All notable changes to this project are documented here. The format is
based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and
this project uses [Semantic Versioning](https://semver.org/).

## [Unreleased]

First public version. Compiles with ESPHome 2026.9.1; hardware testing
pending before the 0.1.0 release.

### Added

- Front and back touch leak probes (GPIO32, GPIO33) with configurable
  thresholds.
- Latched leak trip, restored across reboots, cleared only when both
  probes read dry.
- Reset from Home Assistant, the BOOT button (GPIO0, hold 1-5 s) and
  optional remote reset wires (GPIO4).
- Onboard LED (GPIO2) on while the trip is latched.
- Optional Bluetooth proxy package.
- Example Home Assistant automations.
- GitHub Actions build check.
