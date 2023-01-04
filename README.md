#TorchVision ops for libtorch

##Intruction

This is cpp version of torchvision ops for libtorch

##Usage

Linux 

```
mkdir build
cd build
cmake .. -DWITH_CUDA=on
```

Windows

```
install MSVC v142 - VS2019 C++ x64/x86 (14.28-16.9)
mkdir build
cd build
cmake .. -DWITH_CUDA=on -Tversion=14.28.29910
```



link generated libvisionop.so in your project to use torchvision ops, e.g. roi_align, nms.

[Here](https://github.com/vghost2008/mmdetection_cpp) is a example.
