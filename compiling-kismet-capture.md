# Compiling Kismet Capture for OpenWrt

To follow this guide, you'll need to have access to a Debian-based Linux distro. Ubuntu on WSL works fine.

## Prepare Build Environment
Install packages for Debian system:
```bash
sudo apt update
sudo apt install build-essential clang flex bison g++ gawk \
gcc-multilib g++-multilib gettext git libncurses5-dev libssl-dev \
python3-setuptools rsync swig unzip zlib1g-dev file wget
```

Get sources (note: we are using the kismet-packages repot):
```bash
git clone https://github.com/openwrt/openwrt.git
git clone https://github.com/kismetwireless/kismet-packages.git
cp -r kismet-packages/openwrt/kismet-openwrt/ openwrt/package/network/
cd openwrt
```

Select target OpenWrt version:
```bash
git branch -a
git tag
git checkout v23.05.4
```

## Enable Websockets (Optional)
Edit `package/network/kismet-openwrt/kismet.mk` and remove `--disable-libwebsockets` from the configure options:
```makefile
CONFIGURE_OPTS := \
	--sysconfdir=/etc/kismet \
	--bindir=/usr/bin \
	--disable-python-tools \
	--with-protoc=$(STAGING_DIR_HOSTPKG)/bin/protoc \
	--enable-protobuflite \
	--disable-element-typesafety \
	--disable-debuglibs \
	--disable-libcap \
	--disable-libnm \
	--disable-wifi-coconut
```

Edit `package/network/kismet-openwrt/kismet-capture-linux-wifi/Makefile` and add `+libwebsockets-mbedtls` to its dependencies:
```makefile
define Package/kismet-capture-linux-wifi
  SECTION:=net
  CATEGORY:=Network
  TITLE:=Kismet Wi-Fi Capture Support
  URL:=https://www.kismetwireless.net
  DEPENDS:=+libpthread +libpcap +libnl +libcap +protobuf-lite +libprotobuf-c +libwebsockets-mbedtls
  SUBMENU:=kismet
endef
```

## Finish Setting Up Build Environment
```bash
# Update the feeds
./scripts/feeds update -a
./scripts/feeds install -a

# Configure the firmware image
make menuconfig
```

Select your router's Target System/Subtarget/Profile. For example, the Linksys E8450 will use:
* Target System: MediaTek Ralink ARM
* Subtarget: MT7622
* Target Profile: Linksys E8450

Next, navigate to `Network > kismet > kismet-capture-linux-wifi`. Press `SPACE` to select it as a module `<M>`.

Exit the menu, saving your changes to `.config`.

## Compile the Package
This will take some time. Add the -j option to use multiple cores. For example, `make -j4`.
```bash
make tools/install
make toolchain/install

make package/network/kismet-openwrt/kismet-capture-linux-wifi/compile
```
After compilation, the package can be found at:
```
bin/packages/[TARGET]/base/kismet-capture-linux-wifi*.ipk
```


## Installing

After copying the package to your OpenWrt router.

```bash
opkg update
opkg install ./kismet-capture-linux-wifi*.ipk

kismet_cap_linux_wifi --help
```

Refer to the offical Kismet documentation to set-up a Kismet server and use your OpenWrt router as a datasource.


## Conclusion

This process uses a couple of Makefiles from a Kismet repot, placed into the /packages dir. They can be edited and reused to fit your needs. For example, you may want to edit the Makefiles to use a different version of Kismet.

## References
* https://www.kismetwireless.net/docs/readme/installing/
* https://www.kismetwireless.net/docs/readme/remotecap/remotecap/
* https://openwrt.org/docs/guide-developer/toolchain/install-buildsystem
* https://openwrt.org/docs/guide-developer/toolchain/use-buildsystem
* https://openwrt.org/docs/guide-developer/toolchain/single.package