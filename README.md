# TwisterOS-Imager

TwisterOS Imaging Utility

## Download

- Download the latest version for Raspberry Pi OS from the [TwisterOS downloads page](https://oxmc.github.io/TwisterOS/downloads/).

### Download-other

Windows and MacOS are currently not supported.
They will be supported soon.

# How to compile

## Debian/Ubuntu

### Get dependencies

Install the build dependencies:

```
sudo apt install -y --no-install-recommends build-essential devscripts debhelper cmake git libarchive-dev libcurl4-openssl-dev \
    qtbase5-dev qtbase5-dev-tools qtdeclarative5-dev libqt5svg5-dev qttools5-dev qt5-default libssl-dev \
    qml-module-qtquick2 qml-module-qtquick-controls2 qml-module-qtquick-layouts qml-module-qtquick-templates2 qml-module-qtquick-window2 qml-module-qtgraphicaleffects
```

#### Get the source

```
git clone --depth 1 https://github.com/oxmc/TwisterOS-Imager
```

#### Build the Debian package

```
cd TwisterOS-Imager
debuild -uc -us
```

debuild will compile everything, create a .deb package and put it in the parent directory.
You can install it with apt:

```
cd ..
sudo apt install ./ob-imager*.deb
```

It should create an icon in the start menu under "Utilities" or "Accessories".
The imaging utility will normally be run as regular user, and will call udisks2 over DBus to perform privileged operations like opening the disk device for writing.
If udisks2 is not functional on your Linux distribution, you can alternatively start it as "root" with sudo and similar tools.

### Fedora/RHEL/CentOS Linux

#### Get dependencies

Install the build dependencies:

```
sudo yum install git gcc gcc-c++ make cmake libarchive-devel libcurl-devel openssl-devel qt5-qtbase-devel qt5-qtquickcontrols2-devel qt5-qtsvg-devel qt5-linguist
```

#### Get the source

```
git clone --depth 1 https://github.com/oxmc/TwisterOS-Imager
```

#### Build and install the software

```
cd TwisterOS-Imager
cmake .
make
sudo make install
```

### Windows

#### Get dependencies

- Get the Qt online installer from: https://www.qt.io/download-open-source
During installation, choose a Qt 5.x with Mingw32 32-bit toolchain and CMake.

- If using the official Qt distribution that does NOT have schannel (Windows native SSL library) support, compile OpenSSL libraries ( https://wiki.qt.io/Compiling_OpenSSL_with_MinGW ) and copy the libssl/crypto DLLs to C:\qt\5.x\mingw73_32\bin

- For building installer get Nullsoft scriptable install system: https://nsis.sourceforge.io/Download

- It is assumed you already have a proper code signing certificate, and signtool.exe from the Windows SDK installed.
If NOT and are you only compiling for your own personal use, comment out all lines mentioning signtool from CMakelists.txt and the .nsi installer script.

#### Building

Building can be done manually using the command-line, using "cmake", "make", etc., but if you are not that familar with setting up a proper Windows build environment (setting paths, etc.), it is easiest to use the Qt creator GUI instead.

- Download source .zip from github and extract it to a folder on disk
- Open CMakeLists.txt in Qt creator.
- For builds you distribute to others, make sure you choose "Release" in the toolchain settings and not the debug flavour.
- Menu "Build" -> "Build all"
- Result will be in ../build_rpi-imager_someversion
- Go to the BUILD folder, right click on the .nsi script "Compile NSIS script", to create installer.

Note: the CMake integration in Qt Creator is a bit flaky at times. If you made any custom changes to the CMakeLists.txt file and it subsequently gets in an endless loop where it never finishes the "configures" stage while re-processing the file, delete "build_rpi-imager_someversion" directory and try again.

### Mac OS X

#### Get dependencies

- Get the Qt online installer from: https://www.qt.io/download-open-source
During installation, choose a Qt 5.x edition and CMake.
- For creating a .DMG for distribution you can use an utility like: https://github.com/sindresorhus/create-dmg
- It is assumed you have an Apple developer subscription, and already have a "Developer ID" code signing certificate for distribution outside the Mac Store. (Privileged apps are not allowed in the Mac store)

#### Building

- Download source .zip from github and extract it to a folder on disk
- Start Qt Creator (may need to start "finder" navigate to home folder using the "Go" menu, and find Qt folder to start it manually as it may not have created icon in Applications), and open CMakeLists.txt
- Menu "Build" -> "Build all"
- Result will be in ../build_rpi-imager_someversion
- For distribution to others: code sign the .app, create a DMG, code sign the DMG, submit it for notarization to Apple and staple the notarization ticket to the DMG.

E.g.:

```
cd build-rpi-imager-Desktop_Qt_5_14_1_clang_64bit-Release/
codesign --deep --force --verify --verbose --sign "YOUR KEYID" --options runtime rpi-imager.app
mv rpi-imager.app "Raspberry Pi Imager.app"
create-dmg Raspberry\ Pi\ Imager.app
mv Raspberry\ Pi\ Imager\ .dmg imager.dmg
xcrun altool --notarize-app -t osx -f imager.dmg --primary-bundle-id="org.raspberrypi.imagingutility" -u YOUR-EMAIL-ADDRESS -p YOUR-APP-SPECIFIC-APPLE-PASSWORD -itc_provider TEAM-ID-IF-APPLICABLE
xcrun stapler staple imager.dmg
```

## Other notes

### Debugging

On Linux and Mac the application will print debug messages to console by default if started from console.
On Windows start the application with the command-line option --debug to let it open a console window.

### Custom repository

### NOTE: We are using a url to get the image list, you can change it in [`config.h`](https://github.com/oxmc/TwisterOS-Imager/blob/c90da69aff9fb693c29b51b1a03d5224d09d72ce/config.h#L11)

If the application is started with "--repo [your own URL]" it will use a custom image repository.
So you can simply create another 'start menu shortcut' to the application with that parameter to use the application with your own images.

### Telemetry

### NOTE: I have disabled Telementry by default, every thing below does not apply.

In order to understand which images and operating systems are most popular and re-organise the application accordingly, when using the default image repository, the URL, operating system name and category (if present) of a selected image are sent to https://rpi-imager-stats.raspberrypi.org by [`downloadstatstelemetry.cpp`](https://github.com/oxmc/TwisterOS-Imager/blob/main/downloadstatstelemetry.cpp).

This web service is hosted by [Heroku](https://www.heroku.com) and only stores an incrementing counter using a [Redis Sorted Set](https://redis.io/topics/data-types#sorted-sets) for each URL, operating system name and category per day in the `eu-west-1` region and does not associate any personal data with those counts. This allows us to query the number of downloads over time and nothing else.

The last 1,500 requests to the service are logged for one week before expiring as this is the [minimum log retention period for Heroku](https://devcenter.heroku.com/articles/logging#log-history-limits).

On Windows, you can opt out of telemetry by disabling it in the Registry:

```
reg add "HKCU\Software\Raspberry Pi\Imager" /v telemetry /t REG_DWORD /d 0
```

On Linux, run `ob-imager --disable-telemetry` or add the following to `~/.config/Raspberry Pi/Imager.conf`:

```ini
[General]
telemetry=false
```

On macOS, disable it by editing the property list for the application:

```
defaults write org.raspberrypi.Imager.plist telemetry -bool NO
```
