# Apple Sorting GUI

Open-access operator GUI for real-time in-field multispectral apple grading and pneumatic sorting.
Michigan State University | ASABE Paper No. 2600080

[![Python](https://img.shields.io/badge/Python-3.10-blue)](https://python.org)
[![PyQt6](https://img.shields.io/badge/GUI-PyQt6-green)](https://pypi.org/project/PyQt6/)
[![ASABE](https://img.shields.io/badge/ASABE-AIM%202026-0B5FFF)](https://doi.org/10.13031/aim.202600080)

---

## What It Is

A modular, multi-threaded PyQt6 GUI for a multispectral vision-based apple grading and sorting system. It integrates a **JAI FSFE-3200T-10GE** camera, a **YOLO instance-segmentation** grading pipeline (default weights: YOLOv8m-seg), online size estimation, and **Arduino-based pneumatic sorting** on a three-lane screw conveyor.

Paper: *Design and Development of An Open-Source Graphical User Interface for Real-Time In-Field Multispectral Apple Grading and Sorting* (Amjad, Lu, and Tian, 2026).

## System Overview

```
[JAI Camera] -> [YOLO + tracking/grading] -> [SorterController] -> [Arduino] -> [Pneumatic paddles]
  Color+NIR1+NIR2   Fresh/Processing/Cull      XYZ digit packet      USB serial   Outlets A/B/C
  eBUS MultiSource  multi-view vote + size     per committed apple   115200 baud
```

**Three grades, three outlets** (matches Table 3a in the paper):

| Grade | Outlet | Action digit | Example (Lane 1) |
|---|---|---|---|
| Fresh | A | 1 | `100\n` |
| Processing | B | 2 | `200\n` |
| Cull | C | 3 | `300\n` |

Serial packet format is three ASCII digits plus newline (`XYZ\n`): X = lane 1, Y = lane 2, Z = lane 3. Inactive lanes send `0`. Valid action digits are 1, 2, and 3 only. Arduino acknowledges actuation with `OK`.

---

## Repository

```bash
git clone https://github.com/AgFood-Sensing-and-Intelligence-Lab/Apple-GUI.git
cd Apple-GUI
```

---

## Running on a New Windows Machine

**Requirements:**

- [Miniconda or Anaconda](https://docs.conda.io/en/latest/miniconda.html)
- NVIDIA driver 525 or newer (CUDA / GPU inference)
- Windows 10 or 11 (64-bit)

**Step 1 - Install once**

Double-click `install.bat`.

This will:

- Create or update the `applegui` conda environment from `environment.yml`
- Install PyTorch with CUDA 12.8 support
- Install the JAI/Pleora eBUS Python wheel if an `ebus_python*.whl` is found next to `install.bat` or under the Pleora SDK Python folder
- Create an "Apple Sorter" shortcut on the Desktop

Live camera mode uses the JAI/Pleora eBUS Python package. Obtain the `ebus_python` wheel from the vendor SDK install, place it next to `install.bat` (or keep it under the Pleora SDK Python folder), then run setup. Without eBUS, the app still runs in mock / video-simulation mode.

**Step 2 - Launch**

Double-click `launch.bat` or the Desktop shortcut.

---

## Developer Setup

```bash
git clone https://github.com/AgFood-Sensing-and-Intelligence-Lab/Apple-GUI.git
cd Apple-GUI
conda env create -f environment.yml
conda activate applegui
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu128
python main.py
```

`environment.yml` does not install PyTorch; use the pip command above (same as `install.bat`). For live JAI capture, install `ebus_python` from your Pleora/JAI SDK wheel (not shipped in this repository).

---

## Features

- Synchronized three-channel acquisition (Color, NIR1, NIR2) via the JAI/Pleora eBUS Python SDK
- GenICam camera controls from the GUI: ROI, frame rate, per-channel exposure/gain/black level, white balance
- Runtime YOLO model load/unload and confidence threshold control
- Instance segmentation, ByteTrack + conveyor-aware tracking, multi-view grade aggregation
- Online fruit size estimation from segmentation masks (ridge regression model)
- Pneumatic sorter control over USB serial (or simulation mode without hardware)
- Live multispectral display, grading statistics, and analytics dashboard
- On-demand session logging (CSV grades, per-apple crops, optional raw frames)
- Mock / video-simulation modes for full pipeline testing without the camera

---

## Hardware (lab system)

| Component | Specification |
|---|---|
| Camera | JAI FSFE-3200T-10GE (3-CMOS, 10 GigE Vision) |
| Spectral channels | Color (visible); NIR1 (~800 nm band); NIR2 (~900 nm band) |
| Resolution | 2048 x 1536 pixels per channel |
| Acquisition | 30 FPS stable full-resolution free-run (default exposure 9229 us) |
| Lens | 6 mm, f/2.8 (3CMOS/prism) |
| NIC | 10 Gigabit Ethernet |
| Computing | Intel Core i9-11900K, NVIDIA RTX 5060 Ti (16 GB), 128 GB RAM |
| Conveyor | 3-lane screw conveyor, design throughput 1-3 apples/s/lane |
| Sorter | 3 pneumatic units (solenoid + air cylinder + paddle) |
| Controller | Arduino, USB serial at 115200 baud |
| OS | Windows 10/11 |

Illumination in the imaging chamber: six downward-facing halogen lamps and four white LED strips (as described in the paper).

## Sorter Wiring (logical)

```
PC (COM3) --USB--> Arduino --doAction(digit)--> solenoid pair --> paddle --> Outlet A/B/C
```

Physical arrival timing uses the sorter-gate NIR sensor on the Arduino. With `camera_to_gate_m: 0.0`, the GUI fires the serial command when the grade is committed; the Arduino queues the digit and actuates when the fruit reaches the gate.

---

## Configuration

Edit `config/config.yaml`:

```yaml
camera:
  mode: "mock"        # "mock" | "jai"

sorter:
  mode: "simulation"  # "simulation" | "serial"
  serial:
    port: "COM3"      # Arduino COM port
    baudrate: 115200

conveyor:
  camera_to_gate_m: 0.0          # 0.0 = fire on grade; Arduino NIR gate handles timing
  speed_apples_per_sec: 1        # nominal 1 | 2 | 3 apples/s/lane
```

Default model file: `models/best.pt`. Size model: `models/size_model.pkl`.

---

## Citation

If you use this software, please cite ASABE Paper No. 2600080:

Amjad, H., Lu, Y., & Tian, Y. (2026). Design and development of an open-source graphical user interface for real-time in-field multispectral apple grading and sorting. ASABE Paper No. 2600080. St. Joseph, MI: ASABE. https://doi.org/10.13031/aim.202600080

---

## License note

This repository's application code is released for research use with the paper. Live JAI acquisition depends on the vendor eBUS Python SDK, which users obtain from the camera SDK package and install locally. That SDK is proprietary and is not published in this repository.
