# Huawei Solar Integration - SDongle Enhanced Fork

This is a fork of the [wlcrs/huawei_solar](https://github.com/wlcrs/huawei_solar) Home Assistant integration with enhanced SDongle support using register data from the [esp32_huawei_modbus_sniffer](https://github.com/NemesisSVK/esp32_huawei_modbus_sniffer) project.

## What's New

### Enhanced SDongle Register Support

This fork adds comprehensive support for SDongleA-05 and compatible dongles by incorporating registers discovered through passive Modbus RTU sniffing. The new registers provide:

#### 1. Fast Meter Power Registers (FC0x41 Proprietary)
These registers provide faster update rates than standard Modbus FC03 registers, ideal for real-time monitoring:

- `meter_active_power_fast` (16300) - Total meter active power in Watts
- `grid_a_power_fast` (16305) - Phase A grid power in Watts
- `grid_b_power_fast` (16307) - Phase B grid power in Watts
- `grid_c_power_fast` (16309) - Phase C grid power in Watts
- `meter_reactive_power_fast` (16312) - Total reactive power in var

#### 2. Comprehensive Grid Meter Data
Full meter telemetry visible through the SDongle uplink bus:

**Voltages:**
- Phase voltages: `grid_a_voltage_meter`, `grid_b_voltage_meter`, `grid_c_voltage_meter` (37101-37105)
- Line voltages: `grid_ab_voltage_meter`, `grid_bc_voltage_meter`, `grid_ca_voltage_meter` (37126-37130)

**Currents:**
- Phase currents: `grid_a_current_meter`, `grid_b_current_meter`, `grid_c_current_meter` (37107-37111)

**Power:**
- Active power: `meter_active_power` (37113)
- Reactive power: `meter_reactive_power` (37115)
- Per-phase power: `grid_a_power_meter`, `grid_b_power_meter`, `grid_c_power_meter` (37132-37136)

**Energy:**
- Exported energy: `grid_exported_energy` (37119)
- Imported energy: `grid_imported_energy` (37121)
- Reactive energy: `grid_reactive_energy` (37123)

**Other:**
- Power factor: `meter_power_factor` (37117)
- Frequency: `meter_frequency` (37118)
- Meter type: `meter_type` (37125)
- Meter status: `meter_status` (37100)

## Installation

### Prerequisites

- Home Assistant 2024.1 or later
- Python 3.12+ (for the huawei-solar library)
- Huawei SUN2000 inverter with SDongleA-05 or compatible
- SDongle firmware V200R022C10SPC110 or later recommended

### Installation Steps

1. **Remove the original integration** (if installed):
   - Go to Settings → Devices & Services
   - Find "Huawei Solar" integration
   - Remove it (your configuration will be preserved)

2. **Install this fork via HACS**:
   - Add this repository as a custom repository in HACS
   - Search for "Huawei Solar SDongle Enhanced"
   - Install the integration

3. **Or install manually**:
   ```bash
   cd /config/custom_components
   git clone https://github.com/ics/huawei_solar_fork.git huawei_solar
   ```

4. **Restart Home Assistant**

5. **Re-add the integration**:
   - Settings → Devices & Services → Add Integration
   - Select "Huawei Solar"
   - Follow the configuration wizard

## Configuration

### Connection Settings

When configuring the integration:

1. **Connection Type**: Select "Network" for SDongle connection
2. **Host**: Enter the SDongle's IP address (set a static IP!)
3. **Port**: Typically `502` or `6607`
4. **Slave ID**: Typically `1` for SDongle
5. **Advanced: Elevate Permissions**: Check this for full register access

### Entity Enablement

By default, the following new entities are enabled:
- `meter_active_power_fast` - For real-time power monitoring
- `meter_active_power` - Standard meter active power
- `grid_exported_energy` - For net metering
- `grid_imported_energy` - For consumption tracking

Other diagnostic entities are disabled by default to reduce clutter. Enable them as needed from the entity registry.

## Technical Details

### Register Sources

The new registers come from two sources:

1. **FC0x41 Proprietary Registers** (16300-16312):
   - Fast-update proprietary Modbus function code
   - Discovered through traffic analysis by esp32_huawei_modbus_sniffer
   - Provides sub-second update rates vs. standard 1-5 second polling

2. **Standard FC03 Registers** (37100-37138):
   - Documented in Huawei Modbus interface specifications
   - Available through the SDongle uplink bus
   - Comprehensive meter telemetry

### huawei-solar Library Fork

This integration requires a modified version of the huawei-solar library that includes the new register definitions. The library fork is located at:
[https://github.com/ics/huawei-solar-lib-fork](https://github.com/ics/huawei-solar-lib-fork)

The manifest.json automatically installs this forked version.

### Compatibility

**Tested Hardware:**
- Inverter: SUN2000-8K-MAP0
- Inverter Firmware: V200R024C00SPC106
- Dongle: SDongleA-05
- Dongle Firmware: V200R022C10SPC110
- Power Meter: DTSU666-H

**Likely Compatible:**
- Other SUN2000 MA series inverters
- SDongleA-01, SDongleA-03 (untested)
- Other Huawei meters (DTSU666-W, etc.)

## Troubleshooting

### No Data from New Registers

1. **Check SDongle Connection**: Ensure the SDongle is properly connected and has network access
2. **Verify Slave ID**: The SDongle typically uses slave ID 1
3. **Check Permissions**: Some registers require elevated permissions
4. **Review Logs**: Check Home Assistant logs for Modbus communication errors

### Modbus Timeouts

The SDongle has specific timing requirements:
- Add `connectdelay: 1s` in advanced configuration if experiencing timeouts
- Don't poll faster than once per second
- Ensure no other Modbus clients are connected simultaneously

### Missing Registers

Not all registers may be available on your specific hardware/firmware combination:
- Check your SDongle firmware version
- Some registers require specific meter models
- Proprietary FC0x41 registers may vary by firmware

## Contributing

This fork incorporates data from the excellent [esp32_huawei_modbus_sniffer](https://github.com/NemesisSVK/esp32_huawei_modbus_sniffer) project. If you discover additional registers or have improvements:

1. Test with your hardware
2. Document register addresses and scaling
3. Submit a pull request with test data

## Credits

- Original integration: [wlcrs/huawei_solar](https://github.com/wlcrs/huawei_solar)
- Register discovery: [NemesisSVK/esp32_huawei_modbus_sniffer](https://github.com/NemesisSVK/esp32_huawei_modbus_sniffer)
- huawei-solar library: [wlcrs/huawei-solar-lib](https://github.com/wlcrs/huawei-solar-lib)

## License

Same as the original huawei_solar integration (GPL-3.0)

## Support

- Issues: [GitHub Issues](https://github.com/ics/huawei_solar_fork/issues)
- Documentation: [Original Wiki](https://github.com/wlcrs/huawei_solar/wiki)
- Community: [Home Assistant Community Forum](https://community.home-assistant.io/)
