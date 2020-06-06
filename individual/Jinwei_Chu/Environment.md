### Modifying the Floor
The default RectangleArena PROTO defines a simple floor pinned on the static environment, i.e. without Physics node, and surrounded by walls. Other pre-built floors are available in the Webots objects library. We will now delete the RectangleArena node and add a simple floor that we will manually surround with walls later in this tutorial. 

To remove the RectangleArena, select it either in the 3D view or in the scene tree view with a left click and press the Delete key on your keyboard. Alternatively, you can right click on it in the 3D view and select Delete in the context menu (you can also use the context menu directly in the scene tree view). Select the TexturedBackgroundLight node and click on the Add button. In the open dialog box, and choose PROTO nodes (Webots Projects) / objects / floors / Floor (Solid).

The newly added Floor PROTO has a default size of 10mx10m, but it is possible to adjust its size, its position and texture by changing the corresponding fields.

**However, during the process of development, the color and the texture of th floor is required to be modified. In the end, I decided to package several solid boxes whose color and the texture can be modified easily together to form a whole floor.**

### The Solid Node
This subsection introduces the most important base node in Webots: the Solid node, from which many other nodes derive.

The physics engine of Webots is designed for simulating rigid bodies only. An important step, when designing a simulation, is to break up the various entities into separate rigid bodies.

To define a rigid body, you will have to create a Solid node. Inside this node you will set up different sub-nodes corresponding to the characteristics of the rigid body. The following figure depicts a rigid body and its sub-nodes. The graphical representation of the Solid node is defined by the Shape nodes populating its children list. The collision bounds are defined in its boundingObject field. The graphical representation and the collision shape are often but not necessarily identical. Finally, the physics field defines if the object belongs to the dynamical or to the static environment. All these sub-nodes are optional, but the physics field needs the boundingObject to be defined.

### DEF-USE Mechanism
The DEF-USE mechanism allows to define a node in one place and to reuse that definition elsewhere in the scene tree. This is useful to avoid the duplication of identical nodes in world files. Moreover, it also allows users to modify several objects at the same time. Here is how it works: first a node is labeled with a DEF string. Then copies of this node can be reused elsewhere with the USE keyword. Only the fields of the DEF node can be edited, the fields of the USE inherit from the DEF node and cannot be changed. This mechanism is dependent on the order of the nodes in the world file. A DEF node should be defined before any corresponding USE node.

The two Sphere definitions that we have used earlier to define the ball, are redundant. We will now merge these two Spheres into only once using the DEF-USE mechanism.

### Add Walls
In order to verify your progression, implement by yourself four walls to surround the environment. The walls have to be defined statically to the environment. To understand the difference between static and dynamic, let's take a defined object (the ball) above the ground. If the Physics node is NULL, it will remain frozen in the air during the simulation (static case). If the physics field contains a Physics nodes, it will fall under the effect of gravity (dynamic case).

Use as much as possible the DEF-USE mechanism at the Shape level rather than at the Geometry level. Indeed it's more convenient to add an intermediate Shape node in the boundingObject field of the Solid node. The best Geometry primitive to implement the walls is the Box node. Only one Shape has to be defined for all the walls. The expected result is shown in this figure.

### Lights
The lighting of a world is determined by Light nodes and the Background . There are three types of light nodes: the DirectionalLight, the PointLight and the SpotLight. A DirectionalLight simulates a light which is infinitely far (ex: the sun), a PointLight simulates light emitted from a single point (ex: a light bulb), and a SpotLight simulates a conical light (ex: a flashlight). This figure shows a comparison between them. Each type of light node can cast shadows. You can find their complete documentation in the Reference Manual.

Lights are costly in term of performance and reduce the simulation speed (especially when they cast shadows). Minimizing the number of lights increases the rendering speed. A PointLight is more efficient than a SpotLight, but less than a DirectionalLight.

In this simulation, the Light node is not visible in the scene tree because it is contained in the TexturedBackgroundLight PROTO node. It consists of a DirectionalLight whose intensity and direction is computed automatically according to the background of the scene.

### Texture
The texture URLs must be defined either relative to the worlds directory of your project directory or relative to the default project directory WEBOTS_HOME/projects/default/worlds. In the default project directory you will find textures that are available for every world.

Open the red_brick_wall.jpg texture in an image viewer while you observe how it is mapped onto the Sphere node in Webots.

Textures are mapped onto Geometry nodes according to predefined UV mapping functions described in the Reference Manual. A UV mapping function maps a 2D image representation to a 3D model.


