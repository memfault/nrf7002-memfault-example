![GitHub Workflow Status (with event)](https://img.shields.io/github/actions/workflow/status/memfault/nrf7002-memfault-example/build.yml?style=for-the-badge&logo=githubactions)

# nRF7002 Memfault Example Project

This is an example nRF-Connect SDK project, based on this example:

https://github.com/nrfconnect/sdk-nrf/tree/v2.5.0/samples/debug/memfault

It adds support for performing Memfault OTA on the nrf7002-dk development board,
by adding the necessary configurations for placing the mcuboot secondary
partition in the external SPI NOR flash.

The internal 1MB flash on the nrf53 on the dk is insufficient to perform OTA; a
base HTTP client image for the nrf7002 is approximately 600-700kB, which
prevents placing both the primary and secondary partitions in the internal
flash.

## Building

To build the project:

```bash
# create a workspace directory
❯ mkdir nrf7002-memfault-example
❯ cd nrf7002-memfault-example
❯ git clone https://github.com/memfault/nrf7002-memfault-example.git
❯ west init -l nrf7002-memfault-example
❯ west update
# to build the project, you need to provide a Memfault Project Key
❯ west build \
    --board nrf7002dk_nrf5340_cpuapp \
    --pristine=always nrf7002-memfault-example \
    -- \
    -DCONFIG_MEMFAULT_NCS_PROJECT_KEY=\"${MEMFAULT_PROJECT_KEY}\"
# to flash
❯ west flash
# open a serial port and interact with the device to test
```

## Testing OTA

1. Build and flash the project with a specific version, for example `1.0.0`:

   ```bash
   ❯ west build --board nrf7002dk_nrf5340_cpuapp --pristine=always \
     nrf7002-memfault-example \
     -- \
     -DCONFIG_MEMFAULT_NCS_PROJECT_KEY=\"${MEMFAULT_PROJECT_KEY}\" \
     -DCONFIG_MEMFAULT_NCS_FW_VERSION_STATIC=y \
     -DCONFIG_MEMFAULT_NCS_FW_VERSION=\"1.0.0\"
   ```

2. Rebuild with a different version, for example `1.1.0`:

   ```bash
   ❯ west build --board nrf7002dk_nrf5340_cpuapp --pristine=always \
     nrf7002-memfault-example \
     -- \
     -DCONFIG_MEMFAULT_NCS_PROJECT_KEY=\"${MEMFAULT_PROJECT_KEY}\" \
     -DCONFIG_MEMFAULT_NCS_FW_VERSION_STATIC=y \
     -DCONFIG_MEMFAULT_NCS_FW_VERSION=\"1.1.0\"
   ```

3. Create a release in the Memfault UI matching the new version, here `1.1.0`,
and upload the `build/zephyr/app_update.bin` artifact.

4. Deploy the release to the device's cohort in the Memfault UI.

5. From the device, run the `mflt_nrf get_latest_url` to confirm the new update
   is ready.

6. Run `mflt_nrf fota` to perform the update.

7. After the update is dowloaded and installed, the device should restart.
   Running `mflt get_device_info` should show the new version.

More details on using Memfault OTA can be found here:

https://docs.memfault.com/docs/platform/ota
