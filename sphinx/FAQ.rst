Frequently Asked Questions
==========================


1.    Crashes or reboots the pi:

        Anything using a loadable texture (i.e. nearly all the demos) requires
        the Python Imaging module, some demos need tk see the ReadMe_ on installing
        these.

2.    I see nothing but a White screen:

        Something has gone wrong with an opengles function, sometimes there might
        be an error message in the terminal such as failed to create display. This
        could be caused by running out of gpu memory (see ReadMe_ for how to set
        up memory split) occasionally multi-thread applications can cause this
        problem if an opengles function call is made not from the main thread.

3.    I see nothing but a black screen:

        Something has gone wrong in a shader. There may be a line nunber reference
        output by the shader compiler in the terminal window. Let us know

4.    I see nothing but the background (you will need to set background to
      non-transparent and a color not equal to black to white to determine
      if this is happening):

        Either the shape is behind the near plane, beyond the far plane, is outside
        the field of view or the polygons are facing away from the camera. Often
        this is because you are actually **inside** the object. Try using the
        Camera.point_at([x,y,z]) function (see demos/ClashWalk.py for use) or move
        and rotate the object. Sprite and ImageSprite shapes are one sided so
        cannot be seen from behind, try using a Plane instead
      
5.    The demo loads but the mouse doesn't move the camera as it's supposed to:

        If this only happens on demos using the ``event`` library (such as Silo.py)
        then it could be the hardware configuration is pretending to be something
        it isn't. It's not uncommon for keyboards to say they are mice or joysticks.
        If you have a mouse combined with a keyboard (to save on USB slots) then
        you might need to use ``get_mouse_movements(1)``. If you have problems
        with a device or inputs using the event system it's a good idea to run
        ``python FindDevices.py`` from ``pi3d/event/`` this will give you lots
        of additional information.
        TODO - other specific fixes

6.    How do I use a joystick, gamepad, xbox controller etc with a pi3d
      application:
    
        Often these will just work when plugged into the USB
        sometimes you may need to use a different InputEvents method, for instance
        with an xbox 360 you get the left joystick from ``get_joystickB3d()``
        Also you would need to install the driver and start it running first::
        
          sudo apt-get install xboxdrv
          sudo xboxdrv -s -i 0

7.     How do I make my own 3D model to load into pi3d:

        You will need to 'make' one on a bigger computer using 3D software such
        as ``blender``. This falls outside the scope of this FAQ but your best
        option is to export the model as an obj file. In the options you should
        make sure that you specify normals to be saved. If you want your 3D model
        to be textured (have a picture wrapped around it) you will need to
        define uv mapping. In programs such as blender it is also possible to
        use a more detailed (high polygon) model to create a 'normal map' image
        that can be used to give surface detail to the model in pi3d. Quite
        technical but lots of instructional videos on youtube!
      
8.    I get error messages trying to install from PiStore:

        There was an issue with the early versions of PiStore which should
        be fixed if you update it. If you continue to have problems and you
        are somehow able to read this FAQ somewhere else you should be able
        to download a zipped file from http://pi3d.github.com There is also
        documentation and installation instructions on that webiste.
      
9.    Can I use pi3d for 2D images:

10.   How do I display 2D images in front of a 3D scene:

11.   How do I display an image exactly without anti-aliasing or smoothing
      i.e. pixel perfect::
