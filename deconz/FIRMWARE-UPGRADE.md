# Upgrading RaspBee and ConBee firmware

Firmware upgrades from the web UI will fail silently. Instead, an interactive utility script is provided as part of the Hass.io add-on image that you must use to flash your device's firmware. Follow these instructions to upgrade your firmware.

For more details/the original source of these instructions, see [here](https://github.com/marthoc/docker-deconz#updating-conbeeraspbee-firmware).

## Step 1 - Enable SSH access to the Hass.io host

Follow the steps outlined here to enable SSH access: https://developers.home-assistant.io/docs/en/hassio_debugging.html.

## Step 2 - Run the firmware upgrade script

**Note: You may need to remove other USB devices (e.g. a Z-Wave stick) temporarily while flashing Conbee if the update process below fails. This should not be necessary if you use the device-mapping properly**

1. Check your deCONZ add-on logs for the update firmware file name. Look for lines near the beginning of the log that look like this, noting the .CGF file name listed (you'll need this later):
```
GW update firmware found: /usr/share/deCONZ/firmware/deCONZ_Rpi_0x261e0500.bin.GCF
GW firmware version: 0x261c0500
GW firmware version shall be updated to: 0x261e0500
```

Alternatively you can log into your Hass.io host over SSH (set up in step 1) and run this command to extract the GCF line:
```docker ps | grep deconz | awk '{print $NF}' | while read cont; do echo "Searching in $cont"; docker logs $cont 2>&1 | grep 'GCF\|firmware version'; done

Searching in addon_a0d7b954_deconz
23:59:37:341 GW update firmware found: /usr/share/deCONZ/firmware/deCONZ_Rpi_0x262f0500.bin.GCF
23:59:38:534 Device firmware version 0x261F0500
00:01:47:443 GW firmware version: 0x261f0500
```

2. Stop the deCONZ add-on.
3. Log in to your Hass.io host over SSH (set up in step 1).
4. Invoke the firmware upgrade script:

```bash
# If you have a single usb-device plugged into your machine
docker run -it --rm --entrypoint "/firmware-update.sh" --privileged --cap-add=ALL -v /dev:/dev -v /lib/modules:/lib/modules -v /sys:/sys marthoc/deconz
```

```bash
# If you have more than one device or the update fails, try explicitly mapping the device id found in the deconz add-on configuration into the container
docker run -it --rm --entrypoint "/firmware-update.sh" --privileged --cap-add=ALL -v /dev/serial/by-id/usb-dresden_elektronik_ingenieurtechnik_GmbH_ConBee_II_DExxxxxxx-if00:/dev/ttyACM0  -v /lib/modules:/lib/modules -v /sys:/sys marthoc/deconz
```

5. Follow the prompts:

- Enter the device path. If you have a single usb device plugged in, `/dev/ttyUSB0` for ConBee or /dev/ttyAMA0 for RaspBee. If you used the explicit device mapping, then always use: `/dev/ttyACM0`.  
- Type or paste the full file name (but not the full path) that corresponds to the file name that you found in the deCONZ container logs in step 1 (or, select a different filename, but you should have a good reason for doing this). The file may need to be downloaded.
- If the device/number and file name look OK, type Y to start flashing!

6. You should receive a success message if the flashing is OK. Exit your SSH session and restart the deCONZ add-on.
