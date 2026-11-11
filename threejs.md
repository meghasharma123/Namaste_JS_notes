# Basics

Three.js is a JavaScript 3D graphics library built on top of `WebGL` (and it also supports WebGPU).

## 5 important building blocks

1. `Scene`: It holds all your 3D objects, lights, cameras and other scene elements.
``` const scene = new THREE.Scene(); ```

2. The `camera` decides what the user sees.
```
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
```

3. `Renderer` takes the scene and camera and displays the result on the screen using the graphics API.
```
const renderer = new THREE.WebGLRenderer({
  antialias: true
});

renderer.setSize(
  window.innerWidth,
  window.innerHeight
);

document.body.appendChild(renderer.domElement);
```

4. `Geometry` defines the shape of an object.
```
const geometry = new THREE.BoxGeometry(1, 1, 1);
```

5. `Material` defines how the surface looks: color, roughness, metalness, texture, etc.
```
const material = new THREE.MeshStandardMaterial({
  color: 0x00ff00,
  roughness: 0.5
});
```

- A 3D object is commonly created using Geometry + Material = `Mesh`.
```
const mesh = new THREE.Mesh(geometry, material);
scene.add(mesh);
```

**Finally: Render the scene**
```renderer.render(scene, camera);```

## Main architecture

`Scene` [Contains objects and lights] ->  `Camera` [Defines the viewpoint] -> `Renderer` [Draws the scene using WebGL/WebGPU] -> `Canvas` / Browser [Displays the 3D result]

- Every 3D object has three important transformation properties:
```
mesh.position.set(2, 1, 0);
mesh.rotation.y = Math.PI / 2;
mesh.scale.set(2, 2, 2);
```

position = location.
rotation = orientation.
scale = size.

Three.js rotation values are in radians, not degrees.
- 90° = Math.PI / 2
- 180° = Math.PI
- 360° = Math.PI * 2

## Types of camera

1. `PerspectiveCamera`: Objects farther away appear smaller. This creates a realistic, human-eye-like view. Example: A 3D robot simulator where you want depth and realistic perspective.
2. `OrthographicCamera`: Objects maintain the same apparent size regardless of distance. Parallel lines remain parallel. Example: CAD software, engineering drawings, or a top-down robot layout.

## Types of lights

1. `AmbientLight`: Provides general illumination to the whole scene. It doesn't create directional shadows
2. `DirectionalLight`: Light comes from a particular direction, similar to sunlight. Useful for creating shadows.
3. `PointLight`: Emits light in all directions from a single position, like a light bulb.
4. `SpotLight`: Emits light in a cone-shaped direction, like a flashlight.

## Types of materials

1. `MeshBasicMaterial`: Doesn't react to lighting. It displays its color directly.
2. `MeshStandardMaterial`: Reacts to lighting and supports realistic surface properties such as roughness and metalness.

```
const material = new THREE.MeshStandardMaterial({
  color: 0x888888,
  roughness: 0.4,
  metalness: 0.8
});
```

## Animation Loop

A 3D scene doesn't automatically update continuously when an object's position or rotation changes. We need an animation loop to update and render the scene repeatedly.

```
function animate() {
  requestAnimationFrame(animate);

  robot.rotation.y += 0.01;

  renderer.render(scene, camera);
}

animate();
```

What's happening?
1. requestAnimationFrame(animate) schedules the next frame.
2. We update the robot's rotation.
3. renderer.render() draws the updated scene.
4. This repeats, creating animation.

`Delta time`: `It makes movement independent of frame rate.` Suppose one computer renders 60 frames per second and another renders 120. If you rotate an object by 0.01 every frame, the second computer rotates it faster. To avoid this, use delta time (time elapsed since the previous frame):

```
const clock = new THREE.Clock();

function animate() {
  requestAnimationFrame(animate);

  const delta = clock.getDelta();

  robot.rotation.y += 1 * delta;

  renderer.render(scene, camera);
}

animate();
```

## Loading 3D models

- I would use Three.js `GLTFLoader` [it's a Three.js addon/class] to load the robot.glb file. Once loaded, I can access the model through gltf.scene and add it to the main scene. 

- The model contains a hierarchy of objects representing different robot parts. This hierarchy allows me to control individual joints, while parent transformations also affect their children.

```
import { GLTFLoader } from
  'three/addons/loaders/GLTFLoader.js';

const loader = new GLTFLoader();

loader.load('/models/robot.glb', (gltf) => {
  const robot = gltf.scene;

  scene.add(robot);
});
```

## Raycasting

Raycasting means sending an invisible ray from the camera through the mouse pointer into the 3D scene to detect which objects it intersects.

```
const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();

function onClick(event) {
  mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
  mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

  raycaster.setFromCamera(mouse, camera);

  const intersects = raycaster.intersectObjects(
    scene.children,
    true
  );

  if (intersects.length > 0) {
    console.log(intersects[0].object);
  }
}
```

- Raycaster: Creates and tests the ray.
- setFromCamera(): Sends the ray through the mouse position from the camera.
- intersectObjects(): Checks which objects the ray hits.
- intersects[0]: The closest intersection.
The true argument means recursive checking, so it can check nested child objects too.

Raycasting is used to detect interactions with 3D objects. It sends a ray from the camera through the mouse position and checks which objects it intersects. 
I can use Raycaster.intersectObjects() to identify the clicked robot joint. Passing `true` enables recursive checking of nested child objects.

## Performance optimization

1. `Reuse geometry/materials`: Correct. If multiple objects share the same geometry or material, reuse them instead of creating duplicates. This saves memory and resource creation overhead.

2. `Dispose resources`: Correct. When removing objects permanently, dispose of their geometry, materials, and textures when no longer needed. This frees GPU resources.

3. `Throttle updates`: Correct. If WebSocket sends simulation updates very frequently, avoid processing every update unnecessarily. You can batch or throttle updates and render efficiently.

4. `InstancedMesh`: Efficiently renders many objects sharing the same geometry and material.

5. `Reduce draw calls`: Combine objects where appropriate and avoid unnecessary separate rendering operations.


# React Three Fiber

- `R3F`: React renderer for Three.js

`Canvas` is the root component of React Three Fiber. It creates and manages the Three.js rendering environment.

```
import { Canvas } from '@react-three/fiber';

function App() {
  return (
    <Canvas>
      <ambientLight intensity={0.5} />
      <directionalLight position={[5, 5, 5]} />
      <Cube />
    </Canvas>
  );
}
```

It handles things like:
- Creating the renderer.
- Setting up the scene and camera.
- Rendering the 3D content.
- Managing the rendering lifecycle.

`useFrame` is a React Three Fiber hook that executes a callback on every rendered frame.

Useful for:
- Animating robot joints.
- Updating object positions.
- Synchronizing simulation data.
- Creating smooth animations.

```
useFrame((state, delta) => {
    meshRef.current.rotation.y += delta;
  });
```

@react-three/drei is a helper library that provides useful abstractions for R3F, such as model loading, camera controls, and environment lighting.

## How do you optimize R3F performance?

1. Use `useRef` instead of React state for high-frequency animation updates.

2. Use `useFrame` efficiently and avoid unnecessary calculations.

3. Reuse geometry and materials.

4. Use `InstancedMesh` for repeated objects.

5. Dispose of `unused resources.`

6. Avoid unnecessary React re-renders.

7. Use LOD (Level of Detail) for complex models when appropriate.

Q. What is useFrame?	
- Runs on every rendered frame

Q. Why useRef?	
- Update Three.js objects without React rerenders