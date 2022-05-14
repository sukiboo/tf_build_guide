# Building TensorFlow from source

This is a step-by-step guide for building a CPU version of TensorFlow on Linux and MacOS.
Official instructions are available at https://www.tensorflow.org/install/source but they are not detailed enough and I'm tired of troubleshooting the same issues every time I need to set up a new machine.
I wrote this guide mainly to avoid an unnecessary frustration in the future, though someone else might find it useful as well.


## Why building from source when you can just `pip install`?

Have you seen messages like these:
```
This TensorFlow binary is optimized with oneAPI Deep Neural Network Library (oneDNN) 
to use the following CPU instructions in performance-critical operations:  AVX2 FMA
To enable them in other operations, rebuild TensorFlow with the appropriate compiler flags.
```
Then you can speed up your code by 20+% if you just build TensorFlow from source.
All you need is the terminal window, this guide and several hours of free time!


## Instructions

### 1. Install prerequisites
Make sure you have `python` and `pip` installed already.
On MacOS you will need Xcode 9.2 or later; check your version with `xcodebuild -version`.

### 2. Choose the version of TensorFlow you want to install
It might depend on your particular needs or the version of Python you're using. 
If you're not sure, a good way to find the latest compatible version is to do a pip-install and then check the version:
```
pip install tensorflow
pip show tensorflow
```
On my current machine it is `2.6.2`, which is what I'm going to use for the rest of this guide.

### 3. Find the version of Bazel you will need
First, find the required version of Bazel in the `.bazelversion` file of the repository of your version of TensorFlow: https://github.com/tensorflow/tensorflow/blob/v2.6.2/.bazelversion
In my case it is `3.7.2` so that's what I'm going to install.

### 4. Install Bazel
Download Bazel installer from the relesases page: https://github.com/bazelbuild/bazel/releases/tag/3.7.2 and install it and with `--user` flag:
```
chmod +x bazel-3.7.2-installer-linux-x86_64.sh
./bazel-3.7.2-installer-linux-x86_64.sh --user
```

### 5. Download TensorFlow source files
Download your version (from step 2.) of TensorFlow from https://github.com/tensorflow/tensorflow:
```
git clone -b v2.6.2 https://github.com/tensorflow/tensorflow.git
```

### 6. Configure TensorFlow
Specify the configuration that you need.
I typically use the default options but your needs may vary.
```
cd tensorflow
./configure
```
If Bazel is not found, run `export PATH="$PATH:$HOME/bin"`

### 7. Compile TensorFlow
Specify the optimization flags that you need or choose `arch=native` to use the flags supported by your CPU, which is what I use:
```
bazel build --config=opt --copt=-march=native -k //tensorflow/tools/pip_package:build_pip_package
```
This is the most convoluted step that typically takes several hours to complete and many things can go wrong.
For instance, here are some of the problems I've experienced in the past:
- if a package is missing, i.e. `keras-preprocessing`, install it with `pip install keras-preprocessing`
- if python2 is not found create a symlink via `sudo ln -s /usr/bin/python3 /usr/bin/python`
- if compilation breaks for no appearent reason, check the logs by running `dmesg`; if you see out of memory errors, e.g. `Out of memory: Kill process 23747 (cc1plus)`, try one of the following fixes:
  - specify the memory size by adding `--local_ram_resources=8192` flag
  - reduce the number of workers by adding `-j 1` flag
- on MacOS, make sure that you're using the correct version of Xcode; if you're not, install and switch via `sudo xcode-select -switch /path_to_xcode/Xcode.app`
- if nothing else helps, try running `bazel sync` to check the dependencies

### 8. Build TensorFlow
If the previous step completed without errors, this should be a pretty straightforward process.
```
./bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg
```

### 9. Install TensorFlow
Finally, install the package.
Keep in mind that the tags of your package will likely differ from mine!
If you have the same version installed already, either uninstall it or run pip with the `--force-reinstall` flag.
```
pip install /tmp/tensorflow_pkg/tensorflow-2.6.2-cp36-cp36m-linux_x86_64.whl
```

### 10. Verify successful installation
Import TensorFlow to check your installation -- make sure to leave the current directory first:
```python
import tensorflow as tf
tf.Variable(0)
```
If no warnings appear then all is well and you can enjoy your well-earned speed up!
