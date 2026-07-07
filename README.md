# ESPHome Device Builder Helm Chart

This repository contains a Helm chart for running ESPHome Device Builder on Kubernetes.

The chart lives in [`charts/esphome-device-builder`](charts/esphome-device-builder) and deploys the official ESPHome container image, which includes Device Builder and serves it on port `6052` with `/config` mounted for device configuration files and build state.

See the [chart README](charts/esphome-device-builder/README.md) for installation examples and the full values reference.
