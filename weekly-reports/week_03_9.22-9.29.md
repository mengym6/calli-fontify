## Questions

1. Fontify seems to fail to reconstruct high-frequency details; however, adding a specific high-pass loss function does not work well.

2. We need another method to develop a structure loss for Jieti.

3. A Jieti mask, even when only half of it is used, can still cause the generated glyph to miss many stroke details. For instance, the glyph `如` has two parts, left and right; even using one part of the mask can misguide the model and cause unexpected generation. I would attribute this to an entire glyph region being covered and the model being unable to learn under this high-ratio mask. Finally, it generates very thick and dark strokes, possibly aiming to deceive the loss function(L1&L2).Unfortunately forget to download pictures, but it does perform the worse.

## Progress

1. For Q1, I discovered that the decoder is too shallow and is not a classic U-Net-style upsampling decoder, although it performed well in the 64 × 64 resolution pre-training task. However, CalliPhase offers a higher resolution (448 × 448). Besides, the current decoder taps features from four layers (3, 6, 9, 12). In previous work, blocks 1 to 9 were actually frozen, meaning that their parameters were not trained.

2. For Q2, simply adding a structure loss focusing on row, column, centroid, and foreground area cannot offer enough supervision. I am working on adding a new module to extract Jieti style information, with other loss functions serving as assistants.

3. The new baseline involves totally unfreezing the model, without GAN loss (since GAN loss causes the test loss not to converge), and replacing Jieti masks with random mask, and using Bifa masks. This baseline performs quite well on calli-font(seeing pictures in directory "images"), so whether catastrophic forgetting occurs on printed fonts does not matter(assumption). (It performs better than the frozen baseline.)

4. The GAN loss function causes fluctuations; maybe it will be removed in future work.

## Future Job

1. Build a paired style encoder that takes a reference printed glyph and a font calligraphy glyph of the same character, then extracts a global style representation `g` and local style tokens `S`.

2. Feed the reference printed glyph through the printed-glyph stream to obtain pre-mixed content features `C`. Use `g` to guide the content features and `S` to express region relationships and deformation， producing layout features `L`.

3. Inject a layout residual derived from `L` into the original Fontify features, followed by local style information from `S`, to obtain multi-level generative features `F`.

4. Mix `F` with `L` before a stronger decoder to generate the target calligraphy glyph.

5. Find out a method to capitalize on the semantic annotations in calliphase, leading Fontify from "in-painting" to "writing", as acknowledging GPT-image2 has changed the generation task from "drawing" to "writing".
