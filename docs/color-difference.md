# Color differences

## Euclidean distance

We often need to determine the distance between two colors, for a variety of use cases.
Before most people dive into color science, when they are only familiar with sRGB colors,
their fist attempt to do so usually is the Euclidean distance of colors in sRGB,
like so: `sqrt((r₁ - r₂)² + (g₁ - g₂)² + (b₁ - b₂)²)`.
However, since sRGB is not [perceptually uniform](https://programmingdesignsystems.com/color/perceptually-uniform-color-spaces/),
pairs of colors with the same Euclidean distance can have hugely different perceptual differences:

<div style="background: hsl(30, 100%, 50%)"></div>
<div style="background: hsl(50, 100%, 50%)"></div>
<div style="background: hsl(230, 100%, 50%)"></div>
<div style="background: hsl(250, 100%, 50%)"></div>

```js
let color1 = new Color("hsl(30, 100%, 50%)");
let color2 = new Color("hsl(50, 100%, 50%)");
let color3 = new Color("hsl(230, 100%, 50%)");
let color4 = new Color("hsl(260, 100%, 50%)");
color1.distance(color2, "srgb");
color3.distance(color4, "srgb");
```

Notice that even though `color3` and `color4` are far closer than `color1` and `color2`, their sRGB Euclidean distance is slightly larger!

Euclidean distance *can* be very useful in calculating color difference, as long as the measurement is done in a perceptually uniform color space, such as Lab or Jzazbz:

```js
let color1 = new Color("hsl(30, 100%, 50%)");
let color2 = new Color("hsl(50, 100%, 50%)");
let color3 = new Color("hsl(230, 100%, 50%)");
let color4 = new Color("hsl(260, 100%, 50%)");
color1.distance(color2, "lab");
color3.distance(color4, "lab");

color1.distance(color2, "jzazbz");
color3.distance(color4, "jzazbz");
```

## Delta E (ΔE)

DeltaE (ΔE) is a family of algorithms specifically for calculating the difference (delta) between two colors.
The very first version of DeltaE, [DeltaE 1976](https://en.wikipedia.org/wiki/Color_difference#CIE76) was simply the Euclidean distance of the colors in Lab:

```js
let color1 = new Color("hsl(30, 100%, 50%)");
let color2 = new Color("hsl(50, 100%, 50%)");
let color3 = new Color("hsl(230, 100%, 50%)");
let color4 = new Color("hsl(260, 100%, 50%)");

color1.deltaE76(color2);
color3.deltaE76(color4);
```

However, because Lab turned out to not be as perceptually uniform as it was once thought, the algorithm was revised in 1984 (CMC), 1994, and lastly, 2000, with the most accurate and most complicated DeltaE algorithm to date.
Color.js supports all DeltaE algorithms except DeltaE 94. Each deltaE algorithm comes with its own method (e.g. `color1.deltaECMC(color2)`),
as well as a parameterized syntax (e.g. `color1.deltaE(color2, "CMC")`) which falls back to DeltaE 76 when the requested algorithm is not available, or the second argument is missing.

Note: If you are not using the Color.js bundle that includes everything, you will need to import the modules for DeltaE CMC and DeltaE 2000 manually. DeltaE 76 is supported in the core Color.js.

```js
// These are not needed if you're just using the bundle
// and will be omitted in other code examples
import "https://colorjs.io/src/deltaE/deltaECMC.js";
import "https://colorjs.io/src/deltaE/deltaE2000.js";

let color1 = new Color("blue");
let color2 = new Color("lab", [30, 30, 30]);
let color3 = new Color("lch", [40, 50, 60]);

color1.deltaE(color2, "76");
color2.deltaE(color3, "76");

color1.deltaE(color2, "CMC");
color2.deltaE(color3, "CMC");

color1.deltaE(color2, "2000");
color2.deltaE(color3, "2000");
```

For most DeltaE algorithms, 2.3 is considered the "Just Noticeable Difference" (JND).
Can you notice a difference in the two colors below?

```js
// These are not needed if you're just using the bundle
let color1 = new Color("lch", [40, 50, 60]);
let color2 = new Color("lch", [40, 50, 60]);

color1.deltaE(color2, "76");
color1.deltaE(color2, "CMC");
color1.deltaE(color2, "2000");
```