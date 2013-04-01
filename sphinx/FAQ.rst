Frequently Asked Questions
==========================


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

      Possibly something has gone wrong in a shader.

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

      TODO - other specific fixes

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
        Include Normals (have to tick)
        Include UVs (default but see below)
        Write Materials (default)
        Object as OBJ Objects (default)

        Forward -Z Forward (default)
        Up Y Up (default)
        these last two will mean that..
        Blender.x=>pi3d.x, Blender.y=>pi3d.z, Blender.z=>pi3d.y with no reflection
        of whatever you design

      If you want your 3D model to be textured (have a picture wrapped around it)
      you will need to define uv mapping (after you export you may need to
      edit the ``mtl`` file so the relative path to the image is correct for
      their locations on the pi. In programs such as blender it is also possible to
      use a more detailed (high polygon) model to create a 'normal map' image
      that can be used to give surface detail to the model in pi3d. Quite
      technical but lots of instructional videos on youtube!

#.  I get error messages trying to install from PiStore?

      There was an issue with the early versions of PiStore which should
      be fixed if you update it. If you continue to have problems and you
      are somehow able to read this FAQ somewhere else you should be able
      to download a zipped file from http://pi3d.github.com There is also
      documentation and installation instructions on that webiste.

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

#.  How do I display 2D images in front of a 3D scene?

      Either draw them onto a Canvas object using the 2d_flat shader or
      create two cameras one 3D and one 2D and assign the relevant camera
      to the types of objects you want to be drawn by each method. You
      can move the 3D camera around the scene but leave the 2D one stationary,
      that way you won't have to keep moving and rotating the 2D objects
      to keep them in front of the camera.

      Orthographic (2D) cameras will render objects so they are in front
      of objects rendered by perspective (3D) cameras.

      If you create a camera it will become the default instance so if you
      need more than one you need to explicitly create them and it's a good
      idea to explicitly assign the one you want to each object.

#.  How do I display an image exactly without anti-aliasing or smoothing
    i.e. pixel perfect?

      This can be done by using the 2d_flat shader and spcifying when the
      Texture is loaded that mipmap=False. Because this is a global setting
      it will be overwritten by whichever Texture is the last to be loaded

#.  When the demos start there is a message in the terminal
    ``..echomesh.util.Log: Log level is INFO``

    Where does that come from and what does it mean?

      pi3d uses three utilities developed in parallel with it for the
      echomesh project (see ./echomesh/util/ directory). The Log module is
      started by several of the basic classes (Buffer, EventStream, Display,
      Loadable, Mouse, parse_mtl, Shader, Screenshot) This means that all
      programs using the pi3d modules will create an echomesh Log as a
      by-product. It can be used for debugging and recording errors.

#.  How do I use ``echomesh.util.Log`` to gather or display useful information
    in my application?

      You need to create an instance ``LOGGER = Log.logger(__name__)`` typically
      then call methods of this such as ``LOGGER.info("...")`` or ``LOGGER.debug()``

#.  How do I see the logged information from ``echomesh.util.Log``?

      TODO I can't find any file, but it looks like Log needs something in
      echomesh.config.Config otherwise it will get an exception and LOG_FILE
      will be set to a zero length string.

#.  How do I keep two components (Shapes) 'joined together' as they pitch, roll
    and rotate (yaw), like the TigerTank does with its body, turret and gun?

      First of all it is easiest if you make the zero points of all the shapes
      coincide. When you move and rotate the ojbects you must move and rotate
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

#.  Is it possible to change the shape of an object once it's been made?

      The most efficient way is to use the scale(sx, sy, sz) method. However,
      this obviously limits the shape changing that can take place. If the
      shape needs to be changed more than this then it can be remade as
      a new instance to replace the old one. When you do this you should
      clear the previous opengles buffers using the unload_opengl() method
      before destroying the old one otherwise there will be a memory leak.

      The alternative way of doing it is to use the Buffer.re_init() method
      which takes the same arguments as Buffer.__init__() (see documentation)
      so is a little more technical to use. There is an example of this
      technique demos/IceGrow.py

#.  Sometime, when I move the mouse or the program is loading a file from
    disk, everything slows down or freezes.

      The Display has a frames_per_second argument and if you set this
      lower than the flat out rate it will give the processor some 'slack'
      to accomplish other jobs.

      To do things like file loading in the background (for instance, preloading
      an image or Shape so that it can instantly appear later) you need to use
      Python's threading - demos/Slideshow_2d.py is an example.

.. _ReadMe: http://pi3d.github.com/html/index.html
