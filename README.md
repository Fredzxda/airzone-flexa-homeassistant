# Airzone FLEXA 1.x — Local Home Assistant integration

[Documentation en français](README.fr.md)

**Bring legacy Airzone FLEXA 1.x zoning systems into Home Assistant without an Airzone cloud gateway.**

This community project documents a practical, **local-only** integration for older Airzone FLEXA installations for which a suitable cloud gateway is not available. It reuses the existing FLEXA controller, wireless Tacto thermostats and motorized dampers.

## Why this project?

- **No Airzone cloud dependency:** communication between Home Assistant and the FLEXA controller stays on your local network.
- **Low hardware cost:** an **EW41 serial-to-Ethernet gateway**, approximately **€30** (indicative price; seller and model may vary).
- **Reuse existing equipment:** no replacement of the controller, thermostats or dampers required in the tested installation.
- **Bidirectional control:** changes from Home Assistant and from the existing Tacto controls can be reflected on both sides.
- **Optional voice assistants:** Google Home integration can be added separately, but is **not required** for local control.

## Architecture

```text
Home Assistant
     |
     | Modbus TCP (local LAN)
     v
EW41 serial/Ethernet gateway (~€30)
     |
     | Modbus RTU / RS485
     v
Airzone FLEXA 1.x controller
     |
     +-- Existing wireless Tacto thermostats
     +-- Existing motorized zone dampers
```

The tested installation uses a Modbus TCP connection to the EW41 on port 502, with RTU slave address 1. **Network addresses, passwords and household-specific entity names are deliberately excluded** from published examples.

## Confirmed scope on one installation

- Four individually controlled zones using Home Assistant Modbus and template-based climate entities.
- Zone enable/disable, room temperature and temperature setpoint.
- Night preset and zone state bits.
- Global system mode: off, cooling, ventilation and heating.
- Reading temperature limits from Modbus.
- Home Assistant and physical Tacto control remain usable together.

**Compatibility notice:** these observations come from a legacy FLEXA 1.x installation. They are not a guarantee of compatibility with every FLEXA controller or firmware revision.

## Register map — tested configuration

| Function | Zone 1 | Zone 2 | Zone 3 | Zone 4 |
| --- | ---: | ---: | ---: | ---: |
| Zone state bitfield | 256 | 512 | 768 | 1024 |
| Minimum setpoint | 257 | 513 | 769 | 1025 |
| Maximum setpoint | 258 | 514 | 770 | 1026 |
| Target temperature | 259 | 515 | 771 | 1027 |
| Room temperature | 266 | 522 | 778 | 1034 |

Global mode is read from holding register **0**. All temperature registers shown here use a scale factor of **0.1** in the tested setup.

For the zone-state bitfield, observed meanings include bit 0 (night), bit 1 (schedules disabled), and bit 2 (zone enabled). **Writes must preserve unrelated bits**; do not write a fixed value to a complete bitfield without reading its current value.

## Project status

**Documentation and configuration extraction in progress.** The published README describes a working personal installation; a cleaned, reproducible configuration and step-by-step installation guide are still being prepared. Do not treat this repository as a plug-and-play integration yet.

## Safety and limitations

- Incorrect Modbus writes may interfere with HVAC operation. Verify wiring, register addresses and controller compatibility before enabling writes.
- Start with conservative polling; the legacy bus can be sensitive to excessive requests.
- Keep the EW41 and Home Assistant on a trusted local network. Do not expose Modbus TCP port 502 to the internet.
- This is an independent community project, **not affiliated with or endorsed by Airzone**.
- Third-party documentation and software retain their respective rights.

## License

This project is intended to be distributed under **GNU GPL v3.0**. See the repository's `LICENSE` file for the applicable license text.
