# Platforms tested

* Olimex STM32-e407

# Preparing environment

* Compile toolchain:

```bash
# Step 1

_ crossdev --lenv 'USE="nano -nls -threads -unicode"' -s3 -t arm-none-eabi

# Step 2

_ crossdev --lenv 'USE="nano -nls -threads -unicode"' --genv 'USE="cxx -nls -nptl -pch -pie -ssp" EXTRA_ECONF="--with-multilib-list=rmprofile --disable-decimal-float --disable-libffi --disable-libgomp --disable-libmudflap --disable-libquadmath --disable-shared --disable-threads --disable-tls"' -s4 -t arm-none-eabi

# Step 3

_ emerge cross-arm-none-eabi/newlib

# Step 4: to get gdb

_ crossdev --lenv 'USE="nano -nls -threads -unicode"' --genv 'USE="cxx -nls -nptl -pch -pie -ssp" EXTRA_ECONF="--with-multilib-list=rmprofile --disable-decimal-float --disable-libffi --disable-libgomp --disable-libmudflap --disable-libquadmath --disable-shared --disable-threads --disable-tls"' -s4 --ex-gdb -t arm-none-eabi
```

Remove `-cxx` from gcc in `/etc/portage/package.use/cross-arm-none-eabi`.
Reemerge gcc.

```bash
_ emerge -va cross-arm-none-eabi/gcc
```

* Install application for flashing.

```bash
_ emerge -va openocd
```

* Install application to connect using serial

```bash
_ emerge -va minicom picocom
```

# Preparing Nuttx

* Compile and install kconfig.

```bash
git clone https://bitbucket.org/nuttx/tools.git
cd tools/kconfig-frontends
./configure --prefix=$PWD/install
```

In configuration steps of nuttx exports the location of kconfig.

```bash
export PATH=$PWD/install/bin:$PATH
export LD_LIBRARY_PATH=$PWD/install/lib:$LD_LIBRARY_PATH
```

* Install nuttx.

```bash
git clone https://bitbucket.org/nuttx/NuttX.git nuttx
git clone https://bitbucket.org/nuttx/apps
```

* Configure compilation of NSH

```bash
cd nuttx
tools/configure.sh configs/olimex-stm32-e407/usbnsh
```

* Build nuttx

```bash
make
```

* Flash nuttx

```bash
openocd -f interface/ftdi/olimex-arm-usb-tiny-h.cfg -f target/stm32f4x.cfg -c init -c "reset halt" -c "flash write_image erase nuttx.bin 0x08000000"
```

# Debugging

In order to generate Debug symbols select `Build Setup ---> Debug Options ---> Generate Debug Symbols`.

```bash
arm-none-eabi-gdb

(gdb) target remote localhost:3333
(gdb) file <nuttx_folder>/nuttx
```

# Connection

```bash
picocom /dev/ttyACM0 -b 115200 -l
```
