# Raspberry Pi OpenGL ES 2 without an X server (using EGL)

Have you ever wanted to render things with headless (no screen) Raspberry Pi? Then this is something you might be interested in! 

The following file `triangle.c` contains an example that renders a triangle and saves it into a `output.raw` file. The example uses EGL to create a pixel buffer as a surface. Normally, when using X server, you would create a window surface. The example also uses OpenGL ES 2 to setup a very simple shader to render a triangle.

The `triangle.c` contains comments which should explain to you all the necessary parts. Please note that this example will not teach you fundamentals of OpenGL! The purpose of this file is solely to demonstrate creating an OpenGL ES 2 context and getting the raw pixel output without using any virtual Linux framebuffers or physical screen (no X server).

## Which Raspberry Pi works?

* Raspberry Pi 1 (tested and works)
* Raspberry Pi 2 (tested and works)
* Raspberry Pi 3 (tested and works)
* Raspberry Pi 4 (tested and works, but won't work in headless mode, you need a HDMI output)

## Raspberry Pi 1,2,3

*For the Raspberry Pi 4 instructions see the next section*

**What do I need?**

You need a GCC compiler and EGL and GLES library. If you are using the latest Raspbian distro, all those are already located on the image, no extra `apt-get` needed. You can check if you have the gcc installed by executing `gcc --version`. You can also check if GLES and EGL are installed by executing `ls /opt/vc/lib` which should list `libbrcmEGL.so` and `libbrcmGLESv2.so`. They are all already included in the Jessie/Stretch/Buster Raspbian! If you don't have `libbrcmEGL.so`, then you might have `libEGL.so` in the same folder.

**The problem of mesa apt package**

The following packages `mesa-common-dev` and `mesa-utils` **do NOT work** and instead they may break EGL on your operating system. The libraries installed through any of the `mesa` packages will install incompatible version of the EGL, most likely into `/lib/arm-linux-gnueabihf` folder. Don't use these! Use the ones provided by the official Raspbian OS image in the `/opt/vc/lib` folder!

**How to I try it?**

Copy or download the `triangle.c` file onto your Raspberry Pi. Using any terminal, write the following commands to compile the source file:

```
gcc -c triangle.c -o triangle.o -I/opt/vc/include
gcc -o triangle triangle.o -lbrcmEGL -lbrcmGLESv2 -L/opt/vc/lib
```

To run the executable, type the following:

```
./triangle
```

You should see the following output:

```
Initialized EGL version: 1.4
GL Viewport size: 800x600
```

At the same time, a new file should be created: `output.raw`. This file contains raw 800x600 RGB pixels. You can use Photoshop or any other software to import and view this file. You should be able to see the following purple triangle. Please note that the image is mirrored vertically as the pixel coordinates in OpenGL start from the bottom, not from the top. Example of the image:

![Screenshot of a purple triangle](output.png "Screenshot of a purple triangle")

## Raspberry Pi 4

Raspberry Pi 4, at the moment of writing this, has no full KMS driver, because the GPU is different from the previous ones. Instead of using the `vc` libraries, you will need to use the DRM/GBM.

**What do I need?**

You need a GCC compiler, GDM, EGL, and GLES library. The GCC compiler is already included in the Raspbian image. To install the other libraries, simply run:

```bash
sudo apt-get install libegl1-mesa-dev libgbm-dev libgles2-mesa-dev
```

You will also need to connect your Raspberry Pi to a screen. The boot config from `/boot/config.txt` that I have used for my tests, if it helps in any way:

```bash
dtoverlay=vc4-fkms-v3d
max_framebuffers=2
hdmi_force_hotplug=1
hdmi_group=2
hdmi_mode=81
```

**How to I try it?**

Copy or download the `triangle_rpi4.c` file onto your Raspberry Pi. Using any terminal, write the following commands to compile the source file:

```
gcc -o triangle_rpi4 triangle_rpi4.c -ldrm -lgbm -lEGL -lGLESv2 -I/usr/include/libdrm -I/usr/include/GLES2
```

To run the executable, type the following:

```
./triangle_rpi4
```

You should see the following output:

```
resolution: 1366x768
Initialized EGL version: 1.4
GL Viewport size: 1366x768
```

Note! The resolution depends on the screen you are using!

At the same time, a new file should be created: `output.raw`. This file contains raw 1366x768 RGB pixels. You can use Photoshop or any other software to import and view this file. You should be able to see the following purple triangle. Please note that the image is mirrored vertically as the pixel coordinates in OpenGL start from the bottom, not from the top. Example of the image:

![Screenshot of a purple triangle](output.png "Screenshot of a purple triangle")


## Troubleshooting and Questions

**Failed to get EGL version! Error:**

Your EGL might be faulty! Make sure you are using the libraries provided by Raspbian and not the ones installed through apt-get or other package managers. Use the ones in the `/opt/vc/lib` folder. If that does not work, try reinstalling your Raspberry Pi OS.

**I get "Error! The glViewport/glGetIntegerv are not working! EGL might be faulty!" What should I do?**

Same as above.

**How do I change the pixelbuffer resolution?**

Find `pbufferAttribs` and change `EGL_WIDTH` and `EGL_HEIGHT`.

**I get "undefined reference" on some gl functions!**

Some GL functions may not come from GLES library. You may need to get GL1 library by executing `sudo apt-get install libgl1-mesa-dev` and then simply add: `-lGL` flag to the linker, so you get: `gcc -o triangle triangle.o -lbrcmEGL -lbrcmGLESv2 -lGL -L/opt/vc/lib`

**How do I change the buffer sample size or the depth buffer size or the stencil size?**

Find `configAttribs` at the top of the `triangle.c` file and modify the attributes you need. All config attributes are located here <https://www.khronos.org/registry/EGL/sdk/docs/man/html/eglChooseConfig.xhtml>

**Can I use glBegin() and glEnd()?**

You should not, but you can. However, it seems that OpenGL ES does not like that and nothing will be rendered. For this you will need to link the `GL` library as well. Simply, add `-lGL` to the gcc command.

**Failed to add service - already in use?**

As mentioned in [this issue](https://github.com/matusnovak/rpi-opengl-without-x/issues/1), you have to remove both `vc4-kms-v3d` and `vc4-fkms-v3d` from [R-Pi config](https://elinux.org/R-Pi_configuration_file). Also relevant discussion here: <https://stackoverflow.com/questions/40490113/eglfs-on-raspberry2-failed-to-add-service-already-in-use>. If you are using Raspberry Pi 4 (the `triangle_rpi4.c`) it might happen sometimes. I could not fully figure it out, but it might be due to incorrect EGL or DRM/GBM cleanup in  the code. I say "incorrect", but I don't know where. 

## License

Do whatever you want.

```
This is free and unencumbered software released into the public domain.

Anyone is free to copy, modify, publish, use, compile, sell, or
distribute this software, either in source code form or as a compiled
binary, for any purpose, commercial or non-commercial, and by any
means.

In jurisdictions that recognize copyright laws, the author or authors
of this software dedicate any and all copyright interest in the
software to the public domain. We make this dedication for the benefit
of the public at large and to the detriment of our heirs and
successors. We intend this dedication to be an overt act of
relinquishment in perpetuity of all present and future rights to this
software under copyright law.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS BE LIABLE FOR ANY CLAIM, DAMAGES OR
OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE,
ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR
OTHER DEALINGS IN THE SOFTWARE.

For more information, please refer to <http://unlicense.org/>
```
