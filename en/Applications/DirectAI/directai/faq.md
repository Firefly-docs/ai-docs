# FAQ and Troubleshooting

This chapter covers common issues with device connection, app installation, and everyday use of the DirectAI client.

## Quick Finder

| Symptom | Section |
|:---:|:---:|
| No devices found on the LAN, or no response over USB | [Device Discovery](#device-discovery) |
| Sign-in fails or saved credentials expired | [Sign-in Issues](#sign-in-issues) |
| Blank app page or API errors | [App Runtime](#app-runtime) |
| Metrics missing, terminal not connecting | [Device Management](#device-management) |

## Device Discovery

### No devices found on the LAN

- Make sure the PC and the device are on the same LAN, and that client isolation is disabled on the network.
- Device scanning relies on the device-side service. Check that it is running on the device:

```bash
systemctl status directai-device-server
```

- If the device-side service is not installed or not running, the client cannot discover the device. Get the service package from the [Firefly Download Center](https://community.t-firefly.com/doc/download/428), copy it to the device, and install (the service is enabled at boot automatically):

```bash
sudo dpkg -i ./firefly-directai-device-server_*.deb
```

- Click **Scan** again in the client, or switch to **USB discovery**.

### No response after connecting USB

- Try a different USB cable or port to rule out charge-only cables.
- Sign-in and data transfer over USB still rely on the device-side service running normally.

## Sign-in Issues

### Sign-in fails

- Make sure you are using the Linux username and password of the device.
- When connecting to a device for the first time, complete the device confirmation prompt before entering your credentials.

### Saved credentials expired

After the device password changes or the system is reset, the client shows "saved credentials have expired". Sign in again with the device's Linux password.

## App Runtime

### Blank app page or errors

- Make sure the device is still connected.
- The app frontend runs inside the client — check the app's own log output.
- Calls to `directai.app.*` APIs require a signed-in device. Some APIs that depend on the network (such as `getIp()`) return `null` over USB, which is expected.

### Settings not taking effect

- Some settings require a restart of the corresponding service; check the service status on the app settings page.
- Settings are stored per device and app — reconfigure after switching devices.

## Device Management

### Metrics never show

Metrics come from snapshots reported by the device-side service. The client shows a waiting state until the first snapshot arrives. If nothing shows for a long time, check that the service is running on the device.

### Terminal not connecting

- Make sure the SSH service is enabled on the device.
- Use the same Linux account you signed in with.
- After a network change or device reboot, reconnect the device and try again.
