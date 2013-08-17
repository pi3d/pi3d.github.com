3D Graphics Explanation
=======================

This is my understanding of how the gpu and 3D graphics are implemented
on the Raspberry pi. It is based entirely on what I have had to learn in
order for me to get the pi3d library working and doing what I wanted. i.e.
there will be gaps and possibly misapprehensions! Also I intentionally
skip over many of the more involved aspects such as rendering to off-screen
buffers, using masks, etc.

There are three external shared object libraries imported into pi3d to
allow access to the graphics processor: libbcm_host.so is used to create
and manage a display surface to draw on, libELG.so and libGLESv2.so provide
access to the OpenGL language functions developed to standardise utilisation
of graphics cards. Mobile devices, including the Raspberry pi use a slightly
cutdown version called OpenGLES, specifically version 2.0.

From OpenGLESv2.0 onwards the fundamental graphics donkey work is done by
'shaders' that are defined by the developer and compiled as the program
runs rather than being 'built into' the gpu. This opens up a fantastic
range of possibilities but there are some fundamental limits that may not
be immediately apparent. There are two parts to a shader: the vertex
shader and the fragment (essentially pixel) shader which are written in
a C like language. I will give some more detail to what each actually does
later but in outline; information can be passed from the cpu program (in
our case python pi3d ones) to the shaders and they can pass information
between them however the only output is pixels and they operate solely on
one vertex or pixel. i.e. the vertex shader can't access information about
the location of adjacent vertices and the fragment shader can't determine
the colour, say, of adjacent pixels. This means that some operation such
as blurring or edge detection have to be done as a double pass process.

Information needed to render the scene can be passed to the shader as
'uniform' variables. This generally includes things such as the location
and colour of light sources, fog properties, a sequence  or scale number
for moving patterns etc. In addition there are some specific elements that
I will explain in detail: 1. 'attribute' arrays containing vertex x,y,z
values, normal vectors and texture coordinates, 2. matrices defining how
the object has moved or rotated and how the camera has moved, rotated and
perspectived (!). 3. images or texture samplers to 'clothe' the object or
provide information on bumps or reflections.

1.    3D objects are defined for use in graphics programs starting with a
      list of points or vertices in space each one needing x, y, z coordinates.
      Although not generally essential, in pi3d each vertex has a normal vector
      defined as well. This is effectively an arrow at right angles to the surface
      at that point and it also needs three values to define its magnitude in
      the x, y, z directions. The normal vector can be used by the shader to
      work out how light would illuminate a surface or how reflections would
      appear. If the normals at each corner of a triangular face are all pointing
      in the same direction then the fragment shader will treat the surface as
      flat, but if they are in different directions the surface will appear to
      blend smoothly from one direction to another. 3D models created in
      applications such as blender normally have an option to set faces to look
      either angular or smoothed by calculating different types of normal vectors.
      Each vertex also has a texture coordinate attached to it. This is often
      termed the u, v position from a two dimensional texture that is to be mapped
      to that vertex. Again the fragment shader can interpolate points on a surface
      between vertices and look up what part of a texture to render at each pixel.
      The final piece of information needed by the shader is to define which
      vertices to use for the corners of each triangle. So if I use as an example
      a very simple one sided square this could be defined by the vertices:

      [(x=0.0, y= 0.0, z=0.0), (x=0.0, y=1.0, z=0.0), (x=1.0, y=1.0, z=0.0), (x=1.0, y=0.0, z=0.0)]

      and the normals:

      [(x=0.0, y=0.0, z=-1.0),(x=0.0, y=0.0, z=-1.0),(x=0.0, y=0.0, z=-1.0),(x=0.0, y=0.0, z=-1.0)]

      and texture coordinates:

      [(u=0.0, v=0.0) ,(u=0.0, v=1.0) ,(u=1.0, v=1.0) ,(u=1.0, v=0.0) ]

      and triangle indices (note the order of corners is important. The triange
      'faces' towards a view where the sequence is clock-wise. Normally the backs
      of faces are not rendered by the gpu):

      [(0,1,2), (0,2,3)]

      Draw these out on some paper so you can see how the system works. The gpu
      uses coordinate directions x increases from left to right, y increases
      from bottom to top, z increases going into the screen.

2.    The gpu has been designed to be fantastically efficient at performing
      vector and matrix arithmetic. So rather than the cpu calculating where
      about the vertices have  moved and how these positions can be represented
      on the 2D computer screen it simply calculates a transformation matrix
      to represent this and passes that to the gpu. In pi3d we pass two matrices,
      one representing the object translation, rotation and scale and an additional
      one including the camera movement and perspective calculations. In the
      vertex shader these matrices are used to convert the raw vertex positions
      to screen locations and to work out where the light should come from to
      work out shadows.
      
3.    Image files can be converted into texture arrays that are accessed
      very efficiently by the gpu.

When pi3d/Buffer draw() method is called for a 3D object the python side
of the program sets the shader and necessary uniform variables to draw the
given object. It then works out the 4x4 matrix combining translation, rotation,
scale for the object and an additional matrix incorporating the camera
movement and lens settings. The camera has two basic modes for handling
perspective, the default is 'normal' where things further away are represented
as smaller on the screen and the this is defined by a viewing angle between
the top edge of the screen and bottom edge. If the camera is set to
orthographic mode then objects do not get smaller in the distance and one
unit of object dimension corresponds to a pixel on the screen. An orthographic
camera can be used to do fast 2D drawing.

The glDrawElements function is then called  which sets the vertex shader
to work out the locations of each vertex, normal, lighting, texture in
terms of screen coordinates. The vertex shader then passes the relevant
information to the fragment shader which  calculates what colour and alpha
value to use for each pixel. The fragment shader takes into account the
depth value of each pixel and doesn't draw anything that is behind something
it has already drawn. This means that it is more efficient to draw opaque
objects from near to far but if something is partially transparent then
is must be drawn after anything further away that should 'show through'.

Finally, pi3d uses a double buffer system where everything is drawn onto
an off-screen buffer which, when complete, is swapped 'instantaneously' to
visible. This makes the animation much smoother. 
