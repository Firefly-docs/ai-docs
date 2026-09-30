# Introduction

DirectAI is a Firefly desktop management client for edge devices. It runs on your PC to discover, connect to, and manage devices, bringing device lookup, sign-in, app installation, service configuration, and monitoring together in a single desktop client — so anyone can manage local AI devices without professional skills.

If you are getting started with DirectAI, see [Quick Start](../getting-started/quickstart.md) to install the client and connect your first device.

## What Is DirectAI

Managing an edge AI device usually involves device discovery, secure trust, account sign-in, software installation, service configuration, and monitoring. DirectAI brings all of these steps into one desktop interface: discover a device over the network or USB, sign in, and you are ready to use the app store and the device management panel.

## Problems It Solves

- **Complicated device onboarding**: device discovery and account sign-in are combined into a guided flow, over both LAN and USB.
- **High barrier to app installation**: apps install from the app store in one click — the client handles the deployment on the device side automatically.
- **Scattered maintenance tools**: the device management panel shows CPU, memory, disk, and temperature readings in one place, with a built-in remote terminal.

## Key Features

- **Device discovery and sign-in**: scan devices on the LAN or connect over USB, confirm when prompted, and sign in with your Linux account.
- **App store**: browse, import, and manage device apps, with version replacement.
- **App lifecycle management**: install, update, and uninstall apps, and pin them to the sidebar for quick access.
- **Device monitoring**: live readings of device info, CPU, memory, storage, and temperature.
- **Remote terminal**: operate the device from the built-in terminal in the client.

## Typical Use Cases

- A unified desktop entry point for managing multiple AI devices in classrooms, labs, or offices.
- Deploying local LLM and knowledge-base apps offline through the app store in air-gapped environments.
- Monitoring device load and temperature from the device panel, with the terminal for troubleshooting.
- Batch-installing and updating apps on devices distributed across a team.

## Software Components

DirectAI consists of two parts:

| Component | Role |
|:---:|:---|
| Client | The desktop app on your PC, providing device discovery and sign-in, the app store, device management, and the remote terminal |
| Device server | The required backend service on the device, handling device identity, network discovery, secure connections, account sign-in, status reporting, and app management |

The device server comes pre-installed and starts automatically at boot. Apps are distributed through the app store: when you install one, the client handles the deployment on the device side automatically.

## Documentation

- First time using DirectAI: [Quick Start](../getting-started/quickstart.md)
- Install your first app: [Install Apps](../getting-started/install-app.md)
