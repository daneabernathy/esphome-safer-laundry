# Safer Laundry

An [ESPHome](https://esphome.io) water leak monitor for the floor under a
washing machine. Two bare-wire probes use the ESP32's built-in touch
sensing, so the only parts are an ESP32 board, some wire and a USB power
supply.

When water bridges a probe for 2 seconds, Home Assistant gets a
**Laundry Water Leak** alert and a **Leak Trip** that stays latched,
even across reboots, until someone resets it once the floor is dry.

## Status

| Check | Result |
| --- | --- |
| Config validates (ESPHome 2026.9.1) | ✅ |
| Firmware compiles, all packages on (ESPHome 2026.9.1) | ✅ |
| GitHub Actions build | ❌ Not run yet |
| Leak probes and latched trip on hardware | ❌ Not tested in this version |
| BOOT button and remote wire reset on hardware | ❌ Not tested |
| Bluetooth proxy on hardware | ❌ Not tested |

An earlier, simpler version (probes, latch and reset button only) has
run on the author's board. See [CHANGELOG.md](CHANGELOG.md) for changes.

## Features

- Front and back leak probes, each with its own threshold.
- Latched leak trip that survives power cuts and only resets when both
  probes read dry.
- Three ways to reset: a button in Home Assistant, holding the board's
  BOOT button for 1 to 5 seconds, or touching two remote wires together
  (optional).
- Onboard blue LED lights while the trip is latched.
- Optional Bluetooth proxy to extend Home Assistant's Bluetooth range.
- Example Home Assistant automations: phone alert with a reset button,
  music on a speaker, and an alert if the monitor goes offline.

## Hardware

| Part | Notes |
| --- | --- |
| ESP32 DevKit V1 (ESP-WROOM-32) | Any `esp32dev` board with GPIO32/33 exposed |
| Hookup wire | Bare the ends for the probes |
| USB power supply and cable | Mount the board above floor level |

### Wiring

| Board pin | Connects to |
| --- | --- |
| GPIO32 (D32) | Front probe: a bare wire end on the floor |
| GPIO33 (D33) | Back probe: a bare wire end on the floor |
| GND | A bare ground wire about half an inch (12 mm) from each probe |
| GPIO4 (D4) + GND | Optional remote reset wires, run to somewhere reachable |

Water bridging a probe and its ground wire lowers the touch reading
below the threshold. Keep probe ends clear of anything metal that
could touch both.

## Setup

You need [ESPHome](https://esphome.io/guides/installing_esphome/) on a
computer (or the ESPHome Device Builder add-on in Home Assistant) and a
USB data cable.

1. Clone this repository and copy the secrets template:

   ```sh
   git clone https://github.com/daneabernathy/esphome-safer-laundry.git
   cd esphome-safer-laundry
   cp secrets.example.yaml secrets.yaml
   ```

2. Fill in `secrets.yaml`. The comments in the file explain each value.
3. Choose features: in `safer-laundry.yaml`, comment out any line under
   `packages:` you don't want (remote reset wires, Bluetooth proxy).
4. Validate, then flash over USB the first time:

   ```sh
   esphome config safer-laundry.yaml
   esphome run safer-laundry.yaml
   ```

   After that, updates can go over Wi-Fi.
5. Home Assistant discovers the device. Add it from **Settings > Devices
   & services**, and enter the API encryption key if asked.

If the board can't join your Wi-Fi it starts a hotspot named
**Safer Laundry Fallback** (password from `fallback_ap_password`, default
`safer-laundry`). Connect to it and enter new Wi-Fi details in the page
that opens.

## Calibrating the probes

Touch readings depend on wire length and routing, so check yours:

1. Set `setup_mode: true` under `esp32_touch:` and flash.
2. Watch `esphome logs safer-laundry.yaml`. Note the reading with each
   probe dry, then with a little water bridging it.
3. Set `front_probe_threshold` and `back_probe_threshold` (under
   `substitutions:`) between the wet and dry readings. The default is 700.
4. Set `setup_mode` back to `false` and flash again.

## Home Assistant entities

| Entity | Type | Meaning |
| --- | --- | --- |
| Laundry Water Leak | binary sensor (moisture) | Water detected right now |
| Leak Trip | binary sensor (problem) | Latched until reset |
| Washer Leak Front / Back | binary sensor (moisture) | Each probe on its own |
| Reset Leak Trip | button | Clears the trip if both probes are dry |
| Laundry Monitor Online | binary sensor | Device is connected |
| Reset Button Board / Remote | binary sensor | Physical reset inputs |
| WiFi Signal | sensor | Signal strength in dBm |

Entity IDs start with the device name, for example
`binary_sensor.safer_laundry_leak_trip`. Check yours under **Settings >
Devices & services > ESPHome**.

## Example automations

[homeassistant/automations.example.yaml](homeassistant/automations.example.yaml)
has four automations: alarm, cleared, reset from the phone notification,
and monitor offline. Replace the placeholder entity and action names
marked `CHANGE` before using them.

## Troubleshooting

- **Unstable or rebooting:** comment out the Bluetooth proxy package
  first. It uses the most memory.
- **False alarms:** lower the threshold a little or shorten the probe
  wires.
- **Trip won't reset:** a probe still reads wet. Check the
  Washer Leak Front and Back sensors.

## Contributing

Issues and pull requests are welcome. Please run
`esphome config safer-laundry.yaml` before opening a pull request; the
GitHub Actions workflow also compiles every change.

## Acknowledgements

Written by Dane Abernathy with help from
[Claude](https://www.anthropic.com/claude) (Anthropic), used through
Claude Code to draft configuration and documentation. Commits Claude
helped write carry a `Co-Authored-By: Claude` trailer. Every change is
reviewed by the maintainer; the Status table shows what has been tested
on real hardware.

## License

[MIT](LICENSE)
