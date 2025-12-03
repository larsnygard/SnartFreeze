# SnartFreeze

ESPHome-based smart freezer controller for Home Assistant integration.

## Features

- **Temperature Monitoring**: DS18B20 sensor for accurate freezer temperature readings
- **Compressor Control**: Relay-based control with hysteresis to prevent short-cycling
- **Door Monitoring**: Optional magnetic door sensor to detect when the freezer is open
- **Home Assistant Integration**: Full integration via ESPHome native API
- **Web Interface**: Local web server for status monitoring and control
- **OTA Updates**: Over-the-air firmware updates
- **Configurable Setpoints**: Adjustable target temperature and hysteresis values
- **Safety Features**: Automatic compressor shutoff if temperature sensor fails

## Hardware Requirements

- ESP32 or ESP8266 development board
- DS18B20 waterproof temperature sensor
- Relay module (suitable for your compressor's voltage/current)
- Door/window magnetic sensor (optional)
- 4.7kΩ pull-up resistor for DS18B20 data line

## Wiring Diagram

```
ESP32 Pin Assignments:
┌─────────────────┬───────────────────────────────┐
│ GPIO Pin        │ Connection                    │
├─────────────────┼───────────────────────────────┤
│ GPIO4           │ DS18B20 Data (with 4.7kΩ      │
│                 │ pull-up to 3.3V)              │
│ GPIO16          │ Relay Control (compressor)    │
│ GPIO17          │ Door Sensor                   │
│ 3.3V            │ DS18B20 VCC, Door Sensor VCC  │
│ GND             │ DS18B20 GND, Relay GND,       │
│                 │ Door Sensor GND               │
└─────────────────┴───────────────────────────────┘
```

### DS18B20 Wiring

```
DS18B20 (Waterproof):
  Red    → 3.3V
  Black  → GND
  Yellow → GPIO4 (with 4.7kΩ resistor to 3.3V)
```

## Installation

### Prerequisites

1. Install ESPHome:
   ```bash
   pip install esphome
   ```

2. Clone this repository:
   ```bash
   git clone https://github.com/larsnygard/SnartFreeze.git
   cd SnartFreeze
   ```

3. Create your secrets file:
   ```bash
   cp secrets.yaml.example secrets.yaml
   ```

4. Edit `secrets.yaml` with your credentials:
   ```yaml
   wifi_ssid: "Your_WiFi_Name"
   wifi_password: "Your_WiFi_Password"
   ap_password: "Fallback_AP_Password"
   api_encryption_key: "your-32-byte-base64-key"
   ota_password: "your-ota-password"
   ```

   Generate an API encryption key:
   ```bash
   openssl rand -base64 32
   ```

### Flashing the Device

1. Connect your ESP32 via USB

2. Compile and upload:
   ```bash
   esphome run snartfreeze.yaml
   ```

3. For subsequent OTA updates:
   ```bash
   esphome run snartfreeze.yaml --device snartfreeze.local
   ```

## Home Assistant Integration

After flashing, the device will automatically be discovered by Home Assistant if you have the ESPHome integration installed.

### Available Entities

| Entity | Type | Description |
|--------|------|-------------|
| `sensor.snartfreeze_freezer_temperature` | Sensor | Current freezer temperature |
| `switch.snartfreeze_compressor` | Switch | Compressor on/off control |
| `binary_sensor.snartfreeze_freezer_door` | Binary Sensor | Door open/closed state |
| `number.snartfreeze_target_temperature` | Number | Target temperature setpoint |
| `number.snartfreeze_temperature_hysteresis` | Number | Temperature hysteresis |
| `sensor.snartfreeze_wifi_signal` | Sensor | WiFi signal strength |
| `sensor.snartfreeze_uptime` | Sensor | Device uptime |

### Example Automations

**Alert when door is open too long:**
```yaml
automation:
  - alias: "Freezer Door Open Alert"
    trigger:
      - platform: state
        entity_id: binary_sensor.snartfreeze_freezer_door
        to: "on"
        for: "00:02:00"
    action:
      - service: notify.mobile_app
        data:
          title: "Freezer Alert"
          message: "Freezer door has been open for 2 minutes!"
```

**Alert on high temperature:**
```yaml
automation:
  - alias: "Freezer Temperature Alert"
    trigger:
      - platform: numeric_state
        entity_id: sensor.snartfreeze_freezer_temperature
        above: -10
        for: "00:10:00"
    action:
      - service: notify.mobile_app
        data:
          title: "Freezer Alert"
          message: "Freezer temperature is {{ states('sensor.snartfreeze_freezer_temperature') }}°C!"
```

## Configuration

### Adjusting Default Settings

Edit `snartfreeze.yaml` to change default values:

```yaml
substitutions:
  name: snartfreeze
  friendly_name: SnartFreeze
  target_temp: "-18"    # Default target temperature in °C
  hysteresis: "2"       # Temperature swing before compressor cycles
```

### Changing GPIO Pins

If using different pins, update the configuration:

```yaml
one_wire:
  - platform: gpio
    pin: GPIO4  # Change to your data pin

switch:
  - platform: gpio
    pin:
      number: GPIO16  # Change to your relay pin

binary_sensor:
  - platform: gpio
    pin:
      number: GPIO17  # Change to your door sensor pin
```

## Safety Considerations

⚠️ **IMPORTANT**: Working with mains voltage is dangerous. The relay module controls your freezer's compressor which operates on mains voltage.

- Use appropriate relay modules rated for your freezer's power requirements
- Ensure proper electrical isolation
- Consider using a qualified electrician for mains wiring
- The software includes safety features, but hardware safety should be your primary concern

## Troubleshooting

### Temperature sensor not detected
- Check wiring and ensure the 4.7kΩ pull-up resistor is connected
- Verify the sensor address in ESPHome logs

### WiFi connection issues
- Device creates fallback AP "SnartFreeze Fallback" when WiFi fails
- Connect to the fallback AP to reconfigure

### Compressor not responding
- Check relay wiring and ensure adequate power supply
- Verify GPIO pin configuration matches your hardware

## License

This project is licensed under the GNU General Public License v3.0 - see the [LICENSE](LICENSE) file for details.

## Contributing

Contributions are welcome! Please feel free to submit pull requests or create issues for bugs and feature requests.
