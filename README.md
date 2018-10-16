# About
This repository is forked from [BVLC/caffe](https://github.com/BVLC/caffe).  

### What's New
* Add VS 2017 and CUDA 10.0 support
* Use newer version of dependencies generated from [codelilei/caffe-builder](https://github.com/codelilei/caffe-builder)
* Fix some details to adapt the high version compiler and build components


# Windows Caffe

**This is an experimental, community based branch led by Guillaume Dumont (@willyd). It is a work-in-progress.**

This branch of Caffe ports the framework to Windows.



## Windows Setup

### Requirements

 - Visual Studio 2017
     - Technically only the VS C/C++ compiler is required (cl.exe)
 - [CMake](https://cmake.org/) 3.12 or higher (Currently only Visual Studio generators are supported)


### Optional Dependencies

 - Python for the pycaffe interface. Anaconda Python 3.6 x64 (or Miniconda)
 - Matlab for the matcaffe interface.
 - CUDA 9.x or 10.0
 - cuDNN v7.x

 We assume that `cmake.exe` and `python.exe` are on your `PATH`.


### Install caffe dependencies

Before starting, build caffe dependencies by following the instructions in the [codelilei/caffe-builder](https://github.com/codelilei/caffe-builder)  [README](https://github.com/codelilei/caffe-builder/blob/master/README.md).


### Configuring and Building Caffe

After completing the last step, modify cmake\WindowsDownloadPrebuiltDependencies.cmake at line 49, change the path to the root of what you just built to, e.g. D:/caffe-builder/build.  
Execute the following commands in a `cmd` prompt (we use `C:\Projects` as a root folder for the remainder of the instructions):
```cmd
C:\Projects> git clone https://github.com/codelilei/caffe.git
C:\Projects> cd caffe
C:\Projects\caffe> git checkout windows
:: Edit any of the options inside build_win_vs17.cmd to suit your needs
C:\Projects\caffe> scripts\build_win_vs17.cmd
```
The `build_win_vs17.cmd` script will create the Visual Studio project files and build the Release configuration. By default all the required DLLs will be copied next to the consuming binaries. If you wish to disable this option, you can by changing the command line option `-DCOPY_PREREQUISITES=0`. The prebuilt libraries also provide a `prependpath.bat` batch script that can temporarily modify your `PATH` environment variable to make the required DLLs available.


Below is a more complete description of some of the steps involved in building caffe.



### Use cuDNN

To use cuDNN the easiest way is to copy the content of the `cuda` folder into your CUDA toolkit installation directory. For example if you installed CUDA 10.0 and downloaded cudnn-10.0-windows10-x64-v7.3.0.29.zip you should copy the content of the `cuda` directory to `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v10.0`. Alternatively, you can define the CUDNN_ROOT cache variable to point to where you unpacked the cuDNN files e.g. `C:/Projects/caffe/cudnn-10.0-windows10-x64-v7.3.0.29/cuda`. For example the command in [scripts/build_win_vs17.cmd](scripts/build_win_vs17.cmd) would become:
```
cmake -G"!CMAKE_GENERATOR!" ^
      -DBLAS=Open ^
      -DCMAKE_BUILD_TYPE:STRING=%CMAKE_CONFIG% ^
      -DBUILD_SHARED_LIBS:BOOL=%CMAKE_BUILD_SHARED_LIBS% ^
      -DBUILD_python:BOOL=%BUILD_PYTHON% ^
      -DBUILD_python_layer:BOOL=%BUILD_PYTHON_LAYER% ^
      -DBUILD_matlab:BOOL=%BUILD_MATLAB% ^
      -DCPU_ONLY:BOOL=%CPU_ONLY% ^
      -DCUDNN_ROOT=C:/Projects/caffe/cudnn-10.0-windows10-x64-v7.3.0.29/cuda ^
      -C "%cd%\libraries\caffe-builder-config.cmake" ^
      "%~dp0\.."
```

Alternatively, you can open `cmake-gui.exe` and set the variable from there and click `Generate`.

### Building only for CPU

If CUDA is not installed Caffe will default to a CPU_ONLY build. If you have CUDA installed but want a CPU only build you may use the CMake option `-DCPU_ONLY=1`.

### Using the Python interface

The recommended Python distribution is Anaconda or Miniconda. To successfully build the python interface you need to add the following conda channels:
```
conda config --add channels conda-forge
conda config --add channels willyd
```
and install the following packages:
```
conda install --yes cmake ninja numpy scipy protobuf==3.6.1 six scikit-image pyyaml pydotplus graphviz
```
If Python is installed the default is to build the python interface and python layers. If you wish to disable the python layers or the python build use the CMake options `-DBUILD_python_layer=0` and `-DBUILD_python=0` respectively. In order to use the python interface you need to either add the `C:\Projects\caffe\python` folder to your python path or copy the `C:\Projects\caffe\python\caffe` folder to your `site_packages` folder.

### Using the MATLAB interface

Follow the above procedure and use `-DBUILD_matlab=ON`. Change your current directory in MATLAB to `C:\Projects\caffe\matlab` and run the following command to run the tests:
```
>> caffe.run_tests()
```
If all tests pass you can test if the classification_demo works as well. First, from `C:\Projects\caffe` run `python scripts\download_model_binary.py models\bvlc_reference_caffenet` to download the pre-trained caffemodel from the model zoo. Then change your MATLAB directory to `C:\Projects\caffe\matlab\demo` and run `classification_demo`.

### Using the Ninja generator

Currently not supported for this version cause of some unexpected problems not addressed.

### Building a shared library

CMake can be used to build a shared library instead of the default static library. To do so follow the above procedure and use `-DBUILD_SHARED_LIBS=ON`. Please note however, that some tests (more specifically the solver related tests) will fail since both the test executable and caffe library do not share static objects contained in the protobuf library.

### Troubleshooting

Should you encounter any error please contact the author.

## Known issues

- The `GPUTimer` related test cases always fail on Windows. This seems to be a difference between UNIX and Windows.
- Shared library (DLL) build will have failing tests.
- Shared library build only works with the Ninja generator

## Further Details

Refer to the BVLC/caffe master branch README for all other details such as license, citation, and so on.
