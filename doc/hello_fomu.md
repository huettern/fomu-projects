# Hello Fomu

## First Start

Plug in

```bash
kernel: [11644.494558] usb 2-1.5.1: new full-speed USB device number 8 using ehci-pci
kernel: [11644.606884] usb 2-1.5.1: New USB device found, idVendor=1209, idProduct=5bf0
kernel: [11644.606889] usb 2-1.5.1: New USB device strings: Mfr=1, Product=2, SerialNumber=0
kernel: [11644.606891] usb 2-1.5.1: Product: Fomu DFU Bootloader v1.7.3-1-g82cb20c
kernel: [11644.606893] usb 2-1.5.1: Manufacturer: Foosn
mtp-probe: checking bus 2, device 8: "/sys/devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.5/2-1.5.1"
mtp-probe: bus: 2, device: 8 was not an MTP device
fwupd[25565]: (fwupd:25565): Dfu-WARNING **: DFU version is invalid: 0x0101
```

Add udev rule so that dfu can be used without sudo:

```bash
echo 'SUBSYSTEMS=="usb", ATTRS{idVendor}=="1209", ATTRS{idProduct}=="5bf0", MODE="0666"' | sudo tee /etc/udev/rules.d/42-fomu.rules
sudo udevadm control --reload-rules
sudo udevadm trigger
```

Install dfu util
```bash
sudo apt install dfu-util
```

List dfu devices
```bash
$ dfu-util -l
dfu-util 0.8

Copyright 2005-2009 Weston Schmidt, Harald Welte and OpenMoko Inc.
Copyright 2010-2014 Tormod Volden and Stefan Schmidt
This program is Free Software and has ABSOLUTELY NO WARRANTY
Please report bugs to dfu-util@lists.gnumonks.org

Found DFU: [1209:5bf0] ver=0101, devnum=9, cfg=1, intf=0, alt=0, name="Fomu DFU Bootloader v1.7.3-1-g82cb20c", serial="UNKNOWN"
```

## Toolchain setup

Install [yosys][ys] synthesis tool
```bash
sudo apt-get install build-essential clang bison flex \
	libreadline-dev gawk tcl-dev libffi-dev git \
	graphviz xdot pkg-config python3 libboost-system-dev \
	libboost-python-dev libboost-filesystem-dev
git clone https://github.com/YosysHQ/yosys
cd yosys
make -j4
sudo make install
```

Build and install [icestorm][is] project
```bash
sudo apt-get install build-essential clang bison flex libreadline-dev \
                     gawk tcl-dev libffi-dev git mercurial graphviz   \
                     xdot pkg-config python python3 libftdi-dev \
                     qt5-default python3-dev libboost-all-dev cmake
git clone https://github.com/cliffordwolf/icestorm.git icestorm
cd icestorm
make -j$(nproc)
sudo make install
```

Install build requirements for [nexrpnr](https://github.com/YosysHQ/nextpnr)
```bash
sudo apt install python3.5
sudo apt install libboost-all-dev python3-dev qt5-default clang-format cmake libeigen3-dev
```

Clone, build and install nextpnr
```bash
git clone https://github.com/YosysHQ/nextpnr
cd nextpnr
cmake -DARCH=ice40 .
make -j$(nproc)
sudo make install
```

## Build blinky

Clone my git repository containing a submodule of the official fomu test repo
```bash
git clone https://github.com/noah95/fomu-projects
cd fomu-projects
git submodule update --init --recursive
```

Compile

```bash
cd fomu-tests/blink
make FOMU_REV=hacker
```

## Tool overview

| Tool           | Used for          | Description |
|:---------------|:------------------|:------------|
| [icestorm][is] | Synthesis and pnr | Provides Lattice iCE40 chip documentation to the other tools. Database is located under `/usr/local/share/icebox`            |
| [yosys][ys]    | Synthesis         | Takes Verilog HDL files and synthesises them to hardware compatible code |

[is]: http://www.clifford.at/icestorm/
[ys]: http://www.clifford.at/yosys/


## Build bootloader

Download and install the SiFive RISC-V toolchain:
```bash
wget https://static.dev.sifive.com/dev-tools/riscv64-unknown-elf-gcc-8.2.0-2019.05.3-x86_64-linux-ubuntu14.tar.gz
tar xvf riscv64-unknown-elf-gcc-8.2.0-2019.05.3-x86_64-linux-ubuntu14.tar.gz
sudo mkdir /opt/riscv
sudo cp -r riscv64-unknown-elf-gcc-8.2.0-2019.05.3-x86_64-linux-ubuntu14/* /opt/riscv/
echo 'export PATH=$PATH:/opt/riscv/bin' >> ~/.bashrc
```

Clone the Fomu bootloader
```bash
git clone git@github.com:im-tomu/foboot.git
cd foboot/
```

Compile
```bash
cd hw/
python3 foboot-bitstream.py --revision hacker
```

This will build ...

## FAQ

### Qt problems
I had to install Qt via the [official installer](https://www.qt.io/download) and manually set the `LD_LIBRARY_PATH` value to the Qt5 installation
```bash
export LD_LIBRARY_PATH=/home/noah/Qt/5.13.0/gcc_64/lib
```

/usr/lib/x86_64-linux-gnu/cmake/Qt5/Qt5Config.cmake
/home/noah/Qt/5.13.0/gcc_64/lib/cmake/Qt5/Qt5Config.cmake
/home/noah/anaconda3/pkgs/qt-5.6.2-3/lib/cmake/Qt5/Qt5Config.cmake
/home/noah/anaconda3/lib/cmake/Qt5/Qt5Config.cmake
