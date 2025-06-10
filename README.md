# GoatColorToolbox.js

A compact, performant, and dependency-free color utility library. It handles parsing and conversion for **Hex, RGB, HSL, and OKLCH**, including modern CSS Color Level 4 features. It also provides tools for validation, WCAG contrast checking, and color theory palette generation.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Features

-   **Robust Parsing**: Parses Hex, RGB(A), HSL(A), OKLCH(A), and CSS Named Colors.
-   **Modern & Legacy Syntax**: Understands modern `rgb(R G B / A)` and legacy `rgba(R, G, B, A)` formats.
-   **Full Conversion**: Convert any valid color to any supported format string (`toHex()`, `toRgbaString()`, `toOklchString()`) or object (`toRgb()`).
-   **Alpha Handling**: Smart alpha formatting that respects the original input style (e.g., `50%` vs `.5`).
-   **Color Palettes**: Generate palettes based on color theory (Monochromatic, Analogous, Complementary, and more).
-   **WCAG Contrast**: Calculate the contrast ratio between two colors.
-   **OKLCH Gamut Utility**: Find the maximum sRGB chroma for a given OKLCH Lightness and Hue, perfect for building color pickers.
-   **No Dependencies**: A single, self-contained file.

## Installation

#### Browser

Download `GoatColorToolbox.js` and include it with a script tag. The `GoatColor` factory will be available on the global `window` object.

```html
<script src="GoatColorToolbox.js"></script>
<script>
    const color = GoatColor('rebeccapurple');
    console.log(color.toHex()); // #663399
</script>
```

#### Node.js / Bundlers

```bash
npm install ./path/to/GoatColorToolbox.js
```
```javascript
const GoatColor = require('./GoatColorToolbox.js');
const color = GoatColor('rebeccapurple');
```

## Usage

The library exposes a single factory function, `GoatColor()`, which you use to create new color instances.

```javascript
// Create a color instance from a string
const myColor = GoatColor('hsl(270 50% 40%)');

// Check if the input was valid
if (myColor.isValid()) {
  // Convert to different formats
  console.log(myColor.toString());       // hsl(270 50% 40%)
  console.log(myColor.toHex());          // #663399
  console.log(myColor.toRgbString());    // rgb(102 51 153)
  console.log(myColor.toOklchString());  // oklch(45% 0.14 303)
} else {
  console.error(myColor.error); // "Unrecognized color format: ..."
}
```

---

## API Reference

### Parsing & Validation

#### `GoatColor(colorString)`
Creates a new color instance.
```javascript
GoatColor('#ff7f50');                      // Hex
GoatColor('rgb(255 127 80)');              // Modern RGB
GoatColor('hsla(19, 100%, 66%, .8)');       // Legacy HSLA with number alpha
GoatColor('oklch(75% 0.15 25 / 50%)');      // OKLCH with percent alpha
GoatColor('dodgerblue');                    // CSS Named Color
```

#### `color.isValid()`
Returns `true` if the input color string was successfully parsed.
```javascript
GoatColor('hotpink').isValid();   //=> true
GoatColor('not-a-color').isValid(); //=> false
```

#### `GoatColor.isValidColorString(string)`
A static method to quickly check if a string is a valid format without creating a full instance.
```javascript
GoatColor.isValidColorString('hsl(0 100% 50%)'); //=> true
```

### Output & Conversion

#### `color.toString(format?)`
Converts the color to a string. The default `'auto'` format intelligently picks the best representation (e.g., short hex, or matching the input's format family).

```javascript
const c1 = GoatColor('#f0c');
c1.toString(); // #f0c (short hex)

const c2 = GoatColor('oklch(70% 0.1 50 / 50%)');
c2.toString(); // oklch(70% 0.1 50 / 50%) (matches input family)

// You can also force a specific format
c2.toString('rgba'); // rgb(215 168 153 / 50%)
```

#### Format-specific String Conversions
Methods accept an optional `legacy=true` boolean to use comma-separated syntax.

```javascript
const color = GoatColor('rgba(255, 0, 0, 0.5)');

// Hex
color.toHex();       //=> #ff0000
color.toHexa();      //=> #ff000080
color.toHexShort();  //=> null (not possible for #ff0000)

// RGB
color.toRgbString();        //=> rgb(255 0 0)
color.toRgbaString();       //=> rgb(255 0 0 / .5) (modern syntax, respects input alpha style)
color.toRgbaString(true);   //=> rgba(255, 0, 0, .5) (legacy syntax)

// HSL
color.toHslString();        //=> hsl(0 100% 50%)
color.toHslaString();       //=> hsl(0 100% 50% / .5)
color.toHslaString(true);   //=> hsla(0, 100%, 50%, .5)

// OKLCH
color.toOklchString();      //=> oklch(63% 0.26 29)
color.toOklchaString();     //=> oklch(63% 0.26 29 / .5)
```

#### Object Conversions
Get raw channel values as a plain object.

```javascript
const color = GoatColor('deepskyblue');

color.toRgb();   //=> { r: 0, g: 191, b: 255 }
color.toHsla();  //=> { h: 195, s: 100, l: 50, a: 1 }
color.toOklch(); //=> { l: 79.51..., c: 0.16..., h: 222.68... }
```

### Color Manipulation

#### `color.setAlpha(value, styleHint?)`
Mutates the color instance to set a new alpha value (0-1). The `styleHint` ('percent' or 'number') influences future string outputs.

```javascript
const color = GoatColor('blue'); // a = 1
color.setAlpha(0.4, 'percent');
console.log(color.toRgbaString()); // rgb(0 0 255 / 40%)
```

#### `color.flatten(background)`
Removes transparency by blending the color against a background. Returns a new, fully opaque `GoatColor` instance. **If the background color is invalid, this method will return a new, invalid `GoatColor` instance.**

```javascript
const transparentRed = GoatColor('rgba(255, 0, 0, 0.5)');
const flatColor = transparentRed.flatten('white');

console.log(flatColor.toHex()); // #ff8080
```

### Color Theory Palettes

Generate arrays of new `GoatColor` instances, generally sorted by hue.

```javascript
const base = GoatColor('blue');

// Analogous: colors adjacent on the color wheel
const analogous = base.getAnalogousPalette();
console.log(analogous.map(c => c.toHex()));
//=> [ '#8000ff', '#0000ff', '#0080ff' ]

// Monochromatic with custom spread
const mono = base.getMonochromaticPalette(5, 0.95, 0.95); // Higher factors = more spread
console.log(mono.map(c => c.toHex()));
//=> [ '#000029', '#000080', '#0000ff', '#8787ff', '#d3d3ff' ]

// Other available palettes:
base.getMonochromaticPalette(count?, lightenFactor?, darkenFactor?);
base.getComplementaryPalette();
base.getSplitComplementaryPalette(angle?);
base.getTriadicPalette();
base.getTetradicPalette(offsetAngle?);
```

### Utilities

#### `GoatColor.getContrastRatio(color1, color2)`
Calculates the WCAG contrast ratio between two colors. Handles transparent colors by flattening them first.

```javascript
const ratio = GoatColor.getContrastRatio('navy', '#ffc0cb');
console.log(ratio); //=> 6.25

// AA Large Text: >= 3
// AA Small Text: >= 4.5
// AAA Large Text: >= 4.5
// AAA Small Text: >= 7
```

#### `GoatColor.getMaxSRGBChroma(l, h, maxChroma)`
A utility for UI color pickers. It finds the maximum chroma that is still representable in the sRGB gamut for a given OKLCH lightness (0-100) and hue (0-360).

```javascript
// For a greenish color at 70% lightness, what is the most vibrant I can get?
const maxChroma = GoatColor.getMaxSRGBChroma(70, 150, 0.4);
console.log(maxChroma.toFixed(4)); //=> 0.1611

const inGamutColor = GoatColor(`oklch(70% ${maxChroma} 150)`);
console.log(inGamutColor.toHex()); //=> #00b46f
```