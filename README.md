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

* **model** (**Required**, enum): The model of the range finder sensor. Options are `TFMINI_PLUS`, `TFMINI_S` or `TF_LUNA`.
* **uart_id** (*Optional*, [ID](https://esphome.io/guides/configuration-types/#id)): Manually specify the ID of the [UART Bus](https://esphome.io/components/uart) if you use multiple UART buses.
* **sample_rate** (*Optional*, integer): The frame rate at which the sensor will output sensor data in samples per second. For the TFMINI_PLUS and TFMINI_S the range is 1-1000. For the TF_LUNA the range is 1-500. When low-power mode is enabled for the TFMINI_S or TF_LUNA, the rate must be 10 or less. Default is 100.
* **config_pin** (*Optional*, [Pin Schema](https://esphome.io/guides/configuration-types#config-pin-schema)): This pin when connected will be set high to enable UART mode. (*TF_LUNA only*)
* **low_power** (*Optional*, boolean): Turns on low-power mode. This requires `sample_rate` to be 10 or less and is supported only by `TFMINI_S` and `TF_LUNA`.

## Sensors

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

* **distance** (*Required*): Distance in cm. For the TFMINI_PLUS and TFMINI_S the range is 10-1200cm. For the TF_LUNA the range is 20-800cm. A distance of 10000cm means the sensor is not receiving enough signal, most likely open air. A distance of 0cm means the sensor is saturated and there is no measurement possible. All Options from [Sensor](https://esphome.io/components/sensor/#config-sensor).
* **signal_strength** (*Optional*): Represents the signal strength with a range of 0-65535. The longer the measurement distance, the lower signal strength will be. The lower the reflectivity is, the lower the signal strength will be. When signal strength is less than 100 detection is unreliable and distance is set to 10000cm. When signal strength is 65535 detection is unreliable and distance is set to 0cm. All Options from [Sensor](https://esphome.io/components/sensor/#config-sensor).
* **temperature** (*Optional*): Internal temperature in °C. It's not clear how useful this sensor because it certainly does not measure room temperature. All Options from [Sensor](https://esphome.io/components/sensor/#config-sensor).

## Development with VS Code Dev Containers

Open this component directory as the VS Code workspace and reopen it in the Dev Container. All component checkouts share the same persistent ESPHome, PlatformIO, and pip cache volumes; Docker also reuses locally downloaded ESPHome images.

Run the VS Code task **Clear cached downloads** to empty the shared cache volumes and remove locally downloaded `ghcr.io/esphome/esphome` images. Close the Dev Container before running the task so Docker can remove images that are in use.

On Windows, the VS Code **Terminal > Run Build Task** command runs the default **ML ESPHome: Build ESPHome test configs** task. It runs **ML ESPHome: Validate ESPHome component** first, and only compiles if validation succeeds. Validation applies supported automatic fixes, then runs strict YAML linting, C/C++ formatting checks, Ruff format/lint, flake8, pylint, pyupgrade, and ESPHome-style text convention checks. Compilation covers every `.yaml` and `.yml` file under `test/` with the ESPHome `stable`, `beta`, and `dev` images.

Tool versions are pinned to the versions used by ESPHome CI, and the existing `.yamllint`, `.clang-format`, `.flake8`, and `pyproject.toml` configurations are used. clang-tidy is not included because it requires ESPHome's PlatformIO compilation metadata, which is not present in a standalone component checkout.

The build stage detects the GitHub owner/repository from the checkout's `origin` remote and replaces matching ESPHome GitHub component sources with the local `components/` directory in temporary copies of the test files; original test YAML files are not modified. Pass `-ComponentSource owner/repository` to `.devcontainer\build.ps1` if the checkout has no GitHub `origin`; use `-Versions @("dev")` to select ESPHome tags, or `-Image ghcr.io/esphome/esphome:<tag>` to use one custom image.

Run `.devcontainer\validate.ps1` alone to apply fixes and run validation without compiling. `yamllint`, flake8, pylint, and semantic or unsupported lint findings are not automatically rewritten and will be reported for manual attention. There is no separate unit-test runner; compiling the test configs is the integration/build test.
