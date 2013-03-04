Frequently Asked Questions
==========================


#.  My RPi Crashes or reboots when I try and run a demo:

      Anything using a loadable texture (i.e. nearly all the demos) requires
      the Python Imaging module, some demos need tk, see the ReadMe_ on installing
      these.

#.  I see nothing but a White screen:

      Possibly something has gone wrong with an opengles function, sometimes
      there might be an error message in the terminal such as failed to create
      display. This could be caused by running out of gpu memory (see ReadMe_
      for how to set up memory split) occasionally multi-thread applications
      an cause this problem if an opengles function call is made not from the
      main thread.

#.  I see nothing but a black screen:

      Possibly something has gone wrong in a shader. There may be a line
      nunber reference output by the shader compiler in the terminal window.
      It is great fun experimenting with shaders but they are *?£#%* taciturn
      beasts to debug! The problem could be caused by sending some bad
      render setting to a shader.

#.  I see nothing but the background (you will need to set background to
    non-transparent and a color not equal to black to white to determine
    if this is happening):

      Either the shape is behind the near plane, beyond the far plane, is outside
      the field of view or the polygons are facing away from the camera. Often
      this is because you are actually **inside** the object. Try using the
      Camera.point_at([x,y,z]) function (see demos/ClashWalk.py for use) or move
      and rotate the object. Sprite and ImageSprite shapes are one sided so
      cannot be seen from behind, try using a Plane instead
        
#.  I can just see black silhouettes against the background:

      You may be trying to use a shader that requires light but there isn't
      any, or it's turned down too low. Try switching to a 'flat' shader
      to check. Alternatively, if it's a shape you have generated such as
      a Lathe or a Model, the normal vectors might be pointing in the wrong
      direction. Try re-generating the shape, the path you use for the Lathe
      needs to start at the top of the object and there are functions in
      most 3D modeling applications to recalculate normals, or force them
      to point outwards.
      
#.  The demo loads but the mouse doesn't move the camera as it's supposed to:

      If this only happens on demos using the ``event`` library (such as Silo.py)
      then it could be the hardware configuration is pretending to be something
      it isn't. It's not uncommon for keyboards to say they are mice or joysticks.
      If you have a mouse combined with a keyboard (to save on USB slots) then
      you might need to use ``get_mouse_movements(1)``. If you have problems
      with a device or inputs using the event system it's a good idea to run
      ``python FindDevices.py`` from ``pi3d/event/`` this will give you lots
      of additional information.
      TODO - other specific fixes

#.  How do I use a joystick, gamepad, xbox controller etc with a pi3d
    application:
    
      Often these will just work when plugged into the USB
      sometimes you may need to use a different InputEvents method, for instance
      with an xbox 360 you get the left joystick from ``get_joystickB3d()``
      Also you would need to install the driver and start it running first::
      
        sudo apt-get install xboxdrv
        sudo xboxdrv -s -i 0

#.  How do I make my own 3D model to load into pi3d:

      You will need to 'make' one on a bigger computer using 3D software such
      as ``blender``. This falls outside the scope of this FAQ but your best
      option is to export the model as an obj file. In the options you should
      make sure that you specify normals to be saved. If you want your 3D model
      to be textured (have a picture wrapped around it) you will need to
      define uv mapping. In programs such as blender it is also possible to
      use a more detailed (high polygon) model to create a 'normal map' image
      that can be used to give surface detail to the model in pi3d. Quite
      technical but lots of instructional videos on youtube!
      
#.  I get error messages trying to install from PiStore:

      There was an issue with the early versions of PiStore which should
      be fixed if you update it. If you continue to have problems and you
      are somehow able to read this FAQ somewhere else you should be able
      to download a zipped file from http://pi3d.github.com There is also
      documentation and installation instructions on that webiste.
      
#.  Can I use pi3d for 2D images:

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
      method is that it can use the even simpler Canvas object and 

#.  How do I display 2D images in front of a 3D scene:

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
    i.e. pixel perfect:
    
      This can be done by using the 2d_flat shader and spcifying when the
      Texture is loaded that mipmap=False. Because this is a global setting
      it will be overwritten by whichever Texture is the last to be loaded

.. _ReadMe: http://pi3d.github.com/html/index.html
