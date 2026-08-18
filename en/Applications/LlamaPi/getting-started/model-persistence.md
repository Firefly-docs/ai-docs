# Persistent Deployment

This guide explains how to load a model automatically when the device starts, ensuring that LlamaPi remains available to external applications.

## Enable Automatic Loading

```bash
llamapi enable qwen3:4b
```

This command saves the model's automatic-loading settings. The model loads automatically the next time the device starts.

## Load Immediately and Verify

Enable automatic loading and load the model now:

```bash
llamapi enable qwen3:4b --now
```

Check the model state:

```bash
llamapi ps
```

When the `ENABLE` column shows `yes(N)`, the model is configured for automatic loading, and `N` is the configured instance count.

## Configure Instances and a Runtime ID

```bash
llamapi enable qwen3:4b --instance 2 --id assistant --now
```

The instance count affects hardware resource usage. Select a count appropriate for the device.

## Disable Automatic Loading

```bash
llamapi disable qwen3:4b
```

Disable automatic loading and unload the model now:

```bash
llamapi disable qwen3:4b --now
```

If automatic loading was configured with a custom runtime ID, use that ID to remove the entry:

```bash
llamapi disable assistant --now
```

For additional LlamaPi service configuration and troubleshooting, see [Service Configuration and Operations](../advanced-guides/server-operations.md). See the [Terminal Command Guide](../advanced-guides/cli-command-guide.md) for complete command options.
