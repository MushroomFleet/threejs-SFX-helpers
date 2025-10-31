# Three.js Post-Processing Effects - Quick Reference

A comprehensive guide to implementing built-in post-processing effects in Three.js using EffectComposer.

## Table of Contents
- [Setup: EffectComposer Basics](#setup-effectcomposer-basics)
- [FilmPass - Scanlines, Noise & Film Grain](#filmpass---scanlines-noise--film-grain)
- [DotScreenPass - Halftone/Pixelation](#dotscreenpass---halftonepixelation)
- [GlitchPass - Digital Glitch Effects](#glitchpass---digital-glitch-effects)
- [UnrealBloomPass - Glow Effects](#unrealbloompass---glow-effects)
- [AfterimagePass - Motion Blur/Trails](#afterimagepass---motion-blurtrails)
- [RenderPixelatedPass - Retro Pixelation](#renderpixelatedpass---retro-pixelation)
- [Combining Multiple Effects](#combining-multiple-effects)

---

## Setup: EffectComposer Basics

Before using any post-processing effects, you need to set up the EffectComposer pipeline.

### Step 1: Import Required Modules

```javascript
import * as THREE from 'three';
import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer.js';
import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass.js';
```

### Step 2: Create Composer and Base Render Pass

```javascript
// After creating your renderer, scene, and camera
const composer = new EffectComposer(renderer);

// First pass: render the scene normally
const renderPass = new RenderPass(scene, camera);
composer.addPass(renderPass);
```

### Step 3: Update Your Animation Loop

Replace `renderer.render(scene, camera)` with `composer.render()`:

```javascript
function animate() {
    requestAnimationFrame(animate);
    
    // Update your scene...
    
    // Render with post-processing
    composer.render();
}
```

### Step 4: Handle Window Resizing

```javascript
window.addEventListener('resize', () => {
    const width = window.innerWidth;
    const height = window.innerHeight;
    
    camera.aspect = width / height;
    camera.updateProjectionMatrix();
    
    renderer.setSize(width, height);
    composer.setSize(width, height);
});
```

---

## FilmPass - Scanlines, Noise & Film Grain

Creates vintage film effects with scanlines, random noise, and grayscale conversion.

### Import

```javascript
import { FilmPass } from 'three/examples/jsm/postprocessing/FilmPass.js';
```

### Basic Implementation

```javascript
// Parameters: noiseIntensity, scanlinesIntensity, scanlinesCount, grayscale
const filmPass = new FilmPass(
    0.35,    // noiseIntensity (0.0 - 1.0)
    0.5,     // scanlinesIntensity (0.0 - 1.0)
    648,     // scanlinesCount (number of lines)
    false    // grayscale (true/false)
);

composer.addPass(filmPass);
```

### Parameter Guide

| Parameter | Type | Range | Description |
|-----------|------|-------|-------------|
| noiseIntensity | float | 0.0 - 1.0 | Amount of random noise/grain |
| scanlinesIntensity | float | 0.0 - 1.0 | Visibility of horizontal scanlines |
| scanlinesCount | integer | 100 - 2000 | Number of scanline rows |
| grayscale | boolean | true/false | Convert to black & white |

### Common Presets

**Subtle VHS Effect:**
```javascript
const vhsPass = new FilmPass(0.2, 0.3, 512, false);
```

**Heavy Film Grain:**
```javascript
const grainPass = new FilmPass(0.8, 0.1, 1024, false);
```

**Old TV with Grayscale:**
```javascript
const tvPass = new FilmPass(0.5, 0.7, 648, true);
```

---

## DotScreenPass - Halftone/Pixelation

Creates a halftone printing effect with colored dots.

### Import

```javascript
import { DotScreenPass } from 'three/examples/jsm/postprocessing/DotScreenPass.js';
```

### Basic Implementation

```javascript
// Parameters: center (Vector2), angle (radians), scale
const dotScreenPass = new DotScreenPass(
    new THREE.Vector2(0.5, 0.5),  // center point
    0.5,                           // angle in radians
    1.0                            // scale (larger = bigger dots)
);

composer.addPass(dotScreenPass);
```

### Parameter Guide

| Parameter | Type | Description |
|-----------|------|-------------|
| center | Vector2 | Center point of the effect (0.0 - 1.0) |
| angle | float | Rotation angle of dot pattern in radians |
| scale | float | Dot size (0.1 = fine, 2.0 = coarse) |

### Common Presets

**Comic Book Style:**
```javascript
const comicPass = new DotScreenPass(
    new THREE.Vector2(0.5, 0.5),
    Math.PI / 4,
    0.8
);
```

**Newspaper Print:**
```javascript
const printPass = new DotScreenPass(
    new THREE.Vector2(0.5, 0.5),
    0,
    0.5
);
```

**Animated Rotation:**
```javascript
const dotPass = new DotScreenPass(
    new THREE.Vector2(0.5, 0.5),
    0,
    1.0
);

// In your animation loop
function animate() {
    dotPass.uniforms['angle'].value += 0.01;
    composer.render();
}
```

---

## GlitchPass - Digital Glitch Effects

Creates digital distortion and RGB channel separation effects.

### Import

```javascript
import { GlitchPass } from 'three/examples/jsm/postprocessing/GlitchPass.js';
```

### Basic Implementation

```javascript
const glitchPass = new GlitchPass();

// Optional: disable wild mode for more controlled glitching
glitchPass.goWild = false;

composer.addPass(glitchPass);
```

### Control Options

**Wild Mode (Random Intense Glitches):**
```javascript
glitchPass.goWild = true;  // Random, intense glitches
```

**Mild Mode (Subtle Occasional Glitches):**
```javascript
glitchPass.goWild = false;  // More controlled, subtle
```

### Triggering Glitches Manually

```javascript
// Trigger a glitch effect programmatically
function triggerGlitch() {
    glitchPass.generateTrigger();
}

// Example: glitch on user interaction
document.addEventListener('click', triggerGlitch);
```

### Custom Timing

```javascript
// Glitch every 2 seconds
setInterval(() => {
    glitchPass.generateTrigger();
}, 2000);
```

---

## UnrealBloomPass - Glow Effects

Creates bright, glowing bloom effects on luminous objects.

### Import

```javascript
import { UnrealBloomPass } from 'three/examples/jsm/postprocessing/UnrealBloomPass.js';
```

### Basic Implementation

```javascript
// Parameters: resolution, strength, radius, threshold
const bloomPass = new UnrealBloomPass(
    new THREE.Vector2(window.innerWidth, window.innerHeight),
    1.5,    // strength
    0.4,    // radius
    0.85    // threshold
);

composer.addPass(bloomPass);
```

### Parameter Guide

| Parameter | Type | Range | Description |
|-----------|------|-------|-------------|
| resolution | Vector2 | - | Render resolution (usually window size) |
| strength | float | 0.0 - 3.0 | Bloom intensity |
| radius | float | 0.0 - 1.0 | Bloom spread |
| threshold | float | 0.0 - 1.0 | Brightness threshold (higher = only bright objects glow) |

### Common Presets

**Subtle Glow:**
```javascript
const subtleBloom = new UnrealBloomPass(
    new THREE.Vector2(window.innerWidth, window.innerHeight),
    0.8,   // strength
    0.2,   // radius
    0.9    // threshold
);
```

**Intense Sci-Fi Glow:**
```javascript
const scifiBloom = new UnrealBloomPass(
    new THREE.Vector2(window.innerWidth, window.innerHeight),
    2.5,   // strength
    0.8,   // radius
    0.3    // threshold
);
```

**Animated Pulse:**
```javascript
const pulseBloom = new UnrealBloomPass(
    new THREE.Vector2(window.innerWidth, window.innerHeight),
    1.5, 0.4, 0.85
);

// In animation loop
function animate() {
    pulseBloom.strength = 1.0 + Math.sin(Date.now() * 0.001) * 0.5;
    composer.render();
}
```

---

## AfterimagePass - Motion Blur/Trails

Creates motion trails by blending previous frames with the current frame.

### Import

```javascript
import { AfterimagePass } from 'three/examples/jsm/postprocessing/AfterimagePass.js';
```

### Basic Implementation

```javascript
// Parameter: damp (0.0 - 1.0, higher = longer trails)
const afterimagePass = new AfterimagePass(0.92);

composer.addPass(afterimagePass);
```

### Parameter Guide

| Parameter | Type | Range | Description |
|-----------|------|-------|-------------|
| damp | float | 0.0 - 1.0 | Trail persistence (0.95 = long trails, 0.5 = short) |

### Common Presets

**Subtle Motion Blur:**
```javascript
const motionBlur = new AfterimagePass(0.8);
```

**Long Ghost Trails:**
```javascript
const ghostTrails = new AfterimagePass(0.96);
```

**Speed Effect:**
```javascript
const speedLines = new AfterimagePass(0.85);
```

### Clearing Trails

```javascript
// Reset/clear accumulated trails
afterimagePass.uniforms['damp'].value = 0;  // Clear
// Then restore
afterimagePass.uniforms['damp'].value = 0.92;
```

---

## RenderPixelatedPass - Retro Pixelation

Creates authentic retro-style pixelation by rendering at lower resolution.

### Import

```javascript
import { RenderPixelatedPass } from 'three/examples/jsm/postprocessing/RenderPixelatedPass.js';
```

### Basic Implementation

```javascript
// Parameters: resolution (pixels), scene, camera
const pixelPass = new RenderPixelatedPass(
    4,      // pixelSize (higher = more pixelated)
    scene,
    camera
);

composer.addPass(pixelPass);
```

### Important Notes

- This pass **replaces** RenderPass, don't use both
- Renders scene at lower resolution then scales up

### Setup Example

```javascript
// DON'T use RenderPass with RenderPixelatedPass
// const renderPass = new RenderPass(scene, camera);  // SKIP THIS

const composer = new EffectComposer(renderer);

const pixelPass = new RenderPixelatedPass(6, scene, camera);
pixelPass.normalEdgeStrength = 0.3;  // Edge detection
pixelPass.depthEdgeStrength = 0.4;
composer.addPass(pixelPass);
```

### Parameter Guide

| Parameter | Type | Description |
|-----------|------|-------------|
| pixelSize | integer | Pixel size (2 = subtle, 8 = very blocky) |
| normalEdgeStrength | float | Edge outline intensity (0.0 - 1.0) |
| depthEdgeStrength | float | Depth-based edge intensity (0.0 - 1.0) |

### Common Presets

**Retro Game (Low-Res):**
```javascript
const retroPass = new RenderPixelatedPass(8, scene, camera);
retroPass.normalEdgeStrength = 0;
retroPass.depthEdgeStrength = 0;
```

**Pixel Art with Outlines:**
```javascript
const pixelArtPass = new RenderPixelatedPass(4, scene, camera);
pixelArtPass.normalEdgeStrength = 0.5;
pixelArtPass.depthEdgeStrength = 0.5;
```

---

## Combining Multiple Effects

Stack multiple passes to create complex visual styles.

### Example 1: VHS Aesthetic

```javascript
const composer = new EffectComposer(renderer);

// Base render
const renderPass = new RenderPass(scene, camera);
composer.addPass(renderPass);

// Add scanlines and noise
const filmPass = new FilmPass(0.35, 0.5, 512, false);
composer.addPass(filmPass);

// Slight bloom for glow
const bloomPass = new UnrealBloomPass(
    new THREE.Vector2(window.innerWidth, window.innerHeight),
    0.6, 0.3, 0.9
);
composer.addPass(bloomPass);

// Occasional glitches
const glitchPass = new GlitchPass();
glitchPass.goWild = false;
composer.addPass(glitchPass);
```

### Example 2: Cyberpunk Style

```javascript
const composer = new EffectComposer(renderer);

const renderPass = new RenderPass(scene, camera);
composer.addPass(renderPass);

// Heavy bloom
const bloomPass = new UnrealBloomPass(
    new THREE.Vector2(window.innerWidth, window.innerHeight),
    2.0, 0.6, 0.4
);
composer.addPass(bloomPass);

// RGB separation glitches
const glitchPass = new GlitchPass();
composer.addPass(glitchPass);

// Motion trails
const afterimagePass = new AfterimagePass(0.88);
composer.addPass(afterimagePass);
```

### Example 3: Comic Book Style

```javascript
const composer = new EffectComposer(renderer);

const renderPass = new RenderPass(scene, camera);
composer.addPass(renderPass);

// Halftone dots
const dotPass = new DotScreenPass(
    new THREE.Vector2(0.5, 0.5),
    0,
    0.6
);
composer.addPass(dotPass);
```

### Pass Order Matters!

The order of passes affects the final result:

```javascript
// Order 1: Film grain on top of everything
composer.addPass(renderPass);
composer.addPass(bloomPass);
composer.addPass(filmPass);  // Grain applied last

// Order 2: Film grain before bloom (different look)
composer.addPass(renderPass);
composer.addPass(filmPass);
composer.addPass(bloomPass);  // Bloom on grainy image
```

---

## Performance Tips

1. **Limit Active Passes**: Each pass adds rendering overhead
2. **Disable Unused Passes**: Use `pass.enabled = false` instead of removing
3. **Resolution Scaling**: Lower composer resolution for better performance:
   ```javascript
   composer.setSize(window.innerWidth * 0.5, window.innerHeight * 0.5);
   ```
4. **Conditional Effects**: Enable expensive effects only when needed:
   ```javascript
   if (needsBloom) {
       bloomPass.enabled = true;
   }
   ```

---

## Troubleshooting

**Black Screen:**
- Ensure `composer.render()` is called instead of `renderer.render()`
- Check that RenderPass is added first

**Effects Not Visible:**
- Verify pass is added: `composer.addPass(pass)`
- Check pass parameters aren't at zero/default
- Ensure pass comes after RenderPass

**Pixelated Output:**
- Match composer size to renderer size on resize
- Check renderer pixel ratio: `renderer.setPixelRatio(window.devicePixelRatio)`

**Transparent Background Issues:**
- Set renderer alpha: `new THREE.WebGLRenderer({ alpha: true })`
- Use appropriate clear settings

---

## Additional Resources

- [Three.js Examples - Post-Processing](https://threejs.org/examples/?q=postprocess)
- [Three.js Docs - EffectComposer](https://threejs.org/docs/#examples/en/postprocessing/EffectComposer)
- [Shader Reference for Custom Effects](./threejs-custom-shaders.md)

---

**Happy Effect Processing! 🎨✨**
