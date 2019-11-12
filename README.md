# CARVR - Plane Flying & Sound Effects

In this 2-part-workshop, we will create a plane that flies while constrained by the camera, with matching sound and visual effects.

We will take use of Unity's ViewPort mechanism, which allows us to restrict our movement through only what the camera can see, effectively only letting the plane fly in certain areas. We will also add breaking and accelerating to the project, and script in the visual and sound effects to match the scenary.

### What this Workshop covers
    1. Unity Basics (brushing over)
    2. Piloting system in C#
    3. Camera movement coupled to Player's
    4. Adding SFX and VFX to the Scene.

### What you will need: 
    1. Unity Game Engine
    2. VS-Code / Visual Studio
    3. Mouse

---

## **Unity Basics**


### The Unity Interface

![](/imgs/Interface.png)

#### The Hierarchy Window
The Hierarchy Window shows a list of all game objects in the scene, including game objects, camera, etc.

#### The Project Window
The project window displays all assets we have available to build the game with, such as meshes, scripts, materials.

#### The Inspector Window
The Inspector Window is a context sensitive window which shows us the properties of any object we want to see. It also lets us add components to individual game objects.

#### The Scene View
The Scene View is where we are going to visually build the game. It allow us to view the layout before and after running the game.

#### The Game View
The Game View allows us to preview our game in the editor. 

---

### **Creating the Plane** 

Start off by creating two Cubes. One of them will be the "Wings", and one will be the "Cockpit" of the plane. We're making this simple to start off, but we could create a full mesh in Blender / Maya to insert into the scene.

Make sure both Cubes have their Transform reset to (0, 0, 0) on both Position and Rotation and their scale is set to (1, 1, 1) before you start.

The Wings should have a scale of (3, .1 and .8), while the Cockpit can just be a regular 1x1x1 cube.

To make it easier on the rest of the tutorial, we wlil create a empty GameObject called "Player" and we're parenting down the Plane as follows:

Player > Cockpit > Wing

![](/imgs/Parenting.png)

---

### **Scripting**

Let's start off by making our Plane move.

## Unity Movement System via Axis

Unity uses a Axis movement system imbued into it's code (which can be viewed AND edited at Project Settings --> Input). This Axis movement is tied to a WASD movement=style, where each letter represents a key on the keyboard:

- W --> up
- S --> down
- A --> left
- D --> right

Because of that, we can create a new script attached to the "Player" GameObject called `PlayerController.cs`

Since we're using this method to control movement, and we know that movement goes into the **Update** function, we will declare two variables into Update calling Axis movements.

> Variable of type Float called "h" for an Input on Axis "Horizontal"
> Variable of type Float called "v" for an Input on Axis "Vertical"


Let's also add some movement to the Player. Now, we will call another Method called **LocalMove** (and we will write it) which will move our Player.
**LocalMove** must contain 3 parameters: Both Horizontal and Vertical movement input, and the speed of the movement.

So, under the h and v variables, we call:
`LocalMove(parameter 1, parameter 2, parameter 3);`

# Now to write LocalMove:

**private void LocalMove(float x, float y, float speed)**

Inside LocalMove, we will first change the local position of the Player every Time.deltaTime to position x and y with a certain speed.

**tranform.localPosition += new Vector3(?, ?, ?) * a certain speed * Time.deltaTime;**

Go test your game and see if your plane flies around! 
If it's too fast or too slow, change the speed factor.

## Making the ship's movement natural

Right now the Plane moves like a rigid object, and that's not how planes (at least our Plane doesn't) fly around.

So we will add some rotation to match it's movement, and for that we will use a Sphere that will act sort of like a Joystick to our Plane.

First of all, go back to the scene and create two GameObjects: an empty one, called AimParent, and a (1, 1, 1) sphere called AimSphere.
Don't forget to Reset their transforms.

After you parented AimSphere as a child of AimParent, move AimSphere 3 units in the Z axis in front of the ship, as such:

![](/imgs/Sphering.png)

Beneath our **LocalMove** in Update, add a new **RotationLook** method that we will write beneath.

RotationLook also must have 3 parameters, since it will be used to move the Sphere: Both Horizontal and Vertical movement input and speed of movement (different from the Plane's)

`RotationLook(parameter 1, parameter 2, parameter 3);`

Don't forget to also add a variable for the Sphere in the beggining of the script:

`public Transform aimSphere;`

For **RotationLook** method, we will first need to make sure the parent of the sphere is always in the same spot, so it doesn't move around:

`aimSphere.parent.localPosition = Vector3.zero;`

We also need to add movement to the Sphere via `aimSphere.localPosition = new Vector3(parameter, parameter, parameter);`.
This movement is guided through the "h" and "v" variables, plus a certain distance from the Plane (we created the sphere to be 3 units away from (0, 0, 0) wink wink)

Lastly, we will change the rotation of the Plane itself to match the sphere with this line:

`transform.rotation = Quaternion.RotateTowards(transform.rotation, Quaternion.LookRotation(aimSphere.position), Mathf.Deg2Rad * moveSpeed * Time.deltaTime);`, where moveSpeed is a float that you have to declare and will judge how quickly the sphere moves on the screen.


## Adding HorizontalLean

To make the plane a tad more realistic, we will also add HorizontalLean, through the lines:

`private void HorizontalLean(Transform target, float axis, int leanLimit, float lerpTime)
    {
        Vector3 targetEulerAngles = target.localEulerAngles;
        target.localEulerAngles = new Vector3(targetEulerAngles.x, targetEulerAngles.y, Mathf.LerpAngle(targetEulerAngles.z, -axis * leanLimit, lerpTime));
    }`

where **leanLimit** is how much the plane can lean sideways before getting "stuck", while **lerpTime** is how long the leaning takes.


## Clamping the Movement

Right now the ship can fly anywhere, and that's not what we want. We want the ship to be "stuck" inside the camera view. For that, we will use ClampPosition(), a method created by us. Add it on the last line of the LocalMove method.

ClampPosition() will firstly transform the Worldspace position into a Viewport position, which is what the camera can see
Then we will Clamp01 the position between [0 and 1] so that movement gets stuck inside the camera.
Finally we will return the Viewport to Worldspace so that the ship can fly on the world while still being constrained by the camera.
Code is as follows:

`Vector3 pos = mainCamera.WorldToViewportPoint(transform.position);
 pos.x = Mathf.Clamp01(pos.x);
 pos.y = Mathf.Clamp01(pos.y);
 transform.position = mainCamera.ViewportToWorldPoint(pos);`

## Camera Controlling

Create an empty GameObject to hold the Camera called "CameraParent" and child the Main Camera onto it.
Then, also child the AimParent (and consequentially the AimSphere) inside CameraParent, as such:

![](/imgs/Finishing.png)

Now we will create our last script, to control the camera to go after the Plane while it's flying.

This will be fairly simple script compared to our previous ones, so you can try doing it on your own.

If you feel like you need a hand, here's a "hacked" version of that script:

```
public class cameraController : MonoBehaviour
{
    // Declare a GameObject variable for the Player.

    // Declare a Vector2 called limits for where your camera can move.
    
    // Declare a Vector3 called velocity that's equal to (0, 0, 0).

    // Call a LateUpdate function
    void LateUpdate()
    {
        // Call a new variable called "localPos", a Vector3 that is located on the Camera's current position
        
        // Use this following script to make the movement smooth:
        transform.position = Vector3.SmoothDamp(localPos, new Vector3(localPos.x, localPos.y, playerTarget.transform.position.z), ref velocity, 0.5f);
    }
}
```

## Lets Get Coding

