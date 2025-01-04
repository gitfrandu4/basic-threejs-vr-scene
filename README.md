# Basic Three.js VR Scene

A modern WebXR-enabled virtual reality scene built with Three.js, featuring interactive controllers and object manipulation.

<img src="assets/scene.png" alt="Screenshot of the scene">

## Table of Contents

- [Basic Three.js VR Scene](#basic-threejs-vr-scene)
  - [Table of Contents](#table-of-contents)
  - [Additional Documentation](#additional-documentation)
  - [Features](#features)
  - [Code Highlights](#code-highlights)
    - [1. Enabling WebXR and Adding the VR Button](#1-enabling-webxr-and-adding-the-vr-button)
    - [2. Adding a Sky and Ground](#2-adding-a-sky-and-ground)
    - [3. Loading FBX Models](#3-loading-fbx-models)
    - [4. Controllers and "Grab" Interactions](#4-controllers-and-grab-interactions)
    - [5. Raycast with the Left Controller](#5-raycast-with-the-left-controller)
    - [WebAR Considerations](#webar-considerations)
  - [Usage](#usage)
  - [Contributing](#contributing)
  - [License](#license)

## Additional Documentation
The project draws on the following **WebXR** documentation:
- **WebXR Emulator Extension** (to emulate VR/AR devices on a desktop):  
  [MozillaReality/WebXR-emulator-extension](https://github.com/MozillaReality/WebXR-emulator-extension)
- **iOS App for WebXR** (to run VR/AR experiences on Apple devices):  
  [WebXR Viewer on AppStore](https://apps.apple.com/us/app/webxr-viewer/id1295998056)

For a detailed explanation of the starter scene, VR/AR setup, environment textures, and controller events, you can refer to the documentation text provided in this repo (or the excerpt in the project's docs folder).

## Features
- **Skybox** (using `SphereGeometry` with a texture).
- **Ground** plane (texture repeated over a large area).
- **Multiple FBX Models** (loaded via `FBXLoader` and placed at different coordinates).
- **VR Controllers** using `XRControllerModelFactory`, including events to:
  - **Grab** nearby objects when the right controller is close enough.
  - **Raycast** with the left controller (pull objects in or relocate them).
- Example code for both **WebVR** and **WebAR** (just by switching to `VRButton` or `ARButton`).

## Code Highlights

### 1. Enabling WebXR and Adding the VR Button
```js
import { VRButton } from 'https://unpkg.com/three@0.126.0/examples/jsm/webxr/VRButton.js';

renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
renderer.xr.enabled = true; // <-- Enable WebXR
document.body.appendChild(renderer.domElement);
document.body.appendChild(VRButton.createButton(renderer));

renderer.setAnimationLoop(render);
```

### 2. Adding a Sky and Ground
```js
// Texture loader
let textureLoader = new THREE.TextureLoader();

// Sky
const textureSky = textureLoader.load('path/to/sky-texture.jpg');
const materialSky = new THREE.MeshBasicMaterial({
  map: textureSky,
  side: THREE.DoubleSide,
});
const sky = new THREE.Mesh(
  new THREE.SphereGeometry(30, 32, 32),
  materialSky
);
scene.add(sky);

// Ground
const textureGround = textureLoader.load(
  'path/to/grass-texture.jpg',
  function (texture) {
    texture.wrapS = texture.wrapT = THREE.RepeatWrapping;
    texture.repeat.set(60, 60);
  }
);
const materialGround = new THREE.MeshStandardMaterial({ map: textureGround });
let plane = new THREE.Mesh(new THREE.PlaneGeometry(60, 60), materialGround);
plane.rotation.x = -Math.PI / 2; // Rotar 90 grados
scene.add(plane);
```

### 3. Loading FBX Models
```js
import { FBXLoader } from 'https://unpkg.com/three@0.126.0/examples/jsm/loaders/FBXLoader.js';

new FBXLoader().load(
  'path/to/Tree(1).fbx',
  (object) => {
    object.traverse((child) => {
      object.scale.set(0.004, 0.004, 0.004);
      createTree(object.clone(), [0, 0, -10]);
      createTree(object.clone(), [5, 0, -3]);
      // ...
    });
  }
);

function createTree(tree, position) {
  tree.position.set(position[0], position[1], position[2]);
  scene.add(tree);
}
```

### 4. Controllers and "Grab" Interactions
```js
import { XRControllerModelFactory } from 'https://unpkg.com/three@0.126.0/examples/jsm/webxr/XRControllerModelFactory.js';

// Controllers
const controllerModelFactory = new XRControllerModelFactory();
controllerL = renderer.xr.getControllerGrip(0);
controllerR = renderer.xr.getControllerGrip(1);

scene.add(controllerL);
scene.add(controllerR);

// Right Controller: Grab
let checkGrabbingR = false;
controllerR.addEventListener('selectstart', () => (checkGrabbingR = true));
controllerR.addEventListener('selectend', () => (checkGrabbingR = false));

function CheckObjects() {
  // If within a certain radius, snap object to controllerR
  objects.forEach((obj) => {
    let dist = obj.position.distanceTo(controllerR.position);
    if (dist <= controllerActionRadius) {
      obj.position.copy(controllerR.position);
    }
  });
}

function render() {
  if (checkGrabbingR) CheckObjects();
  renderer.render(scene, camera);
}
```

### 5. Raycast with the Left Controller
```js
controllerL.addEventListener('squeezestart', SelectEventRay);

function SelectEventRay() {
  const intersections = getIntersections(controllerL);
  intersections.forEach((intersection) => {
    if (objects.includes(intersection.object)) {
      intersection.object.position.copy(controllerL.position);
    }
  });
}

function getIntersections(controller) {
  const tempMatrix = new THREE.Matrix4();
  const raycaster = new THREE.Raycaster();

  tempMatrix.identity().extractRotation(controller.matrixWorld);
  raycaster.ray.origin.setFromMatrixPosition(controller.matrixWorld);
  raycaster.ray.direction.set(0, 0, -1).applyMatrix4(tempMatrix);

  return raycaster.intersectObjects(scene.children);
}
```

### WebAR Considerations
If you’d like to switch to **AR** instead of **VR**, you can import:
```js
import { ARButton } from 'https://unpkg.com/three@0.126.0/examples/jsm/webxr/ARButton.js';
```
...and replace the VR button with:
```js
document.body.appendChild(ARButton.createButton(renderer, {
  requiredFeatures: ['hit-test'] // or []
}));
```
Your `renderer.xr.enabled = true;` remains the same, but AR mode uses the device camera as a background. In advanced AR experiences, you can detect surfaces (plane detection), place objects on them, and interact by tapping or by using controllers if available.

## Usage
1. **Clone** the repository:
```bash
git clone https://github.com/usuario/basic-threejs-vr-scene.git
```
2. **Install** a local server or open the file in a compatible environment:
   - For example, serve it with `npx http-server` (or any other simple web server).
3. **Open** the `index.html` in your **WebXR-compatible browser**. If using mobile:
   - On **Android**, use **Chrome** (with WebXR flags enabled if necessary).
   - On **iOS**, use the **WebXR Viewer** app.
4. **Enter VR** or **Enter AR** by clicking the button added by `VRButton` or `ARButton`.

## Contributing
1. **Fork** the repository.
2. **Create** a new branch:
```bash
git checkout -b feature/new-feature
```
3. **Commit** your changes:
```bash
git commit -m "Add new feature"
```
4. **Push** the branch:
```bash
git push origin feature/new-feature
```
5. **Open** a Pull Request for review.

## License
This project is available under the [MIT License](LICENSE).
