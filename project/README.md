# Building PipeCNN for Arm

Here are some notes on buildig the project for an OpenCL project

## Requirements

Quartus 18.1
OpenCL for 18.1
Embedded Development Suite 18.1

## Makefile

The Makefile has some 'configuration' stuff related to the boards.
I've left most of that out, and I've used the 'default' option in the `board_env.xml`

## Board Environment

You must have a `board_env.xml` file, and then a `hardware` directory in the same place as that.
Within that `hardware` directory, you  need to have 1 or more quartus projects.

So, a part structure like this:

./
   board_env.xml
   /hardware
      de10_nano_sharedonly_hdii
      de10_nano_sharedonly
      ... etc


Here is my `board_env.xml`


```
<?xml version="1.0"?>
<board_env version="19.0" name="dhs">
  <hardware dir="hardware" default="de10_nano_sharedonly">

  </hardware>
  <platform name="arm32">
    <linkflags>-L%b/arm32/lib</linkflags>
    <linklibs>-lintel_soc32_mmd</linklibs>
    <utilbindir>%b/arm32/bin</utilbindir>
  </platform>
  <platform name="linux64">
    <linkflags>-L%b/arm32/lib -L%a/host/arm32/lib</linkflags>
    <linklibs>-lintel_soc32_mmd -lstdc++</linklibs>
    <utilbindir>%b/arm32/bin</utilbindir>
  </platform>
  <platform name="windows64">
    <linkflags>-L%b\arm32\lib</linkflags>
    <linklibs>-lintel_soc32_mmd</linklibs>
    <utilbindir>%b\arm32\bin</utilbindir>
  </platform>
</board_env>
```

I'm only cross-compiling, so only the `platform name="arm32"` is relevant. I did not change that stuff at all though.

### Environment Variables

You need to set the `AOCL_BOARD_PACKAGE_ROOT` variable to point to where your `board_env.xml` file is.

You also need a handful of other variables.
I have this all scripted, and it would be difficult to transfer that over, but here are the ones you need to set. The exact values may very based on your system

export TOOLCHAIN= ** this is root driectory of your toolchain ** 
For me, I do this


```
LINARO_VER=${LINARO_VER:='4.9'}

toolchain=$(find ~/linuxproj/toolchain -maxdepth 1 -type d -name "gcc*${LINARO_VER}*")

echo "Linaro version is ${LINARO_VER}"

export TOOLCHAIN=${toolchain}
echo "Toolchain set to ${TOOLCHAIN}"
```

export ARCH=arm
export CROSS_COMPILE=arm-linux-gnueabihf-
export BR2_TOOLCHAIN_EXTERNAL_PATH=${TOOLCHAIN}


The following can be set with the 'embedded_command_shell.sh' script

`source <root for embedded tools>/embedded_command_shell.sh`

```
export SOCEDS_DEST_ROOT="${_SOCEDS_ROOT}"
source "${_SOCEDS_ROOT}/env.sh"
```

Don't forget to setup PATH, and LD_LIBRARY_PATH for your Linaro toolchain..

### Other Environment variables

 
```
	export QUARTUS_VERSION=${QUARTUS_VERSION:-19.1}
	base=~/intelFPGA/${QUARTUS_VERSION} # Your path may vary
    export INTELFPGAOCLSDKROOT="${base}/hld"
    export ALTERAFPGAOCLSDKROOT=$INTELFPGAOCLSDKROOT
    export PATH=$PATH:${base}/hld/bin
    export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}${LD_LIBRARY_PATH:+:}$INTELFPGAOCLSDKROOT/host/linux64/lib
```

Please review these of course.

##

Build the library
You have to build some RTL libraries to proceed. This should be simple

	cd project/device/RTL
	make

Now, go to the `PipeCNN/project` directory, and try and make it

	make
	
God speed!!


