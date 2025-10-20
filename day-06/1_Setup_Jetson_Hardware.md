# Jetson Configuration (GStreamer / FFmpeg / OpenCV / NVDEC)

## 🎯 Learning objectives
- Prepare a Jetson (JetPack) device for ML and multimedia workloads.  
- Install and verify GStreamer, FFmpeg (with NVDEC/NVENC), OpenCV with CUDA/DNN support, and NV codec headers.  

---

## 📘 Overview
This guide collects the common commands and steps to configure a Jetson device (JetPack) and build/install a multimedia stack: GStreamer, FFmpeg (with NVDEC/NVENC), NV codec headers, and OpenCV with CUDA/FFmpeg/GStreamer support.

Notes:
- Building FFmpeg and OpenCV on Jetson may take considerable time and may require swap.  
- Follow each section in order. If you already installed JetPack via NVIDIA SDK Manager, some components may already be present.

---

## ⚙️ Prerequisites
- Jetson device with JetPack installed (CUDA, cuDNN, TensorRT, Multimedia API).  
- sudo access on the device.  
  
---

## 1) Install JetPack components
Install the JetPack meta-package (this installs CUDA, cuDNN, TensorRT, multimedia libraries if not already provided by SDK Manager):

```bash
sudo apt install nvidia-jetpack
```

---

## 2) Install jetson-stats (jtop)
Useful for monitoring CPU/GPU/temperature/IO while building and testing:

```bash
sudo -H pip3 install -U jetson-stats
```

Run the monitor:

```bash
jtop
```

---

## 3) Install GStreamer (system packages)
Install development and plugin packages so OpenCV and apps can use GStreamer pipelines:

```bash
sudo apt install -y \
  libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev \
  libgstreamer-plugins-bad1.0-dev gstreamer1.0-plugins-base \
  gstreamer1.0-plugins-good gstreamer1.0-plugins-bad \
  gstreamer1.0-plugins-ugly gstreamer1.0-libav gstreamer1.0-tools
```

Verify GStreamer:

```bash
gst-inspect-1.0 --version
```

---

## 4) Install NV codec headers (nv-codec-headers)
FFmpeg needs NV codec headers for NVDEC/NVENC support. Build and install them:

```bash
cd ~
git clone https://github.com/FFmpeg/nv-codec-headers.git
cd nv-codec-headers
make
sudo make install
```

---

## 5) Build FFmpeg with CUDA / NVDEC / NVENC
Clone FFmpeg, configure with CUDA/NV codecs and common libs, then build. This may take a long time on the Jetson.

```bash
cd ~
git clone https://git.ffmpeg.org/ffmpeg.git ~/ffmpeg
cd ~/ffmpeg

./configure --prefix=/usr/local/ffmpeg \
  --enable-gpl --enable-nonfree \
  --enable-cuda --enable-cuvid --enable-nvenc \
  --enable-libnpp --enable-libdrm \
  --extra-cflags="-I/usr/local/cuda/include" \
  --extra-ldflags="-L/usr/local/cuda/lib64" \
  --enable-libx264 --enable-libx265 --enable-libvpx \
  --enable-libfdk-aac --enable-libmp3lame --enable-libopus

make -j$(nproc)

sudo make install
sudo ldconfig
```

Verify FFmpeg installation:
From the ffmpeg source folder
```bash
cd ~/ffmpeg
./ffmpeg -version
./ffmpeg -hwaccels
```
or check installed package list
```bash
dpkg -l | grep ffmpeg
```

---

## 6) Build & install OpenCV with CUDA / GStreamer / FFmpeg support
Clone OpenCV + contrib and build with the flags enabling GStreamer, FFmpeg, CUDA and DNN CUDA support.

```bash
cd ~
git clone https://github.com/opencv/opencv.git
git clone https://github.com/opencv/opencv_contrib.git

cd opencv
mkdir build && cd build

cmake -D CMAKE_BUILD_TYPE=RELEASE \
  -D CMAKE_INSTALL_PREFIX=/usr/local \
  -D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib/modules \
  -D WITH_GSTREAMER=ON \
  -D WITH_FFMPEG=ON \
  -D FFMPEG_DIR=/usr/local/ffmpeg \
  -D WITH_CUDA=ON \
  -D WITH_TBB=ON \
  -D WITH_OPENMP=ON \
  -D WITH_CUDNN=ON \
  -D OPENCV_DNN_CUDA=ON \
  -D ENABLE_FAST_MATH=1 \
  -D CUDA_FAST_MATH=1 \
  -D WITH_CUBLAS=1 \
  -D BUILD_opencv_python3=ON \
  -D PYTHON3_EXECUTABLE=$(which python3) \
  ..

make -j$(nproc)
sudo make install
sudo ldconfig
```

Verify OpenCV installation:

```bash
python3 -c "import cv2; print(cv2.__version__)"
```

---

## 7) Full verification
Run a small script to confirm the build flags (CUDA, FFMPEG, GStreamer) are enabled in OpenCV:

```bash
python3 - <<'PY'
import cv2
print([line for line in cv2.getBuildInformation().splitlines() if 'CUDA' in line or 'FFMPEG' in line or 'GStreamer' in line])
PY
```

Also check ffmpeg and GStreamer:

```bash
cd ~/ffmpeg
./ffmpeg -version
./ffmpeg -hwaccels
gst-inspect-1.0 --version
```

## 🛠 Troubleshooting
- Builds can fail due to low memory. Add swap before building (see prerequisites).  
- If `nv-codec-headers` fails, ensure you have the kernel headers and a compatible toolchain.  
- If OpenCV CMake cannot find FFMPEG/CUDA, check `FFMPEG_DIR` and your CUDA install path.  
- Use `jtop` while building to monitor thermals and throttling.

---

## 🔗 References
- NVIDIA JetPack / SDK Manager documentation  
- nv-codec-headers: https://github.com/FFmpeg/nv-codec-headers  
- FFmpeg source: https://git.ffmpeg.org/ffmpeg.git  
- OpenCV: https://github.com/opencv/opencv  

