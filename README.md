# WARNING WORK IN PROGRESS!!! INFORMATION MAY BE INCORRECT OR INCOMPLETE

#Building your own Linux distribution for embedded devices

In this series of exercises you will build a Linux distribution for Raspberry Pi. We will do this in three steps. We will begin by setting up our build system. We will then build the reference implementation called Poky and run it with an emulator (QEMU). After that we will customize Poky, effectively making it our own distribution. In the final step we will obtain the Raspberry Pi hardware support and build our own custom image. After this final step will have a bootable image. For example we could write it to a sd card and boot a Raspberry Pi with it. 

##Prerequisites
### Few recommendations
Make sure to have at least(!) 50 gb of hard disk space available.


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

Congratulations! You have just built the reference implamentation (Poky) for the first time.

##Step 2: Making our own layer and recipe
In the previous step we build the reference implementation without any changes. In this step we’re going to customize it by adding a recipe. In order to do this will will add a layer where the recipe will be placed. We will also create our own image that will contain the recipe. 

    yocto-layer create iot-tech-day
    Please enter the layer priority you'd like to use for the layer: [default: 6] 
    Would you like to have an example recipe created? (y/n) [default: n] y
    Please enter the name you'd like to use for your example recipe: [default: example] helloIotTech
    Would you like to have an example bbappend file created? (y/n) [default: n] n

    New layer created in meta-iot-tech-day.

    Don't forget to add it to your BBLAYERS (for details see meta-iot-tech-day\README).

You will notice that a folder meta-iot-tech-day has been added. 

TODO Add image of file structure here

TODO Inspect our new recipe: helloIotTech

The last thing we'll do is to add our custom image

     mkdir -p recipes-core/images
     cd recipes-core/images/
     touch qemu-iot-tech-image.bb
     
Using your favorit editor open the recipe:     
     
     nano qemu-iot-tech-image.bb

Enter the following information:

     require recipes-core/images/core-image-minimal.bb
     IMAGE_INSTALL += " helloIotTech"

As a final step we need to add the layer to our conf/bblayers.conf so that bitbake can find it. We’ll do this using

    bitbake-layers add-layer $HOME/yocto/poky/meta-iot-tech-day/

The last argument is the path to our created layer. I happen to have this in my home folder under yocto/poky. __Make sure that it reflects the path that you have.__

Now we can build our custom image by running

    bitbake qemu-iot-tech-image

When the build has completed you can again run it with QEMU

    runqemu qemux86

__NOTE__ Login as: __root__

You can now execute the helloworld program in the QEMU:

    helloworld
    
Should print "Hello IoT Tech Day!"

##Step 3: Building our distribution for Raspberry Pi

TODO Clone RPI layer
TODO Add image for RPI

TODO Handle RPI 1 and 2

TODO Build image
TODO Execute img


## More information
If you would like more information I can warmly recommend reading the following resources. Good luck!

[Yocto Project Documentation](https://www.yoctoproject.org/documentation "Yocto Project Documentation") 

[Example recipe: mtr](http://cgit.openembedded.org/meta-openembedded/tree/meta-networking/recipes-support/mtr/mtr_0.86.bb)


