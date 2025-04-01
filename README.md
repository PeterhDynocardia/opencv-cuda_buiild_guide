# opencv-cuda_buiild_guide
The document shows how to build opencv-cuda with RTX-40XX GPUs.
# OpenCV with CUDA in a Conda Environment

This guide walks through building **OpenCV with CUDA support** and integrating it with a **Conda environment** on a Linux system. It includes GPU acceleration, resolves library conflicts, and ensures clean Conda usage.

---

## 🧪 Environment Setup

### 1. Create and Activate Conda Environment
```bash
conda create -n dyno python=3.10
conda activate dyno
conda install numpy
```

### 2. Install System Dependencies
```bash
sudo apt install build-essential cmake git pkg-config \
    libjpeg-dev libpng-dev libtiff-dev libwebp-dev \
    libopenexr-dev libtbb-dev libgtk-3-dev \
    libavcodec-dev libavformat-dev libswscale-dev \
    gcc-10 g++-10
```

### 3. (Optional) Install CUDA Toolkit 12.6
If your system uses NVIDIA Driver 555+ and supports CUDA 12.5+, you can install the latest CUDA Toolkit (12.6) manually:

```bash
wget https://developer.download.nvidia.com/compute/cuda/12.6.0/local_installers/cuda_12.6.0_555.58.02_linux.run
chmod +x cuda_12.6.0_555.58.02_linux.run
sudo ./cuda_12.6.0_555.58.02_linux.run
```

When prompted, deselect the driver installation if you already have the correct version installed.

After installation, add CUDA to your path:
```bash
export PATH=/usr/local/cuda-12.6/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-12.6/lib64:$LD_LIBRARY_PATH
```

Make it permanent by adding the above two lines to your `~/.bashrc` or `~/.zshrc`.

Verify:
```bash
nvcc --version
nvidia-smi
```

---

## ⬇️ Clone OpenCV Repositories

```bash
git clone https://github.com/opencv/opencv.git
git clone https://github.com/opencv/opencv_contrib.git
cd opencv
mkdir build && cd build
```

---

## ⚙️ Configure CMake for Build

```bash
cmake .. \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_INSTALL_PREFIX=/home/peter/.conda/envs/dyno \
  -DPYTHON_EXECUTABLE=/home/peter/.conda/envs/dyno/bin/python \
  -DPYTHON3_INCLUDE_DIR=/home/peter/.conda/envs/dyno/include/python3.10 \
  -DPYTHON3_LIBRARY=/home/peter/.conda/envs/dyno/lib/libpython3.8.so \
  -DPYTHON3_PACKAGES_PATH=/home/peter/.conda/envs/dyno/lib/python3.8/site-packages \
  -DBUILD_opencv_python3=ON \
  -DBUILD_TESTS=OFF \
  -DBUILD_PERF_TESTS=OFF \
  -DBUILD_EXAMPLES=OFF \
  -DOPENCV_EXTRA_MODULES_PATH=~/opencv_contrib/modules \
  -DWITH_CUDA=ON \
  -DWITH_CUDNN=ON \
  -DWITH_NVCUVID=ON \
  -DCUDA_NVCC_FLAGS="-Wno-deprecated-gpu-targets --expt-relaxed-constexpr --expt-extended-lambda -std=c++14" \
  -DBUILD_ZLIB=ON \
  -DBUILD_PNG=ON \
  -DBUILD_TIFF=ON \
  -DBUILD_JPEG=ON \
  -DBUILD_WEBP=ON \
  -DBUILD_JASPER=ON \
  -DWITH_OPENEXR=OFF \
  -DWITH_IPP=ON \
  -DWITH_TBB=ON \
  -DCMAKE_CXX_STANDARD=14 \
  -DCMAKE_C_COMPILER=/usr/bin/gcc-10 \
  -DCMAKE_CXX_COMPILER=/usr/bin/g++-10
```

---

## 🧱 Build and Install OpenCV

```bash
make -j$(nproc)
make install
```

---

## 🐍 Install Python Bindings into Conda

```bash
cd ~/opencv/build/python_loader
pip install .
```

---

## 🩹 Fix GLib Runtime Conflicts (libgio/libglib)

Create activation hook:
```bash
mkdir -p $CONDA_PREFIX/etc/conda/activate.d
nano $CONDA_PREFIX/etc/conda/activate.d/env_vars.sh
```

Paste:
```bash
export LD_PRELOAD="/usr/lib/x86_64-linux-gnu/libglib-2.0.so.0 /usr/lib/x86_64-linux-gnu/libgio-2.0.so.0"
```

This ensures OpenCV uses the system's GTK-related libraries instead of Conda's stripped-down versions.


---

### If you don't want to set LD_PRELOAD because it destroys conda environment:

You can create a config in VSCode and only preload the opencv library when you need to run a opencv denpendent python file

- if you don't see any launch.json file:

Create or edit the .vscode/launch.json file.

You can auto-generate one by:

Opening a Python script

Clicking the "Run and Debug" button

Choosing "Python File" → It will create .vscode/launch.json for you

Then add multiple configurations like this:
launch.json:
```
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Python: Clear Preload",
      "type": "python",
      "request": "launch",
      "program": "${file}",
      "console": "integratedTerminal",
      "env": {
        "LD_PRELOAD": ""
      }
    },
    {
      "name": "Python: With OpenCV Preload",
      "type": "python",
      "request": "launch",
      "program": "${file}",
      "console": "integratedTerminal",
      "env": {
        "LD_PRELOAD": "/usr/lib/x86_64-linux-gnu/libglib-2.0.so.0 /usr/lib/x86_64-linux-gnu/libgio-2.0.so.0"
      }
    }
  ]
}
```

✅ Step-by-Step: Select a Run Config in VS Code

Open the Run and Debug Panel

1. Click the Run & Debug icon on the left sidebar (looks like a play button with a bug).

    Or press Ctrl + Shift + D (Windows/Linux) or Cmd + Shift + D (macOS).

2. Choose Your Config
At the top of the panel, you'll see a dropdown — this is your launch configuration selector.

3. Select the config you want to use.

---

## ✅ Test the Installation

```bash
conda activate dyno
python -c "import cv2; print(cv2.__version__); print(cv2.cuda.getCudaEnabledDeviceCount())"
```

You should see:
- ✅ Correct OpenCV version (e.g. `4.11.0`)
- ✅ Non-zero CUDA device count if GPU is supported

---

## 🎉 Done!
You now have a fully functional OpenCV + CUDA setup inside your Conda environment, cleanly isolated and free of GTK/GLib conflicts.

---

Feel free to customize and share this `README.md` with anyone who needs to repeat this setup!


