# Local ESPHome Component Build Checks

Run ESPHome component builds through the Windows Docker Desktop CLI (`docker.exe`), not from a WSL shell.

## Use the ESPHome checkout, not an external component reference

When validating a change to this component, compile the checked-out component source against ESPHome itself:

1. Refresh the ESPHome Docker image tags with `docker pull ghcr.io/esphome/esphome:stable` and
   `docker pull ghcr.io/esphome/esphome:dev`. Docker reuses unchanged image layers and downloads only layers
   that changed. Do this before the builds so a newly released stable image is picked up.
2. Check the version reported by each image. Check out the matching ESPHome stable tag and the `dev` branch in
   separate local directories. Update existing checkouts without overwriting uncommitted changes.
3. Copy this repository's `components/tfmini` directory into each checkout at `esphome/components/tfmini`.
4. Make temporary copies of the test YAML files and remove their `external_components` and `ref` substitution
   entries. Do not edit the tracked test files. Without an external component declaration, ESPHome loads the
   overlaid component from its own `esphome/components` tree.
5. Run the ESPHome CLI from the mounted checkout, setting `PYTHONPATH=/esphome`. This makes Python load the
   checked-out ESPHome source rather than the version packaged in the image.

## Preserve Docker caches

Use short-lived containers (`--rm` is fine); the Docker images and named volumes are what preserve downloaded
data between runs. Mount persistent named volumes at both `/config/.esphome` for generated builds and
`/root/.cache/esphome` for frameworks, compilers, and other ESPHome tools. Keep stable and dev caches separate.
Do not remove these images or volumes during routine cleanup.

Example using Windows PowerShell variables for the ESPHome checkout and temporary test configuration paths:

```powershell
docker run --rm `
  --entrypoint python `
  --workdir /esphome `
  --env PYTHONPATH=/esphome `
  --mount "type=bind,source=$core,target=/esphome" `
  --mount "type=bind,source=$configs,target=/config" `
  --mount "type=volume,source=tfmini-esphome-dev-build,target=/config/.esphome" `
  --mount "type=volume,source=tfmini-esphome-dev-tools,target=/root/.cache/esphome" `
  ghcr.io/esphome/esphome:dev `
  -m esphome compile /config/test-tfmini-esp32-s3-idf.yaml
```

The CI-equivalent matrix is `test-tfmini-esp32-s3-idf.yaml`, `test-tfmini-esp32-s3-ard.yaml`, and
`test-tfmini-rp2.yaml`, each built against stable and dev ESPHome source.
