# superformula-js

An interactive browser-based visualization of the **Gielis Superformula** — a mathematical generalization of the circle that can describe a wide variety of natural and geometric shapes by tuning six parameters.

## Screenshot

![Superformula visualization – default parameters (m=6, a=b=1, n₁=n₂=n₃=1)](https://github.com/user-attachments/assets/c310e5ed-2bb2-472e-b015-a34b9bb3cfba)

## The Formula

The Superformula, introduced by Johan Gielis in 2003, is expressed in polar coordinates as:

$$r(\varphi) = \left[ \left|\frac{\cos\left(\frac{m\varphi}{4}\right)}{a}\right|^{n_2} + \left|\frac{\sin\left(\frac{m\varphi}{4}\right)}{b}\right|^{n_3} \right]^{-\frac{1}{n_1}}$$

where $r$ is the radial distance for a given angle $\varphi \in [0, 2\pi)$.

In plain text:

```
r(φ) = [ |cos(m·φ/4) / a|^n2  +  |sin(m·φ/4) / b|^n3 ] ^ (-1/n1)
```

## Parameters

| Parameter | Description | Typical range |
|-----------|-------------|---------------|
| `m`       | Rotational symmetry — number of lobes/petals | 1 – 20 |
| `a`       | Horizontal scaling factor | 0.1 – 5 |
| `b`       | Vertical scaling factor | 0.1 – 5 |
| `n₁`      | Overall shape exponent | 0.1 – 20 |
| `n₂`      | Cosine exponent — controls curvature of lobes | 0.1 – 20 |
| `n₃`      | Sine exponent — controls curvature between lobes | 0.1 – 20 |

By varying these six parameters the formula can model shapes found in nature (flowers, starfish, snowflakes, leaves) as well as regular polygons, ellipses, and many abstract curves.

## Usage

Open `index.html` directly in any modern browser — no build step or server required.  
Use the sliders at the bottom of the page to adjust each parameter in real time.  
The **Stroke** and **Fill** colour pickers let you customise the appearance; the **Filled** toggle switches between outline-only and filled rendering.  
Click the **SAVE** button to save the current configuration to the gallery. The gallery displays up to the last **5** saved configurations as thumbnail previews; clicking any thumbnail instantly restores that configuration to the canvas.  
Click the **RAND** button to instantly randomize all six parameters and discover unexpected shapes.

## References

### Scientific papers

- Gielis, J. (2003). **A generic geometric transformation that unifies a wide range of natural and abstract shapes.** *American Journal of Botany*, 90(3), 333–338. <https://doi.org/10.3732/ajb.90.3.333>
- Gielis, J. (2017). **The Geometrical Beauty of Plants.** Atlantis Press / Springer. ISBN 978-94-6239-150-5.
- Matsuura, M. (2015). Gielis' superformula and regular polygons. *Journal of Geometry*, 106(2), 383–403. <https://doi.org/10.1007/s00022-014-0251-6>

### Web resources

- [Superformula – Wikipedia](https://en.wikipedia.org/wiki/Superformula) — overview with interactive examples and parameter tables.
- [Superformula – Wolfram MathWorld](https://mathworld.wolfram.com/Superformula.html) — mathematical treatment with formula derivations.
- [Paul Bourke – Superformula](https://paulbourke.net/geometry/supershape/) — early gallery of shapes and C source code.
- [The Nature of Code – Chapter 3 (Daniel Shiffman)](https://natureofcode.com/oscillation/) — accessible introduction with p5.js examples.

## License

[MIT](LICENSE)
