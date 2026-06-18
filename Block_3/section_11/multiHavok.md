# Multiple scenes with Physics

The previous section covered how to create multiple scenes when each scene was simply created by a single createScene call. This section will cover how to create multiple scenes where a scene might depend on multiple files and include physics.

The output of the section Player04 will be adapted as the fourth scene in a multiple display.

Up to now the index.ts file has been responsible for creating scenes and rendering them.  This was done by calling createScene.ts.  However in this section the file start.ts will be called and this will be responsible for creating scenes depending on multiple files and including physics, and returning these to the index.ts file.

The start.ts file will be responsible for creating the scenes and returning them to the index.ts file.  The index.ts file will then be responsible for rendering these scenes. 

The structure of the multiHavok folder will now incude the full file sturcture of player04 but with the original index.ts replaced by start.ts.

When the start function is called it will create a scene with a character controller, collisions and a gui and return this to the index.ts file.

**start.ts**
```javascript
import { Engine } from "@babylonjs/core";
import createStartScene from "./createStartScene";
import './main.css';
import {createCharacterController} from "./createCharacterController";
import { gui } from "./gui";
import { setupCollisions } from "./collisions";

export default async function start(eng: Engine) {
 const startScene = await createStartScene(eng);
    createCharacterController(startScene.scene);
    setupCollisions(startScene);
    gui(startScene.scene);
    return startScene;
}
```

Note that the start function is asynchronous and await is used to wait for the createStartScene function to complete before continuing with the rest of the function. This is necessary because the createStartScene function is also asynchronous and returns a promise. The await keyword is used to wait for the promise to resolve before continuing with the rest of the function. This ensures that the rest of the function only runs after the createStartScene function has completed and the scene has been created.

Now scene[3] in index.ts is modified to call the start function instead of createStartScene.ts.

**index.ts**
```javascript
import { Engine} from "@babylonjs/core";
import createScene1  from "./scene1/createStartScene";
import createScene2  from "./scene2/createStartScene";
import createScene3  from "./scene3/createStartScene";
import start4  from "./scene4/src/start";
import menuScene from "./gui/guiScene";
import "./main.css";

const CanvasName = "renderCanvas";

let canvas = document.createElement("canvas");
canvas.id = CanvasName;

canvas.classList.add("background-canvas");
document.body.appendChild(canvas);

let scene;
let scenes: any[] = [];


let eng = new Engine(canvas, true, {}, true);
let gui = menuScene(eng);

(async function main(){
scenes[0] = createScene1(eng);
scenes[1] = createScene2(eng);
scenes[2] = createScene3(eng);
scenes[3] = await start4(eng);
scene = scenes[0].scene;
setSceneIndex(0);
})();

export default function setSceneIndex(i: number) {
  console.log("Switching to scene index:", i, scenes[i]);
  eng.stopRenderLoop();
  eng.runRenderLoop(() => {
      scenes[i].scene.render();
      gui.scene.autoClear = false;
      gui.scene.render();
  });
}   
```
Note that the calls are to uniqely named functions createScene1 - createScene4 and stat4.  This naming is added on import to avoid conflicts with other files.

In principle that should be enough to run any multifile scene, however some modifications are needed to prevent the gui in the scene from interfering with the gui in the menu.

This ammounts to the removal of the lines which saught to render the gui with high resolution.

**scene4/src/startgui.ts**
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
    AdvancedDynamicTexture.CreateFullscreenUI("scene4UI", true, scene);
  

  let button1: Button = createSceneButton(
    "button1",
    "Click Me!",
    "0px",
    "0px"
  );

  heading1 = createTextBlock("heading1", "Hello World", "1px", "1px");
  text1 = createTextBlock("text1", "Debug", "1px", "1px");
  text2 = createTextBlock("text2", "Debug", "1px", "1px");
  text3 = createTextBlock("text3", "Debug", "1px", "1px");
  text4 = createTextBlock("text4", "Debug", "1px", "1px");


  //https://doc.babylonjs.com/features/featuresDeepDive/gui/gui#grid
  // Create a grid, Pointer will then only apply to the grid and not the whole screen.

  const grid = new Grid();
  grid.width = "100%";
  grid.height = "100px";
  grid.verticalAlignment = Control.VERTICAL_ALIGNMENT_TOP;
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
    console.log("gui made")
  }
}
```

This now runs as intended.

When building there can only be one assets folder so the assets for the scene need to be moved into the common assets folder.  This assets folder must be copied to the Public folder prior to building.





