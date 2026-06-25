# Materials & Texture

Materials are applied to objects in a BabylonJS scene and textures are then added to these materials.

There are two types of materials:
1. BabylonJS features some [standard materials](https://doc.babylonjs.com/features/featuresDeepDive/materials/using/masterPBR/#introduction) which can be used with textures
2. Physically based rendering materials: [PBR materials](https://doc.babylonjs.com/features/featuresDeepDive/materials/using/masterPBR/#introduction)
   * These are more advanced and give extra control over material features such as Refraction, Standard Light Falloff, LightMaps and Dedicated image processing
    
A smple set of textures are available in the BabylonJS [texture library](https://doc.babylonjs.com/toolsAndResources/assetLibraries/availableTextures/). Of these the Diffuse/Albedo maps are the ones relevant to this section.

## Standard materials

The standard materials interact with lighting in to generate colour in a scene.

All types of ligt have shared parameters which include the diffuse color and the specular color.  

* The diffuse light tints the base color of all illuminated surfaces and interacts with the diffuse colour and texture of a material.
* The specular light produces highlight reflections on *standard* materials. (Note that this is not used on PBR materials)

In addition, only the Hemispherical Light diffuse color sets the ambient color on the scene.  This is still named as diffuse light but you might find it useful to call it ambient light.

```javascript
// Create the hemispheric light
const light = new BABYLON.HemisphericLight("light", new BABYLON.Vector3(0, 1, 0), scene);

// Set the sky (diffuse) and ground colors to yellow to create a full yellow AMBIENT glow
light.diffuse = new BABYLON.Color3(1, 1, 0);       // Sky/Main glow
light.groundColor = new BABYLON.Color3(0.5, 0.5, 0); // Ground reflection
```

The standard material has color of its own added as a texture:
* The diffuse (albedo) texture material color interacts with the diffuse light illuminating it.
* The specular texture material color interacts with the specular light illuminating it.
* The ambient texture color interacts with the ambient (diffuse hemispherical) light illuminating it.
* The emissive texture color allows a material to glow with its own color.

Other textures which can be added to standard materials are 

* bumpTexture: Typically holds a normal map (or heightmap) to simulate intricate surface depth, cracks, and bumps without adding real geometry.  
* opacityTexture: A grayscale map used to control per-pixel alpha transparency (black is fully transparent, white is fully opaque).
* reflectionTexture: Used to project reflections onto the mesh, often via a CubeTexture or a mirror texture.
* refractionTexture: Simulates light warping through the object, ideal for glass or water effects.


   