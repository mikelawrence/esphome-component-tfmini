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

When both ESPHome `stable` and `beta` images are selected, the build script skips `beta` if its reported version matches `stable`. Build and validation use Docker's default bridge network and remove unused build networks left by older script versions for this checkout. Networks with attached containers and networks belonging to other projects are left alone. Cache-clearing also uses Docker's default bridge network.

Open this component directory as the VS Code workspace and reopen it in the Dev Container. All component checkouts share the same persistent ESPHome, PlatformIO, and pip cache volumes; Docker also reuses locally downloaded ESPHome images.

Run the VS Code task **Clear cached downloads** to empty the shared cache volumes and remove locally downloaded `ghcr.io/esphome/esphome` images. Close the Dev Container before running the task so Docker can remove images that are in use.

On Windows, the VS Code **Terminal > Run Build Task** command runs the default **ML ESPHome: Build ESPHome test configs** task. It runs **ML ESPHome: Validate ESPHome component** first, and only compiles if validation succeeds. Validation applies supported automatic fixes, then runs strict YAML linting, C/C++ formatting checks, Ruff format/lint, flake8, pylint, pyupgrade, and ESPHome-style text convention checks. Compilation covers every `.yaml` and `.yml` file under `test/` with the ESPHome `stable`, `beta`, and `dev` images.

Tool versions are pinned to the versions used by ESPHome CI, and the existing `.yamllint`, `.clang-format`, `.flake8`, and `pyproject.toml` configurations are used. clang-tidy is not included because it requires ESPHome's PlatformIO compilation metadata, which is not present in a standalone component checkout.

The build stage detects the GitHub owner/repository from the checkout's `origin` remote and replaces matching ESPHome GitHub component sources with the local `components/` directory in temporary copies of the test files; original test YAML files are not modified. Pass `-ComponentSource owner/repository` to `.devcontainer\build.ps1` if the checkout has no GitHub `origin`; use `-Versions @("dev")` to select ESPHome tags, or `-Image ghcr.io/esphome/esphome:<tag>` to use one custom image.

Run `.devcontainer\validate.ps1` alone to apply fixes and run validation without compiling. `yamllint`, flake8, pylint, and semantic or unsupported lint findings are not automatically rewritten and will be reported for manual attention. There is no separate unit-test runner; compiling the test configs is the integration/build test.
