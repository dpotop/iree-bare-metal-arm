# IREE Bare-Metal Arm Sample on Raspberry Pi 3

This is a fork of the original repository meant to allow execution on Raspberry Pi 3 SBCs (instead of the various types of platforms proposed by renode).

This projects demonstrates how to build [IREE](https://github.com/google/iree) with the [GNU Arm Embedded Toolchain](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm) for bare-metal Raepberry Pi targets.

**DISCLAIMER**:
This project is not intended for everyday use and made available without any support.
However, we welcome any kind of feedback via the issue tracker or if appropriate via IREE's [communication channels](https://github.com/google/iree#communication-channels), e.g. via the Discord server.

## Getting Started

### Prerequisites

You need CMake and the [GNU Arm Embedded Toolchain](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm) installed on your host machine. Further, a C/C++ compiler is required to build IREE for your host machine.

### Clone and Build
#### Clone

```shell
git clone https://github.com/dpotop/iree-bare-metal-arm.git
cd iree-bare-metal-arm
git submodule update --init
cd third_party/iree
git submodule update --init
cd ../../
```
> Note:<br>
> &nbsp;&nbsp;&nbsp;&nbsp;The submodules used within IREE themself include submodules, so that we advice to avoid an recursive update.

> Note:<br>
> &nbsp;&nbsp;&nbsp;&nbsp;It may happen that new submodules are added to IREE.<br>
> &nbsp;&nbsp;&nbsp;&nbsp;Thus it might not be sufficient to only pull the latest master and you may need update the submodules manually.

#### Host Build

You will need an installation of IREE on your host machine. The lines below can be used to build IREE from the already cloned submodule:

```shell
mkdir build-iree-host
cd build-iree-host

cmake -GNinja \
      -DCMAKE_C_COMPILER=clang \
      -DCMAKE_CXX_COMPILER=clang++ \
      -DIREE_HAL_DRIVERS_TO_BUILD="VMVX_Sync" \
      -DIREE_TARGET_BACKENDS_TO_BUILD="VMVX;DYLIB-LLVM-AOT" \
      -DIREE_BUILD_SAMPLES=OFF \
      -DIREE_BUILD_TESTS=OFF \
      -DCMAKE_INSTALL_PREFIX=../build-iree-host-install \
      ../third_party/iree/
cmake --build . --target install
cd ..
```

For further information, see the [Getting started](https://google.github.io/iree/building-from-source/getting-started/) guide.

#### Target Build

One can choose to either build the sample with libopencm3 or with CMSIS by either setting `BUILD_WITH_LIBOPENCM3` or `BUILD_WITH_CMSIS` to `ON`.
Depending on whether you build with libopencm3 or CMSIS, you need to pass the correct linker flags via `CUSTOM_ARM_LINKER_FLAGS`
and need to specify the appropriate linker script via `PATH_TO_LINKER_SCRIPT`.

```shell
mkdir build
cd build

# Set the path to the GNU Arm Embedded Toolchain, e.g.
# export PATH_TO_ARM_TOOLCHAIN="/usr/local/gcc-arm-none-eabi-10-2020-q4-major"

# To build with CMSIS
# export CUSTOM_ARM_LINKER_FLAGS="-lnosys"
# export PATH_TO_LINKER_SCRIPT="`realpath ../build_tools/stm32f407.ld`"

# To build with libopencm3
# export CUSTOM_ARM_LINKER_FLAGS="-nostartfiles"
# export PATH_TO_LINKER_SCRIPT="`realpath ../build_tools/stm32f4-discovery.ld`"


# export PATH_TO_IREE_HOST_BINARY_ROOT="`realpath ../build-iree-host-install`"

cmake -GNinja \
#     -DBUILD_WITH_CMSIS=ON \
#     -DBUILD_WITH_LIBOPENCM3=ON \
      -DCMAKE_TOOLCHAIN_FILE="`realpath ../build_tools/cmake/arm.toolchain.cmake`" \
      -DARM_TOOLCHAIN_ROOT="${PATH_TO_ARM_TOOLCHAIN}" \
      -DARM_CPU="armv8-a" \
      -DIREE_HAL_DRIVERS_TO_BUILD="VMVX_Sync" \
      -DIREE_HOST_BINARY_ROOT="${PATH_TO_IREE_HOST_BINARY_ROOT}" \
      -DCUSTOM_ARM_LINKER_FLAGS="${CUSTOM_ARM_LINKER_FLAGS}" \
      -DLINKER_SCRIPT="${PATH_TO_LINKER_SCRIPT}" \
      ..
cmake --build . --target simple_embedding
```
