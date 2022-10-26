#TorchVision ops for libtorch

##Intruction

This is cpp version of torchvision ops for libtorch

##Usage

```
mkdir build
cd build
cmake .. -DWITH_CUDA=on
```

link generated libvisionop.so in your project to use torchvision ops, e.g. roi_align, nms.

[Here](https://github.com/vghost2008/mmdetection_cpp) is a example.
