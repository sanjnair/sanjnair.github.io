
![Tensorflow_Lite_C++](/assets/images/tensorflow_lite_c++.png)
# Android App using Tflite C++ API

In this blog, I'll show you how to build an Android app that uses Tflite C++ API for loading and running tflite models.
Please note that this tutorial assumes you are using Ubuntu 16.04 system.
<!--more-->

## Building Tflite shared library

We'll build Tflite shared library from tensorflow sources. Following steps illustrate how to build Tflite shared library from sources.

#### Get tensorflow code

```bash
mkdir -p $HOME/tf
cd $HOME/tf
git clone https://github.com/tensorflow/tensorflow.git

```

#### Install Bazel to build tensorflow
Follow instructions from https://docs.bazel.build/versions/master/install-ubuntu.html.

#### Install Tensorflow from Sources
Follow instructions from https://www.tensorflow.org/install/install_sources

#### Build Tflite

* Edit `$HOME/tf/tensorflow/tensorflow/contrib/lite/BUILD`. Add the following code to the end of the file:

```
# Build Tensorflowlite Library
cc_binary(
    name = "libtflite.so",
    deps = [
        ":framework",
        "//tensorflow/contrib/lite/kernels:builtin_ops",
    ],
    linkshared=1
)
```

* From shell, run the following commands:

```bash
cd $HOME/tf/tensorflow/
bazel build -c opt //tensorflow/contrib/lite:libtflite.so --crosstool_top=//external:android/crosstool --host_crosstool_top=@bazel_tools//tools/cpp:toolchain --config=android_arm64 --cpu=arm64-v8a --fat_apk_cpu=arm64-v8a --cxxopt="-std=c++14"
```

* Copy tflite.so file from `/home/sanjay/tf/tensorflow/bazel-bin\*` folder to a local folder.


## Integrating `libtflite.so` with Android app
I'm using Android Studio version 3.14 and Android NDK Version 17.1


#### Make changes to `CmakeLists.txt`

```
set(HOME_DIR                $ENV{HOME})
set(TF_DIR                  ${HOME_DIR}/tf/tensorflow)
set(FLATBUFFER_DIR          ${TF_DIR}/tensorflow/contrib/lite/downloads/flatbuffers/include)
set(TFLITE_LIB_DIR          ${HOME_DIR}/android-app/jniLibs/arm64-v8a)
...

include_directories(
    <other include dirs>,
    ${FLATBUFFER_DIR}
    ${TF_DIR}
)

link_directories(${TFLITE_LIB_DIR})
find_library(libtflite tflite)
...

target_link_libraries (
    <other libraries>,
    tflite
)

```

#### Coding C++ with Tensorflow lite

Follow the interface code https://github.com/tensorflow/tensorflow/blob/master/tensorflow/contrib/lite/interpreter.h

From Google's documentation:
```C++
// An interpreter for a graph of nodes that input and output from tensors.
// Each node of the graph processes a set of input tensors and produces a
// set of output Tensors. All inputs/output tensors are referenced by index.
//
// Usage:
//
// -- Create basic model
// Interpreter foo(2, 1);
// foo.SetTensorParametersReadWrite(0, ...);
// foo.SetTensorParametersReadOnly(1, ...);
// foo.SetNodeParameters(0, ...)
//
// -- Resize input array to 1 length.
// foo.ResizeInputTensor(0, 1);
// foo.AllocateTensors();
// -- Install array data
// foo.typed_tensor<float>(0)[0] = 3;
// foo.Invoke();
// foo.typed_tensor<float>(0)[0] = 4;
// foo.Invoke();
// -- Resize input array and set data.
// foo.ResizeInputTensor(0, 2);
// foo.AllocateTensors();
// foo.typed_tensor<float>(0)[0] = 4;
// foo.typed_tensor<float>(0)[1] = 8;
// foo.Invoke();
//
```
