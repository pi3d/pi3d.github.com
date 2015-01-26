Using pi3d with Android devices
===============================


1.  Although building applications to run on Android devices may seem a
    really neat thing to do it is not plain sailing. The development process
    is significantly more time consuming and debugging is hard. pi3d has
    only just been ported to Android so there are likely to be lots of
    bugs and limitations. These are the prerequisites:

      1. linux. either on its own, dual boot or as a virtual machine. This
      is because..
      
      2. kivy and python-for-android have to run from linux (they do have
      a ready-made virtual machine image you can use on windows)
      
      2.a several dependencies such as cython, android sdk, ndk itemised on
      the python-for-android website (and included in the virtual machine
      image, also packaged in easy to use format as 'buildozer')
      
      3. android studio, not completely essential but very handy for running
      on virtual devices etc.
      
      4. to get pi3d to run you have to hack three of the standard files
      in the python-for-android standard download. This is because pi3d
      opted to use the python ctypes mechanism for running the OpenGL ES2
      libraries rather than requiring stuff to be compiled. And python-for-
      android had opted not to use ctypes at all.
      
      5. It probably only works on recent versions of android. When testing
      I have used android API-14 (aka Android 4.0 aka Ice-cream-sandwich)
      but only tested on a phone running KitKat.

2.  Because pi3d uses kivy to get touch input (and, potentially other
    android functionality) it has to fit in with kivy's event callback
    structure. This is like pygame, tk, qt etc etc and means that the
    standard 'simple' while DISPLAY.loop_running(): system used in most
    of the pi3d demos, has to be converted to a function that can be
    passed to the kivy clock. So there has to be a way of maintaining the
    state of the app, for instance with global variables or having the
    function as a method of an instance of a class.

3.  So if you're sure you want to continue, this is a list of the steps.
    I will not cover setting up linux, there is plenty of info
    on the internet. It takes a little while but generally isn't too
    difficult.

    3.1 Follow the instructions here
    https://github.com/kivy/python-for-android
    including the links to the documentation pages. Basically you need to
    install various applications and clone the python-for-android repository
    onto your computer. I had used the kivy suggestion, previously, to install
    buildozer and already had most of the required applications installed
    but I was missing ant. When you start to try to use it you will get
    error messages telling you which components are missing, if any.

    3.2 Add a recipe for pi3d. You need to create the file in this location
    ``whereveryouput/python-for-android/recipes/pi3d/recipe.sh`` with
    contents::

        #!/bin/bash

        VERSION_pi3d=${VERSION_pi3d:-stable}
        URL_pi3d=https://github.com/tipam/pi3d/archive/android.zip
        DEPS_pi3d=(pil kivy numpy android)
        MD5_pi3d=
        BUILD_pi3d=$BUILD_PATH/pi3d/$(get_directory $URL_pi3d)
        RECIPE_pi3d=$RECIPES_PATH/pi3d

        function prebuild_pi3d() {
          true
        }

        function shouldbuild_pi3d() {
          if [ -d "$SITEPACKAGES_PATH/pi3d" ]; then
            DO_BUILD=0
          fi
        }

        function build_pi3d() {

          cd $BUILD_pi3d

          push_arm

          export LDFLAGS="$LDFLAGS -L$LIBS_PATH"
          export LDSHARED="$LIBLINK"
            
          # fake try to be able to cythonize generated files
          $HOSTPYTHON setup.py build_ext
          try find . -iname '*.pyx' -exec $CYTHON {} \;
          try $HOSTPYTHON setup.py build_ext -v
          #try find build/lib.* -name "*.o" -exec $STRIP {} \;
          try $HOSTPYTHON setup.py install -O2

          try rm -rf $BUILD_PATH/python-install/lib/python*/site-packages/pi3d/tools

          unset LDSHARED
          pop_arm
        }

        function postbuild_pi3d() {
          true
        }

    At the moment the android-capable version of pi3d is in the android
    branch as specified above, but this will be merged into master as
    soon as it becomes stable.

    3.3 See posts here: https://github.com/kivy/python-for-android/issues/301
    To get python-for-android to be able to use ctypes you need to
    edit ``whereveritis/android/platform/android-ndk-r9c/platforms/android-14/arch-arm/usr/include/malloc.h``
    and change::

      //extern size_t malloc_usable_size(const void*); //this
      extern size_t malloc_usable_size(void*); //to this

    You also need to edit ``whereveryouput/python-for-android/distribute.sh``
    and comment out the lines::

      # try rm -rf ctypes
      ...
      # try rm -rf lib-dynload/_ctypes_test.so

    ``whereveryouput/python-for-android/recipes/python/patches/disable-modules.patch``
    and take ctypes out of the line::

      +disabled_module_list = ['spwd','bz2','ossaudiodev',...

    ``whereveryouput/python-for-android/recipes/python/recipe.sh`` and
    replace the lines that look like::

      try ./configure --host=arm-eabi OPT=$OFLAG --prefix="$BUILD_PATH/python-install" --enable-shared ...
      echo ./configure --host=arm-eabi  OPT=$OFLAG --prefix="$BUILD_PATH/python-install" --enable-shared ...

    with the
    following::

        export HOSTARCH=arm-eabi
        export BUILDARCH=x86_64-linux-gnu
        export CFLAGS="$CFLAGS -DNO_MALLINFO"
        try ./configure --host=$HOSTARCH --build=$BUILDARCH --prefix="$BUILD_PATH/python-install" --enable-shared ...
        echo ./configure --host=$HOSTARCH --build=$BUILDARCH --prefix="$BUILD_PATH/python-install" --enable-shared ...

    3.4 Now you should be able to generate the distribution framework of
    support files that will allow pi3d packages to be exported to android.
    This takes quite a while to run as it downloads and compiles a whole
    hierarchy of dependencies. On my bottom-of-the-range but 2014 laptop
    it takes 25 minutes. Once this has been done successfully it shouldn't
    need to be re-done until there are modifications in the support packages
    (that you want to include)::

      $ cd whereveryouput/python-for-android
      $ ./distribute.sh -m "pi3d"

    If you need to force it to recompile something (i.e. you alter something
    in the pi3d library or there was a typo in one of the mods above) then
    you need to put in an additional -f also if you have problems it might
    be useful to see the output, you can direct this to a file by adding
    ``> path/to/file.txt 2>&1`` on the end. Note that the question asking
    for you to press any key will stop the script indefinitely so do several
    'Enters' after starting it running.

    To make distribute.sh re-download the source code from the remote
    repository you seem to have to delete the files in
    ``whereveryouput/python-for-android/.packages/pi3d/``

    3.5 Set up test project before launching into anything too complicated:
    Create a directory in your home directory with an appropriate name
    create a subdirectory called ``textures`` and copy into it the image
    file from pi3d_demos (you can use whatever you want but change the
    program appropriately)::

      $ mkdir ~/pi3d_android
      $ mkdir ~/pi3d_android/pi3dtest/
      $ mkdir ~/pi3d_android/pi3dtest/
      $ cp ~/pi3d_demos/textures/PATRN.PNG ~/pi3d_android/pi3dtest/textures/

    Now create a minimal demo file in ~/pi3d_android/pi3dtest/ (in this example)
    called main.py and fill it with::

      #!/usr/bin/python
      from __future__ import absolute_import, division, print_function, unicode_literals

      import math
      import pi3d

      DISPLAY = pi3d.Display.create(depth=16) # NB need to set here, see build.py below
      tex = pi3d.Texture('textures/PATRN.PNG')
      ball = pi3d.Sphere(z=5.0)
      shader = pi3d.Shader('uv_light')
      ball.set_draw_details(shader, [tex])
      t = 0.0
      rotate = False

      def pi3dloop(dt):
        global DISPLAY, ball, t, rotate
        DISPLAY.loop_running()
        ball.draw()
        t += 0.01
        ball.positionY(math.sin(t * 03.13))
        ball.positionZ(4.0 + math.sin(t * 0.53))
        if rotate:
          ball.rotateIncY(0.5)
        if DISPLAY.android.screen.moved:
          ball.translateX(DISPLAY.android.screen.touch.dx * 0.01)
          DISPLAY.android.screen.moved = False
        if DISPLAY.android.screen.tapped:
          rotate = not rotate
          DISPLAY.android.screen.tapped = False

      DISPLAY.android.set_loop(pi3dloop)
      DISPLAY.android.run()

      DISPLAY.destroy()
      
    3.6 Running ./distribute.sh (in step 3.4) should have generated a
    default distribution in ``whereveryouput/python-for-android/dist/default/``
    this needs to have a ``project.properties`` file with a minimum single
    line ``target=android-14`` (as per default version created in the directory.
    I have to recreate this file each time I run distribute.sh) From this
    directory run::

      $ ./build.py --dir ~/pi3d_android/pi3dtest --package org.demo.pi3dtest --name "pi3dtest" --meta-data surface.depth=16 --version 1.0.0 debug

    Which shouldn't take too long and will put the android apk package
    file into the ``bin`` subdirectory. I found that some phones needed to
    have the bits set for the depth buffer so if the app needs to be deployed
    generally you need to include this in the build meta-data. You will need
    to set the same value when you run Display.create(), see above.

    The quickest way to run this on my computer is to download it to a phone
    (you will need to enable PC connection and Security/Unknown sources from
    settings. Quite a hack on some phone but google details). However if
    it doesn't work you will have no information as to why. For proper
    development and debugging you will need to install an emulator.

    3.7 I used google's Android Studio which is big and powerful, but lots
    of other people have used it and had the same problems you will so it's
    possible to find answers relatively easily and most components can be
    run independently from the command line. My steps were

      3.7.1 Install Android Studio then actually set up a new project -
      even though it's not really going to be used. You won't get the
      window with menu options otherwise.

      3.7.1 Set up a device to emulate. Tools->android->AVD_manager->new You
      will need to select a system image to download and use. I tried 14
      (icecream) but it didn't run so did 19 (kitkat) which seemed to run
      (and the phone was running kitkat). You will need to Show Advanced
      Settings to see what the device id is (probably just replaced spaces
      with underlines) if you are going to run the emulator from command line.

      3.7.2 Enable logcat. Run->Edit_Configurations->Android->app->Logcat
      uncheck the option to Filter only for this application.

      3.7.3 If you're using linux you can now close Android Studio which
      will free a lot of resources. You can start the emulator::

        $ cd whereveritis/Android/Sdk/tools
        $ ./emulator -avd Nexus_One_API_19

      To start logcat
      (from a different terminal)::

        $ cd whereveritis/Android/Sdk/platform-tools
        $ ./adb logcat

      To install new versions of your android
      package onto the emulator::

        $ .adb install -r whereveryouput/python-for-android/dist/default/bin/pi3dtest-0.0.1-debug.apk

4.  For info on changing the icon or loading image, building to enable
    other features, publishing on play.google; read the docs on the
    python-for-android website.

5.  At the moment I am aware of the following deficiencies or incompatible
    modules when running pi3d on android::

      Mouse 
      Keyboard
      Event
      Font (However Pngfont *does* work and pi3d_demos/fonts/Arial.png has been tidied)

.. _ReadMe: http://pi3d.github.com/html/index.html

