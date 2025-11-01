# Three.js Fragment Shader Examples

A collection of 7 custom GLSL fragment shaders for testing with your Three.js shader app.

**Note:** These are fragment shaders only. Your app should handle the vertex shader and uniforms setup.

---

## 1. Sepia Tone

Creates a warm, vintage photograph effect.

### Shader Code

```glsl
uniform sampler2D tDiffuse;
varying vec2 vUv;

void main() {
  vec4 color = texture2D(tDiffuse, vUv);
  
  // Sepia tone transformation
  float r = color.r * 0.393 + color.g * 0.769 + color.b * 0.189;
  float g = color.r * 0.349 + color.g * 0.686 + color.b * 0.168;
  float b = color.r * 0.272 + color.g * 0.534 + color.b * 0.131;
  
  gl_FragColor = vec4(r, g, b, color.a);
}
```

### Usage Notes

- Classic sepia tone using standard transformation matrix
- Creates warm brown tones reminiscent of old photographs
- Adjust the matrix values for different warm/cool tones

---

## 2. Edge Detection (Sobel Filter)

Highlights edges in the scene using convolution - perfect for stylized, toon-like, or technical visualizations.

### Shader Code

```glsl
uniform sampler2D tDiffuse;
varying vec2 vUv;

void main() {
  // Fixed texel size (adjust if needed)
  vec2 texel = vec2(0.001, 0.001);
  
  // Sample surrounding pixels
  float tl = dot(texture2D(tDiffuse, vUv + vec2(-texel.x, texel.y)).rgb, vec3(0.299, 0.587, 0.114));
  float t  = dot(texture2D(tDiffuse, vUv + vec2(0.0, texel.y)).rgb, vec3(0.299, 0.587, 0.114));
  float tr = dot(texture2D(tDiffuse, vUv + vec2(texel.x, texel.y)).rgb, vec3(0.299, 0.587, 0.114));
  
  float l  = dot(texture2D(tDiffuse, vUv + vec2(-texel.x, 0.0)).rgb, vec3(0.299, 0.587, 0.114));
  float r  = dot(texture2D(tDiffuse, vUv + vec2(texel.x, 0.0)).rgb, vec3(0.299, 0.587, 0.114));
  
  float bl = dot(texture2D(tDiffuse, vUv + vec2(-texel.x, -texel.y)).rgb, vec3(0.299, 0.587, 0.114));
  float b  = dot(texture2D(tDiffuse, vUv + vec2(0.0, -texel.y)).rgb, vec3(0.299, 0.587, 0.114));
  float br = dot(texture2D(tDiffuse, vUv + vec2(texel.x, -texel.y)).rgb, vec3(0.299, 0.587, 0.114));
  
  // Sobel operator
  float sobelX = -tl - 2.0*l - bl + tr + 2.0*r + br;
  float sobelY = -tl - 2.0*t - tr + bl + 2.0*b + br;
  
  float edge = sqrt(sobelX * sobelX + sobelY * sobelY);
  edge *= 3.0; // Edge strength
  
  gl_FragColor = vec4(vec3(edge), 1.0);
}
```

### Usage Notes

- Uses Sobel operator for gradient detection
- White lines = edges, black = flat areas
- Adjust `texel` size for thicker/thinner edges
- Can invert: `vec3(1.0 - edge)` for black lines on white
- Change `edge *= 3.0;` to adjust intensity

---

## 3. Simple Bloom/Glow

Makes bright areas glow and emit light - perfect for sci-fi, neon, and magical effects.

### Shader Code

```glsl
uniform sampler2D tDiffuse;
varying vec2 vUv;

void main() {
  vec4 color = texture2D(tDiffuse, vUv);
  
  // Calculate brightness
  float brightness = dot(color.rgb, vec3(0.299, 0.587, 0.114));
  
  // Extract bright areas using smoothstep (threshold at 0.7)
  float bloomAmount = smoothstep(0.7, 1.0, brightness);
  vec3 bloom = color.rgb * bloomAmount * 2.0;
  
  // Add bloom to original
  gl_FragColor = vec4(color.rgb + bloom, color.a);
}
```

### Usage Notes

- Bright areas (above 0.7) glow smoothly
- Change `0.7` to adjust brightness threshold
- Change `* 2.0` to adjust glow intensity
- Works great on emissive materials and lights

---

## 4. Kaleidoscope Effect

Creates mesmerizing mirror/symmetry patterns by manipulating UV coordinates in polar space.

### Shader Code

```glsl
uniform sampler2D tDiffuse;
varying vec2 vUv;

void main() {
  vec2 center = vec2(0.5, 0.5);
  vec2 uv = vUv - center;
  
  // Convert to polar coordinates
  float radius = length(uv);
  float theta = atan(uv.y, uv.x);
  
  // Create 6-segment kaleidoscope
  float segments = 6.0;
  float segment = 3.14159265 * 2.0 / segments;
  theta = mod(theta, segment);
  theta = abs(theta - segment * 0.5);
  
  // Convert back to cartesian
  vec2 newUv = vec2(cos(theta), sin(theta)) * radius + center;
  
  gl_FragColor = texture2D(tDiffuse, newUv);
}
```

### Usage Notes

- Creates 6-segment mirror pattern
- Change `segments = 6.0;` to other values (try 4.0, 8.0, 12.0)
- Even numbers create symmetrical patterns
- Odd numbers create more dynamic patterns

---

## 5. Posterization (Color Quantization)

Reduces colors to specific levels creating a graphic, poster-like, or retro video game aesthetic.

### Shader Code

```glsl
uniform sampler2D tDiffuse;
varying vec2 vUv;

void main() {
  vec4 color = texture2D(tDiffuse, vUv);
  
  // Quantize to 5 levels per channel
  float levels = 5.0;
  vec3 posterized = floor(color.rgb * levels) / levels;
  
  gl_FragColor = vec4(posterized, color.a);
}
```

### Usage Notes

- Reduces colors to 5 levels per channel
- Change `levels = 5.0;` to other values (try 3.0, 8.0, 16.0)
- Lower values = stronger poster effect
- Higher values = subtle quantization

---

## 6. Comic Book (Combined Effect)

Combines posterization and edge detection for a classic comic book / graphic novel look.

### Shader Code

```glsl
uniform sampler2D tDiffuse;
varying vec2 vUv;

void main() {
  vec4 color = texture2D(tDiffuse, vUv);
  
  // Step 1: Posterization
  float levels = 5.0;
  vec3 posterized = floor(color.rgb * levels) / levels;
  
  // Step 2: Edge Detection
  vec2 texel = vec2(0.001, 0.001);
  
  float tl = dot(texture2D(tDiffuse, vUv + vec2(-texel.x, texel.y)).rgb, vec3(0.299, 0.587, 0.114));
  float t  = dot(texture2D(tDiffuse, vUv + vec2(0.0, texel.y)).rgb, vec3(0.299, 0.587, 0.114));
  float tr = dot(texture2D(tDiffuse, vUv + vec2(texel.x, texel.y)).rgb, vec3(0.299, 0.587, 0.114));
  float l  = dot(texture2D(tDiffuse, vUv + vec2(-texel.x, 0.0)).rgb, vec3(0.299, 0.587, 0.114));
  float r  = dot(texture2D(tDiffuse, vUv + vec2(texel.x, 0.0)).rgb, vec3(0.299, 0.587, 0.114));
  float bl = dot(texture2D(tDiffuse, vUv + vec2(-texel.x, -texel.y)).rgb, vec3(0.299, 0.587, 0.114));
  float b  = dot(texture2D(tDiffuse, vUv + vec2(0.0, -texel.y)).rgb, vec3(0.299, 0.587, 0.114));
  float br = dot(texture2D(tDiffuse, vUv + vec2(texel.x, -texel.y)).rgb, vec3(0.299, 0.587, 0.114));
  
  float sobelX = -tl - 2.0*l - bl + tr + 2.0*r + br;
  float sobelY = -tl - 2.0*t - tr + bl + 2.0*b + br;
  float edge = sqrt(sobelX * sobelX + sobelY * sobelY);
  edge *= 3.0;
  
  // Combine: darken posterized image with edges
  vec3 final = posterized * (1.0 - edge);
  
  gl_FragColor = vec4(final, color.a);
}
```

### Usage Notes

- Classic comic book effect with bold colors and dark outlines
- Posterization creates flat color areas
- Edge detection adds black ink lines
- Adjust `levels` for more/fewer colors
- Adjust edge multiplier for thicker/thinner lines

---

## 7. Technical Visualization (Combined Effect)

Combines inverted edge detection and posterization for a technical blueprint or CAD-style look.

### Shader Code

```glsl
uniform sampler2D tDiffuse;
varying vec2 vUv;

void main() {
  vec4 color = texture2D(tDiffuse, vUv);
  
  // Step 1: Edge Detection
  vec2 texel = vec2(0.001, 0.001);
  
  float tl = dot(texture2D(tDiffuse, vUv + vec2(-texel.x, texel.y)).rgb, vec3(0.299, 0.587, 0.114));
  float t  = dot(texture2D(tDiffuse, vUv + vec2(0.0, texel.y)).rgb, vec3(0.299, 0.587, 0.114));
  float tr = dot(texture2D(tDiffuse, vUv + vec2(texel.x, texel.y)).rgb, vec3(0.299, 0.587, 0.114));
  float l  = dot(texture2D(tDiffuse, vUv + vec2(-texel.x, 0.0)).rgb, vec3(0.299, 0.587, 0.114));
  float r  = dot(texture2D(tDiffuse, vUv + vec2(texel.x, 0.0)).rgb, vec3(0.299, 0.587, 0.114));
  float bl = dot(texture2D(tDiffuse, vUv + vec2(-texel.x, -texel.y)).rgb, vec3(0.299, 0.587, 0.114));
  float b  = dot(texture2D(tDiffuse, vUv + vec2(0.0, -texel.y)).rgb, vec3(0.299, 0.587, 0.114));
  float br = dot(texture2D(tDiffuse, vUv + vec2(texel.x, -texel.y)).rgb, vec3(0.299, 0.587, 0.114));
  
  float sobelX = -tl - 2.0*l - bl + tr + 2.0*r + br;
  float sobelY = -tl - 2.0*t - tr + bl + 2.0*b + br;
  float edge = sqrt(sobelX * sobelX + sobelY * sobelY);
  edge *= 3.0;
  
  // Invert edges (black lines on white background)
  float invertedEdge = 1.0 - edge;
  
  // Step 2: Posterization on the edge result
  float levels = 3.0;
  float posterized = floor(invertedEdge * levels) / levels;
  
  gl_FragColor = vec4(vec3(posterized), color.a);
}
```

### Usage Notes

- Technical/blueprint aesthetic with clean lines
- Inverted edge detection creates white backgrounds
- Posterization adds distinct tonal steps
- Adjust `levels` for different contrast steps
- Perfect for architectural or technical visualizations

---

## Quick Start Tips

1. **Copy and paste** - Each shader is ready to use as-is
2. **Start simple** - Test shaders 1-5 individually first
3. **Try combined effects** - Shaders 6-7 show how to combine techniques in a single shader
4. **Stack effects** - Apply multiple shader passes in sequence for even more effects
5. **Modify values** - Adjust hardcoded numbers in the shader to experiment
6. **Performance** - Edge Detection is expensive (9 texture samples), combined effects are even more so

---

## Combining Effects

The document now includes two pre-built combined effects:

- **Comic Book** (Shader #6): Posterization + Edge Detection for graphic novel style
- **Technical Visualization** (Shader #7): Inverted Edge Detection + Posterization for blueprint style

Other powerful combinations to try by stacking multiple shader passes:

- **Psychedelic**: Kaleidoscope → Sepia
- **Neon Dreams**: Bloom → Kaleidoscope
- **Retro Game**: Posterization (low levels) → Bloom (subtle)
- **Vintage Tech**: Sepia → Edge Detection

Happy shader experimenting! 🎨✨
