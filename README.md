# Sensor Fusion Platform Manifest

This repository contains the [Google repo](https://gerrit.googlesource.com/git-repo/) manifest for the C++ sensor fusion platform. It lets you clone and build the thermal camera (CTS Gen3), radar, and fusion components in one step.

## Quick Start

### 1. Install the repo tool

```bash
# Linux / macOS
mkdir -p ~/bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
export PATH=~/bin:$PATH

# Add to your shell profile (~/.bashrc or ~/.zshrc) to make it permanent:
# export PATH=~/bin:$PATH
```

> **Note:** The repo tool requires Python 3.6+.

### 2. Initialize and sync

```bash
# Replace with your manifest repo URL (e.g. https://github.com/yourorg/sensor-platform-manifest)
repo init -u https://github.com/yourorg/sensor-platform-manifest -m default.xml
repo sync
```

### 3. Install dependencies

```bash
# Ubuntu / Debian
sudo apt install build-essential cmake libzmq3-dev

# Fedora / RHEL
sudo dnf install gcc-c++ cmake zeromq-devel
```

### 4. Build

```bash
./build.bash
```

That's it. The script configures CMake, builds all components, and prints the executable paths.

---

## What You Get

After `repo sync`, your workspace looks like:

```
.
├── build.bash           # One-command build script
├── CMakeLists.txt       # Top-level CMake
├── .gitignore
├── hal/                 # SPI + I2C hardware abstraction
├── transport/           # ZMQ message transport
├── sensors/
│   ├── cts-gen3/        # CTS Gen3 thermal camera (FT4222)
│   └── radar/           # Radar sensor
└── fusion/              # Multi-sensor fusion
```

## Running the Sensors

### Thermal camera only

```bash
./build/sensors/cts-gen3/cts_gen3_publisher tcp://*:5555
```

### Radar only

```bash
./build/sensors/radar/radar_publisher tcp://*:5556
```

### Fusion (thermal + radar)

Run in separate terminals:

```bash
# Terminal 1
./build/sensors/cts-gen3/cts_gen3_publisher tcp://*:5555

# Terminal 2
./build/sensors/radar/radar_publisher tcp://*:5556

# Terminal 3
./build/fusion/fusion_node tcp://localhost:5555 tcp://localhost:5556
```

---

## Manifest Configuration

The `default.xml` manifest lists all projects. To use your own repositories, update the `fetch` URL:

```xml
<remote name="origin" fetch="https://github.com/yourorg" />
<default remote="origin" revision="main" sync-j="4" />

<project name="platform" path="." />
<project name="sensor-hal" path="hal" />
<project name="sensor-transport" path="transport" />
<project name="cts-gen3-driver" path="sensors/cts-gen3" />
<project name="radar-driver" path="sensors/radar" />
<project name="sensor-fusion" path="fusion" />
```

The clone URL for each project is `{fetch}/{name}.git`.

---

## Repo Layout

| Project   | Path            | Description                    |
|-----------|-----------------|--------------------------------|
| platform  | . (root)        | Build script, top-level CMake  |
| hal       | hal/            | SPI + I2C hardware abstraction |
| transport | transport/      | ZMQ publish/subscribe          |
| cts-gen3  | sensors/cts-gen3| CTS Gen3 thermal camera       |
| radar     | sensors/radar   | Radar sensor                   |
| fusion    | fusion/         | Multi-sensor fusion            |

---

## Troubleshooting

### "repo: command not found"

Add `~/bin` to your PATH:
```bash
export PATH=~/bin:$PATH
```

### "Could not find workspace root"

Run `./build.bash` from the directory created by `repo sync` (the workspace root). The script looks for `CMakeLists.txt` and `hal/` in the current or parent directories.

### "libzmq not found"

Install the ZeroMQ development package:
- Ubuntu/Debian: `sudo apt install libzmq3-dev`
- Fedora: `sudo dnf install zeromq-devel`

### Build fails in a subproject

You can build components individually:

```bash
cd hal && mkdir build && cd build && cmake .. && make
cd ../transport && mkdir build && cd build && cmake .. && make
# etc.
```

---

## Optional: CTS Gen3 Hardware

To use the CTS Gen3 thermal camera with the FT4222 USB bridge:

1. Install the [FTDI FT4222 SDK](https://ftdichip.com/drivers/d3xx-drivers/) (libft4222)
2. Uncomment the `libft4222` link in `hal/CMakeLists.txt`
3. Connect the FT4222 dual-port adapter and CTS Gen3 EVK
