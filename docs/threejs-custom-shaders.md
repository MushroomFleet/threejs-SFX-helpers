# Three.js Custom Shader Passes - Developer Guide

A practical guide to creating custom post-processing effects using GLSL shaders in Three.js.

## Table of Contents
- [Why Custom Shaders?](#why-custom-shaders)
- [The Shader Workflow](#the-shader-workflow)
- [Complete Working Example: Solarization Effect](#complete-working-example-solarization-effect)
- [Understanding the Code](#understanding-the-code)
- [Common Shader Patterns](#common-shader-patterns)
- [Animation with Uniforms](#animation-with-uniforms)
- [Advanced Techniques](#advanced-techniques)

---

## Why Custom Shaders?

Custom shaders give you pixel-level control over your post-processing effects. While built-in passes are convenient, custom shaders let you:

- Create unique visual effects not available in Three.js
- Optimize performance for specific needs
- Combine multiple techniques in a single pass
- Access precise mathematical transformations
- Animate effects over time with full control

---

## The Shader Workflow

### Step 1: Understand the Architecture

```
Scene → RenderPass → [Your Custom ShaderPass] → Screen
                      ↓
                  Fragment Shader
                      ↓
                  (Process Each Pixel)
```

Each pixel of the rendered scene is processed by your fragment shader.

### Step 2: Core Components

1. **ShaderPass**: Three.js wrapper that applies your shader
2. **Fragment Shader (GLSL)**: Code that processes each pixel
3. **Uniforms**: Variables you can pass from JavaScript to GLSL
4. **tDiffuse**: The input texture (previous rendering pass)

### Step 3: Basic Structure

```javascript
import { ShaderPass } from 'three/examples/jsm/postprocessing/ShaderPass.js';

// Define your shader
const MyShader = {
    uniforms: {
        tDiffuse: { value: null },  // Input texture (required)
        // Your custom uniforms here
    },
    
    vertexShader: `
        varying vec2 vUv;
        void main() {
            vUv = uv;
            gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
        }
    `,
    
    fragmentShader: `
        uniform sampler2D tDiffuse;
        varying vec2 vUv;
        
        void main() {
            vec4 color = texture2D(tDiffuse, vUv);
            // Process color here
            gl_FragColor = color;
        }
    `
};

// Create and add the pass
const myPass = new ShaderPass(MyShader);
composer.addPass(myPass);
```

---

## Complete Working Example: Solarization Effect

Let's build a complete solarization effect with animated threshold control.

### Full Implementation

```javascript
import * as THREE from 'three';
import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer.js';
import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass.js';
import { ShaderPass } from 'three/examples/jsm/postprocessing/ShaderPass.js';

// Define the Solarization Shader
const SolarizeShader = {
    uniforms: {
        tDiffuse: { value: null },
        threshold: { value: 0.5 },
        intensity: { value: 1.0 },
        time: { value: 0.0 }
    },
    
    vertexShader: `
        varying vec2 vUv;
        
        void main() {
            vUv = uv;
            gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
        }
    `,
    
    fragmentShader: `
        uniform sampler2D tDiffuse;
        uniform float threshold;
        uniform float intensity;
        uniform float time;
        
        varying vec2 vUv;
        
        void main() {
            // Sample the input texture
            vec4 color = texture2D(tDiffuse, vUv);
            
            // Calculate luminance (brightness)
            float luma = dot(color.rgb, vec3(0.299, 0.587, 0.114));
            
            // Solarization: invert colors above threshold
            vec3 solarized = color.rgb;
            if (luma > threshold) {
                solarized = 1.0 - color.rgb;
            }
            
            // Mix original with solarized based on intensity
            vec3 finalColor = mix(color.rgb, solarized, intensity);
            
            gl_FragColor = vec4(finalColor, color.a);
        }
    `
};

// Setup Scene, Camera, Renderer (your existing code)
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// Setup Post-Processing
const composer = new EffectComposer(renderer);

// Add render pass
const renderPass = new RenderPass(scene, camera);
composer.addPass(renderPass);

// Add custom solarization pass
const solarizePass = new ShaderPass(SolarizeShader);
composer.addPass(solarizePass);

// Animation Loop
function animate() {
    requestAnimationFrame(animate);
    
    // Update time uniform for potential animation
    solarizePass.uniforms['time'].value += 0.01;
    
    // Animate threshold (optional)
    solarizePass.uniforms['threshold'].value = 0.5 + Math.sin(Date.now() * 0.001) * 0.3;
    
    // Render with post-processing
    composer.render();
}

animate();

// Handle resize
window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
    composer.setSize(window.innerWidth, window.innerHeight);
});
```

---

## Understanding the Code

### Uniforms

Uniforms are variables passed from JavaScript to GLSL:

```javascript
uniforms: {
    tDiffuse: { value: null },      // Input texture (auto-populated)
    threshold: { value: 0.5 },       // Custom parameter
    intensity: { value: 1.0 },       // Mix amount
    time: { value: 0.0 }            // For animation
}
```

Access in JavaScript:
```javascript
solarizePass.uniforms['threshold'].value = 0.7;
```

### Vertex Shader

Usually stays the same for post-processing:

```glsl
varying vec2 vUv;  // Pass UV coordinates to fragment shader

void main() {
    vUv = uv;
    gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
}
```

- `vUv`: Texture coordinates (0,0 to 1,1)
- `varying`: Shared between vertex and fragment shader

### Fragment Shader

Where the magic happens - processes each pixel:

```glsl
uniform sampler2D tDiffuse;  // Input texture
varying vec2 vUv;            // UV coordinates from vertex shader

void main() {
    // Get the color at this pixel
    vec4 color = texture2D(tDiffuse, vUv);
    
    // Process the color
    // ...
    
    // Output final color
    gl_FragColor = vec4(finalColor, color.a);
}
```

---

## Common Shader Patterns

### 1. Pixelation Effect

```javascript
const PixelateShader = {
    uniforms: {
        tDiffuse: { value: null },
        resolution: { value: new THREE.Vector2(512, 512) },
        pixelSize: { value: 4.0 }
    },
    
    vertexShader: `
        varying vec2 vUv;
        void main() {
            vUv = uv;
            gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
        }
    `,
    
    fragmentShader: `
        uniform sampler2D tDiffuse;
        uniform vec2 resolution;
        uniform float pixelSize;
        varying vec2 vUv;
        
        void main() {
            // Calculate pixel grid
            vec2 dxy = pixelSize / resolution;
            vec2 coord = dxy * floor(vUv / dxy);
            
            // Sample at pixel center
            gl_FragColor = texture2D(tDiffuse, coord);
        }
    `
};
```

### 2. Scanline Effect

```javascript
const ScanlineShader = {
    uniforms: {
        tDiffuse: { value: null },
        lineCount: { value: 400.0 },
        intensity: { value: 0.3 },
        time: { value: 0.0 }
    },
    
    vertexShader: `
        varying vec2 vUv;
        void main() {
            vUv = uv;
            gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
        }
    `,
    
    fragmentShader: `
        uniform sampler2D tDiffuse;
        uniform float lineCount;
        uniform float intensity;
        uniform float time;
        varying vec2 vUv;
        
        void main() {
            vec4 color = texture2D(tDiffuse, vUv);
            
            // Create scanlines with optional scroll
            float scanline = sin((vUv.y + time * 0.1) * lineCount * 3.14159) * 0.5 + 0.5;
            
            // Apply scanline darkening
            color.rgb -= scanline * intensity;
            
            gl_FragColor = color;
        }
    `
};
```

### 3. RGB Shift (Chromatic Aberration)

```javascript
const RGBShiftShader = {
    uniforms: {
        tDiffuse: { value: null },
        amount: { value: 0.005 },
        angle: { value: 0.0 }
    },
    
    vertexShader: `
        varying vec2 vUv;
        void main() {
            vUv = uv;
            gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
        }
    `,
    
    fragmentShader: `
        uniform sampler2D tDiffuse;
        uniform float amount;
        uniform float angle;
        varying vec2 vUv;
        
        void main() {
            vec2 offset = vec2(cos(angle), sin(angle)) * amount;
            
            // Sample RGB channels at different positions
            float r = texture2D(tDiffuse, vUv + offset).r;
            float g = texture2D(tDiffuse, vUv).g;
            float b = texture2D(tDiffuse, vUv - offset).b;
            
            gl_FragColor = vec4(r, g, b, 1.0);
        }
    `
};
```

### 4. Noise Effect

```javascript
const NoiseShader = {
    uniforms: {
        tDiffuse: { value: null },
        time: { value: 0.0 },
        intensity: { value: 0.5 }
    },
    
    vertexShader: `
        varying vec2 vUv;
        void main() {
            vUv = uv;
            gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
        }
    `,
    
    fragmentShader: `
        uniform sampler2D tDiffuse;
        uniform float time;
        uniform float intensity;
        varying vec2 vUv;
        
        // Pseudo-random function
        float random(vec2 st) {
            return fract(sin(dot(st.xy, vec2(12.9898, 78.233))) * 43758.5453123);
        }
        
        void main() {
            vec4 color = texture2D(tDiffuse, vUv);
            
            // Generate time-based noise
            float noise = random(vUv + time) * intensity;
            
            // Add noise to color
            color.rgb += noise - intensity * 0.5;
            
            gl_FragColor = color;
        }
    `
};
```

---

## Animation with Uniforms

### Time-Based Animation

Update uniforms in your animation loop:

```javascript
function animate() {
    requestAnimationFrame(animate);
    
    // Update time uniform
    myPass.uniforms['time'].value += 0.016; // ~60fps
    
    composer.render();
}
```

### Sine Wave Animation

```javascript
function animate() {
    requestAnimationFrame(animate);
    
    const time = Date.now() * 0.001;
    
    // Oscillate between 0.3 and 0.7
    myPass.uniforms['intensity'].value = 0.5 + Math.sin(time) * 0.2;
    
    composer.render();
}
```

### Interactive Control

```javascript
// Mouse-based control
document.addEventListener('mousemove', (e) => {
    const x = e.clientX / window.innerWidth;
    const y = e.clientY / window.innerHeight;
    
    myPass.uniforms['threshold'].value = x;
    myPass.uniforms['intensity'].value = y;
});

// Keyboard control
document.addEventListener('keydown', (e) => {
    if (e.key === 'ArrowUp') {
        myPass.uniforms['intensity'].value += 0.1;
    } else if (e.key === 'ArrowDown') {
        myPass.uniforms['intensity'].value -= 0.1;
    }
});
```

---

## Advanced Techniques

### Multi-Pass Effects

Some effects need multiple passes:

```javascript
// Pass 1: Blur horizontally
const blurHPass = new ShaderPass(HorizontalBlurShader);
composer.addPass(blurHPass);

// Pass 2: Blur vertically
const blurVPass = new ShaderPass(VerticalBlurShader);
composer.addPass(blurVPass);
```

### Accessing Previous Frame

Store and access previous frame data:

```javascript
const FeedbackShader = {
    uniforms: {
        tDiffuse: { value: null },
        tPrevious: { value: null },
        blend: { value: 0.9 }
    },
    // ... (blend current with previous frame)
};
```

### Using Multiple Textures

```javascript
const CompositeShader = {
    uniforms: {
        tDiffuse: { value: null },
        tOverlay: { value: null },  // Second texture
        opacity: { value: 0.5 }
    },
    
    fragmentShader: `
        uniform sampler2D tDiffuse;
        uniform sampler2D tOverlay;
        uniform float opacity;
        varying vec2 vUv;
        
        void main() {
            vec4 base = texture2D(tDiffuse, vUv);
            vec4 overlay = texture2D(tOverlay, vUv);
            
            // Blend textures
            gl_FragColor = mix(base, overlay, opacity);
        }
    `
};

// Set the overlay texture
myPass.uniforms['tOverlay'].value = myTexture;
```

### Performance Optimization

```javascript
// Lower resolution for expensive effects
const lowResComposer = new EffectComposer(renderer);
lowResComposer.setSize(window.innerWidth * 0.5, window.innerHeight * 0.5);

// Conditional rendering
if (needsExpensiveEffect) {
    expensivePass.enabled = true;
} else {
    expensivePass.enabled = false;
}
```

---

## GLSL Quick Reference

### Data Types

```glsl
float a = 1.0;           // Floating point
int b = 1;               // Integer
bool c = true;           // Boolean
vec2 d = vec2(1.0, 2.0); // 2D vector
vec3 e = vec3(1.0, 2.0, 3.0);  // 3D vector (RGB)
vec4 f = vec4(1.0, 2.0, 3.0, 1.0);  // 4D vector (RGBA)
```

### Common Functions

```glsl
// Math
sin(x), cos(x), tan(x)
abs(x), sign(x)
min(a, b), max(a, b), clamp(x, min, max)
mix(a, b, t)  // Linear interpolation
smoothstep(edge0, edge1, x)

// Vector
dot(a, b)     // Dot product
length(v)     // Vector length
normalize(v)  // Unit vector
distance(a, b)

// Texture
texture2D(sampler, uv)  // Sample texture
```

### Swizzling

```glsl
vec4 color = vec4(1.0, 0.5, 0.0, 1.0);
vec3 rgb = color.rgb;    // Get RGB
float r = color.r;       // Get red channel
vec2 rg = color.rg;      // Get red & green
color.rgb = vec3(0.0);   // Set RGB to black
```

---

## Debugging Shaders

### Visualize Values

Output values as colors to see what's happening:

```glsl
// Visualize UV coordinates
gl_FragColor = vec4(vUv, 0.0, 1.0);

// Visualize a float value (0 to 1)
float value = someCalculation();
gl_FragColor = vec4(vec3(value), 1.0);

// Visualize with color coding
if (value > threshold) {
    gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0);  // Red
} else {
    gl_FragColor = vec4(0.0, 1.0, 0.0, 1.0);  // Green
}
```

### Browser Console

Check for shader compilation errors in browser console:

```javascript
// Three.js will log shader errors automatically
// Look for: "THREE.WebGLProgram: shader error..."
```

---

## Best Practices

1. **Start Simple**: Begin with basic color manipulation before complex effects
2. **Test Incrementally**: Add one feature at a time to isolate issues
3. **Normalize Values**: Keep shader values between 0.0 and 1.0 when possible
4. **Use Uniforms**: Don't hardcode values - make them adjustable
5. **Optimize**: Minimize texture lookups and calculations
6. **Document**: Comment your shader code, especially math
7. **Reuse**: Build a library of shader snippets for common operations

---

## Resources

- [The Book of Shaders](https://thebookofshaders.com/) - Excellent GLSL learning resource
- [Shader Toy](https://www.shadertoy.com/) - Community shader examples
- [Three.js Built-in Shaders](https://github.com/mrdoob/three.js/tree/dev/examples/jsm/shaders) - Reference implementations
- [WebGL Reference Card](https://www.khronos.org/files/webgl/webgl-reference-card-1_0.pdf) - GLSL syntax reference

---

## Next Steps

1. Experiment with the example shaders provided
2. Modify parameters and observe the results
3. Combine multiple effects in creative ways
4. Build your own shader library
5. Share your creations!

**Happy Shader Coding! 🎨✨**
