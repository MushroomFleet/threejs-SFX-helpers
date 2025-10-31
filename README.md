# Three.js SFX Helpers 🎨✨

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Three.js](https://img.shields.io/badge/Three.js-r170+-blue.svg)](https://threejs.org/)
[![LLM-Friendly](https://img.shields.io/badge/LLM-Friendly-green.svg)](https://github.com/MushroomFleet/threejs-SFX-helpers)

> Comprehensive documentation and "cheat sheets" for implementing post-processing effects in Three.js — designed for developers working with LLMs (Claude, ChatGPT, etc.) and those exploring shader-based visual effects.

## 📖 Overview

This repository provides two essential reference guides for Three.js post-processing effects:

1. **Quick Reference Guide** — Complete implementation guide for built-in Three.js post-processing passes
2. **Custom Shaders Guide** — Starter documentation for creating custom GLSL shader effects

These documents are specifically formatted to be **LLM-friendly**, making it easy for AI assistants to help you implement effects quickly and correctly. They're also excellent standalone references for human developers.

## 📚 Documentation

### [`threejs-sfx-quick-reference.md`](./docs/threejs-sfx-quick-reference.md)

A comprehensive step-by-step guide covering all major built-in post-processing effects:

- **EffectComposer Setup** — Complete initialization workflow with resize handling
- **FilmPass** — VHS effects, scanlines, noise, and film grain with multiple presets
- **DotScreenPass** — Halftone/comic book printing effects
- **GlitchPass** — Digital corruption and RGB channel separation
- **UnrealBloomPass** — Glow and bloom effects for luminous objects
- **AfterimagePass** — Motion blur and ghost trails
- **RenderPixelatedPass** — Authentic retro pixelation
- **Combining Effects** — Real-world examples (VHS aesthetic, cyberpunk, comic book styles)
- **Performance Tips** — Optimization strategies
- **Troubleshooting** — Common issues and solutions

**Perfect for:** Quickly implementing professional post-processing effects without deep shader knowledge.

### [`threejs-custom-shaders.md`](./docs/threejs-custom-shaders.md)

A practical introduction to creating custom shader-based post-processing effects:

- **Complete Working Example** — Full solarization effect implementation
- **Shader Architecture** — Understanding the post-processing pipeline
- **Ready-to-Use Patterns** — Pixelation, scanlines, RGB shift, and noise effects
- **Animation Techniques** — Time-based and interactive uniform control
- **GLSL Fundamentals** — Quick reference for shader programming
- **Advanced Techniques** — Multi-pass effects, texture blending, optimization
- **Debugging Strategies** — Visualization and troubleshooting methods

**Perfect for:** Developers ready to create unique, custom visual effects with full pixel-level control.

## 🚀 Quick Start

### For Developers

1. Browse the documentation in the [`/docs`](./docs) folder
2. Copy the relevant code snippets into your Three.js project
3. Adjust parameters to achieve your desired effect
4. Refer to the troubleshooting sections if needed

### For LLM-Assisted Development

When working with AI assistants (Claude, ChatGPT, etc.), reference these documents to get accurate, copy-paste-ready code:

**Example prompts:**

```
"Using the threejs-sfx-quick-reference.md, implement a VHS aesthetic 
with scanlines and bloom in my Three.js scene"
```

```
"Based on the custom shaders guide, help me create a pixelation effect 
that I can control with mouse position"
```

```
"Show me how to combine FilmPass and GlitchPass according to the quick 
reference, and explain the pass order"
```

The structured format ensures AI assistants provide:
- ✅ Correct import statements
- ✅ Proper parameter usage
- ✅ Complete implementation code
- ✅ Best practices and gotchas

## 💡 Use Cases

### Built-in Effects (Quick Reference)
- **Game Development** — Retro aesthetics, VHS/CRT effects
- **Music Visualizers** — Bloom, trails, and glitch effects
- **Web Experiences** — Cinematic post-processing
- **Prototyping** — Quick visual enhancement

### Custom Shaders
- **Unique Visual Styles** — Create effects not available in Three.js
- **Brand-Specific Effects** — Custom color grading and treatments
- **Performance Optimization** — Combine multiple effects in one pass
- **Interactive Experiences** — Real-time parameter control

## 🛠️ Requirements

- **Three.js** r170 or later (most examples work with r128+)
- Basic understanding of Three.js scene setup
- For custom shaders: Willingness to learn GLSL (guides included!)

## 📋 Code Examples

### Quick Built-in Effect

```javascript
import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer.js';
import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass.js';
import { FilmPass } from 'three/examples/jsm/postprocessing/FilmPass.js';

const composer = new EffectComposer(renderer);
composer.addPass(new RenderPass(scene, camera));

// VHS effect with scanlines
const filmPass = new FilmPass(0.35, 0.5, 512, false);
composer.addPass(filmPass);

// In your animation loop
function animate() {
    requestAnimationFrame(animate);
    composer.render();
}
```

### Custom Shader Effect

```javascript
import { ShaderPass } from 'three/examples/jsm/postprocessing/ShaderPass.js';

const MyShader = {
    uniforms: {
        tDiffuse: { value: null },
        intensity: { value: 1.0 }
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
        uniform float intensity;
        varying vec2 vUv;
        
        void main() {
            vec4 color = texture2D(tDiffuse, vUv);
            gl_FragColor = vec4(color.rgb * intensity, color.a);
        }
    `
};

const myPass = new ShaderPass(MyShader);
composer.addPass(myPass);
```

## 🎯 Design Philosophy

These guides follow several key principles:

1. **LLM-Optimized Structure** — Clear headings, complete code blocks, and explicit parameter documentation
2. **Copy-Paste Ready** — All examples are fully functional without hidden dependencies
3. **Progressive Complexity** — Start simple, build to advanced techniques
4. **Real-World Examples** — Practical presets and combinations, not just theory
5. **Troubleshooting First** — Common issues addressed upfront

## 🤝 Contributing

Contributions are welcome! If you've discovered:

- New effect combinations
- Performance optimizations
- Additional shader patterns
- Clearer explanations
- Bug fixes or corrections

Please open an issue or submit a pull request.

## 📖 Additional Resources

- [Three.js Official Documentation](https://threejs.org/docs/)
- [Three.js Post-Processing Examples](https://threejs.org/examples/?q=postprocess)
- [The Book of Shaders](https://thebookofshaders.com/) — Learn GLSL
- [ShaderToy](https://www.shadertoy.com/) — Community shader examples

## 📄 License

MIT License - See [LICENSE](LICENSE) for details

## 📚 Citation

### Academic Citation
If you use these guides in your research, educational materials, or project documentation, please cite:

```bibtex
@software{threejs_sfx_helpers,
  title = {Three.js SFX Helpers: Comprehensive Post-Processing Reference for LLM-Assisted Development},
  author = {Drift Johnson},
  year = {2025},
  url = {https://github.com/MushroomFleet/threejs-SFX-helpers},
  version = {1.0.0}
}
```

### Support This Project

If these guides saved you time or helped your project, consider supporting continued development:

[![Ko-Fi](https://cdn.ko-fi.com/cdn/kofi3.png?v=3)](https://ko-fi.com/driftjohnson)

---

**Built with ❤️ for the Three.js and AI-assisted development community**
