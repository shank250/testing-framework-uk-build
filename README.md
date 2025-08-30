# Testing Framework for Unikraft Builds

![License](https://img.shields.io/badge/license-BSD--3--Clause-blue.svg)
![Python](https://img.shields.io/badge/python-3.10%2B-blue.svg)
![GSoC](https://img.shields.io/badge/GSoC-2025-orange)
![Code Style: Black](https://img.shields.io/badge/code%20style-black-000000.svg)

A modular and extensible Python-based testing framework for validating Unikraft unikernel builds across various configurations, platforms, and environments.
This tool is designed to be seamlessly integrated into the CI/CD pipelines of the Unikraft ecosystem.

## Overview

This project aims to:

- Automate configuration, building, and testing of Unikraft unikernels.
- Support a wide range of configuration options: VMMs, hypervisors, architectures, and boot protocols.
- Integrate tightly with the [`catalog`](https://github.com/unikraft/catalog) and [`catalog-core`](https://github.com/unikraft/catalog-core) repositories.
- Run as part of Unikraft's CI/CD workflows to validate pull requests automatically.

## Requirements

For running the framework, you need packages required to build and run Unikraft applications, and the frameworks own dependencies.
Follow the steps:

1. Install base packages, QEMU, Docker Firecracker, Clang as instructed in the [`Requirements` section in the `catalog-core` repository](https://github.com/unikraft/catalog-core#requirements).
   Be sure to follow the [instructions to configure QEMU bridged networking](https://github.com/unikraft/catalog-core#qemu):

   ```console
   test -d /etc/qemu || sudo mkdir /etc/qemu
   echo "allow all" | sudo tee /etc/qemu/bridge.conf
   ```

1. Install [KraftKit](https://github.com/unikraft/kraftkit):

   ```console
   curl --proto '=https' --tlsv1.2 -sSf https://get.kraftkit.sh | sh
   ```

1. Install Python3 and Python3 package management tools: `python3`, `python3-pip`, `python3-venv`.
   On Ubuntu/Debian or other `apt`-based distributions, use the following command:

   ```console
   sudo apt install python3 python-is-python3 python3-pip python3-venv
   ```

## Set Up and Run the Testing Framework

### Initial Setup

Follow the steps below to set up the testing framework:

1. Clone the repository:

   ```console
   git clone https://github.com/shank250/testing-framework-uk-build.git
   cd testing-framework-uk-build
   ```

1. Create a Python virtual environment:

   ```console
   python -m venv .venv
   source .venv/bin/activate
   ```

1. Install Python dependencies:

   ```console
   pip install -r requirements.txt
   ```

1. Configure the framework:

   Create the configuration file `src/config.yaml`, as a copy of the the `src/config.yaml.template` file:

   ```console
   cp src/config.yaml.template src/config.yaml
   ```

   Edit the `src/config.yaml` file and update the `base` path to point to the absolute path of your local Unikraft working directory.
   A sample configuration would be:

   ```yaml
   source:
     base: /home/monkey/razvand/orgs/unikraft/maintainer-tools/workdir
   ```

   > **Note:**
   > If local Unikraft repository isn't already set up then you need to clone and setup this [repo](https://github.com/unikraft-upb/maintainer-tools/tree/staging/workdir).

### Sudo Setup

This project uses a shell script that requires `sudo` access.
To avoid being prompted for a password every time the script runs, follow these steps:

- Open the sudoers file using the safe editor:

  ```console
  sudo visudo
  ```

- Add the following line at the end (replace `your_username` and `/path/to/dir`):

  ```text
  your_username ALL=(ALL) NOPASSWD: /path/to/dir
  ```

  ```text
  machine02 ALL=(ALL) NOPASSWD: /usr/bin/pkill, /usr/bin/kraft, /usr/sbin/ip, /usr/bin/rm, /usr/local/bin/firecracker-x86_64, /usr/bin/qemu-system-x86_64, /usr/bin/qemu-system-arm, /usr/bin/qemu-system-aarch64

  Defaults env_keep += "KRAFTKIT_NO_WARN_SUDO KRAFTKIT_BUILDKIT_HOST"
  ```

### Local Docker Registry Setup

The local Docker registry is **required** for testing catalog examples.
Set up a simple Docker registry with the following steps:

1. Create persistent storage directory:

   ```console
   mkdir -p ~/local-registry/data
   ```

1. Run the official registry container:

   ```console
   docker run -d --name local-registry \
     -p 5000:5000 \
     -v ~/local-registry/data:/var/lib/registry \
     registry:2
   ```

1. Verify the registry is running:

   ```console
   docker ps --filter name=local-registry
   ```

1. Test the registry API:

   ```console
   curl http://localhost:5000/v2/_catalog
   # Should return: {"repositories":[]}
   ```

The registry will be accessible at `http://localhost:5000`.
It is used by the framework when testing catalog examples that require container image operations.

### Running the Framework

To run the framework, use the following command:

```console
python src/main.py /absolute/path/to/app/dir
```

Make sure the path you provide is absolute (i.e., it starts with `/`).
This should point to the specific application directory inside your catalog repository.

#### Command Line Arguments

The framework supports the following optional arguments:

- `-n, --test-session-name`:
  Specify a custom test session name.
  If not provided, a 'session' will be used as default name.

  ```console
  python src/main.py /absolute/path/to/app/dir -n my_test_session
  ```

- `-t, --target-no`:
  Run tests for specific targets only. Supports multiple formats:
  - Comma-separated: `1,3,5` - Run targets 1, 3, and 5
  - Range with colon: `1:5` - Run targets 1 through 5
  - Range with dash: `1-5` - Run targets 1 through 5
  - Mixed formats: `1,3:5,7` - Run target 1, targets 3-5, and target 7
  - Space-separated: `1 3 5` - Run targets 1, 3, and 5

  ```console
  # Run specific targets
  python src/main.py /absolute/path/to/app/dir -t 1,3,5

  # Run a range of targets
  python src/main.py /absolute/path/to/app/dir -t 1:5

  # Combined example with custom session name
  python src/main.py /absolute/path/to/app/dir -n my_session -t 2:4,7
  ```

- `-v, --verbose`:
  Enable verbose output with debug-level logging.

  ```console
  python src/main.py /absolute/path/to/app/dir -v
  ```

  > **Note:**
  > Target numbers use 1-based indexing (first target is 1, not 0).
  > If no target numbers are specified, all available targets will be tested.

## 🤝 Contributing

We welcome contributions from the community!
If you're interested in contributing to this testing framework, please read our [CONTRIBUTING.md](./CONTRIBUTING.md) guide for instructions on how to get started.

Whether it's reporting bugs, suggesting features, or submitting pull requests - your input is appreciated!

## 📄 License

This project is licensed under the [BSD 3-Clause License](./COPYING.md).

By contributing to this repository, you agree that your contributions will be licensed under the same terms.
