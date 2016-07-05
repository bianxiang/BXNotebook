[toc]

## 构建Android

The build produces a few main files inside `android_src/out/target/product/<DEVICE>/`, which are as follows:

- `system.img`: The system image file
- `boot.img`: Contains the kernel
- `recovery.img`: Contains code for recovery partition of the device

In the case of an emulator build, the preceding files will appear at `android_src/
out/target/product/generic/`.






