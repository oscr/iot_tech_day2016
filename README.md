# WARNING WORK IN PROGRESS!!! INFORMATION MAY BE INCORRECT OR INCOMPLETE

#Building your own Linux distribution for embedded devices

In this series of exercises you will build a Linux distribution for Raspberry Pi. We will do this in three steps. We will begin by setting up our build system. We will then build the reference implementation called Poky and run it with an emulator (QEMU). After that we will customize Poky, effectively making it our own distribution. In the final step we will obtain the Raspberry Pi hardware support and build our own custom image. After this final step will have a bootable image. For example we could write it to a sd card and boot a Raspberry Pi with it. 

##Prerequisites
### Few recommendations
Make sure to have at least(!) 50 gb of hard disk space available.

TODO Explain this in more detail 
I will work from a folder called __yocto__ in my home folder. If you choose a different location please take care to change paths. 

###A supported distribution:
In order to use Yocto you need to install a Linux distribution that is supported. You can find the full list here: 

http://www.yoctoproject.org/docs/2.0/ref-manual/ref-manual.html#detailed-supported-distros

I will use Ubuntu 14.04 LTS as an example because I really like it.

###Install required packages
The following packages are required for Ubuntu. 

    sudo apt-get install gawk wget git-core diffstat unzip texinfo gcc-multilib \
    build-essential chrpath socat libsdl1.2-dev xterm

If you are using any other supported distribution you’ll find the required packages here:

http://www.yoctoproject.org/docs/2.0/ref-manual/ref-manual.html#required-packages-for-the-host-development-system


##Step 1: Building Poky and using QEMU
First we need to get the reference implementation Poky

    git clone -b jethro git://git.yoctoproject.org/poky
    cd poky

At the current time jethro is the newest release so we will use it. However things might change fast so keep an eye on the documentation for a newer release. After we’ve cloned Poky we need to set up our environment with

    source oe-init-build-env

We are now ready to build!

    bitbake core-image-minimal

Note that this step will take hours. While waiting take a look at [Yocto Project Quick Start documentation](http://www.yoctoproject.org/docs/2.0.1/yocto-project-qs/yocto-project-qs.html "Yocto Project Quick Start")

Now we are ready to run. 

    runqemu quemux86

__NOTE__ When login prompt. Enter: __root__ 

__NOTE__ If you're running on a system without display you may have to use: __runqemu qemux86 nographic__

Congratulations! You have just built the reference implamentation (Poky) for the first time.

##Step 2: Making our own layer and recipe
In the previous step we build the reference implementation without any changes. In this step we’re going to customize it by adding a recipe. In order to do this will will add a layer where the recipe will be placed. We will also create our own image that will contain the recipe. 

Make sure your current directory is poky folder.

    cd ..
    pwd
    /home/oscar/yocto/poky

We will now create our own layer

    yocto-layer create iot-tech-day
    Please enter the layer priority you'd like to use for the layer: [default: 6] 6
    Would you like to have an example recipe created? (y/n) [default: n] y
    Please enter the name you'd like to use for your example recipe: [default: example] helloIotTech
    Would you like to have an example bbappend file created? (y/n) [default: n] n

    New layer created in meta-iot-tech-day.

    Don't forget to add it to your BBLAYERS (for details see meta-iot-tech-day\README).

You will notice that a folder meta-iot-tech-day has been added. 

After adding the layer you should have folder with the following structure:

    tree meta-iot-tech-day/
    meta-iot-tech-day/
    ├── conf
    │   └── layer.conf
    ├── COPYING.MIT
    ├── README
    └── recipes-example
        └── example
            ├── helloIotTech-0.1
            │   ├── example.patch
            │   └── helloworld.c
            └── helloIotTech_0.1.bb

Note the generated recipe: helloIotTech_0.1.bb 

For fun lets edit the helloworld.c

    nano meta-iot-tech-day/recipes-example/example/helloIotTech-0.1/helloworld.c

If changed mine to print "Hello IoT Tech Day!" instead.

The last thing we'll do is to add our custom image

    mkdir -p meta-iot-tech-day/recipes-core/images

Using your favorit editor open the recipe:     
     
    nano meta-iot-tech-day/recipes-core/images/qemu-iot-tech-image.bb

Enter the following information:

    require recipes-core/images/core-image-minimal.bb
    IMAGE_INSTALL += " helloIotTech"

As a final step we need to add the layer to our conf/bblayers.conf so that bitbake can find it. We’ll do this using

    cd build
    bitbake-layers add-layer $HOME/yocto/poky/meta-iot-tech-day/

The last argument is the path to our created layer. I happen to have this in my home folder under yocto/poky. __Make sure that it reflects the path that you have.__

Now we can build our custom image by running

    bitbake qemu-iot-tech-image

When the build has completed you can again run it with QEMU

    runqemu qemux86
    
When startup is completed you will see the following:

    Poky (Yocto Project Reference Distro) 2.0.1 qemux86 /dev/ttyS0

    qemux86 login: root
    root@qemux86:~# helloworld
    Hello IoT Tech Day!


##Step 3: Building our distribution for Raspberry Pi


The first step to get our Linux distribution running on Raspberry Pi is to obtain a __Board Support Package__. We will find that in the Raspberry Pi layer: 

    cd .. 
    git clone -b jethro git://git.yoctoproject.org/meta-raspberrypi
    
If we check inside the meta-raspberrypi layer we will see that there are three available images.

    ls meta-raspberrypi/recipes-core/images/
    rpi-basic-image.bb  rpi-hwup-image.bb  rpi-test-image.bb

We will base our new image on the rpi-basic-image
    
    nano meta-iot-tech-day/recipes-core/images/rpi-iot-tech-image.bb
 
Make sure the file contains:        

    require /home/oscar/yocto/poky/meta-raspberrypi/recipes-core/images/rpi-basic-image.bb

    IMAGE_INSTALL += " helloIotTech"

__NOTE__ Make sure to replace the full path above!

    cd build
    nano conf/local.conf
    
If you have a Raspberry Pi enter the following at the top of the file:

    MACHINE="raspberrypi" 

Or if you have a Raspberry Pi 2:

    MACHINE="raspberrypi2

After that we want to make sure to add the Raspberry Pi layer

    bitbake-layers add-layer $HOME/meta-raspberrypi/
    
If we look at the 

    cat conf/bblayers.conf
    
You should see both the something like this 

    BBLAYERS ?= " \
    /home/oscar/yocto/poky/meta \
    /home/oscar/yocto/poky/meta-yocto \
    /home/oscar/yocto/poky/meta-yocto-bsp \
    /home/oscar/yocto/poky/meta-iot-tech-day \
    /home/oscar/yocto/poky/meta-raspberrypi \
    "

We are now ready to build our image:

    bitbake rpi-iot-tech-image

TODO Execute img

TODO Summary!

## More information
If you would like more information I can warmly recommend reading the following resources. Good luck!

[About meta-raspberrypi layer](http://git.yoctoproject.org/cgit/cgit.cgi/meta-raspberrypi/about/)

[Yocto Project Documentation](https://www.yoctoproject.org/documentation "Yocto Project Documentation") 

[Example recipe: mtr](http://cgit.openembedded.org/meta-openembedded/tree/meta-networking/recipes-support/mtr/mtr_0.86.bb)
