Frequently Asked Questions
==========================


#.  When starting any demo I get an AssertionError in DisplayOpenGL
    ``assert s >= 0``

      This is generally caused by the graphics memory allocation on the
      Raspberry Pi being too low (less than 32)

#.  When starting any demo I get an AssertionError in DisplayOpenGL
    ``assert self.surface != EGL_NO_SURFACE``

      This is generally caused by the graphics memory allocation on the
      Raspberry Pi being too low (less than 64)

#.  When running ConferenceHall (or other program with a large number of
    textures) it appears to start ok then shuts down (byebye 3 message) just
    before the main display loop should appear.

      This is generally caused by the graphics memory allocation on the
      Raspberry Pi being too low (less than 128)

#.  When starting TigerTank, ConferenceHall or MarsStation demos I get an
    error ``_tkinter.TclError: couldn't connect to display ":0"``

      tkinter is trying to use a display provided by the X-server. Run
      startx from the command line.

#.  I am trying to install using PiStore but it's been running for hours
    with no sign of completing.

      The PiStore install adds `pip` and `Pillow` which take quite a bit
      of the cpu resources. It could be that you have set the graphics memory share
      to the higher level needed to run pi3d. In the long run it will be quicker
      to abort the installation, remove the half installed pi3d, use
      raspi-config to set the graphics memory low, re-install pi3d, then
      set the graphics memory back up again!

#.  I have installed using PiStore and run the demos from the Menu. Now
    I would like to play around for myself.

      There is a PiStore button next to `launch` that lets you see the source
      code. It should open a file browser window to
      `/usr/local/indiecity/InstalledApps/skillmanmedia/Full/pi3d_demos`
      which is a very obscure location and also protected against modification.
      To play around with the code you should either copy the whole
      of this directory to your home directory (i.e. so you have
      `/home/pi/pi3d_demos/`) or clone or download the demos from
      http://github.com/pi3d/pi3d_demos (which will include a couple of
      other demos excluded from PiStore because they use very large resource
      files.)

      Before going too far it would be a good idea to `ReadMe`_

#.  When I try and run the demos I just get a load of error messages such as
    ``libEGL warning: GLX/DRI2 is not supported/failed to authenticate etc``

      The chances are this is because 'something' (such as gedit) has installed
      mesa which added its own versions of libEGL and libGLESv2. If
      you run::

        $ sudo find / -name libEGL*
        $ sudo find / -name libGLESv2*

      you should just get /opt/vc/lib/libEGL.so and /opt/vc/lib/libGLESv2.so
      if other ones turn up i.e. /usr/lib/arm-linux-gnueabihf/libEGL.so.1
      you could try creating symbolic links for them all like this::

        $ sudo ln -fs /opt/vc/lib/libEGL.so /usr/lib/arm-linux-gnueabihf/libEGL.so
        $ sudo ln -fs /opt/vc/lib/libEGL.so /usr/lib/arm-linux-gnueabihf/libEGL.so.1
        $ sudo ln -fs /opt/vc/lib/libGLESv2.so /usr/lib/arm-linux-gnueabihf/libGLESv2.so
        $ sudo ln -fs /opt/vc/lib/libGLESv2.so /usr/lib/arm-linux-gnueabihf/libGLESv2.so.2

      Using the actual paths as listed by find. Or just delete them as I did
      **NB DON'T DELETE THE ONES IN /opt/vc/lib** After creating symbolic links
      or deleting the new files you will need to run::

        $ sudo ldconfig

      This issue is being looked into by the maintainers of Raspbian so,
      hopefully, it will be fixed in later releases.

#.  My RPi Crashes or reboots when I try and run a demo.

      Any program using a loadable texture, which includes nearly all the demos,
      requires the Python Imaging module (PIL). Additionally, some demos require tk.

      See the ReadMe_ for information on how to install these packages.

#.  I see nothing but a white screen.

      Possibly something has gone wrong with an opengles function, sometimes
      there might be an error message in the terminal such as failed to create
      display. This could be caused by running out of gpu memory (see ReadMe_
      for how to set up memory split).

      Occasionally multi-threaded applications can cause this problem if an
      opengles function call is made not from the main thread.  If you encounter
      this, please contact the pi3d team so we can protect against this in
      future.

#.  I see nothing but a black screen.

      Possibly something has gone wrong in a shader, such as using a shader
      requiring texture coords (i.e. mat_relfect) on a Model exported with
      no uv mapping.

      There may be a line number reference output by the shader compiler in the
      terminal window.  It is great fun experimenting with shaders but they are
      *?£#%* taciturn beasts to debug! The problem could be caused by sending
      some bad render setting to a shader.

#.  I see nothing but the background.

      You will need to set background to non-transparent and a color not equal
      to black or white to determine if this is happening.

      Either the shape is behind the camera, too far away, is outside the field
      of view, is too small, too large or the polygons are facing away from the
      camera. Often this is because you are actually **inside** the object.

      Try using the Camera.point_at([x,y,z]) function (see demos/ClashWalk.py
      for use) or move and rotate the object and camera. Sprite and ImageSprite
      shapes are one sided so cannot be seen from behind, try using a Plane
      instead

#.  I see only black silhouettes against the background.

      You may be trying to use a shader that requires light but there isn't
      any, or it's turned down too low. Try switching to a 'flat' shader
      to check.

      Alternatively, if it's a shape you have generated such as
      a Lathe or a Model, the normal vectors might be pointing in the wrong
      direction. Try re-generating the shape, the path you use for the Lathe
      needs to start at the top of the object and there are functions in
      most 3D modeling applications to recalculate normals, or force them
      to point outwards.

#.  The demo loads but the mouse doesn't move the camera as it's supposed to.

      If this only happens on demos using the ``event`` library (such as Silo.py)
      then it could be the hardware configuration is pretending to be something
      it isn't. It's not uncommon for keyboards to say they are mice or
      joysticks.

      If you have a mouse combined with a keyboard (to save on USB slots) then
      you might need to use ``get_mouse_movements(1)``. If you have problems
      with a device or inputs using the event system it's a good idea to run
      ``python FindDevices.py`` from ``pi3d/event/`` - this will give you lots
      of additional information.

      There is also an application ``demos/TestEvents.py`` that you can run to
      find what information is being returned by your input devices. In some
      circumstances you might need to modify the values returned by the
      ``pi3d/event/Event.py InputEvents`` methods. TODO at the moment this
      involves hacking the file but it will use a lookup table.

      When running on my laptop (lenovo T420, ubuntu 13.10), occasionally, the
      mouse doesn't work with the ``event`` input, but starts to do after
      running ``demos/TestEvents.py`` and changing the number in
      ``get_mouse_movements()`` a few times. It's not clear what causes this
      but it might be when the USB mouse is plugged in after the computer
      has been booted up.

#.  It appears from the demos that there are some arguments that are optional.
    For example, can a Shape be drawn without specifying a shader and a texture?

      There are (almost too) many ways to set Shapes up to draw. The draw method
      needs to have a **Shader**, a **Light** and a **Camera** specified but if
      you neglect to create a Light and Camera when you first draw a Shape it
      will generate 'default instances' which most of the time are just what you
      want. (These default instances can be accessed to change settings such as
      color or direction for a Light or field of view for a Camera by using the
      syntax: ``Camera.instance()``.

      You do, however need to explicitly
      create the Shader so it does the kind of rendering you want, but you
      can feed that in by various means, many of which also cater for specifying
      the Texture(s) to use as well:

        Set them directly in the Buffer array - the other methods are
        really just wrappers for this i.e.::

          myshape.buf[0].shader = myshader
          myshape.buf[0].textures = [mytexture]

        Include them
        at draw time::

          myshape.draw(myshader, [mytexture])

        Set them beforehand
        (probably the most usual way)::

          myshape.set_draw_details(myshader, [mytexture])

        For Model objects the ambient texture or material shade will normally
        be defined in the 3D object file (egg or obj/mtl) In these cases
        you could use::

          myshape.set_shader(myshader)
          ...
          myshape.set_normal_shine(normtex, ntiles..) # leaves the first texture if there
          ...
          myshape.set_material(mtrl)

#.  How can I blend objects, why do objects vanish when they go behind a transparent
    object and other questions to do with transparency (or apha property)

      Transparency of Shapes can be altered by 1. the set_alpha() method 2. the
      alpha value of pixels in a png type image file 3. alpha value of the fog.
      The blending of the pixels with alpha less than 1.0 is controlled by setting
      Texture.blend to True or False.

      The way that transparency is handled is quite hard to understand. Here is
      some good information http://www.opengl.org/wiki/Transparency_Sorting

      The graphics processor has a global setting to enable blending that is
      switched on or off as each Shape is drawn, allowing or preventing the pixels
      to be blended with whatever's behind them. In pi3d this can be controlled by
      setting the ``blend=True`` argument when the Texture is created or at a later
      point by ``mytexture.blend = True`` In addition to this setting there is a check
      in the draw() method so that blend is enabled when alpha is set to less than 1.0.

      When the gpu is rendering an object there is a depth buffer that holds
      information on how far from the camera each pixel has been drawn. Because
      of this it is normally optimal to draw foreground objects first as there
      is then less of the background to fill in. If the background was drawn
      first then the same pixel might have to be redrawn several times as the
      gpu found something else nearer to the view point. However the gpu
      **doesn't** take into account the transparency of the pixel when it's
      deciding if something is nearer or further away, so for blending
      you have to draw things on top of other things...

      Which sounds obvious but to give an example; if a slideshow tries to blend
      between two images, one drawn in front of the other:

      If you **first** draw the canvasFront (z=0.1) with alpha=0.1
      **then** draw the canvasBack (z=0.2) with alpha=0.9 the result will
      be a very faint image on canvasFront and nothing on canvasBack. Wrong!

      i.e. canvasBack always has to be drawn first and if the application is purely
      fading from one image to another it can leave canvasBack at apha=1.0 (i.e.
      default value) and just increase then decrease the alpha of canvasFront

      In addition to blending, when the Shader is rendering an object it discards
      some pixels without drawing anything at all. The decision is based on the
      alpha value of the pixel as read from the Texture. If blend is True then
      pixels with alpha < 0.05 are discarded if blend is False then pixels with
      alpha < 0.6 are discarded. This allows objects to be drawn after nearer objects
      but still be seen through 'holes' in the image. i.e. the trees in ForestWalk


#.  How do I use a joystick, gamepad, xbox controller etc with a pi3d
    application?

      Often these will just work with the event module when plugged into the USB,
      sometimes you may need to use a different InputEvents method, for instance
      with an xbox 360 you get the left joystick from ``get_joystickB3d()``
      Also you would need to install the driver and start it running first::

        sudo apt-get install xboxdrv
        sudo xboxdrv -s -i 0

#.  How do I make my own 3D model to load into pi3d?

      You will need to 'make' one on a bigger computer using 3D software such
      as ``blender``. This falls outside the scope of this FAQ but your best
      option is to export the model as an obj file. In Bl2.6 options I specify::

        Apply Modifiers (default)
        Include Edges (default)
        Include Normals (tick this) <<<<<<<<<<<<<<<<<<<<< *
        Include UVs (default but see below)
        Write Materials (default)
        Object as OBJ Objects (default)

        Forward -Z Forward (default)
        Up Y Up (default)
        these last two will mean that..
        Blender.x=>pi3d.x, Blender.y=>pi3d.z, Blender.z=>pi3d.y with no reflection
        of whatever you design

      ``*`` If you export without getting blender to Include Normals then pi3d
      will have to generate them when the model is loaded. This is not a
      good idea for several reasons: It will be slower to do on the pi then
      on a 'big' computer, it will have to be done every time the model is
      loaded rather than just once, it will not give the fine control
      available in blender to define the sharpness of edges.

      NB You will need to define uv mapping even if you define a material
      colour and don't intend to use a texture but might want to use a normal
      mapping shader. To do this in blender you need to tab to edit mode, select
      all vertices (a), unwrap (u, Unwrap). If the model has multiple objects
      you will need to do this for each one. After you export you may need to
      edit the ``mtl`` file so the relative path to the image is correct for
      their locations on the pi. In programs such as blender it is also possible to
      use a more detailed (high polygon) model to create a 'normal map' image
      that can be used to give surface detail to the model in pi3d. Quite
      technical but lots of instructional videos on youtube!

#.  Can I use pi3d for 2D images?

      There are various ways of doing this. The easiest way is to use the
      image to texture a simple rectangle. The simplest shape to do this
      is the Sprite which is also utilised by the ImageSprite shape to
      allow the texture to be specified as it is created. The Plane object
      is similar but is two sided. The advantage and disadvantage of this
      method is that images will be different when viewed from different
      locations.

      If you specify an orthogrphic camera (set the argument
      is_3d=False) then there will be no perspective (the image will not
      get smaller as it moves away from the camera) and each unit of the
      dimensions of the object will be one pixel on the screen. With both
      these methods the shape can be rotated, moved and scaled in all
      dimensions.

      You can also use the shader 2d_flat which takes pixels from an image
      and maps them to the screen, see below. The advantage of this
      method is that it can use the even simpler Canvas object and it always
      stays in the same place relative to the camera so you only need one
      camera, which can be the default one that you don't have to bother
      creating. See below.

#.  How do I display 2D images in front of a 3D scene? (or behind, for that
    matter)

      Either draw them onto a Canvas object using the 2d_flat shader or
      create two cameras one 3D and one 2D and assign the relevant camera
      to the types of objects you want to be drawn by each method. You
      can move the 3D camera around the scene but leave the 2D one stationary,
      that way you won't have to keep moving and rotating the 2D objects
      to keep them in front of the camera.

      Orthographic (2D) cameras will render objects with a z value that is
      severely non linear and does not relate in a simple way to the z values
      for the perspective camera. Generally 2D objects will be in front
      of objects rendered by perspective (3D) cameras unless you assign
      z values in the thousands. Too large a z value, though, and they will
      disappear beyond the 'far plane'

      If you create a camera it will become the default instance so if you
      need more than one you need to explicitly create them and it's a good
      idea to explicitly assign the one you want to each object.

#.  How do I display an image exactly without anti-aliasing or smoothing
    i.e. pixel perfect?

      This can be done by using the 2d_flat shader and spcifying when the
      Texture is loaded that mipmap=False. Because this is a global setting
      it will be overwritten by whichever Texture is the last to be loaded

#.  Some of my Textures look a bit blurred or pixely.

      Early GPUs had to have image sizes of powers of 2 pixels. i.e.
      2,4,8..1024,2048 because of the algorithm used for texture sampling,
      but modern ones can manage with any dimensions. With the raspberry
      pi we have found that some widths can cause rows of pixels to be
      offset unless they fall on certain sizes (below). **If the image
      width is a value not in this list then it will be rescaled with a
      resulting loss of clarity**

      Allowed widths 4, 8, 16, 32, 48, 64, 72, 96, 128, 144, 192, 256, 288,
      384, 512, 576, 640, 720, 768, 800, 960, 1024, 1080, 1920

#.  When the demos start there is sometimes a message in the terminal
    looking like:
    ``2013-08-19 15:36:46,232 INFO: __main__: Starting CollisionBalls``
    Where does that come from and what does it mean?

      The Log module is started by several of the basic classes (Buffer,
      EventStream, Display, Loadable, Mouse, parse_mtl, Shader, Screenshot)
      This means that all programs using the pi3d modules will create a Log
      as a by-product. It can be used for debugging and recording errors.

#.  How do I use ``pi3d.Log`` to gather or display useful information
    in my application?

      See the documentation
      `here <http://pi3d.github.io/html/pi3d.util.html#module-pi3d.util.Log/>`_.

#.  How do I keep two components (Shapes) 'joined together' as they pitch, roll
    and rotate (yaw), like the TigerTank does with its body, turret and gun?

      First of all it is easiest if you make the zero points of all the shapes
      coincide. When you move and rotate the objects you must move and rotate
      them all by the same amount. If one component is rotated about the y axis
      by a different amount from the others (i.e. the turret and gun) then
      the difference is just added to the y rotation for that component.
      However if the component is rotated about the y axis and the x axis
      (i.e. the gun) then you have to adjust the x axis and the z axis rotation
      by an amount that depends on the degree of y axis rotation. See the
      drawTiger function in demos/TigerTank.py for the kind of formula to use.

#.  I want to give my shape an angle of bank (z-axis rotation) which it
    maintains as it turns (y-axis rotation) - like an aeroplane. However the
    z-rotation is always relative to the absolute frame of reference so the shape
    pitches backwards and forwards as it turns. How do I make the frame of
    reference rotate with the shape?

      This is because of the order of the transformations done prior to
      redrawing the scene (z, then x, then y). You have to work out what the pitch
      and roll would have to be prior to rotating them about their own y axis!
      To see what I mean watch the behaviour of the tanks in demos/TigerTank.py
      You have to figure out the 'slope of the ground' so that when your
      aeroplane (or boat) is rotated it ends up with the correct pitch and
      roll. For a shape with zero pitch you can use something like::

          absheel = degrees(asin(sin(radians(heel)) * cos(radians(heading))))
          abspitch = degrees(asin(-sin(radians(heel)) * sin(radians(heading))))
          hull.position(xm, ym, zm)
          hull.rotateToX(abspitch)
          hull.rotateToY(-heading)
          hull.rotateToZ(absheel)

      And see the demos/DogFight.py version which has an extra degree
      of freedom.

#.  Is it possible to change the shape of an object once it's been made?

      The most efficient way is to use the scale(sx, sy, sz) method. However,
      this obviously limits the shape changing that can take place. If the
      shape needs to be changed more than this then it can be remade as
      a new instance to replace the old one. (At one stage it was necessary to
      clear the previous opengles buffers using the unload_opengl() method
      before destroying the old shape to stop a graphics memory leak.
      This issue seems to be fixed but if you run into memory problems
      it might be worth trying this. Plus, obviously, report it to us!)

      The alternative way of doing it is to use the Buffer.re_init() method
      which takes the same arguments as Buffer.__init__() (see documentation)
      so is a little more technical to use.

#.  Sometime, when I move the mouse or the program is loading a file from
    disk, everything slows down or freezes.

      The Display has a frames_per_second argument and if you set this
      lower than the flat out rate it will give the processor some 'slack'
      to accomplish other jobs.

      To do things like file loading in the background (for instance, preloading
      an image or Shape so that it can instantly appear later) you need to use
      Python's threading - demos/Slideshow_2d.py is an example.

#.  I am running pi3d on a non-raspberry pi Linux machine but it's running
    at a very slow frame rate.

      Probably the GPU can't run the OpenGL2+ code that mesa interprets
      from the pi3d OpenGLES2 commands. Check the specification for the
      graphics card. ``lspci -v`` and ``feedback.wildfiregames.com/report/opengl/``

#.  Some of the demos on a non-raspberry pi Linux machine work fine but
    other don't run and give an error::

      IOError: [Errno 13] Permission denied: u'/dev/input/mice'

    what is the
    cause of this

      The Mouse gets its info from the operating system file described in
      the error message. This requires it to be run from root, you can
      do this by ``sudo python ForestWalk.py``.

#.  Using python3 and the InputEvents mouse input (Silo and DogFight demos)
    I get very ragged and unresponsive camera movment.

      This should be fixed as of v1.5, try upgrading to the latest
      version of pi3d

#.  How do I do post-rendering processing on a scene, such as blurring,
    edge detection or fancier effects such as oil painting.

      There is a class PostProcess that can be used to render a scene to
      a texture. The Post.py demo shows a simple 3x3 convolution matrix
      shader and there are a host of post process filter shaders that
      are in the pi3d_demos/shaders directory. These wll be loaded in
      turn by FilterDemo.py but the pi will run out of graphics memory
      if you leave the full list in. For more complicated effects it's
      over to you!

#.  OK the example for post processing (Post.py) is quite hard to follow
    how exactly does the PostProcess class work.

      PostProcess inherits from Texture (via OffScreenTexture) so you can
      use an instance of it anywhere you would use a texture, i.e. you
      could uv map it onto any other shape or use it as a bump or
      reflection map. Or use it with your own shader to do something I
      haven't thought of. PostProcess.sprite is a Sprite shape that can
      be used just as any other Shape in your program, you could rotate
      it or change its alpha value or z location to draw it in front of
      other objects. There is also a 2D camera created in PostProcess
      which is used to draw the sprite at full screen using the saved
      texture and the shader you supply in the constructor or post_base
      if you don't supply one.

      PostProcess.draw({48:1.1414, 49:2013, 50:0.0}) will set the unif
      array in PostProcess.sprite as unif[48] = 1.1414 unif[49] = 2013
      unif[50] = 0.0 you can then access these values as uniform
      variables in your shader as vec3 unif[16][0] unfi[16][1]
      unfi[16][2]. If the array indices are contiguous you could do the
      same thing using PostProcess.sprite.set_custom_data(48, [1.1414,
      2013, 0.0]) or even PostProcess.sprite.unif[48] = 1.1414 etc

      I see no reason why you shouldn't do something like:
      render the scene to a texture once a second draw it off-screen using
      a shader to extract edges as dayglo on white, blur them to a second
      texture, draw this onto a foreground sprite fading from alpha 0 to
      1 back to 0 over 1s cycle. Use a different shader to draw the original
      texture onto a spherical surface that gradually changes shape in
      the background. etc etc. 

#.  And why does python set Shape.unif[48] but the shader use
    vec3 unif[16][0].

      On the shader side it's really efficient to define variables as
      vec3, vec4, mat4 etc. and at one stage I tried doing a lot of the
      matrix manipulation in the vertex shader. There were pros and
      cons but in the end I found that using python's numpy library
      was the best bet. But in the mean time I had started storing
      much of the shape information in a form that allowed it to be
      accessible by the shader i.e. location x,y,z was vec3 unif[0]
      in the shader, rotation was vec3 unif[1], scale unif[2], origin
      offset unif[3] etc. Although I no longer needed these for normal
      rendering I thought that they may come in useful for someone at
      some stage so I just left them. I only needed to pass one array
      pionter so there was no cost to having 60 floats available!

      Meanwhile back in the python description of the Shape I had to
      make the unif array a ctypes.c_float array and that seemed to
      have to be one-dimensional. So after a long story unif[16][0]
      in the shader is (same name but different) unif[16*3 + 0] in python

#.  How can you render points like a star field
    or sparks from an explosion.

      If you use the method set_point_size() on a Shape to a value other
      than 0.0 then the vertices of the Shape will be rendered as points.
      The size will actually vary with distance but will be the size you
      specified at 1 unit of distance from the camera.

      pi3d.Points can be used to render points using the mat_flat shader

#.  How can I set up an SD card without all of Raspbian's clutter that will
    boot quickly and allow me to run a dedicated pi3d application.

      I decided that Arch would be tidiest for this as it will comfortably
      fit onto 2GB SD and boots in a few seconds. These were the steps:

      1.  download and unzip the image from
      http://www.raspberrypi.org/downloads

      2. follow the instructions from http://elinux.org/RPi_Easy_SD_Card_Setup
      to get the image onto the SD card

      3. put card in Pi and boot it up.
      log in as ``root``, password ``root`` I didn't change these or set
      up a normal user account with sudo etc. as the card will just be
      used for running one application not connected to the net. You may
      want to do otherwise in which case look at this
      http://elinux.org/ArchLinux_Install_Guide

      4.
      ``# pacman-key --init``

      4a.
      <Alt><F2> ``# ls -R / && ls -R / && ls -R /``

      4b. <Alt><F1> to get back to normal terminal, this is all to do with
      generating entropy to get a random key (apparently).

      5.
      ``# pacman -Syu`` [update packages]

      6.
      ``# pacman -S python2``

      7.
      ``# pacman -S python2-numpy``

      8.
      ``# pacman -S python2-pillow``

      9.
      ``# pacman -S python2-pip``

      10.
      ``# pacman -S git``

      11. ``# pip2 install pi3d --pre`` [the --pre flag tells it to install
      even if pre-release version i.e. 1.7a]

      12.
      ``# cd /home/``

      13. ``# git clone https://github.com/paddywwoof/sailsim.git`` [this would
      be your actual repository, alternatively you could just copy the files
      onto the SD card from a local machine]

      if you need to access the RPi.GPIO
      system from your application then you also need to

      14.
      ``# pacman -S gcc``

      15.
      ``# pip2 install RPi.GPIO``

      if you want to make it a bit easier to start up the application
      then you could make a little script file like this::

        #!/bin/bash
        cd /home/sailsim/
        python2 sailsim.py

      called ``sailsim`` and you then put that file in the /usr/bin/ directory
      and make it executable ``# chmod +x sailsim`` then after logging in
      you will just be able to type ``# sailsim`` and start the app.

      I did managage to get the app to start 'automatically' *before* logging
      in by adding the file below as /etc/systemd/system/start_sailsim.service ::

        [Unit]
        Description=Run sailsim on boot
        After=network.target
        [Service]
        Type=oneshot
        ExecStart=/usr/bin/sailsim
        [Install]
        WantedBy=multi-user.target

      Then run ``# systemctl enable start_sailsim.service`` However there
      were unsatisfactory side effects to do with timing which meant I
      could not use it in this way.
      
#.  Does pi3d work
    with pypy

      pi3d relies on some of the functionality and speed of numpy and this
      only really became useable as of pypy-2.2 and I have managed to get
      pi3d working to some extent with that. At the moment that isn't the
      current version you get with apt-get so these were the steps I took:

      1. download the relevant version from http://pypy.org/download.html
      for your machine (Ubuntu, raspbian etc) extract it into a new directory
      i.e. /home/me/pypy-2.2.1-linux64

      2. in a
      terminal::

        sudo apt-get install pypy-dev

      3. download and install pypy-numpy so it's also in a subdirectory
      of pypy-x.x.x-etc I did this cd to that directory then using::
      
        git clone https://bitbucket.org/pypy/numpy.git
        cd numpy
        sudo ../bin/pypy setup.py install

      4.* download Pillow from https://pypi.python.org/pypi/Pillow and
      extract it into its own subdirectory of pypy-x.x.x-etc i.e.
      /home/me/pypy-2.2.1-linux64/Pillow-2.2.1

      5.* download http://python-distribute.org/distribute_setup.py to
      pypy-x.x.x-etc/bin and run it::

        sudo ./pypy distribute_setup.py

      6.* either cd to pypy-x.x.x-etc/bin
      and run::

        sudo ./easy_install Pillow

      7.* or cd to the Pillow-x.x directory
      and run::
      
        sudo ../bin/pypy setup.py install

      I did different permutations of these things but confused myself as
      to which I was 'really' doing (by occasionally forgetting to type
      ``./pypy`` and thereby running a debian package version that was
      also installed) so some of these steps are redundant. Also other
      steps may be missing.

      At the moment (Dec13
      https://github.com/tipam/pi3d/commit/ce5febc6693115872c7e4653dfea503e029fa0d5)
      the changes to Shape.draw() have been commented out because they
      look to add some extra processing at an expensive location. If
      you want to try pypy you will have to swap the two lines (search
      for pypy to find them)

#.  How can I make my own EnvironmentCube images using pictures of my
    garden or school playground?
  
      Option 1. Using an EnvironmentCube (as the question says) but see
      below for using a Sphere, which is probably easier.
      
      There are lots of ways of doing this and different software as well
      as special cameras. However this is the method I have followed using
      freely available software: gimp and blender (running on a 'normal'
      computer rather than the pi at this stage).
    
      The first half of the job is to get a set of images into a 'seamless'
      band. Obviously you need to have taken a set of pictures that overlap
      25% to 50%. In gimp make a new image that is higher and wider than
      you will need to paste all the images side by side. You will need to
      have the same image repeated at the left end and the right end.
    
      Open each image in gimp then copy it, go to the new 'wide strip'
      image and paste as new layer. Use the four headed arrow to position
      each layer so it 'joins up'. When you put the duplicate left most
      image at the right end you need to make sure that it is at exactly
      the same vertical position as it is on the left.
    
      Working down from the top layer add layer masks (default white, full
      opacity) then using gradient fill tool make the mask fade from
      transparent to opaque across the overlapping portion. You might need
      to slightly rotate some images to make them join up nicely from one
      side to the other.

      When it looks perfect (!) merge the layers down then crop the image
      so there are no gaps at the top and bottom and so the left and right
      edges join seamlessly. You will probably have to zoom to maximum and
      choose an easily identifiable pixel. The rectangular selection tool
      in gimp allows the edges to be dragged to fine tune it. Export the
      image to jpg or png possibly after reducing to a reasonable size. Have
      some suitable sky only image to patch into the top of the sphere you
      will create in blender...

      I used blender 2.69, it's not a trivial application if you've not used
      it before and it might take a bit of effort to figure out what I'm
      referring to [tab] means tab key, otherwise it's probably a menu
      item or an icon in the right hand. Lots of youtube videos to look at.
      In blender:

      1. [del] delete the
      startup cube
      
      2. ``Add Mesh UV Sphere``, on left tools
      set ``Shading Smooth``
      
      3. [s] to scale up
      to about 10x

      4. [tab] to edit mode [a] to deselect all vertices. R-click on top
      vertex the Ctrl-numpad+ to select vertices down to about 45 degrees
      north (or use [b] and box select) [del] delete vertices. You should
      now have a sphere with the top cut off

      5. [tab] back to object mode then create another sphere at the same
      location but scale it up very slightly bigger and chop off the bottom
      but so they overlap just a little.

      6. [tab] back to object mode then ``Add Empty Cube`` at the same location
      (NB if you accidentally left click on the view window you will move
      the starting point marker where new things appear). You should be able
      to zoom in with the mouse wheel and see this cube inside the spheres.

      7. still in object mode right click to select the bottom (inner and larger)
      sphere. The edge should go yellow to indicate it's been selected.

      8. on the right properties window click the Materials icon (CofG circle
      4th from right), then + new.

      9. then click the Textures icon (red/white check 3rd from right),
      then + new, ``Type Image or movie``, ``Image New`` browse to the wide horizon
      image you made, ``Mapping Projection Tube``

      10. still in object mode right click on the top sphere, add material and
      texture exactly as for the bottom sphere but select the patch of sky
      image mentioned above and choose ``Mapping Projection Flat``

      11. in object mode right click on the Empty Cube and add a new Texture (you
      should see a reduced list of options so it's 2nd from right in the list)

      12. select under ``Type Environment Map`` then under ``Environment Map Static``,
      ``Mapping Cube`` and ``Viewpoint Object  Empty``

      13. in the properties icons select render (camera left most) then under
      Render press the render button. This should flash up a series of six
      smaller images then go black!

      14. re-select the Texture icon (all of these steps should have the Empty
      Cube as the selected object) and the little down arrow under Environment
      Map should produce a drop-down menu with an option to save the image.

      The texture can then be used in pi3d with EnvironmentMap type BLENDER. However
      there will be a sharp line where the edge of the bottom sphere fell. You can
      smooth this out using clone, repair, blur and blend tools in gimp; be
      careful not to blur the boundaries between the six images.
    
#.  How do I make an Environment Sphere (such as can use the Photo Sphere
    images created by later versions of Android)
    
      First you need an image very much like the one outlined in the previous
      question. If you have the software on your phone or tablet to do a
      Photo Sphere that's going to be a lot easier but you can do something
      similar with a series of panoramas as modern cameras can make. The
      image needs to be twice as wide as it is high using a standard cylindrical 
      projection http://en.wikipedia.org/wiki/Equirectangular_projection
      
      This image is used for a Texture uv mapped to a standard pi3d.Sphere
      but the Texture needs to have the argument ``flip=True`` and the Sphere
      needs the argument ``invert=True``
      
#.  How can I speed up loading Models. Even quite low polygon counts
    seem to take ages on the Raspberry Pi
    
      Thanks to Avishay https://github.com/avishorp it is possible to use 
      the python pickle functionality to serialise pi3d Shapes including
      Model. [develop branch as at 2014-05-07]
      
      There is an example on github.com/pi3d/pi3d_demos
      LoadModelPickle.py which shows the process but basically:
      
        load the models once normally, create a file (has to be 
        binary for python3) to write to, then ``pickle.dump(mymodel, f)``
        
        subsequently open the file to read from and ``mymodel = pickle.read(f)``
        the loaded file will have any required Textures included automatically
        including bump and reflection maps. However the shader will still
        need to be set with ``set_shader()``
        
      Loading from a pickle file is significantly faster than parsing a
      wavefront obj file but (because of the less efficient image compression)
      the disk space used will be much higher.

.. _ReadMe: http://pi3d.github.com/html/index.html
