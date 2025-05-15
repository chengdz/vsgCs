# Build and Installation

## Prerequisites

vsgCs manages its dependencies using the
[vcpkg](https://github.com/microsoft/vcpkg) package manager. You
should clone that repository and follow the steps described in [vcpkg
with CMake](https://learn.microsoft.com/en-us/vcpkg/get_started/get-started?pivots=shell-bash).

On Linux, vcpkg will complain if some development packages aren't
installed in the system, although it's unclear if this makes any
practical difference. On Fedora, for example, you can install
them with:
```
sudo dnf install xorg-macros xproto xcb-proto libXdmcp-devel libXau-devel
```

## Command line build instructions:

Once you have set the `VCPKG_ROOT` environment variable, you can build
vsgCS using a CMake preset defined in `CMakePresets.json`, for example:

```
export VCPKG_ROOT=<location of vcpkg repository>
cmake --preset vcpkg-debug
cmake --build build-debug --target install -j 10
```

The `cmake` files for vsgCs don't explicitly depend on vcpkg, so you
can install vsgCs' dependencies yourself e.g., using your system's
package manager. Look in vsgCs' `CMakeLists.txt` files for calls to
`find_package` for prerequisites, as well as in the `vcpkg.json` files
in the `extern/vcpkg-overlays` subdirectories.

## OpenSSL and Linux

OpenSSL is a core library that, while not used directly by vsgCs, is
used by several of its dependencies. In particular libCurl, which does
all of vsgCs' http transfers, uses it to set up SSL connections. By
default the OpenSSL library is supplied by vcpkg. However, there may
be a need to use the host system's own OpenSSL implementation, for
example to take advantage of a configuration option that isn't
implemented in the vcpkg version of OpenSSL. You can set the
`VSGCS_USE_SYSTEM_OPENSSL` CMake variable to enable this behavior.

## Cesium Native

vsgCs uses the official sources of Cesium Native and tracks its
monthly releases. The commit of Cesium Native used in the build is set
in [extern/vcpkg-overlays/cesium-native/portfile.cmake](extern/vcpkg-overlays/cesium-native/portfile.cmake).

vsgCs is often used as a vehicle for developing Cesium Native, and it
would be quite painful to create a vcpkg port and package for each
incremental change. You can include Cesium Native as a CMake
subdirectory from its source subdirectory using the
`VSGCS_CESIUM_NATIVE_DIRECTORY` cache variable. The CMake
`local-cesium-native` preset in `CMakePresets.json` uses vcpkg to get
Cesium Native's dependencies without Cesium Native itself, and so is
ideal for this usage. You can create a preset in a local
`CMakeUserPresets.json` file that configures this case:
```
{
  "version": 3,
  "configurePresets": [
    {
      "name": "my-local-native",
        "inherits": ["vcpkg-debug", "local-cesium-native"],
      "binaryDir": "${sourceDir}/build-debug",
      "cacheVariables": {
        "VSGCS_CESIUM_NATIVE_DIRECTORY": "/home/foo/CesiumGS/cesium-native"
      }
    }
  ]
}
```
