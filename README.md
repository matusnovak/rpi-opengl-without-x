# Raspberry Pi OpenGL ES 2 without an X server (using EGL)

Have you ever wanted to render things with headless (no screen) Raspberry Pi? Then this is something you might be interested in! 

The following file `triangle.c` contains an example that renders a triangle and saves it into a `output.raw` file. The example uses EGL to create a pixel buffer as a surface. Normally, when using X server, you would create a window surface. The example also uses OpenGL ES 2 to setup a very simple shader to render a triangle.

The `triangle.c` contains comments which should explain to you all the necessary parts. Please note that this example will not teach you fundamentals of OpenGL! The purpose of this file is solely to demonstrate creating an OpenGL ES 2 context and getting the raw pixel output without using any virtual Linux framebuffers or physical screen (no X server).

## Which Raspberry Pi works?

* Raspberry Pi 1 (tested and works)
* Raspberry Pi 2 (tested and works)
* Raspberry Pi 3 (tested and works)
* Raspberry Pi 4 (todo)

## What do I need?

You need a GCC compiler and EGL and GLES library. If you are using the latest Raspbian distro, all those are already located on the image, no extra `apt-get` needed. You can check if you have the gcc installed by executing `gcc --version`. You can also check if GLES and EGL are installed by executing `ls /opt/vc/lib` which should list `libEGL.so` and `libGLESv2.so`. They are all already included in the Jessie/Stretch/Buster Raspbian!

## Lite (no Desktop) Raspbian:

I was unsuccessful to install the EGL and GLES libraries on the Lite version. All my attempts resulted in a broken EGL library. The following packages `mesa-common-dev` and `libgl1-mesa-dev` **do NOT work** and instead they have broken my system. The only thing I can recommend is to install the full desktop version of Raspbian, then disable the X-server (see this stackoverflow post [here](https://raspberrypi.stackexchange.com/questions/1318/boot-without-starting-x-server)).

## How to I try it?

Copy or download the `triangle.c` file onto your Raspberry Pi. Using any terminal, write the following commands to compile the source file:

```
gcc -c triangle.c -o triangle.o -I/opt/vc/include
gcc -o triangle triangle.o -lEGL -lGLESv2 -L/opt/vc/lib
```

To run the file, type the following:

```
./triangle
```

You should see the following output:

```
Initialized EGL version: 1.4
GL Viewport size: 800x600
```

At the same time, a new file should be created: `output.raw`. This file contains raw 800x600 RGB pixels. You can use Photoshop or any other software to import and view this file. You should be able to see the following purple triangle. Please note that the image is mirrored vertically as the pixel coordinates in OpenGL start from the bottom, not from the top.

![Screenshot of a purple triangle](/../master/output.png?raw=true "Screenshot of a purple triangle")

## Troubleshooting and Questions

**Failed to get EGL version! Error:**

Your EGL might be faulty! Try reinstalling your Raspberry Pi OS.

**I get "Error! The glViewport/glGetIntegerv are not working! EGL might be faulty!" What should I do?**

Your EGL might be faulty! Try reinstalling your Raspberry Pi OS.

**How do I change the pixelbuffer resolution?**

Find `pbufferAttribs` and change `EGL_WIDTH` and `EGL_HEIGHT`.

**I get "undefined reference" on some gl functions!**

Some GL functions may not come from GLES library. You may need to get GL1 library by executing `sudo apt-get install libgl1-mesa-dev` and then simply add: `-lGL` flag to the linker, so you get: `gcc -o triangle triangle.o -lEGL -lGLESv2 -lGL -L/opt/vc/lib`

**How do I change the buffer sample size or the depth buffer size or the stencil size?**

Find `configAttribs` at the top of the `triangle.c` file and modify the attributes you need. All config attributes are located here <https://www.khronos.org/registry/EGL/sdk/docs/man/html/eglChooseConfig.xhtml>

**Can I use glBegin() and glEnd()?**

You should not, but you can. However, it seems that OpenGL ES does not like that and nothing will be rendered.

**Failed to add service - already in use?**

As mentioned in [this issue](https://github.com/matusnovak/rpi-opengl-without-x/issues/1), you have to remove both `vc4-kms-v3d` and `vc4-fkms-v3d` from [R-Pi config](https://elinux.org/R-Pi_configuration_file). Also relevant discussion here: <https://stackoverflow.com/questions/40490113/eglfs-on-raspberry2-failed-to-add-service-already-in-use>

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
