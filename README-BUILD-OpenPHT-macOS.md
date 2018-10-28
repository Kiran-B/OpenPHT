# Building instructions for Open PHT on a macOS

## Generic instructions

### Get the source

```
git clone https://github.com/Kiran-B/OpenPHT.git
git checkout openpht-1.9
```

Or download a tarfile from https://github.com/Kiran-B/OpenPHT/archive/openpht-1.9.zip

### CMake options

Our CMake project handles the following variables:

* OSX_SDK_VERSION=[10.8] - Only available on OSX, decides which SDK version you should use.
* OSX_ARCH=[i386/x86_64] - Only available on OSX, decide what arch you to build for. Defaults to x86_64
* COMPRESS_TEXTURES=[on/off] - If we should compress skin textures. It's expensive and takes a lot of time. If you are just building for yourself or for development, I suggest you turn this off.
* CREATE_BUNDLE=[on/off] - Generates a finalized release bundle and signing the code. You usually want to set this to off.
* ENABLE_AUTOUPDATE=[on/off] - If you want to include the autoupdate code. Should be disabled on Linux since it won't work there.
* ENABLE_DUMP_SYMBOLS=[on/off] - Enable crashreporter system. Should be set to off on developer builds and on Linux.
* CMAKE_INSTALL_PREFIX=[path] - When you run the install command this is where it will be installed.

You also need to pass the -G option to CMake to tell what generator to use, I suggest Ninja since it's fast.

Example, if you want to build a developer build locally:

```
cmake -GNinja -DCMAKE_INSTALL_PREFIX=/usr/local/PlexHomeTheater -DCOMPRESS_TEXTURES=off -DCREATE_BUNDLE=off -DENABLE_DUMP_SYMBOLS=off [path to source]
```

## MacOSX

You need to install the following things:

* [Xcode 10](https://developer.apple.com/download/)
* [Homebrew](https://brew.sh)

Homebrew should install Xcode CLI. Verify that CLI is installed by running the below command
```
xcode-select --install
```
If CLI is installed, it would put up the following message; otherwise it would offer to install:
```
xcode-select: error: command line tools are already installed, use "Software Update" to install updates
```

When you have those installed you need to install the following things from Homebrew

* git
* cmake
* gnu-tar
* xz
* ninja (optional)
* ccache (optional)

```
brew install git cmake gnu-tar xz ninja ccache
```

### Fetch dependencies
This will try to download the dependencies from our(?) server so you don't need to build them. But if they can't be found you might need to build them yourself:

```
cd ./OpenPHT
plex/scripts/build_osx_depends.sh osx64 (or osx for 32bit build)
```

When that is done (takes a long time dudes) you may use Xcode or Ninja to build the app.

Note:
- The above command needs Internet connectivity. It downloads about ~200 MB of data.
- The above command would use the path /Users/Shared/xbmc-depends by default. Ensure that it is accessible.
- This command could fail(it failed for me with a crash of m4). If it happens, run the following command:
```
mv /Users/Shared/xbmc-depends/toolchain/bin/m4 /Users/Shared/xbmc-depends/toolchain/bin/m4.bak && ln -s /usr/bin/m4 /Users/Shared/xbmc-depends/toolchain/bin/m4
```

## Ninja build (if you prefer building over terminal; i.e. without dealing with Xcode IDE)

Below steps assumes you are in OpenPHT directory.

```
cd ..
mkdir pht-ninja-build
cd pht-ninja-build
cmake -GNinja -DCMAKE_INSTALL_PREFIX=/Users/Shared/OpenPHT -DCOMPRESS_TEXTURES=off -DCREATE_BUNDLE=on -DENABLE_AUTOUPDATE=off -DENABLE_DUMP_SYMBOLS=off ../OpenPHT
ninja install
```

This will generate a build at location specified by the parameter (-DCMAKE_INSTALL_PREFIX; which is /Users/Shared/OpenPHT in the example above)

Note:
- If you do see ```Verifing signature
/Users/Shared/OpenPHT/OpenPHT.app: code object is not signed at all``` the app is probably not codesigned as you do not have a 'Developer ID' certificate installed. This could be ignored. However, firewall (if enabled) could keep asking about allowing incoming connections repeatedly. A poassible workaround would be:
```
cd /Users/Shared/OpenPHT/OpenPHT.app/Contents/Frameworks
find . -name '*.dylib' -exec sudo codesign --force --sign - {} \; -prune
cd ../../..
sudo codesign --force --sign - ./OpenPHT.app
```


### Xcode build

Below steps assumes you are in OpenPHT directory.

```
cd .. (Move out of OpenPHT repo)
mkdir pht-xcode-build
cd pht-xcode-build
cmake -GXcode -DCMAKE_INSTALL_PREFIX=/Users/Shared/OpenPHT -DCOMPRESS_TEXTURES=off -DCREATE_BUNDLE=on -DENABLE_AUTOUPDATE=off -DENABLE_DUMP_SYMBOLS=off ../OpenPHT
```


When CMake is finished you should now have a OpenPHT.xcodeproj that you can open in Xcode and build, make sure you build the "Install" target. To run OpenPHT inside Xcode you need to edit schemes, select the Run option and then select the binary that is installed into the path you specified in CMAKE_INSTALL_PATH. When you press run now it will run the install target and then run the binary we output.

