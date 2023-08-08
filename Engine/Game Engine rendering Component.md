# Game Engine rendering Component

### Foundations of depth-buffered triangle rasterization

- A list of component of a 3D scene
  - Virtual scene description: A set of 3D surfaces vectors
  - Visual properties for each surfaces, defines color, reflective, transparency, texture and other properties (shaders) 
  - Virtual camera: Placed within the scene, a focal point, uses a virtual light censor to capture the pixels information onto the near plane, The camera can be moved and rotated
  - A set of light source, provides light ray which reflects of surface in the environment, eventually captured by the virtual camera
  - Solve the shading equation for each pixel on the near plane, calculate the color + intensity of light on each pixel
- rendering engine takes all component into consideration, generate images at high frame rate (30+) to provide the illusion of motion

#### Describing a scene

- 3 types of objects in the scene
  - Opaque: Light completely blocked
  - Transparent: Light pass through without being scattered
  - Translucent: Light pass through scattered in all direction, only a blur of color is showing
    - Where light is not directly reflected
- Transparent + translucent most of the time does not use the interior structure + properties of the object
  - Treated the same way as the opaque object with a lower "alpha" (numeric opacity) value
- Most game engine only deal with surface

##### Representation used by high-end rendering packages

- Some simple, arbitrary shapes can be represented by parametric surface equation
- The film industry represent with rectangular patches
	- 2D surface segment, edges represent by splines, controlled by # of control points
	- Bezier surface: Just like how points interpolate Bezier curves, Bezier surface is create using interpolating between multiple Bezier curves in 3D space
		- Using a grid of control point

	- nonuniform rational B-splines: Also known as NURBS, a mathematical ways of using equation to produce ultra fine splines, and in extension, surfaces
		- The spline is controlled by control point and a the degree of polynomial of the equation. E ach control point has a weight and can fine tune the curve.

	- Bezier triangles: An extension of Bezier surface, surface in 3D is defined over triangular domain
	- N-patches: Using multiple patches to create complex surfaces, using N Bezier + NURB to create surface

- Covering models with tiny rectangles
- Film rendering engine has the ability to sub-divide mesh of control points into smaller polygons, until each polygon is smaller than a pixel, to complete smooth out surface

##### Triangle meshes

- Game usually use triangle meshes to represent surfaces
	- A chain of connected line segment to approximate curve
- Triangle is the polygon of choice for the following reason
	- Simplest type of polygon, require the least amount vertices
	- Triangle is always flat
	- When triangle transforms, it will stay as a triangle / line segment
	- All GPU are all designed + optimized for triangle rasterization

###### Tessellation

- Divide surface up into a collection of triangles
- If the tessellation is fixed, it can cause silhouette edges, ultimately make the object look blocky
	- The solution is to use dynamic tessellation, keep to consistent triangle-to-pixel density
		- Nanite system does the opposite, it reduce triangle count by edge collapse as distance increase 


###### Silhouette edges

- Silhouette edges appear when 2 triangle shares the same edge + 1 faces the viewport + the other faces away from it
	- Silhouette can be determined using the normal of the triangles + measure the angle towards the view vector
		- If the dot product is positive, then facing towards the viewport
- LODs (level of details): Creating a chain of alternate version of the triangle angle, create a approximate dynamic tessellation
	- LOD0: the highest level of tessellation
	- Subsequent LODs will have lower resolution + triangles counts
	- Allow the game engine to focus lighting + transformation of object close to the camera
- Dynamic tessellation: Usually applied to large, expansive meshes (Terrain, water) The region of meshes that are close to the camera are displayed with full resolution, farther away region are tessellated with less
- Progressive meshes: Start with a high-resolution mesh, object then gets automatically DE tessellated as it gets further away by collapsing edges
	- Nanite in addition has the ability to keep the outer boundary of the cluster to be the same through upward recursion
		- The goal of nanite is to achieve constant-time rendering by only render 2 * the pixel amount of triangles for each frame

##### Constructing a triangle mesh

###### Winding order

- Triangles are defined by 3 vertices, with 3 edges connect connecting the vertices together
	- Subtract the position vector of adjacent vertices
- The normal of the triangle is defined by the cross product of any 2 edges (normalized)
- Winding order: The order of triangle's vertices, determines the front + back of the triangle
	- Most graphics can use winding order to determine which side of triangle does not need to be rendered (cull)
		- Only 1 surface of the triangle will be rendered
- Both winding order are okay, as long as consistent

###### Triangle lists

- Lists the vertices in group of 3, each represent a triangle
- The simplest way to represent a surface of mesh

###### Indexed triangle lists

- A more efficient triangle list, store all vertices in a list, and use the index in the list to represent each vertices
	- A vertex buffer (DirectX) / vertex array (OpenGL) + index buffer + index array

###### Strips + fans

- Specialized mesh data structure remove the need of triangle lists
- Pre-define the order of vertices, to form triangles
- Strips: The first 3 vertices form a new triangle, every new vertices form a triangle with the previous 2 vertices
	- Kind of like a Fibonacci without the sum
- Fan: The first 3 vertices form a new triangle, every new vertices form a triangle with the previous vertex + the first vertex

###### Vertex cache optimization

- Triangle can use any vertex within the vertex buffer
- The vertices must be processed in the order they appear, to preserve the shape of the triangle
	- in group of 3

- The GPU can cache some most recently processed vertex for re-use
	- Faster access time
	- Fans + strips benefit from re-using vertex from previous calculations, improve cache coherency (access hit rate)
		- Strip + fan can be indexed too, to avoid duplications

- Normal indexed triangle can also benefit from vertex cache optimization
	- Change the order of the triangles, optimize vertex re-use within the cache


##### Model space

- Model space (local space / object space): A local coordinate system, provide position vectors for vertices
	- The original is either at the center of the object / some convenient locations
	- Axes typically align with front, right + up direction of the object
		- `x,y,z` axes -> 3 basis unit vector `i,j,k`
			- Commonly, `L = i`, `U = j`, `F = k`
		- Arbitrary as long as they are consistent throughout the engine

##### World space + mesh instancing

- World space: The common coordinate system for each level
- Mesh instance: Each object that exist in the world space.
  - Each mesh instances reference to a some shared data
- World matrix (modul-to-world-matrix): A transformation matrix included with every instance, transform object from model space -> world space
  - A 4 by 4 transformation matrix, Each column represent the Rotation + scale transformation of each directional vector
  - The 4 column represent the translation offset
- When apply the world matrix to surface normal, use the inverse to remove the effect of scaling
- Some meshes will only have world axis, they will be static + unique in the level + does not needs to the transformed

#### Describing the visual properties of a surface

- 3 sections of visual properties
	- Geometric information: Surface normal of various point on the surface
	- Lighting information: How light interact with the surface
	- Surface change over time: How the water body moves, how to interact with joint, etc.

##### Introduction of light + color

- Electromagnetic radiation, wave + particles
- The color of light is determined by intensity + wave length / frequency
	- Visible light is in between 740nm to 380nm (Spectral colors)
- Light can be single color / mix of colored light
	- Represented by a spectral plot
		- White light has box-shaped plot, bit of every color

###### Light-object interactions

- Light behavior is determined by the medium (the space light travels through) + interface (shape + properties of the surface)
- Light in engine have 3 different behavior, absorbed, reflected + transmitted (refracted) in the process
	- Light also diffracted, but often un-noticeable in most scene
- Certain wavelength gets absorbed, other gets reflected
- Reflection diffuse: Incoming ray can scatter, and spread in all directions
- Reflection specular: Light ray will only reflect directly / spread in narrow area
- Reflection anisotropic: Lights could be diffused / specular depends on the angle of the surface
- Light travelling through volume can scatter (subsurface scattering), partially absorbed, re-fracted (prism)
	- Refraction depends on the frequency of the light
	- subsurface scattering is commonly used on skin, wax to create a warm appearance

###### Color spaces + color models

- RGB is the most common color models
	- 3 channels, each range from  0 - 255 (RGB888) / RGB565
- Other color model is also available, log-LUV used to HDR lighting

###### Opacity + the alpha channel (RGBA)

- `alpha` measures the opacity of the pixel
- RGBA8888 is the most common, 8 bits for the the opacity channel

##### Vertex attributes

- Surface properties are stored on discrete points on the surface, vertices of the mesh (vertex attributes)
- Each vertex include the following attributes, more could be added per vertex
	- Position vector: 3D position of the vertex in local space
	- Vertex normal: 3D vector, Define the unit surface normal at the vertex, typically used for lighting / rendering calculation
	- Vertex tangent: 2 3D vector, 2 vectors perpendicular to each other + the normal (tangent + bitangent)
		- Vertex tangent + vertex normal = tangent space (X, Y, Z axes)
		- Used to calculate normal mapping + environment mapping
	- Diffuse color: 4D vector, The color of the surface, RGB + alpha channel
		- Color can be calculated at run time (dynamic lighting) / offline (static lighting)
	- Specular color: 4D vector, represent the color of the surface when a light directly reflect from the surface to the camera's near plane
	- Texture coordinates (UV coordinates): 2D vector, Represent a 2D coordinate space on the texture, wrap the texture around the 3D space
		- Object can be wrapped with more than 1 textures, each vertex can have multiple texture coordinates
	- Skinning weights: 2D vector, Each vertex specifies which joints its attached to
		- Each vertex can be affected by multiple joint, position of the vertex depends on the average weight of the joints

##### Vertex formats

- The vertex attributes data structure, usually as `struct` or `class`
- Different type of meshes require different vertex format
	- For simple calculation (Shadow volume, silhouette edge detection, cartoon rendering), only the position of the vertex is needed
	- Typical vertex contains location info, normal info + texture coordinate
	- Skinned vertex typically have position info, color info (diffuse + specular), one or more texture coordinates, joints weight and joints location
- In real world, many vertex formats are not useful / not practical, does not fit with modern hardware + software
	- Game developers can also limit the amount of available properties for meshes
	- Limit the # of weight per vertex
	- Some engine let the hardware select the relevant attribute base on the shaders

##### Attribute interpolation

- Vertices attribute are just approximation of the visual properties of the surface
	- Visual properties of the interior pixel ultimately displays on the screen
- Achieved through linear interpolate between vertex, smooth value transition (LERP)
	- Known as Gouraud shading

###### Vertex normal + smoothing

- Lighting essentially calculate the color of an object per pixel
	- Use surface properties + lighting information to calculate diffuse color at each vertex, then use Gouraud shading to apply the vertex color across the surface
	- Use vertex normal to calculate the impact of the light ray, vertex normal will have huge impact on the final rendering
- If the normal points outwards directly (perpendicular to the surface), then the surface will appear flat + sharp edges, light direction changes abruptly at the edge
- If the normal points out from the edge of the mesh, the lighting of box will be smoother like curves surface, the change in lighting is smoother when move across surface anomalies

##### Textures

- Large triangle does not look nice with vertex basis lighting calculation, can lead to undesirable visual
	- Especially when tessellation is tessellation (the shapes that covers a plane with no gap) density is low
- Solution: Use texture maps
	- Usually contain color information to be applied to triangles in a mesh
	- Texture does not have to applied directly to the mesh, each `texels` (texture pixels) can be used as data table to calculate pixels on screen
	- Texture dimension should always be power of 2, variations between game engine

###### Types of textures

- Diffuse map (albedo map): Describe the diffuse color of each Texel, can be applied like decals onto the surface
- Normal map: Describe the vector normal of each pixel, in RGB channel (Bumps)
- Gloss map: Describe the level of shininess of the mesh at each pixel (Roughness)
- Environment maps: Contain picture of surrounding environment for rendering
- Texture could be used to store any information for lighting calculations

###### Texture coordinates

- Use texture space (a 2D coordinate system) to project texture onto mesh
	- Pair of float value range from 0 - 1 (`u,v`)
- Specify a texture coordinate for each vertex of the mesh, map the triangle to plane in texture space

###### Texture addressing modes

- Texture coordinate can go beyond the 0 - 1 range, different engine will have 1 or more ways of handling that
	- Wrap: Texture will be repeated in all direction
	- Mirror: Similar to wrap mode, except each new tiles are mirrored, mirror on `v-axis` when `u` is odd + `u-axis` when `v` is odd
	- Clamp: Color of the Texel on the edges extends out
	- Border Color: Similar to clamp, except a user-specified color is used to fill the surrounding that are outside the texture coordinate range

###### Texture formats

- Texture can be any image format as long as it can be decoded + stored into memory by the engine
	- `.tga`, `.png`, `.bmp`, `.tip`, etc.
- Usually a 2D array, pixel location + color in different color format
- Most modern graphics cards + API supports compressed textures
	- Break textures into small chunks + represent chunk with less color pallete
	- Faster to render (More texture can fit into cache at once) + smaller in memory

###### Texel density + mipmapping

- Texel density: The ratio of texel to pixel, not a fixed quantity
	- equals to 1 when each texel is matched by 1 pixel
	- Greater than 1 when more than 1 texel is used by a pixel, could create the `Moire banding pattern` / flickering due to different texel taking pixel dominance, waste of memory
	- Lesser than 1 when each texel contribute to more than 1 pixel, able to see the edge of texel + ruin the illusion
- Affects memory consumption + visual quality, ideally want to keep the texel density at 1 at all time
	- Could be approximated via mipmapping
- mipmapping: Each texture has a sequence of lower resolution bitmaps, each with 1/2 the width + height
	- Graphics hardware can automatically select the appropriate version of mip level base on distance from the camera

###### World-space texel density

- Ratio of texel to world-space area on a textured surface
- Does not have to be 1, but should be consistent, especially on the same mesh, quite noticeable to the player

###### Texture filtering

- The graphics hardware sample the texture map + determined where the pixel center lands on the texture space
	- Not a 1-to-1 mapping, could even be in between 2 texel
- Texture filtering: Graphics hardware usually sample more than 1 texel, blend the result + output to the pixel
- Mostly 4 types of texture filtering supported by graphics card
	- Nearest neighbor: The brute force approach. Select the texel that is closest to the center of the pixel + select the mip level closest + greater than the ideal resolution needed to achieve screen-space texel density of 1
	- Bilinear: The weighted average between 4 texels surrounding the center of the pixel, the weight depends on the distance of each texel to the pixel center, use the same mip level selection process as nearest neighbor
	- Trilinear: Apply bilinear filtering between the higher res + lower res of mip level + linearly interpolate the result, eliminate visual boundaries between different mip levels
	- Anisotropic: Bilinear + trilinear filtering a 2 * 2 square blocks of texels, this only works when looking at the surface direct ahead, incorrect when the surface is at angle from the screen plane

##### Materials

- A higher level wrap for the texture
	- Specify the texture used on the surface
	- Define which shader program to use
		- Define some of the parameters for the shader
	- Control the graphics card itself
- Mesh-material pair: All the information needed to render an object, render packets (geometric primitive)
- Each mesh may have multiple different materials, divide mesh up to smaller mesh to apply material individually

#### Lighting basics

- Shading: Include both lighting calculation + other visual effects
	- i.e. Water movement, tessellation of high-order surface, calculations to render a scene

##### Local + global illumination models

- Light transport models: Series of light-surface + light-volume models interacting with each other
- Direct lighting (local illumination models): The simplest model, light bounces once from a single surface to the imaging plane of the virtual camera
	- Only considering the lighting effect from the source, does not affect other nearby objects
- Indirect lighting (Global illumination model): Light can bounce off multiple surfaces before reaching the imaging plane
	- Some models are specialized in certain areas (shadows, reflective surfaces, interreflection, etc.), some are more generalized + account for everything (ray-tracing)
- Global illumination model has be solved mathematically back in 1986, every model attempts to partially + fully solve the equation

##### The Phong lighting model

- The most common local lighting model
- Light have 3 component to be considered
  - Ambient: The overall lighting level of the scene, approximate the amount of indirect light in the scene
  	- Make shadows not completely dark
  - Diffuse: How light from the source reflect in all directions when hit the surface, High diffuse result in a matte, non-shiny appearance
  - Specular: The bright highlight on a glossy surface, result by the direct reflection from the light source
- Each component can be considered its own image, combined them together to create the final intensity
- Phong lighting model requires several input parameters, usually in 3D vector (R,G,B)
  - Viewing direction vector: Normalized direction from the reflection point to camera's focal point
  - Ambient light intensity: light intensity in 3 color channel
  - Surface normal:
  - Surface reflectance properties, include ambient, diffuse, specular reflectivity + a glossiness exponent
  - Light source properties, determine light color + intensity + a direction vector
- The intensity of the light at any point is determined by the following
	- The intensity of the ambient light * the reflectivity of the surface
	- For every light source, the diffuse of the light on the surface (Determined by the angle of the light, normal of the surface + diffuse reflectivity) * light intensity + color
	- For every light source, the specular of the light on the surface (Determined by the Viewing, light angle in reverse + specular reflectivity) * light intensity + color

- The calculation is performed on each RGB channel
	- Note the reflected light has the same normal but opposite tangential component, therefore, needs to be re-calculated


###### Blinn-Phong

- Variation of the Phong model, but changes how specular reflection are calculated
	- Takes the half way point between the angle to camera + angle of the reflection, calculate the specular reflection in that direction
	- Use the surface normal + half way vector to approximate the light direction

###### BRDF plots

- 3 components the Phong model uses are special case of general local reflection model
	- Bidirectional reflection distribution function (BRDF)
- The diffuse reflection will stay constant regardless the viewing angle
- The specular reflection will create a "hot spot" effect, when the reflection angle + viewing angle aligns up closely, but also falls off quickly when angle diverges

##### Modeling light sources

- Describe the sources light in the scene using simplified models

###### Static lights

- Light pre-calculation, calculated offline
- Pre-computer Phong lighting reflection model, store as vertex color attributes
	- Could be later translated into a light map, applied additionally to texture of objects
		- Compare to baking light information directly into the textureUsing light map create more flexibility, ability to calculate light map for each light source separately + are easier to compress
			- An hybrid between texture baking + light map could be considered
	- However, static light is baked in to the level, does not change when the levels / lighting changes

###### Ambient lights

- An universal lighting value applied to all objects in level
	- Single color, scaled by surface ambient reflectivity at run-time
	- Can be divided into regions, different regions might have different intensity + color

###### Directional lights

- Light source that are infinite distance away from the surface
	- All light rays are parallel in direction + does not have original starting location in level + usually are single color

###### Point lights (omnidirectional)

- Located in level, radiates uniformly in all direction
	- Light falloff with square of distance from the light source
	- Has a defined max length + clamp to 0 after the max distance
	- has 3 variables, max distance, light color/intensity + world position
	- Points light is only applied to object within the max distance

###### Spot lights

- Located in level, the light is restricted to a cone-shaped region
	- Define the cone with inner + outer angles
		- Inner cone has full light intensity, intensity decal between inner + outer angles + intensity drops to 0 outside the outer cone
		- The same light intensity distance fall off applies

###### Area lights

- In real life, light source has non-zero area
	- Create the illusion of area lights with casting multiple shadows + blur some of the sharp edges

###### Emissive objects

- Some materials can emit light + become light source
	- Created using emissive texture map
	- Texture with full light intensity even without light sources
- Any emissive object can use combination of multiple light source + multiple techniques

#### The virtual camera

- An ideal focal point + rectangular virtual sensing surface (imaging rectangle)
	- Imaging rectangle has multiple light censors, represent each pixel on screen.
		- Every rendered frame determine color + intensity of each light censor (each pixel)

##### View space

- Camera focal point is the original in view space
- Z-axis usually represent looking up / down, y-axis represent represent forward / back + x-axis represent left / right
	- Z-axis determine how close the imaging plane is to the focal point
- Camera position + rotation can be translated to the world space using a view-to-world matrix, similar to model-to-world matrix
- Triangle mesh's model space -> world space -> view space (Using the world-to-view matrix), inverse V-T-W matrix
- Model-to-view matrix can be pre-calculate the combination of world-to-view matrix, form a model-view matrix (in OpenGL) for faster calculation

##### Projections

- Render 3D scene onto 2D imaging plane
- Perspective projection: mimics images produced by IRL camera, objects that are further appear smaller
- Orthographic projection: Length preserving at any distance, usually provide the player with a different perspective + level design editing process

##### The view volume + the frustum

- View volume: The space the camera can see, defined by 6 planes
	- Near plane: The imaging sensing surface
	- Far plane: Used for rendering optimization, any object beyond the far plane will not be rendered
	- 4 side planes: formed by connection the edges of near plane + far plane
- Perspective projection's planes will form a pyramid shape (frustum)
- Orthographic projection planes will form a rectangular prism
- Planes can be represented with either plane normal + distance from origin / point plane on plane + plane normal

##### Projection + homogeneous clip space

- Homogeneous clip space: A warped version of view space, used to convert camera-space view volume to a canonical (standard) view volume
	- Independent from aspect ratio + resolution + projection used
	- Clip space ranges from (-1, 1) on all axis, a 2 unit cube

- Convenient to clip triangles on the edge of the space
- In the form of left hand axis system, Increase in Z-axis increase depth into the screen

###### Perspective projection

- The matrix used to perform transformation between view space -> homogeneous clip space
- 

509-516