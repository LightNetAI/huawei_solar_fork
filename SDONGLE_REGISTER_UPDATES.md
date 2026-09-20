# SDongle Register Updates from esp32_huawei_modbus_sniffer

## Source
Registers extracted from: https://github.com/NemesisSVK/esp32_huawei_modbus_sniffer
- REGISTER_LIST_MODBUS_SDONGLE_UPLINK.md
- REGISTER_CATALOG.md

## Current SDongle Registers (in huawei_solar)
From sensor.py lines 1850-1881:
- `SDONGLE_TOTAL_INPUT_POWER` (37498) - kW
- `SDONGLE_LOAD_POWER` (37500) - kW  
- `SDONGLE_GRID_POWER` (37502) - kW
- `SDONGLE_TOTAL_BATTERY_POWER` (37504) - kW
- `SDONGLE_TOTAL_ACTIVE_POWER` (37516) - kW

## New SDongle Registers to Add

### From REGISTER_LIST_MODBUS_SDONGLE_UPLINK.md

#### Fast Meter Power Registers (FC0x41 proprietary)
These provide faster update rates than standard FC03 registers:
- Address 16300: `meter_active_power_fast` - I32, 1W, GRP_METER
- Address 16305: `grid_a_power_fast` - I16, 1W, GRP_METER
- Address 16307: `grid_b_power_fast` - I16, 1W, GRP_METER
- Address 16309: `grid_c_power_fast` - I16, 1W, GRP_METER
- Address 16312: `meter_reactive_power_fast` - I16, 1var, GRP_METER

#### SDongle Aggregates (from REGISTER_CATALOG.md section 0.2)
Already partially implemented, but verify these match:
- 37498: `sdongle_pv_power` - U32, 1000kW (matches SDONGLE_TOTAL_INPUT_POWER)
- 37500: `sdongle_load_power` - U32, 1000kW (matches SDONGLE_LOAD_POWER)
- 37502: `sdongle_grid_power` - I32, 1000kW (matches SDONGLE_GRID_POWER)
- 37504: `sdongle_battery_power` - I32, 1000kW (matches SDONGLE_TOTAL_BATTERY_POWER)
- 37516: `sdongle_total_power` - I32, 1000kW (matches SDONGLE_TOTAL_ACTIVE_POWER)

#### Additional Meter Registers (via SDongle)
These are visible on the SDongle uplink bus and should be added:

**Grid Meter Basic (371xx series):**
- 37100: `meter_status` - U16
- 37101: `grid_a_voltage` - I32, 10V
- 37103: `grid_b_voltage` - I32, 10V
- 37105: `grid_c_voltage` - I32, 10V
- 37107: `grid_a_current` - I32, 100A
- 37109: `grid_b_current` - I32, 100A
- 37111: `grid_c_current` - I32, 100A
- 37113: `meter_active_power` - I32, 1W
- 37115: `meter_reactive_power` - I32, 1var
- 37117: `meter_power_factor` - I16, 1000
- 37118: `meter_frequency` - I16, 100Hz
- 37119: `grid_exported_energy` - I32ABS, 100kWh
- 37121: `grid_imported_energy` - I32, 100kWh
- 37123: `grid_reactive_energy` - I32, 100kvarh
- 37125: `meter_type` - U16
- 37126: `grid_ab_voltage` - I32, 10V
- 37128: `grid_bc_voltage` - I32, 10V
- 37130: `grid_ca_voltage` - I32, 10V
- 37132: `grid_a_power` - I32, 1W
- 37134: `grid_b_power` - I32, 1W
- 37136: `grid_c_power` - I32, 1W
- 37138: `meter_type_check` - U16

**Direct Meter Channel (21xx series - F32 floats):**
These are from the direct DTSU666-H meter bus (FC03 float registers):
- 2102-2222: Various F32 registers for meter data
- Includes: currents, voltages, powers, energy totals, power factors
- All IEEE754 float32 format with scale=1

## Implementation Notes

### huawei-solar Library Dependency
The integration imports register names from `huawei_solar` package:
```python
from huawei_solar import register_names as rn
```

To add new registers, we need to either:
1. Fork and modify the huawei-solar library (github.com/wlcrs/huawei-solar-lib)
2. Request the register additions upstream
3. Use a custom register mapping in the integration

### Recommended Approach
1. Fork huawei-solar-lib and add the new register definitions
2. Update the integration's manifest.json to use the forked library
3. Add new sensor entities in sensor.py for the SDongle-specific registers
4. Update the device detection to properly identify SDongle capabilities

### Register Naming Convention
Following the existing pattern in huawei-solar-lib:
- Use uppercase with underscores: `METER_ACTIVE_POWER_FAST`
- Prefix with device type: `SDONGLE_`, `METER_`, `GRID_`
- Match register address ordering in sensor.py

### Priority Implementation
Based on the sniffer project's priority scale:
- **P1 (High)**: Fast power registers (16300-16312) - real-time monitoring
- **P1 (High)**: Grid meter registers (371xx) - essential for net metering
- **P2 (Medium)**: Direct meter F32 registers (21xx) - alternative data source
- **P3 (Low)**: Proprietary FC0x41 addresses - experimental

## Testing Considerations
- Requires SDongleA-05 or compatible dongle
- Test with firmware V200R022C10SPC110 or later
- Verify register addresses match your inverter model (SUN2000-8K-MAP0 tested)
- Some registers may require elevated permissions

## Files to Modify

### huawei-solar-lib fork:
- `src/huawei_solar/registers.py` - Add new register definitions

### huawei_solar integration fork:
- `manifest.json` - Point to forked library
- `sensor.py` - Add SDONGLE_SENSOR_DESCRIPTIONS entries
- Possibly `__init__.py` - Device detection updates
- Possibly `update_coordinator.py` - Register polling updates
