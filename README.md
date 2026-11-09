# ESPHome Component TFMini

These ToF (time of flight) range finder sensors are compact, self-contained range finders. They support both UART and I<sub>2</sub>C communication but this component only supports UART communication.

This component supports the following Benewake LiDAR Range Finder Sensors.

TFMini Plus|TFMini-S| TFLuna
:---------:|:------:|:-----:
<img src="https://en.benewake.com/uploadfiles/2023/03/20230328182059608.png"> | <img src="https://en.benewake.com/uploadfiles/2023/03/20230328182039063.jpg"> | <img src="https://en.benewake.com/uploadfiles/2023/03/20230328182002559.jpg">

```yaml
# Sample configuration entry example
external_components:
  - source:
      type: git
      url: https://github.com/mikelawrence/esphome-component-tfmini

uart:
  - id: tfmini_uart
    tx_pin: GPIO17
    rx_pin: GPIO18
    baud_rate: 115200
    data_bits: 8
    parity: NONE
    stop_bits: 1

sensor:
  - platform: tfmini
    uart_id: tfmini_uart
    model: TFMINI_S
    distance:
      name: "TFMini Distance"
    signal_strength:
      name: "TFMini Signal Strength"
```

## Component

The UART bus must provide both TX and RX pins and use 115200 baud, 8 data bits, no parity, and one stop bit.

### Configuration Variables

* **model** (**Required**, enum): The model of the range finder sensor. Options are `TFMINI_PLUS`, `TFMINI_S` or `TF_LUNA`.
* **uart_id** (*Optional*, [ID](https://esphome.io/guides/configuration-types/#id)): Manually specify the ID of the [UART Bus](https://esphome.io/components/uart) if you use multiple UART buses.
* **sample_rate** (*Optional*, integer): The frame rate at which the sensor will output sensor data in samples per second. For the TFMINI_PLUS and TFMINI_S the range is 1-1000. For the TF_LUNA the range is 1-500. When low-power mode is enabled for the TFMINI_S or TF_LUNA, the rate must be 10 or less. Default is 100.
* **config_pin** (*Optional*, internal GPIO output pin): This pin is set high to enable UART mode. Only valid for `TF_LUNA`.
* **low_power** (*Optional*, boolean): Turns on low-power mode. This requires `sample_rate` to be 10 or less and is supported only by `TFMINI_S` and `TF_LUNA`. Defaults to `false`.

## Sensor

```yaml
# Sample configuration entry example

sensor:
  - platform: tfmini
    model: TFMINI_S
    sample_rate: 10
    low_power: true
    temperature:
      id: tfmini_temperature
      name: "Temperature"
    distance:
      id: tfmini_distance
      internal: true
      name: "TFMini Distance"
      accuracy_decimals: 1
    signal_strength:
      id: tfmini_signal_strength
      name: "TFMini Signal Strength"
      filters:
        - throttle: 1s
```

### Configuration Variables

* **distance** (*Required*): Distance in cm. For the TFMINI_PLUS and TFMINI_S the range is 10-1200cm. For the TF_LUNA the range is 20-800cm. The component publishes an unavailable (`NaN`) distance if signal strength is below 100 or equals 65535, or if the sensor reports a decoded distance of -4. All options from [Sensor](https://esphome.io/components/sensor/#config-sensor).
* **signal_strength** (*Optional*): Signal strength from 0 to 65535. It generally decreases with distance and with lower target reflectivity. The raw strength is still published when the distance is unavailable. All options from [Sensor](https://esphome.io/components/sensor/#config-sensor).
* **temperature** (*Optional*): Internal sensor temperature in °C, not ambient room temperature. All options from [Sensor](https://esphome.io/components/sensor/#config-sensor).
* **temperature** (*Optional*): Internal temperature in °C. It's not clear how useful this sensor because it certainly does not measure room temperature. All Options from [Sensor](https://esphome.io/components/sensor/#config-sensor).

## Development with VS Code Dev Containers

Open the component folder in VS Code and select **Reopen in Container** for development. Docker Desktop is required. The tasks below run from the Windows host workspace using **Terminal > Run Task**:

- **ML ESPHome: Build ESPHome test configs**: The default build task. Validates first, then compiles the test configurations with ESPHome stable, beta, and dev using the local component source.
- **ML ESPHome: Validate ESPHome component**: Applies supported automatic fixes and checks the component without building test configurations.
- **ML ESPHome: Clear cached downloads**: Clears shared ESPHome, PlatformIO, and pip caches and removes downloaded ESPHome Docker images. Close the Dev Container before running it.

Use **Terminal > Run Build Task** to run the default build task. Test configuration builds serve as integration tests; there is no separate unit-test runner.
