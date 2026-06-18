# Rough notes

This section is roughed out to allow you to get access to newly developed code I will add more comments later.


## Player01

Use the [asset manager(https://doc.babylonjs.com/features/featuresDeepDive/importers/assetManager/)] to place trees onto a scene as gltf.  Clone some of these.

This needs the glTF loader to be imported to handle glTF2.0 format models.

A function addAssets within createStartScene creates and returns an assets manager for the scene.  Items are added to this through the assetsManager.addMeshTask function.  So in this example, tree1 is not a model but is a MeshAssetTask.

The [AddMeshTask function](https://doc.babylonjs.com/typedoc/classes/BABYLON.AssetsManager#addmeshtask) takes a list of mandatory parameters: taskName, meshesName, rootUrl,sceneFilename.  There are also optional parameter which may be added including: extension, filename and pluginOptions.

One of the glTF options controls the start mode to determine which, if any, of the built in animations will run when the model is loaded.  Another option is the GLTFLoaderCoordinateSystemMode.  If not included this defaults to Auto mode in which it will automatically confert the glTF if it is based on a right handed coordinate stystem into the coordinate stystem mode of the scene.  This is a significant improvement on the older file loading system where models having a mismatched coordinate system could fall through the ground and dissapear.

The glTF files may be compressed by codecs which include KHR_mesh_draco_compression there are a range of extension options which relate to this.  So the loading of glTF files may sometimes need additional details.

The meshesName defines the name of meshes to load and may be left blank to load whatever meshes are in a particular package.

The plugin options in detail offer options for different file formats including bvh, gltf, obj, splat and stl.  

The [MeshAssetsTask](https://doc.babylonjs.com/typedoc/classes/BABYLON.MeshAssetTask) has a number of properties which include onSucess and onError.  These will be set to functions which will be called when the task succeeds of fails.  The MeshAssetTask is passed into the function as ```task```.

Within this callback function the task properties such as loadedMeshes can be manipulated.  loadedMeshes[0] is the root mesh which can be positioned and scaled.

The root mesh can also be cloned to form an <AbstractMesh>

The assetsManager.onTaskEfforObservable is used to pass back messages if any of the tasks leads to an error condition.

The addAssets function returns the assetsManager

** player01/createStartScene (extract)**
```javascript
function addAssets(scene: Scene) {
  // add assets here
  const assetsManager = new AssetsManager(scene);
  const tree1:MeshAssetTask = assetsManager.addMeshTask(
    "tree1 task",
    "",
    "./assets/nature/gltf/",
    "CommonTree_1.gltf"
  );
  tree1.onSuccess = function (task) {
    task.loadedMeshes[0].position = new Vector3(3, 0, 2);
    task.loadedMeshes[0].scaling = new Vector3(0.5, 0.5, 0.5);
        // Clone tree1
    const tree1Clone = task.loadedMeshes[0].clone("tree1_clone", null);
    tree1Clone!.position = new Vector3(0, 0, 5);
  };

 //  ...

  assetsManager.onTaskErrorObservable.add(function (task) {
    console.log(
      "task failed",
      task.errorObject.message,
      task.errorObject.exception
    );
  });
  return assetsManager;
}
```

The tasks which have been added to the assetsManager are executed by the load() method.

** player01/createStartScene (extract)**
```javascript
   let that: SceneData = { scene: new Scene(engine) };
  //that.scene.debugLayer.show();

  that.light = createLight(that.scene);
  that.ground = createGround(that.scene);
  that.camera = createArcRotateCamera(that.scene);
  const assetsManager = addAssets(that.scene);
  assetsManager.load();
```

The other components of the scene are standard so the full listing of createStartScene is:



** player01/createStartScene **
```javascript
//import "@babylonjs/core/Debug/debugLayer";
//import "@babylonjs/inspector";
import "@babylonjs/loaders/glTF/2.0";
import {
  Scene,
  ArcRotateCamera,
  AssetsManager,
  Vector3,
  HemisphericLight,
  MeshBuilder,
  Mesh,
  Camera,
  Engine,
} from "@babylonjs/core";
import { taaPixelShader } from "@babylonjs/core/Shaders/taa.fragment";

function createLight(scene: Scene) {
  const light = new HemisphericLight("light", new Vector3(0, 1, 0), scene);
  light.intensity = 0.7;
  return light;
}

function createGround(scene: Scene) {
  let ground = MeshBuilder.CreateGround(
    "ground",
    { width: 16, height: 16 },
    scene
  );
  return ground;
}

function createArcRotateCamera(scene: Scene) {
  let camAlpha = -Math.PI / 2,
    camBeta = Math.PI / 2.5,
    camDist = 10,
    camTarget = new Vector3(0, 0, 0);
  let camera = new ArcRotateCamera(
    "camera1",
    camAlpha,
    camBeta,
    camDist,
    camTarget,
    scene
  );
  camera.attachControl(true);
  return camera;
}

function addAssets(scene: Scene) {
  // add assets here
  const assetsManager = new AssetsManager(scene);
  const tree1 = assetsManager.addMeshTask(
    "tree1 task",
    "",
    "./assets/nature/glTF/",
    "CommonTree_1.gltf"
  );
  tree1.onSuccess = function (task) {
    task.loadedMeshes[0].position = new Vector3(3, 0, 2);
    task.loadedMeshes[0].scaling = new Vector3(0.5, 0.5, 0.5);
        // Clone tree1
    const tree1Clone = task.loadedMeshes[0].clone("tree1_clone", null);
    tree1Clone!.position = new Vector3(0, 0, 5);
  };

  const tree2 = assetsManager.addMeshTask(
    "tree1 task",
    "",
    "./assets/nature/glTF/",
    "CommonTree_2.gltf"
  );
  tree2.onSuccess = function (task) {
    task.loadedMeshes[0].position = new Vector3(0, 0, 2);
    task.loadedMeshes[0].scaling = new Vector3(0.5, 0.5, 0.5);
        // Clone tree2
    const tree2Clone = task.loadedMeshes[0].clone("tree2_clone", null);
    tree2Clone!.position = new Vector3(-3, 0, 5);
  };

  const tree3 = assetsManager.addMeshTask(
    "tree1 task",
    "",
    "./assets/nature/glTF/",
    "CommonTree_3.gltf"
  );
  tree3.onSuccess = function (task) {
    task.loadedMeshes[0].position = new Vector3(-3, 0, 2);
    task.loadedMeshes[0].scaling = new Vector3(0.5, 0.5, 0.5);
    // Clone tree3
    const tree3Clone = task.loadedMeshes[0].clone("tree3_clone", null);
    tree3Clone!.position = new Vector3(3, 0, 5);
  };

  assetsManager.onTaskErrorObservable.add(function (task) {
    console.log(
      "task failed",
      task.errorObject.message,
      task.errorObject.exception
    );
  });
  return assetsManager;
}

export default function createStartScene(engine: Engine) {
  interface SceneData {
    scene: Scene;
    light?: HemisphericLight;
    ground?: Mesh;
    camera?: Camera;
  }

  let that: SceneData = { scene: new Scene(engine) };
  //that.scene.debugLayer.show();

  that.light = createLight(that.scene);
  that.ground = createGround(that.scene);
  that.camera = createArcRotateCamera(that.scene);
  const assetsManager = addAssets(that.scene);
  assetsManager.load();
  return that;
}
```
Keep interface in step with scene contents.

** interface.ts **
```javascript
import {
  Scene,
  Mesh,
  HemisphericLight,
  Camera,
} from "@babylonjs/core";

export interface SceneData {
      scene: Scene;
      light?: HemisphericLight;
      ground?: Mesh;
      camera?: Camera;
}
```

Not using any other scene files so index.ts is at its simplest.

**  index.ts **
```javascript
import { Engine } from "@babylonjs/core";
import createStartScene from "./createStartScene";
import './main.css';

const CanvasName = "renderCanvas";

let canvas = document.createElement("canvas");
canvas.id = CanvasName;

canvas.classList.add("background-canvas");
document.body.appendChild(canvas);

let eng = new Engine(canvas, true, {}, true);
let startScene = createStartScene(eng);
eng.runRenderLoop(() => {
    startScene.scene.render();
});            
```

## Player 02

Now add in a character controller and add a physics aggregate to the ground so that the character does not fall through the floor.

Note the initialisation of havok.

** createStartScene **
```javascript
//import "@babylonjs/core/Debug/debugLayer";
//import "@babylonjs/inspector";
import "@babylonjs/loaders/glTF/2.0";
import HavokPhysics, { HavokPhysicsWithBindings } from "@babylonjs/havok";
import {
  Scene,
  ArcRotateCamera,
  AssetsManager,
  Vector3,
  HemisphericLight,
  MeshBuilder,
  Mesh,
  Camera,
  Engine,
  HavokPlugin,
  PhysicsCharacterController,
  Quaternion,
  CharacterSupportedState,
  KeyboardEventTypes,
  PhysicsAggregate,
  PhysicsShapeType,
} from "@babylonjs/core";
import { taaPixelShader } from "@babylonjs/core/Shaders/taa.fragment";



function createLight(scene: Scene) {
  const light = new HemisphericLight("light", new Vector3(0, 1, 0), scene);
  light.intensity = 0.7;
  return light;
}

function createGround(scene: Scene) {
  let ground = MeshBuilder.CreateGround(
    "ground",
    { width: 16, height: 16 },
    scene
  );
  
    // Create a static box shape.
  let groundAggregate = new PhysicsAggregate(ground, PhysicsShapeType.BOX, { mass: 0 }, scene);
  return ground;
}

function createArcRotateCamera(scene: Scene) {
  let camAlpha = -Math.PI / 2,
    camBeta = Math.PI / 2.5,
    camDist = 10,
    camTarget = new Vector3(0, 0, 0);
  let camera = new ArcRotateCamera(
    "camera1",
    camAlpha,
    camBeta,
    camDist,
    camTarget,
    scene
  );
  camera.attachControl(true);
  return camera;
}

function addAssets(scene: Scene) {
  // add assets here
  const assetsManager = new AssetsManager(scene);
  const tree1 = assetsManager.addMeshTask(
    "tree1 task",
    "",
    "./assets/nature/glTF/",
    "CommonTree_1.gltf"
  );
  tree1.onSuccess = function (task) {
    task.loadedMeshes[0].position = new Vector3(3, 0, 2);
    task.loadedMeshes[0].scaling = new Vector3(0.5, 0.5, 0.5);
    // Clone tree1
    const tree1Clone = task.loadedMeshes[0].clone("tree1_clone", null);
    tree1Clone!.position = new Vector3(0, 0, 5);
  };

  const tree2 = assetsManager.addMeshTask(
    "tree1 task",
    "",
    "./assets/nature/glTF/",
    "CommonTree_2.gltf"
  );
  tree2.onSuccess = function (task) {
    task.loadedMeshes[0].position = new Vector3(0, 0, 2);
    task.loadedMeshes[0].scaling = new Vector3(0.5, 0.5, 0.5);
    // Clone tree2
    const tree2Clone = task.loadedMeshes[0].clone("tree2_clone", null);
    tree2Clone!.position = new Vector3(-3, 0, 5);
  };

  const tree3 = assetsManager.addMeshTask(
    "tree1 task",
    "",
    "./assets/nature/glTF/",
    "CommonTree_3.gltf"
  );
  tree3.onSuccess = function (task) {
    task.loadedMeshes[0].position = new Vector3(-3, 0, 2);
    task.loadedMeshes[0].scaling = new Vector3(0.5, 0.5, 0.5);
    // Clone tree3
    const tree3Clone = task.loadedMeshes[0].clone("tree3_clone", null);
    tree3Clone!.position = new Vector3(3, 0, 5);
  };

  assetsManager.onTaskErrorObservable.add(function (task) {
    console.log(
      "task failed",
      task.errorObject.message,
      task.errorObject.exception
    );
  });
  return assetsManager;
}


export default async function createStartScene(engine: Engine) {
  interface SceneData {
    scene: Scene;
    light?: HemisphericLight;
    ground?: Mesh;
    camera?: Camera;
  }

  let that: SceneData = { scene: new Scene(engine) };

  let initializedHavok: any;

  HavokPhysics().then((havok) => {
    initializedHavok = havok;
  });

  const havokInstance: HavokPhysicsWithBindings = await HavokPhysics();
  const hk: HavokPlugin = new HavokPlugin(true, havokInstance);
  that.scene.enablePhysics(new Vector3(0, -9.81, 0), hk);

  //that.scene.debugLayer.show();

  that.light = createLight(that.scene);
  that.ground = createGround(that.scene);
  that.camera = createArcRotateCamera(that.scene);
  const assetsManager = addAssets(that.scene);
  assetsManager.load();
  return that;
}

```

Now add charactercontroller in a new file

** createCharacterController **
```javascript
import "@babylonjs/loaders/glTF/2.0";

import {
  Scene,
  Vector3,
  MeshBuilder,
  PhysicsCharacterController,
  Quaternion,
  CharacterSupportedState,
  KeyboardEventTypes,
  StandardMaterial,
  Color3,
} from "@babylonjs/core";

export function createCharacterController(scene: Scene) {
  // Character state machine
  let characterState = "ON_GROUND";
  const inAirSpeed = 8.0;
  const onGroundSpeed = 10;
  const jumpHeight = 1.5;
  const characterGravity = new Vector3(0, -18, 0);
  
  // Input tracking
  let keyInput = new Vector3(0, 0, 0);
  let wantJump = false;
  
  // Character orientation
  let characterOrientation = Quaternion.Identity();
  let forwardLocalSpace = new Vector3(0, 0, 1);

  // Create visual capsule mesh
  const h = 1.8;
  const r = 0.6;
  let displayCapsule = MeshBuilder.CreateCapsule(
    "CharacterDisplay",
    { height: h, radius: r },
    scene
  );
  displayCapsule.position = new Vector3(0, h / 2, 0);
  
  // Apply material for visibility
  const capsuleMat = new StandardMaterial("capsuleMat", scene);
  capsuleMat.diffuseColor = new Color3(0.8, 0.2, 0.2);
  capsuleMat.emissiveColor = new Color3(0.3, 0.1, 0.1);
  displayCapsule.material = capsuleMat;
  
  // Debug: log initial position
  console.log("Capsule initial position:", displayCapsule.position);

  // Create physics character controller
  let characterController = new PhysicsCharacterController(
    displayCapsule.position.clone(),
    { capsuleHeight: h, capsuleRadius: r },
    scene
  );

  // Compute desired velocity based on input and state
  const getDesiredVelocity = function (
    deltaTime: number,
    supportInfo: {
      supportedState: CharacterSupportedState;
      averageSurfaceNormal: Vector3;
      averageSurfaceVelocity: Vector3;
    },
    currentVelocity: Vector3
  ): Vector3 {
    // Update state
    if (characterState === "ON_GROUND" && supportInfo.supportedState !== CharacterSupportedState.SUPPORTED) {
      characterState = "IN_AIR";
    } else if (characterState === "IN_AIR" && supportInfo.supportedState === CharacterSupportedState.SUPPORTED) {
      characterState = "ON_GROUND";
    }

    // Check for jump transition
    if (characterState === "ON_GROUND" && wantJump) {
      characterState = "START_JUMP";
    } else if (characterState === "START_JUMP") {
      characterState = "IN_AIR";
    }

    let upWorld = characterGravity.normalizeToNew();
    upWorld.scaleInPlace(-1.0);
    let forwardWorld = forwardLocalSpace.applyRotationQuaternion(characterOrientation);

    if (characterState === "IN_AIR") {
      let desiredVelocity = keyInput
        .scale(inAirSpeed)
        .applyRotationQuaternion(characterOrientation);
      let outputVelocity = characterController.calculateMovement(
        deltaTime,
        forwardWorld,
        upWorld,
        currentVelocity,
        Vector3.ZeroReadOnly,
        desiredVelocity,
        upWorld
      );
      // Restore vertical component and apply gravity
      outputVelocity.addInPlace(upWorld.scale(-outputVelocity.dot(upWorld)));
      outputVelocity.addInPlace(upWorld.scale(currentVelocity.dot(upWorld)));
      outputVelocity.addInPlace(characterGravity.scale(deltaTime));
      return outputVelocity;
    } else if (characterState === "ON_GROUND") {
      let desiredVelocity = keyInput
        .scale(onGroundSpeed)
        .applyRotationQuaternion(characterOrientation);

      let outputVelocity = characterController.calculateMovement(
        deltaTime,
        forwardWorld,
        supportInfo.averageSurfaceNormal,
        currentVelocity,
        supportInfo.averageSurfaceVelocity,
        desiredVelocity,
        upWorld
      );
      // Project velocity onto ground plane
      outputVelocity.subtractInPlace(supportInfo.averageSurfaceVelocity);
      let inv1k = 1e-3;
      if (outputVelocity.dot(upWorld) > inv1k) {
        let velLen = outputVelocity.length();
        outputVelocity.normalizeFromLength(velLen);
        let horizLen = velLen / supportInfo.averageSurfaceNormal.dot(upWorld);
        let c = supportInfo.averageSurfaceNormal.cross(outputVelocity);
        outputVelocity = c.cross(upWorld);
        outputVelocity.scaleInPlace(horizLen);
      }
      outputVelocity.addInPlace(supportInfo.averageSurfaceVelocity);
      return outputVelocity;
    } else if (characterState === "START_JUMP") {
      let u = Math.sqrt(2 * characterGravity.length() * jumpHeight);
      let curRelVel = currentVelocity.dot(upWorld);
      return currentVelocity.add(upWorld.scale(u - curRelVel));
    }
    return Vector3.Zero();
  };

  // Sync visual mesh with physics controller every frame
  scene.onBeforeRenderObservable.add(() => {
    displayCapsule.position.copyFrom(characterController.getPosition());
  });

  // Update physics each frame
  scene.onAfterPhysicsObservable?.add(() => {
    if (scene.deltaTime === undefined) return;
    let dt = scene.deltaTime / 1000.0;
    if (dt === 0) return;

    let down = new Vector3(0, -1, 0);
    let support = characterController.checkSupport(dt, down);

    let desiredLinearVelocity = getDesiredVelocity(
      dt,
      support,
      characterController.getVelocity()
    );
    characterController.setVelocity(desiredLinearVelocity);
    characterController.integrate(dt, support, characterGravity);
  });

  // Keyboard input handler
  scene.onKeyboardObservable.add((kbInfo) => {
    const key = kbInfo.event.key;
    
    switch (kbInfo.type) {
      case KeyboardEventTypes.KEYDOWN:
        if (key === "w" || key === "ArrowUp") {
          keyInput.z = 1;
        } else if (key === "s" || key === "ArrowDown") {
          keyInput.z = -1;
        } else if (key === "a" || key === "ArrowLeft") {
          keyInput.x = -1;
        } else if (key === "d" || key === "ArrowRight") {
          keyInput.x = 1;
        } else if (key === " ") {
          wantJump = true;
        }
        break;

      case KeyboardEventTypes.KEYUP:
        if (key === "w" || key === "s" || key === "ArrowUp" || key === "ArrowDown") {
          keyInput.z = 0;
        }
        if (key === "a" || key === "d" || key === "ArrowLeft" || key === "ArrowRight") {
          keyInput.x = 0;
        }
        if (key === " ") {
          wantJump = false;
        }
        break;
    }
  });
}
```

Make sure that index.ts now includes the character controller

** index.ts **
```javascript
import { Engine } from "@babylonjs/core";
import createStartScene from "./createStartScene";
import './main.css';
import {createCharacterController} from "./createCharacterController";

const CanvasName = "renderCanvas";

let canvas = document.createElement("canvas");
canvas.id = CanvasName;

canvas.classList.add("background-canvas");
document.body.appendChild(canvas);

let eng = new Engine(canvas, true, {}, true);

(async function main() {
    const startScene = await createStartScene(eng);
    createCharacterController(startScene.scene);
    eng.runRenderLoop(() => {
        startScene.scene.render();
    });
})();
```

## Player 03

Add two boxes with physics which interact with the character controller.  Added a gui which shows pre defined text and a button, but not changing display.

** createStartScene.ts **
```javascript
//import "@babylonjs/core/Debug/debugLayer";
//import "@babylonjs/inspector";
import "@babylonjs/loaders/glTF/2.0";
import HavokPhysics, { HavokPhysicsWithBindings } from "@babylonjs/havok";
import {
  Scene,
  ArcRotateCamera,
  AssetsManager,
  Vector3,
  HemisphericLight,
  MeshBuilder,
  Mesh,
  Camera,
  Engine,
  HavokPlugin,
  PhysicsAggregate,
  PhysicsShapeType,
  Color3,
  StandardMaterial,
  Texture,
} from "@babylonjs/core";


function createLight(scene: Scene) {
  const light = new HemisphericLight("light", new Vector3(0, 1, 0), scene);
  light.intensity = 0.7;
  return light;
}

function createGround(scene: Scene) {
  let ground = MeshBuilder.CreateGround(
    "ground",
    { width: 16, height: 16 },
    scene
  );
  
    // Create a static box shape.
  let groundAggregate = new PhysicsAggregate(ground, PhysicsShapeType.BOX, { mass: 0 }, scene);
  return ground;
}

function createArcRotateCamera(scene: Scene) {
  let camAlpha = -Math.PI / 2,
    camBeta = Math.PI / 2.5,
    camDist = 10,
    camTarget = new Vector3(0, 0, 0);
  let camera = new ArcRotateCamera(
    "camera1",
    camAlpha,
    camBeta,
    camDist,
    camTarget,
    scene
  );
  camera.attachControl(true);
  return camera;
}

function createBox1(scene: Scene) {
  let box = MeshBuilder.CreateBox("box", { width: 1, height: 1 }, scene);
  box.position.x = -1;
  box.position.y = 3;
  box.position.z = 1;

  var texture = new StandardMaterial("reflective", scene);
  texture.ambientTexture = new Texture(
    "./assets/textures/wood.jpg",
    scene
  );
  texture.diffuseColor = new Color3(1, 1, 1);
  box.material = texture;
  new PhysicsAggregate(box, PhysicsShapeType.BOX, {mass: 0.2, restitution:0.1, friction:0.4}, scene);
  return box;

}

function createBox2(scene: Scene) {
  let box = MeshBuilder.CreateBox("box", { width: 1, height: 1 }, scene);
  box.position.x = -0.7;
  box.position.y = 5;
  box.position.z = 1;

  var texture = new StandardMaterial("reflective", scene);
  texture.ambientTexture = new Texture(
    "./assets/textures/wood.jpg",
    scene
  );
  texture.diffuseColor = new Color3(1, 1, 1);
  box.material = texture;
  new PhysicsAggregate(box, PhysicsShapeType.BOX, {mass: 0.2, restitution:0.1, friction:0.4}, scene);
  return box;
}

function addAssets(scene: Scene) {
  // add assets here
  const assetsManager = new AssetsManager(scene);
  const tree1 = assetsManager.addMeshTask(
    "tree1 task",
    "",
    "./assets/nature/glTF/",
    "CommonTree_1.gltf"
  );
  tree1.onSuccess = function (task) {
    const root = task.loadedMeshes[0];
    root.position = new Vector3(3, 0, 2);
    root.scaling = new Vector3(0.5, 0.5, 0.5);
    // Ensure all child meshes are visible
    task.loadedMeshes.forEach((mesh: any) => {
      mesh.isVisible = true;
    });
    //new PhysicsAggregate(root, PhysicsShapeType.MESH, {mass: 0}, scene);
    
    // Clone tree1
    const tree1Clone = root.clone("tree1_clone", null);
    tree1Clone!.position = new Vector3(0, 0, 5);
    //new PhysicsAggregate(tree1Clone!, PhysicsShapeType.MESH, {mass: 0}, scene);
  };

  const tree2 = assetsManager.addMeshTask(
    "tree1 task",
    "",
    "./assets/nature/glTF/",
    "CommonTree_2.gltf"
  );
  tree2.onSuccess = function (task) {
    task.loadedMeshes[0].position = new Vector3(0, 0, 2);
    task.loadedMeshes[0].scaling = new Vector3(0.5, 0.5, 0.5);
    // Clone tree2
    const tree2Clone = task.loadedMeshes[0].clone("tree2_clone", null);
    tree2Clone!.position = new Vector3(-3, 0, 5);
  };

  const tree3 = assetsManager.addMeshTask(
    "tree1 task",
    "",
    "./assets/nature/glTF/",
    "CommonTree_3.gltf"
  );
  tree3.onSuccess = function (task) {
    task.loadedMeshes[0].position = new Vector3(-3, 0, 2);
    task.loadedMeshes[0].scaling = new Vector3(0.5, 0.5, 0.5);
    // Clone tree3
    const tree3Clone = task.loadedMeshes[0].clone("tree3_clone", null);
    tree3Clone!.position = new Vector3(3, 0, 5);
  };

  assetsManager.onTaskErrorObservable.add(function (task) {
    console.log(
      "task failed",
      task.errorObject.message,
      task.errorObject.exception
    );
  });
  return assetsManager;
}


export default async function createStartScene(engine: Engine) {
  interface SceneData {
    scene: Scene;
    light?: HemisphericLight;
    ground?: Mesh;
    camera?: Camera;
    box1?:Mesh;
    box2?:Mesh;
  }

  let that: SceneData = { scene: new Scene(engine) };

  let initializedHavok: any;

  HavokPhysics().then((havok) => {
    initializedHavok = havok;
  });

  const havokInstance: HavokPhysicsWithBindings = await HavokPhysics();
  const hk: HavokPlugin = new HavokPlugin(true, havokInstance);
  that.scene.enablePhysics(new Vector3(0, -9.81, 0), hk);

  //that.scene.debugLayer.show();

  that.light = createLight(that.scene);
  that.ground = createGround(that.scene);
  that.camera = createArcRotateCamera(that.scene);
  that.box1 = createBox1(that.scene);
  that.box2 = createBox2(that.scene);
  const assetsManager = addAssets(that.scene);
  assetsManager.load();
  return that;
}
```

Add a gui

** gui.ts **
```javascript
import { Scene } from "@babylonjs/core";
import {
  Button,
  AdvancedDynamicTexture,
  TextBlock,
  Control,
  Grid,
  Rectangle,
} from "@babylonjs/gui/2D";




  var text1!: TextBlock; // recieves external messages
  var text2!: TextBlock; // recieves external messages
  var text3!: TextBlock; // recieves external messages
  var text4!: TextBlock; // recieves external messages
  var heading1!: TextBlock;
  

  function createSceneButton(
    name: string,
    index: string,
    x: string,
    y: string,
    //advtex: { addControl: (arg0: Button) => void }
  ) {
    var button: Button = Button.CreateSimpleButton(name, index);
    button.left = x;
    button.top = y;
    button.width = "180px";
    button.height = "35px";
    button.color = "white";
    button.cornerRadius = 20;
    button.background = "green";

    button.onPointerClickObservable.add(function () {
      console.log("click event");
      let toggle:string = button.textBlock!.text == "clicked" ? "Click me!" :"clicked";
      button.textBlock!.text = toggle;
      console.log(toggle);
    });
    return button;
  }

  function createTextBlock(
    name: string,
    index: string,
    left: string,
    top: string
  ) {
    let text: TextBlock = new TextBlock(name, index);
    text.text = index;
    text.color = "white";
    text.fontSize = 24;
    text.left = left;
    text.top = top;
    text.width = "200px";
    text.height = "46px";
    text.fontFamily = "Verdana";
    text.textWrapping = true;
    text.highlightColor = "red";
    text.horizontalAlignment = TextBlock.HORIZONTAL_ALIGNMENT_CENTER;
    text.verticalAlignment = TextBlock.VERTICAL_ALIGNMENT_CENTER;
    // event handling
    text.onPointerEnterObservable.add(function () {
      text.isHighlighted = true;
    });
    text.onPointerOutObservable.add(function () {
      text.isHighlighted = false;
    });
    return text;
  }

  export function gui(scene:Scene): void {
    // add a button
    //https://doc.babylonjs.com/typedoc/modules/BABYLON.GUI  // GUI API

    let advancedTexture: AdvancedDynamicTexture =
      AdvancedDynamicTexture.CreateFullscreenUI("myUI", true, scene);
    let button1: Button = createSceneButton(
      "button1",
      "Click Me!",
      "0px",
      "0px",
      //advancedTexture
    );
    //advancedTexture.addControl(button1); // button 1 could be added to the scene or grid

    //add text block
    //https://playground.babylonjs.com/#2ARI2W#10 //high resolution text//
    scene.getEngine().setHardwareScalingLevel(1 / window.devicePixelRatio);
    advancedTexture.rootContainer.scaleX = window.devicePixelRatio;
    advancedTexture.rootContainer.scaleY = window.devicePixelRatio;

    heading1 = createTextBlock("heading1", "Hello World", "1px", "1px");
    text1 = createTextBlock("text1", "Debug", "1px", "1px");
    text2 = createTextBlock("text2", "Debug", "1px", "1px");
    text3 = createTextBlock("text3", "Debug", "1px", "1px");
    text4 = createTextBlock("text4", "Debug", "1px", "1px");

   
    // advancedTexture.addControl(this.heading1); // text1 block could be added to the scene or grid


//https://doc.babylonjs.com/features/featuresDeepDive/gui/gui#grid
// Create a grid, Pointer will then only apply to the grid and not the whole screen.

const grid = new Grid();
grid.addColumnDefinition(100, true);
grid.addColumnDefinition(0.25);
grid.addColumnDefinition(0.25);
grid.addColumnDefinition(0.25);
grid.addColumnDefinition(0.25);
grid.addColumnDefinition(100, true);
grid.addRowDefinition(50, true);
grid.addRowDefinition(50, true);

// This rect will be on first row and second column
const rect1 = new Rectangle();
rect1.background = "#76d56e88"; //rgba
rect1.thickness = 0;
rect1.addControl(heading1);  // rect is a container which can contain other controls
const rect2 = new Rectangle();
rect2.background = "#60955b88";
rect2.thickness = 0;
rect2.addControl(button1);
const rect3 = new Rectangle();
rect3.background = "#76d56e88";
rect3.thickness = 0;
//empty rect
const rect4 = new Rectangle();
rect4.background = "#60955b88";
rect4.thickness = 0;
//empty rect
const rect5 = new Rectangle();
rect5.background = "#76d56e88";
rect5.thickness = 0;
rect5.addControl(text1);
const rect6 = new Rectangle();
rect6.background = "#60955b88";
rect6.thickness = 0;
rect6.addControl(text2);
const rect7 = new Rectangle();
rect7.background = "#76d56e88";
rect7.thickness = 0;
rect7.addControl(text3);
const rect8 = new Rectangle();
rect8.background = "#60955b88";
rect8.thickness = 0;
rect8.addControl(text4);

grid.addControl(rect1, 0, 1);
grid.addControl(rect2, 0, 2);
grid.addControl(rect3, 0, 3);
grid.addControl(rect4, 0, 4);
grid.addControl(rect5, 1, 1);
grid.addControl(rect6, 1, 2);
grid.addControl(rect7, 1, 3);
grid.addControl(rect8, 1, 4);

advancedTexture.addControl(grid);

scene.registerBeforeRender(() => {
  // cant get to gui
  let mystash = scene.getExternalData("stash") as { [key: string]: string };
  try { text1.text = mystash.message; } catch {}
  try { text2.text = mystash.x; } catch {}// Desired direction
  try { text3.text = mystash.z; } catch {}// Desired direction
  
});
}
```

Index.ts starts up the gui
** index.ts**

```javascript
import { Engine } from "@babylonjs/core";
import createStartScene from "./createStartScene";
import './main.css';
import {createCharacterController} from "./createCharacterController";
import { gui } from "./gui";

const CanvasName = "renderCanvas";

let canvas = document.createElement("canvas");
canvas.id = CanvasName;

canvas.classList.add("background-canvas");
document.body.appendChild(canvas);

let eng = new Engine(canvas, true, {}, true);

(async function main() {
    const startScene = await createStartScene(eng);
    createCharacterController(startScene.scene);
    gui(startScene.scene);
    eng.runRenderLoop(() => {
        startScene.scene.render();
    });
})();
```

## Player 4

added boxes and collision detection which is listed on the gui.

** CreateStartScene **
```javascript
//import "@babylonjs/core/Debug/debugLayer";
//import "@babylonjs/inspector";
import "@babylonjs/loaders/glTF/2.0";
import HavokPhysics, { HavokPhysicsWithBindings } from "@babylonjs/havok";
import {
  Scene,
  ArcRotateCamera,
  AssetsManager,
  Vector3,
  HemisphericLight,
  MeshBuilder,
  Mesh,
  Camera,
  Engine,
  HavokPlugin,
  PhysicsAggregate,
  PhysicsShapeType,
  Color3,
  StandardMaterial,
  Texture,
} from "@babylonjs/core";


function createLight(scene: Scene) {
  const light = new HemisphericLight("light", new Vector3(0, 1, 0), scene);
  light.intensity = 0.7;
  return light;
}

function createGround(scene: Scene) {
  let ground = MeshBuilder.CreateGround(
    "ground",
    { width: 16, height: 16 },
    scene
  );
  
    // Create a static box shape.
  let groundAggregate = new PhysicsAggregate(ground, PhysicsShapeType.BOX, { mass: 0 }, scene);
  return groundAggregate;
}

function createArcRotateCamera(scene: Scene) {
  let camAlpha = -Math.PI / 2,
    camBeta = Math.PI / 2.5,
    camDist = 10,
    camTarget = new Vector3(0, 0, 0);
  let camera = new ArcRotateCamera(
    "camera1",
    camAlpha,
    camBeta,
    camDist,
    camTarget,
    scene
  );
  camera.attachControl(true);
  return camera;
}

function createBox1(scene: Scene) {
  let box = MeshBuilder.CreateBox("box", { width: 1, height: 1 }, scene);
  box.position.x = -1;
  box.position.y = 3;
  box.position.z = 1;

  var texture = new StandardMaterial("reflective", scene);
  texture.ambientTexture = new Texture(
    "./assets/textures/wood.jpg",
    scene
  );
  texture.diffuseColor = new Color3(1, 1, 1);
  box.material = texture;
  let box1Aggregate = new PhysicsAggregate(box, PhysicsShapeType.BOX, {mass: 0.2, restitution:0.1, friction:0.4}, scene);
  box1Aggregate.body.setCollisionCallbackEnabled(true);
  return box1Aggregate;
}

function createBox2(scene: Scene) {
  let box = MeshBuilder.CreateBox("box", { width: 1, height: 1 }, scene);
  box.position.x = -0.7;
  box.position.y = 5;
  box.position.z = 1;

  var texture = new StandardMaterial("reflective", scene);
  texture.ambientTexture = new Texture(
    "./assets/textures/wood.jpg",
    scene
  );
  texture.diffuseColor = new Color3(1, 1, 1);
  box.material = texture;
  let box2Aggregate = new PhysicsAggregate(box, PhysicsShapeType.BOX, {mass: 0.2, restitution:0.1, friction:0.4}, scene);
  box2Aggregate.body.setCollisionCallbackEnabled(true);
  return box2Aggregate;
}

function addAssets(scene: Scene) {
  // add assets here
  const assetsManager = new AssetsManager(scene);
  const tree1 = assetsManager.addMeshTask(
    "tree1 task",
    "",
    "./assets/nature/glTF/",
    "CommonTree_1.gltf"
  );
  tree1.onSuccess = function (task) {
    const root = task.loadedMeshes[0];
    root.position = new Vector3(3, 0, 2);
    root.scaling = new Vector3(0.5, 0.5, 0.5);
    // Ensure all child meshes are visible
    task.loadedMeshes.forEach((mesh: any) => {
      mesh.isVisible = true;
    });
    //new PhysicsAggregate(root, PhysicsShapeType.MESH, {mass: 0}, scene);
    
    // Clone tree1
    const tree1Clone = root.clone("tree1_clone", null);
    tree1Clone!.position = new Vector3(0, 0, 5);
    //new PhysicsAggregate(tree1Clone!, PhysicsShapeType.MESH, {mass: 0}, scene);
  };

  const tree2 = assetsManager.addMeshTask(
    "tree1 task",
    "",
    "./assets/nature/glTF/",
    "CommonTree_2.gltf"
  );
  tree2.onSuccess = function (task) {
    task.loadedMeshes[0].position = new Vector3(0, 0, 2);
    task.loadedMeshes[0].scaling = new Vector3(0.5, 0.5, 0.5);
    // Clone tree2
    const tree2Clone = task.loadedMeshes[0].clone("tree2_clone", null);
    tree2Clone!.position = new Vector3(-3, 0, 5);
  };

  const tree3 = assetsManager.addMeshTask(
    "tree1 task",
    "",
    "./assets/nature/glTF/",
    "CommonTree_3.gltf"
  );
  tree3.onSuccess = function (task) {
    task.loadedMeshes[0].position = new Vector3(-3, 0, 2);
    task.loadedMeshes[0].scaling = new Vector3(0.5, 0.5, 0.5);
    // Clone tree3
    const tree3Clone = task.loadedMeshes[0].clone("tree3_clone", null);
    tree3Clone!.position = new Vector3(3, 0, 5);
  };

  assetsManager.onTaskErrorObservable.add(function (task) {
    console.log(
      "task failed",
      task.errorObject.message,
      task.errorObject.exception
    );
  });
  return assetsManager;
}


export default async function createStartScene(engine: Engine) {
  interface SceneData {
    scene: Scene;
    light?: HemisphericLight;
    ground?: PhysicsAggregate;
    camera?: Camera;
    box1?:PhysicsAggregate;
    box2?:PhysicsAggregate;
  }

  let that: SceneData = { scene: new Scene(engine) };

  let initializedHavok: any;

  HavokPhysics().then((havok) => {
    initializedHavok = havok;
  });

  const havokInstance: HavokPhysicsWithBindings = await HavokPhysics();
  const hk: HavokPlugin = new HavokPlugin(true, havokInstance);
  that.scene.enablePhysics(new Vector3(0, -9.81, 0), hk);

  //that.scene.debugLayer.show();

  that.light = createLight(that.scene);
  that.ground = createGround(that.scene);
  that.camera = createArcRotateCamera(that.scene);
  that.box1 = createBox1(that.scene);
  that.box2 = createBox2(that.scene);
  const assetsManager = addAssets(that.scene);
  assetsManager.load();
  return that;
}
```

When collisions happen a response should be displayed in the GUI.  For this purpose a fuction  setText(newtext: string, index: number) is added to the gui.ts code.  So the full listing of gui.ts is:

**gui.ts**
```javascript
import { Scene } from "@babylonjs/core";
import {
  Button,
  AdvancedDynamicTexture,
  TextBlock,
  Control,
  Grid,
  Rectangle,
} from "@babylonjs/gui/2D";

var text1!: TextBlock; // modified by setText(newtext: string, index: number)
var text2!: TextBlock; // modified by setText(newtext: string, index: number)
var text3!: TextBlock; // modified by setText(newtext: string, index: number)
var text4!: TextBlock; // modified by setText(newtext: string, index: number)
var heading1!: TextBlock;

function createSceneButton(
  name: string,
  index: string,
  x: string,
  y: string
  //advtex: { addControl: (arg0: Button) => void }
) {
  var button: Button = Button.CreateSimpleButton(name, index);
  button.left = x;
  button.top = y;
  button.width = "180px";
  button.height = "35px";
  button.color = "white";
  button.cornerRadius = 20;
  button.background = "green";

  button.onPointerClickObservable.add(function () {
    console.log("click event");
    let toggle: string =
      button.textBlock!.text == "clicked" ? "Click me!" : "clicked";
    button.textBlock!.text = toggle;
    console.log(toggle);
  });
  return button;
}

function createTextBlock(
  name: string,
  index: string,
  left: string,
  top: string
) {
  let text: TextBlock = new TextBlock(name, index);
  text.text = index;
  text.color = "white";
  text.fontSize = 24;
  text.left = left;
  text.top = top;
  text.width = "200px";
  text.height = "46px";
  text.fontFamily = "Verdana";
  text.textWrapping = true;
  text.highlightColor = "red";
  text.horizontalAlignment = TextBlock.HORIZONTAL_ALIGNMENT_CENTER;
  text.verticalAlignment = TextBlock.VERTICAL_ALIGNMENT_CENTER;
  // event handling
  text.onPointerEnterObservable.add(function () {
    text.isHighlighted = true;
  });
  text.onPointerOutObservable.add(function () {
    text.isHighlighted = false;
  });
  return text;
}

export function gui(scene: Scene): void {
  // add a button
  //https://doc.babylonjs.com/typedoc/modules/BABYLON.GUI  // GUI API

  let advancedTexture: AdvancedDynamicTexture =
    AdvancedDynamicTexture.CreateFullscreenUI("myUI", true, scene);
  let button1: Button = createSceneButton(
    "button1",
    "Click Me!",
    "0px",
    "0px"
    //advancedTexture
  );
  //advancedTexture.addControl(button1); // button 1 could be added to the scene or grid

  //add text block
  //https://playground.babylonjs.com/#2ARI2W#10 //high resolution text//
  scene.getEngine().setHardwareScalingLevel(1 / window.devicePixelRatio);
  advancedTexture.rootContainer.scaleX = window.devicePixelRatio;
  advancedTexture.rootContainer.scaleY = window.devicePixelRatio;

  heading1 = createTextBlock("heading1", "Hello World", "1px", "1px");
  text1 = createTextBlock("text1", "Debug", "1px", "1px");
  text2 = createTextBlock("text2", "Debug", "1px", "1px");
  text3 = createTextBlock("text3", "Debug", "1px", "1px");
  text4 = createTextBlock("text4", "Debug", "1px", "1px");

  // advancedTexture.addControl(this.heading1); // text1 block could be added to the scene or grid

  //https://doc.babylonjs.com/features/featuresDeepDive/gui/gui#grid
  // Create a grid, Pointer will then only apply to the grid and not the whole screen.

  const grid = new Grid();
  grid.addColumnDefinition(100, true);
  grid.addColumnDefinition(0.25);
  grid.addColumnDefinition(0.25);
  grid.addColumnDefinition(0.25);
  grid.addColumnDefinition(0.25);
  grid.addColumnDefinition(100, true);
  grid.addRowDefinition(50, true);
  grid.addRowDefinition(50, true);

  // This rect will be on first row and second column
  const rect1 = new Rectangle();
  rect1.background = "#76d56e88"; //rgba
  rect1.thickness = 0;
  rect1.addControl(heading1); // rect is a container which can contain other controls
  const rect2 = new Rectangle();
  rect2.background = "#60955b88";
  rect2.thickness = 0;
  rect2.addControl(button1);
  const rect3 = new Rectangle();
  rect3.background = "#76d56e88";
  rect3.thickness = 0;
  //empty rect
  const rect4 = new Rectangle();
  rect4.background = "#60955b88";
  rect4.thickness = 0;
  //empty rect
  const rect5 = new Rectangle();
  rect5.background = "#76d56e88";
  rect5.thickness = 0;
  rect5.addControl(text1);
  const rect6 = new Rectangle();
  rect6.background = "#60955b88";
  rect6.thickness = 0;
  rect6.addControl(text2);
  const rect7 = new Rectangle();
  rect7.background = "#76d56e88";
  rect7.thickness = 0;
  rect7.addControl(text3);
  const rect8 = new Rectangle();
  rect8.background = "#60955b88";
  rect8.thickness = 0;
  rect8.addControl(text4);

  grid.addControl(rect1, 0, 1);
  grid.addControl(rect2, 0, 2);
  grid.addControl(rect3, 0, 3);
  grid.addControl(rect4, 0, 4);
  grid.addControl(rect5, 1, 1);
  grid.addControl(rect6, 1, 2);
  grid.addControl(rect7, 1, 3);
  grid.addControl(rect8, 1, 4);

  advancedTexture.addControl(grid);

  scene.registerBeforeRender(() => {});
}

export function setText(newtext: string, index: number) {
  switch (index) {
    case 1:
      text1.text = newtext;
      break;
    case 2:
      text2.text = newtext;
      break;
    case 3:
      text3.text = newtext;
      break;
    case 4:
      text4.text = newtext;
      break;

    default:
    // code block
  }
}
```
All the physics aggregated added to the scene are able to trigger collisions.  To enable the various collisions to be sorted out,collision filter groups are used.  Each physics aggregate is assigned a filter membership mask and a filter collide mask.  The membership mask defines what group the object belongs to, and the collide mask defines what groups it will collide with.

The filterMembershipMask is used to define what filter group the aggregate belongs to.  The filterCollideMask is used to define what filter groups the aggregate will collide with.

When a collision occurs the collision callback function is called. 

Each of the colliding aggregates has an collision obserbable and different callback functions can be added to each observable so that different responses can be generated by different collision events.  In this example, the collision callback functions are named collideCB and collideCB1 and correspond to collisions involving the ground and the boxes.

**collisions.ts**
```javascript
import { PhysicsAggregate } from "@babylonjs/core";
import { SceneData } from "./interfaces";
import { gui, setText} from "./gui";

// Collision callback function
const collideCB = (collision: {
  collider: { transformNode: { name: any } };
  collidedAgainst: { transformNode: { name: any } };
  point: any;
  distance: any;
  impulse: any;
  normal: any;
}): void => {
  console.log(
    "collideCB",
    collision.collider.transformNode.name,
    collision.collidedAgainst.transformNode.name
  );
};

const collideCB1 = (collision: {
  collider: { transformNode: { name: any } };
  collidedAgainst: { transformNode: { name: any } };
  point: any;
  distance: any;
  impulse: any;
  normal: any;
}): void => {
  console.log(
    "collideCB1",
    collision.collider.transformNode.name,
    collision.collidedAgainst.transformNode.name
  );
  setText(collision.collider.transformNode.name,1);
  setText(collision.collidedAgainst.transformNode.name,2);
  setText(collision.point.x.toFixed(2),3)
  setText(collision.point.z.toFixed(2),4);

};

export function setupCollisions(sceneData: SceneData): void {
  // Collision filter groups
  const FILTER_GROUP_GROUND = 1;
  const FILTER_GROUP_PLATFORM = 2;
  const FILTER_GROUP_CUBE = 3;
  const FILTER_GROUP_OBSTACLE = 4;
  const FILTER_GROUP_PLAYER = 5;

  // Apply masks and collisions to physics agggregates
  if (sceneData.ground) {
    sceneData.ground.shape.filterMembershipMask = FILTER_GROUP_GROUND;
    sceneData.ground.shape.filterCollideMask = FILTER_GROUP_CUBE | FILTER_GROUP_GROUND;
    sceneData.ground.body.getCollisionObservable().add(collideCB1);
  }

  if (sceneData.box1) {
    sceneData.box1.shape.filterMembershipMask = FILTER_GROUP_CUBE;
        sceneData.box1.shape.filterCollideMask = FILTER_GROUP_CUBE | FILTER_GROUP_GROUND;
    sceneData.box1.body?.getEventMask();
    sceneData.box1.body?.getCollisionObservable().add(collideCB);
  }

  if (sceneData.box2) {
    sceneData.box2.shape.filterMembershipMask = FILTER_GROUP_CUBE;
        sceneData.box2.shape.filterCollideMask = FILTER_GROUP_CUBE | FILTER_GROUP_GROUND;
    sceneData.box2.body?.getEventMask();
    sceneData.box2.body?.getCollisionObservable().add(collideCB);
  }
}
```

Then interfaces.ts keeps up

** interfaces.ts **
```javascript
import {
  Scene,
  Mesh,
  HemisphericLight,
  Camera,
  PhysicsAggregate,
} from "@babylonjs/core";

export interface SceneData {
      scene: Scene;
      light?: HemisphericLight;
      ground?: PhysicsAggregate;
      camera?: Camera;
      box1?:PhysicsAggregate;
      box2?:PhysicsAggregate;
}
```

index.ts runs it all

** index.ts **
```javascript
import { Engine } from "@babylonjs/core";
import createStartScene from "./createStartScene";
import './main.css';
import {createCharacterController} from "./createCharacterController";
import { gui } from "./gui";
import { setupCollisions } from "./collisions";
import { SceneData } from "./interfaces";

const CanvasName = "renderCanvas";

let canvas = document.createElement("canvas");
canvas.id = CanvasName;

canvas.classList.add("background-canvas");
document.body.appendChild(canvas);

let eng = new Engine(canvas, true, {}, true);

(async function main() {
    const startScene = await createStartScene(eng);
    createCharacterController(startScene.scene);
    setupCollisions(startScene);
    gui(startScene.scene);
    eng.runRenderLoop(() => {
        startScene.scene.render();
    });
})();
```


<iframe 
    height="460" 
    width="100%" 
    scrolling="no" 
    title="Player 04" 
    src="Block_3/section_8b/player04build/index.html" 
    frameborder="no" 
    loading="lazy" 
    allowtransparency="true" 
    allowfullscreen="true">
</iframe>



