# OpenGL ES 2 rendering without an X server on Raspberry Pi using EGL

Have you ever wanted to render things with headless (no screen) Raspberry Pi? Then this is something you might be interested in! 

The following file `triangle.c` contains an example that renders a triangle and saves it into a `output.raw` file. The example uses EGL to create a pixel buffer as a surface. Normally, when using X server, you would create a window surface. The example also uses OpenGL ES 2 to setup a very simple shader to render a triangle.

The `triangle.c` contains comments which should explain to you all the necessary parts. Please note that this example will not teach you fundamentals of OpenGL! The purpose of this file is solely to demonstrate creating an OpenGL ES 2 context and getting the raw pixel output without using any virtual Linux framebuffers or physical screen (no X server).

**Please note: Upgrading or downloading different EGL library other than the one distributed via Raspberry Pi distros may break your EGL! Do NOT run `apt-get install libegl1-mesa-dev`!**

# What do I need?

You need a GCC compiler and EGL and GLES library. If you are using the newest Raspbian distro, all those are already located on the image, no extra `apt-get` needed.

# How to I try it?

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

At the same time, a new file should be created: `output.raw`. This file contains raw 800x600 RGB pixels. You can use Photoshop or any other software to import and view this file. You should be able to see the following purple triangle:

![Screenshot of a purple triangle](/../master/screenshot.png?raw=true "Screenshot of a purple triangle")

# Troubleshooting 

**I get "Error! The glViewport/glGetIntegerv are not working! EGL might be faulty!" What should I do?**

Your EGL might be faulty! Try reinstalling your Raspberry Pi OS.