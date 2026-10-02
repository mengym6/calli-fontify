# Weekly Report 04 — Linear Probe & Single Label-Head

## Targets

1. In the current Fontify, can the encoder features be *linearly read out* into semantic regions (linear probe)?
2. Based on the best baseline, does adding a single semantic label-prediction head improve generation?

**Linear probe.**  Model frozen, lower-half tokens taken from taps [2, 5, 8, 11]. Only train a 1*1 linear layer to predict the semantic classes and report IoU.

Controls: 
1.random-init ViT (learned vs. structural), 
2.positional prior
3.`checkpoint-14` 
4.9-layer-frozen baseline, 
5.fully-unfrozen best baseline


**Label-head comparison.** 

Baseline = current best (from `checkpoint-14`, unfrozen, no GAN, BF semantic mask + JT random mask)

Experiment = same hyperparameters, adding one label head (4 taps → `Linear(3072→70)`, 70 semantic labels). The label head directly predicts the semantic classes from the encoder's tokens through one extra encoder forward pass, acting as an auxiliary head on the encoder.


Loss = sample-type-masked BCE + Dice over present classes
(BF samples only score BF channels, JT samples score JT channels), weight 0.1 (about 7–11% of total loss). 

**Generation eval.** 
 Checkpoints ep45 and ep50 compared on L1 and structure metrics (centroid, row/col projection, area, aspect ratio, edge F1 …). Visual evidence: see directory `Images`.

## Results

### Linear probe (val mIoU, 4 taps concatenated)

| Model | JT val | JT train | BF val | BF train |
|---|---:|---:|---:|---:|
| Random init | 0.079 | 0.086 | 0.037 | 0.045 |
| Positional prior | 0.076 | – | 0.007 | – |
| `checkpoint-14` | 0.189 | 0.384 | 0.146 | 0.231 |
| 9-layer-frozen ep45 | 0.188 | 0.385 | 0.150 | 0.240 |
| Best baseline ep45 | 0.221 | 0.427 | 0.172 | 0.289 |
| Label head ep45 | **0.262** | 0.492 | 0.170 | 0.310 |
| Head itself (from joint training) | 0.180 | 0.194 | 0.151 | 0.191 |
| Best ep45, ink visible | 0.206 | 0.372 | 0.213 | 0.308 |

JT main-class val IoU (right / bottom / left / top):

| Model | right | bottom | left | top |
|---|---:|---:|---:|---:|
| Random init | 0.20 | 0.20 | 0.24 | 0.18 |
| `checkpoint-14` | 0.40 | 0.43 | 0.46 | 0.37 |
| Best baseline ep45 | 0.41 | 0.44 | 0.49 | 0.37 |
| Label head ep45 | 0.49 | 0.52 | 0.53 | 0.44 |

### Generation comparison

Per-glyph paired difference Δ = label head − baseline
(edge_f1 higher is better, others should be lower)

| | `l1` | `centroid` | `col` | `aspect` | `edge_f1` |
|---|---|---|---|---|---|
| ep50 JT | −0.0038 [−0.0085, +0.0008] | −0.0016 [−0.0085, +0.0046] | **−0.0041 [−0.0068, −0.0011]** | −0.0074 [−0.0191, +0.0037] | +0.0032 [−0.0014, +0.0077] |
| ep50 BF | +0.0016 [−0.0014, +0.0048] | −0.0007 [−0.0076, +0.0066] | −0.0011 [−0.0029, +0.0008] | −0.0047 [−0.0152, +0.0054] | +0.0041 [−0.0017, +0.0098] |
| ep45 JT | −0.0031 [−0.0079, +0.0015] | −0.0002 [−0.0070, +0.0060] | **−0.0038 [−0.0065, −0.0008]** | −0.0066 [−0.0181, +0.0045] | +0.0036 [−0.0010, +0.0080] |

On JT, every structure metric leans toward the label head

## Conclusions

1. Encoder can successfully read out semantic regions, results are better than others.

2. Label supervision further writes JT region into features.

3. In the visual evidence, the label head improves generation quality on some fonts, which proves that the semantic label works, though it has not achieved a full improvement across all fonts yet.

After all, these experiments demonstrate that using semantic labels is a possible way to improve the model's generation ability, and finally end two weeks of failures.

## Future Job

1. Move structure supervision directly onto the generated image.(Building new JT loss)
2. Layout auxiliary head + feed the predicted layout back into the target tokens.
