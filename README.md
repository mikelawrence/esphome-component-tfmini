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
      url: https://github.com/mikelawrence/esphome-components-tfmini

uart:
  - id: tfmini_uart
    tx_pin: GPIO25
    rx_pin: GPIO24
    baud_rate: 115200
    data_bits: 8
    parity: NONE
    stop_bits: 1
```

## Configuration Variables

* **model** (**Required**, string): The model of the Range Finder Sensor. Options are `TFMINI_PLUS` , `TFMINI_S` or `TF_LUNA` .
* **uart_id** (*Optional*, [ID](https://esphome.io/guides/configuration-types/#id)): Manually specify the ID of the [UART Bus](https://esphome.io/components/uart) if you use multiple UART buses.
* **sample_rate** (*Optional*, integer): The frame rate at which the sensor will output sensor data in samples per sec. For the TFMINI_PLUS and TFMINI_S the range is 1-1000. For the TFLuna the range is 1-500. Note when low_power mode is set to true for the TFMINI_S and the TFLuna model the is significantly lower from 1-10. Default is 100.
* **config_pin** (*Optional*, [Pin Schema](https://esphome.io/guides/configuration-types#config-pin-schema)): This pin when connected will be set high to enable UART mode. (*TF_LUNA only*)
* **low_power** (*Optional*, boolean): Turns on low power mode. This also requires sample_rate to be 10 or less. (*TF_LUNA, TFMini-S only*)

## Sensors

```yaml
# Sample configuration entry example

sensor:
  - platform: tfmini
    model: TFMINI_PLUS
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

* **distance** (*Optional*): Distance in cm. For the TFMINI_PLUS and TFMINI_S the range is 10-1200cm. For the TFLuna the range is 20-800cm. A distance of 10000cm means the sensor is not receiving enough signal, most likely open air. A distance of 0cm means the sensor is saturated and there is no measurement possible. All Options from [Sensor](https://esphome.io/components/sensor/#config-sensor).
* **signal_strength** (*Optional*): Represents the signal strength with a range of 0-65535. The longer the measurement distance, the lower signal strength will be. The lower the reflectivity is, the lower the signal strength will be. When signal strength is less than 100 detection is unreliable and distance is set to 10000cm. When signal strength is 65535 detection is unreliable and distance is set to 0cm. All Options from [Sensor](https://esphome.io/components/sensor/#config-sensor).
* **temperature** (*Optional*): Internal temperature in °C. It's not clear how useful this sensor because it certainly does not measure room temperature. All Options from [Sensor](https://esphome.io/components/sensor/#config-sensor).

## Development with VS Code Dev Containers

Open the component checkout in VS Code and reopen it in the Dev Container. The setup uses Docker Compose with a persistent cache volume shared by all builds.

On Windows, run the VS Code task **Build ESPHome test configs** to compile every `.yaml` and `.yml` file under `test/` with the ESPHome `stable`, `beta`, and `dev` images. The script detects the GitHub owner/repository from the checkout's `origin` remote and replaces matching ESPHome GitHub component sources with the local `components/` directory in temporary copies of the test files. The original test YAML files are not modified.

Run the separate VS Code task **Validate ESPHome component** for strict YAML linting, C/C++ formatting, Ruff format/lint, flake8, pylint, pyupgrade, and ESPHome-style text convention checks. Tool versions are pinned to the versions used by ESPHome CI, and the existing `.yamllint`, `.clang-format`, `.flake8`, and `pyproject.toml` configurations are used. clang-tidy is not included because it requires ESPHome's PlatformIO compilation metadata, which is not present in a standalone component checkout.

Use **Fix ESPHome component formatting/lint** to apply clang-format, Ruff formatting and safe Ruff lint fixes, pyupgrade, and portable text fixes, then rerun validation. `yamllint`, flake8, pylint, and semantic or unsupported lint findings are not automatically rewritten and will be reported for manual attention.

The build script can also be run directly with `-ConfigDirectory`, `-ComponentsDirectory`, and `-Versions` to select ESPHome tags, or with `-Image` to use one custom image. If the checkout does not have a GitHub `origin`, pass `-ComponentSource owner/repository` explicitly.
