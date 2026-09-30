# Quick Start

This section explains how to install LlamaPi and run your first model.

## Install LlamaPi

If the Firefly APT repository is configured on the device, install LlamaPi directly:

```bash
sudo apt install llamapi
```

Alternatively, download the deb packages from the [Firefly website](https://community.t-firefly.com/doc/download/428), copy them to the device, open their directory, and install them locally:

```bash
sudo dpkg -i ./firefly-llamapi-*.deb
```

## Use LlamaPi to Run a Model

Run the `qwen3.5:4b` model with the `llamapi run` command:

```bash
llamapi run qwen3.5:4b
```

The `llamapi run` command automatically selects and downloads a model available on the current hardware:

![quickstart-1](./images/quickstart/quick​start​-1.png)

After the model is downloaded and loaded, the terminal enters an interactive chat. Chat with the model at the prompt:

![quickstart-2](./images/quickstart/qu​i​ckstart​-2.png)

For models with multimodal capabilities, use `@` to attach files in the conversation:

![quickstart-3](./images/quickstart/qu​i​ckstart​-3.png)

Use `/exit` or `Ctrl+D` to leave the chat.

## View Models Available on the Current Hardware

List models available on the current hardware:

```bash
llamapi list --online
```

The following example uses an RK3588 + RK1828 hardware platform:

![quickstart-4](./images/quickstart/qu​i​ckstart​-4.png)

## Next Steps

- Download and remove local models: [Download and Manage Models](./model-download-and-management.md)
- Run and deploy models: [Run and Deploy Models](./model-load-and-run.md)
- Configure automatic loading at service startup: [Persistent Deployment](./model-persistence.md)
- Connect a model to an existing application: [Connect Third-Party Applications](./third-party-integration.md)
- Use LlamaPi from a PC with a graphical interface: [Using the Client](./client-usage.md)
