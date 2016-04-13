# WARNING WORK IN PROGRESS!!! INFORMATION MAY BE INCORRECT OR INCOMPLETE

#Building your own Linux distribution for embedded devices

In this series of exercises you will build a Linux distribution for Raspberry Pi. We will begin by setting up our build machine environment. After that we will build the reference implementation called Poky and run it with QEMU (en emulator). In the next step we will customize Poky by adding a layer containing a helloworld example recipe and image. In the final step we will obtain Raspberry Pi hardware support and build an image. This image could for example be written to an SD card and used to boot a Raspberry Pi.


##Prerequisites
### Advice
Make sure to have at least(!) 50 gb of hard disk space available.

In my examples I work from a folder called yocto in my home folder. Therefore in the examples my path will start with 

    /home/oscar/yocto

Please make sure to change the path where required.

###A supported distribution:
In order to use Yocto you need to install a Linux distribution that is supported. You can find the full list here: 

http://www.yoctoproject.org/docs/2.0.1/ref-manual/ref-manual.html#detailed-supported-distros

In my examples I will use Ubuntu 14.04 LTS.

###Install required packages
The following packages are required for Ubuntu. 

    sudo apt-get install gawk wget git-core diffstat unzip texinfo gcc-multilib \
    build-essential chrpath socat libsdl1.2-dev xterm

If you are using any other supported distribution you’ll find the required packages here:

http://www.yoctoproject.org/docs/2.0.1/ref-manual/ref-manual.html#required-packages-for-the-host-development-system


##Step 1: Building Poky and using QEMU
First we need to get the reference implementation Poky

    git clone -b jethro git://git.yoctoproject.org/poky
    cd poky

__Note__ Currently Jethro is the newest release. However things might change fast so keep an eye out for newer releases. 

After we've cloned Poky we need to set up our environment.

    source oe-init-build-env

We are now ready to build!

    bitbake core-image-minimal

Note that this step will take hours. While waiting take a look at [Yocto Project Quick Start documentation](http://www.yoctoproject.org/docs/2.0.1/yocto-project-qs/yocto-project-qs.html "Yocto Project Quick Start")

Now we are ready to run. 

    runqemu qemux86

__Note__ When login prompt. Enter: __root__ 

__Note__ If you're running on a system without display you may have to use: __runqemu qemux86 nographic__

##Step 2: Making our own layer and recipe
In the previous step we built Poky without any changes. But in this step we're going to customize it by adding a layer which will contain our custom image and a recipe for a helloworld application.

Make sure that you're in the `poky` folder (for me that's `/home/oscar/yocto/poky`)

We will now create our own layer which we will call `iot-tech-day` and at the same time generate a recipe which we will call `helloIotTech`. Rember that if you open a new terminal you need to `source oe-init-build-env` again.

    yocto-layer create iot-tech-day
    Please enter the layer priority you'd like to use for the layer: [default: 6] 6
    Would you like to have an example recipe created? (y/n) [default: n] y
    Please enter the name you'd like to use for your example recipe: [default: example] helloIotTech
    Would you like to have an example bbappend file created? (y/n) [default: n] n

    New layer created in meta-iot-tech-day.

    Don't forget to add it to your BBLAYERS (for details see meta-iot-tech-day\README).

You will notice that a folder `meta-iot-tech-day` has been created. The `meta-` is added as a naming prefix by convention. So the name of our layer is `meta-iot-tech-day`.

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

You will now have a generated recipe `helloIotTech_0.1.bb` with the source code: `helloworld.c` and `example.patch`

For fun lets edit `helloworld.c`

    nano meta-iot-tech-day/recipes-example/example/helloIotTech-0.1/helloworld.c

I change my example to print "Hello IoT Tech Day!" instead.

We also need an image that we will add our recipe to

    mkdir -p meta-iot-tech-day/recipes-core/images

Using nano (or your favorit editor) create the following image recipe     
     
    nano meta-iot-tech-day/recipes-core/images/qemu-iot-tech-image.bb

Enter the following information:

    require recipes-core/images/core-image-minimal.bb
    
    IMAGE_INSTALL += " helloIotTech"

__Note__ It's important to add a space in front of `helloIotTech`!

Go back to the `build` directory. For me that's `/home/oscar/yocto/poky/build`

    cd build
    
Now as a final step we need to add our new layer `meta-iot-tech-day` to our configuration file `conf/bblayers.conf`.
    
    bitbake-layers add-layer $HOME/yocto/poky/meta-iot-tech-day/

The last argument is the path to our created layer. __Make sure that it reflects the path that you have.__

Now we can build our custom image by running:

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

We can confirm it works by typing the following:

    bitbake-layers show-layers
    layer                 path                                      priority
    ==========================================================================
    meta                  /home/oscar/yocto/poky/meta               5
    meta-yocto            /home/oscar/yocto/poky/meta-yocto         5
    meta-yocto-bsp        /home/oscar/yocto/poky/meta-yocto-bsp     5
    meta-iot-tech-day     /home/oscar/yocto/poky/meta-iot-tech-day  6
    meta-raspberrypi      /home/oscar/yocto/poky/meta-raspberrypi   9

We are now ready to build our image:

    bitbake rpi-iot-tech-image

TODO Execute img

TODO Summary!

## More information
If you would like more information I can warmly recommend reading the following resources. Good luck!

[About meta-raspberrypi layer](http://git.yoctoproject.org/cgit/cgit.cgi/meta-raspberrypi/about/)

[Yocto Project Documentation](https://www.yoctoproject.org/documentation "Yocto Project Documentation") 

[Example recipe: mtr](http://cgit.openembedded.org/meta-openembedded/tree/meta-networking/recipes-support/mtr/mtr_0.86.bb)
