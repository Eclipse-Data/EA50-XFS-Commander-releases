# ea50_xfs_commander

## Overview

`ea50_xfs_commander` is a Rust library with Python bindings (via `pyo3`) for communicating with XFS devices. It allows users to send pre-defined commands to a device, supporting both Rust and Python interfaces.

## Features

- Rust-native API for direct device communication
- Python bindings using `pyo3` for easy integration
- Built-in testing support with `pytest`

## Installation

### Python

To install the Python package, download the wheel for your architecture and Python version, and then run:

```sh
pip install <PATH_TO_WHL>
```

## Usage

#### Setup

**Skip this step if you are using a real device**
We create a virtual serial port for this example using sotcat below.

```zsh
socat -d -d pty,raw,echo=0 pty,raw,echo=0
```

We can then use one of the paired ports for sending commands and one for receiving. e.g. if the above command returned /dev/ttys001 and /dev/ttys002, we can monitor /dev/ttys002 with the below on a separate terminal window.

```zsh
cat < /dev/ttys002
```

### Python Example

```python
import logging
import os, pty
from ea50_xfs_commander import Connection

logging.basicConfig(level=logging.DEBUG, format='%(asctime)s - %(levelname)s - %(message)s')
logger = logging.getLogger(__name__)

def manage_device():
    try:
        reader_pty_port, writer_pty_port = pty.openpty()
        device = Connection(os.ttyname(writer_pty_port), 0, 5)
        device.open()

        device.set_comm_frequency("COMM1", "123.110")
        device.set_altitude_bug(99)
        device.set_heading_bug("LEFT", 12)
        device.set_speed_bug(100)

        print("READ FOLLOWING COMMANDS:")
        print("\t"+os.read(reader_pty_port, 1024).decode().replace("\n", "\n\t"))
    except Exception as e:
        logger.error(f"An error occurred: {e}")
    finally:
        try:
            device.close()
        except Exception as e:
            logger.error(f"Failed to close the device connection: {e}")

if __name__ == "__main__":
    manage_device()
```

## Running Tests

### Python Tests

To run Python tests, ensure Maturin and UV are installed:

```sh
uv run pytest
```
