---
layout: post
title: "Install OpenCV3 on Anaconda3 & python3.6 Mac"
date: 2017-12-24 00:00:01
---

```bash
$ git clone https://github.com/opencv/opencv
$ git clone https://github.com/opencv/opencv_contrib
$ cd ~/opencv
$ mkdir build
$ cd build
$ cmake -D CMAKE_BUILD_TYPE=RELEASE \
        -D PYTHON3_LIBRARY=/Users/<username>/anaconda/envs/<env>/lib/libpython3.6m.dylib \
        -D PYTHON3_INCLUDE_DIR=/Users/<username>/anaconda/envs/<env>/include/python3.6m/ \
        -D PYTHON_DEFAULT_EXECUTABLE=/Users/<username>/anaconda/envs/<env>/bin/python3.6 \
        -D PYTHON_PACKAGES_PATH=/Users/<username>/anaconda/envs/<env>/lib/python3.6/site-packages \
        -D INSTALL_C_EXAMPLES=OFF \
        -D INSTALL_PYTHON_EXAMPLES=ON \
        -D BUILD_EXAMPLES=ON \
        -D BUILD_opencv_python3=ON \
        -D BUILD_opencv_python2=OFF \
        -D INCLUDE_DIRECTORIES=/Users/<username>/anaconda/include/python3.6m/ \
        -D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib/modules ..
$ make -j4
$ make install
```

```bash
$ python
import cv2
ImportError: No module named cv2
```

Fix python imports.

```bash
$ cd ~/opencv/lib/python3.6/site-packages
$ mv cv2.cpython-36m-darwin.so cv2.so
$ cp cv2.so /Users/<username>/anaconda3/lib/python3.6/site-packages
```

```python
$ python
import cv2
cv2.__version__
'3.2.0-dev'
```

* [Can't install OpenCV3 on Anaconda3 python3.6 on macOS][r1]
[r1]: https://stackoverflow.com/questions/41873941/cant-install-opencv3-on-anaconda3-python3-6-on-macos/42704182#42704182
