# Lab 11: Textures & FBOs

## 1. Intro

 In lab 9, you learned how OpenGL stores vertex data in two types of objects called vertex buffer objects and vertex array objects. You learned about how to work with scene data in real time. But what about working with 2 dimensional data? In previous projects you worked with a canvas object that displayed your results onto the screen. But how does this work in OpenGL? So far, we have seen the real time pipeline up to the final step in this diagram, the framebuffer.

In an oversimplification, the framebuffer is the 2D canvas that opengl works with when using your shader program you wrote in the last lab. So far, you haven’t had to worry about this since you have been working with the default framebuffer that OpenGL provides which happens to be your application window. But what if you don’t want to draw on the screen? What if you want to draw onto a texture and save it for later? This is where making your own framebuffer objects comes in!
By the end of this lab, you will be able to:
1. Understand how textures work in OpenGL
2. Understand framebuffers and when to use them
3. Draw your framebuffers onto your screen
4. Apply cool post-processing effects in real-time!

## 2. Textures

Before we dive into the 2D data we draw to, let’s think about a common form of 2D data we use in our own scenes…Textures!

### 2.0. What is a Texture

In short, a texture is a 2D image that has the ability to be read and written to. In many cases a texture will be pixelized and each pixel can store anything from RGBA color data to booleans.

### 2.1. Creating a Texture in the CPU

#### 2.1.1 QImages

In Qt creator, the most common form of an image is called a QImage. Let’s create one of our own!

The QImage constructor takes in two parameters, a file path formatted as a QString, and a format specification which is optional. 

|**_Task 1:_**|
|:---|
|Store a QImage in the <code>m_image</code> member variable using the relative file path of the kitten.png image in our project|

Now let’s format our QImage to fit OpenGL standards. Unlike OpenGL which has its UV coordinate (which you will learn about soon) origin at the bottom left, a QImage stores it in the top left. Therefore one of our tasks is to mirror the image vertically. The second is we need to ensure that we have 4, 8-bit color channels for R, G, B, and A.

|**_Task 2:_**|
|:---|
|To do this let’s overwrite our <code>m_image</code> to be:<br/>`m_image = m_image.convertToFormat(QImage::Format_RGBA8888).mirrored();`|

#### 2.1.2 OpenGL Textures

Great, now we have our Qt formatted image, let’s put it in an OpenGL texture. To start, we need to generate a texture using the following function:

`void glGenTextures(GLsizei n, GLuint * textures)`

<details>
  <summary>void:</summary>
</details>
<details>
  <summary>GLsizei n:</summary>
</details>
<details>
  <summary>GLuint * textures:</summary>
</details>

|**_Task 3:_**|
|:---|
|Generate a texture and store it’s id in <code>m_test_texture</code>|

Before we work with the texture, we need to bind it to the state machine using:

`void glBindTexture(GLenum target, GLuint texture)`

<details>
  <summary>void:</summary>
</details>
<details>
  <summary>GLenum target:</summary>
</details>
<details>
  <summary>GLuint texture:</summary>
</details>

Now we have an empty texture sitting in the GL_TEXTURE_2D target in our state machine. Let’s fill it with our QImage using:

`glTexImage2D(GLenum target, GLint level, GLint internalformat, GLsizei width, GLsizei height, GLint border, GLenum format, GLenum type, const void * data)`

<details>
  <summary>void:</summary>
</details>
<details>
  <summary>GLenum target:</summary>
</details>
<details>
  <summary>GLint level:</summary>
</details>
<details>
  <summary>GLint internalformat:</summary>
</details>
<details>
  <summary>GLsizei width:</summary>
</details>
<details>
  <summary>GLsizei height:</summary>
</details>
<details>
  <summary>GLint border:</summary>
</details>
<details>
  <summary>GLenum format:</summary>
</details>
<details>
  <summary>GLenum type:</summary>
</details>
<details>
  <summary>const void * data:</summary>
</details>

|**_Task 4:_**|
|:---|
|Load our <code>m_image</code> variable into <code>m_test_texture</code> which is currently stored in the GL_TEXTURE_2D target|

Before we use the texture, we need to specify some behavior it should take on in particular if the image needs to be scaled up or down. Consider the situation where our fragment lies between two pixels in our texture. Which color should it output? These are parameters we can control and in our case we can ask for OpenGL to linearly interpolate between the nearby pixels.

How do we set these parameters? The function to do so is:

`void glTexParameteri(GLenum target, GLenum pname, GLint param)`

<details>
  <summary>void:</summary>
</details>
<details>
  <summary>GLenum target:</summary>
</details>
<details>
  <summary>GLenum pname:</summary>
</details>
<details>
  <summary>GLint param:</summary>
</details>

|**_Task 5:_**|
|:---|
|Use this function to set the minimize and magnify filters to use linear interpolation|

|**_Task 6:_**|
|:---|
|Unbind our texture from the GL_TEXTURE_2D target|

### 2.2. Passing A Texture To The GPU: Uniforms 2 Electric Boogaloo

Now, how do we work with the texture we just created? We can create a uniform variable for it just how we did in the shaders lab for different data types! 

Let’s begin by creating a uniform variable in our shader that will hold our texture. The data type of a texture is known as sampler2D.

|**_Task 7:_**|
|:---|
|Add a sampler2D uniform variable to the fbo.frag shader file|

Now how do we set this variable? How textures work in OpenGL is using a concept called **texture units**, which you can read more about below. 

<details>
  <summary>Texture Units</summary>
  Texture slots serve as references to OpenGL texture objects which allow them to be sampled in a shader program. Typically there are around 32 slots but the number depends on the graphics card. This means in any shader you can sample at most 32 different textures.
</details>

To set our uniform, we first need to load our texture into a texture unit, and then indicate which slot index should be sampled in our shader.

To load a texture into a texture slot, the steps are:
1. Set the current active texture slot
2. Bind the texture to the appropriate target representing its type

The first call is:
`void glActiveTexture(GLenum texture)`

<details>
  <summary>void:</summary>
</details>
<details>
  <summary>GLenum texture:</summary>
  This is the texture unit your OpenGL texture is bound to.
</details>

The second call is the same binding call we have seen before. 

|**_Task 8:_**|
|:---|
|Before binding our texture, manually set the active texture slot to slot 0|

To set the uniform value, it is represented by an int that correlates to the texture slot we want to use and sample from.

|**_Task 9:_**|
|:---|
|Set the uniform value for your sampler2D you created in your fragment shader|

### 2.3. Using A Texture In The GPU: Fullscreen Quads and UV coordinates

[Shaders]

#### 2.3.1 Fullscreen Quads

If you recall the OpenGL coordinate system, we can see the limits of the screen as follows:

What we can do with this is use it to construct what is known as a fullscreen quad. Think of a fullscreen quad as a projector screen that we drape down in our scene that happens to be just the right size to cover the entire screen so we can’t see behind it but also can see it in its entirety.

[ give them a VAO/VBO thing that sets up a 0.8-0.8 fullscreen quad ]

|**_Task 10:_**|
|:---|
|Edit m_fullscreen_vbo's data to have appropriately placed vertices to cover the screen|

[ at this point, the fragment shader should just be printing some color different from the background ]

|**_Task 11:_**|
|:---|
|Mess around with the m_fullscreen_vbo data. In particular, try moving the z values to different negative values and see what happens!|

#### 2.3.2 UV Coordinates

Great! Now we have the shape on which we’ll plaster our texture, but how do we (texture-)map the image to the surface? In steps a new vertex attribute: UV coordinates! 

The UV attribute tells us at what point in the sampled texture should each vertex correspond to. The lower left corner is set to be (0, 0) and the upper right corner as (1, 1) in an image. 

|**_Task 12:_**|
|:---|
|Pick 6 corresponding UV coordinates to pair with each vertex position you choose which would allow a texture to span across the entire fullscreen quad. That is, the bottom left corner of the fullscreen quad should correlate with the bottom left corner of the texture and the upper right corner of the fullscreen quad correlates with the upper right corner of the texture|

|**_Task 13:_**|
|:---|
|Generate a VBO/VAO pair to store our fullscreen quad in which the UV coordinates are only 2 consecutive floats but the positions remain as 3 consecutive floats|

|**_Task 14:_**|
|:---|
|Add texture UV and position layout variables to the fbo.vert shader|

|**_Task 15:_**|
|:---|
|Create an in/out variable pair to pass the UV coordinates from the vertex to the fragment shader|

#### 2.3.3 Sampling a Texture In A Shader

Now we want to set our fragment color to be our texture at our pre-selected UV coordinates. In GLSL, the function to sample a texture uniform which is of type sampler2D is:

`vec4 texture(sampler2D sampler, vec2 UV)`

<details>
  <summary>vec4:</summary>
</details>
<details>
  <summary>sampler2D:</summary>
</details>
<details>
  <summary>vec2 UV:</summary>
</details>

|**_Task 16:_**|
|:---|
|Set the fragment color based on the texture color at the fragment’s UV coordinate|

At this stage, you should see an image of a cat covering your application window!


## 3 FRAME BUFFER OBJECTS

### 3.1 What Is an FBO?

This is the general definition:
> A framebuffer is a portion of memory containing bitmaps that can drive displays.


FRAME + BUFFER
Buffer = data
Frame = canvas, rgba, originally for the “next/current frame”

OBJECT
Contains

### 3.2 What does an FBO contain?

Attached buffers, which are either texture or renderbuffer. What’s the difference?

Collapsible section: caveats: depth

### 3.3 What does an FBO’s attached buffer contain?

Have seen fragColor before, know about color buffer. Unknown is depth and stencil which have been behind the scenes.
This collection of information (color, depth, and stencil) 

### 3.4 Tasks

Group together tasks in order to create a framebuffer with attachments and sample it’s color attachment as a texture

## 4 Post Processing

Have students write these as functions in their shader program

### 4.1 Per-Pixel Operations: Greyscale

Seen this before in filter, should be simplistic

### 4.2 Kernel Operations: Box-Blur

Kernels in a SHADER? :O

