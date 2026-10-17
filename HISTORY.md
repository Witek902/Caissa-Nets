# Caissa nets history

Every net in this repository is also published as a GitHub release asset, with the net name as the tag. Caissa
downloads its default net, set by `NET_VERSION` in `CMakeLists.txt` and `src/makefile`, from these releases at build
time. Until 2026-05-16 the nets were stored in the Caissa repository itself.

Names format: usually `eval-<run>-<positions>B`, where the number is the training position count of the checkpoint;
`-permuted` means the accumulator was reordered for sparse inference with `utils permuteNet` (bit-exact outputs);
`-spsa` / `-spsa-transplant` mean an SPSA-tuned last layer.

## Architectures

| Nets | Architecture | Net format |
|---|---|---|
| eval-71, eval-75 | `(32x768 -> 1024)x2 -> 1`, CReLU, 32 king buckets, 8 output buckets by piece count | 12 |
| eval-82-383B, eval-84-144B | same, SCReLU | 12 |
| eval-ml-5-200B-permuted, eval-ml-6-160B | `(32x768 -> 1024)x2 -> pairwise CReLU -> 16 -> 32 -> 1`, 8 output buckets at every head layer | 14 |
| eval-ml-8-152B-* and later | `(32x768 -> 1536)x2 -> pairwise CReLU -> 16 -> 32 -> 1` | 15 |

## Summary

Elo is against the previous master net, from the SPRTs quoted in the Caissa commit that switched to the net.

| Net | Default in Caissa since | Release | How it was made | STC | LTC |
|---|---|---|---|---|---|
| eval-71 | 2026-02-10 (`45f1ee7`) | 1.25 | first net with 32 king buckets, 348B positions | +2.07 ± 1.45 | +2.78 ± 1.98 |
| eval-75 | 2026-05-16 (`d8e94e1`) | | 4 sessions, 884B positions, endgame and fortress data added | +2.62 ± 1.68 | +2.79 ± 1.98 |
| eval-82-383B | 2026-06-19 (`d0fdf16`) | 1.26 | SCReLU, 383B positions in 4 LR phases | +7.17 ± 3.01 | +3.93 ± 2.78 |
| eval-84-144B | 2026-08-24 (`7aa469b`) | | fine-tune of eval-82-383B | +1.35 ± 1.06 | |
| eval-ml-5-200B-permuted | 2026-09-12 (`7d820a9`) | | first multilayer net, 2 stages | +18.21 ± 4.88 | +20.58 ± 6.13 |
| eval-ml-6-160B | 2026-09-13 (`590bb44`) | | head-only fine-tune of eval-ml-5, FT frozen | +3.30 ± 1.92 | |
| eval-ml-8-152B-permuted | 2026-09-16 (`0117173`) | | first 1536-wide net, 2 stages | +1.58 ± 1.19 | +7.32 ± 3.54 |
| eval-ml-8-152B-spsa | 2026-09-19 (`34082cc`) | 2.0 | last layer SPSA-tuned (231 params, 108K games) | +5.97 ± 2.73 | +6.09 ± 3.21 |
| eval-ml-11a-100B-spsa-transplant | added 2026-10-17 | | stage-3 fine-tune of eval-ml-8 + the SPSA tune "transplanted" | +3.47 ± 2.00 | +0.98 ± 1.25 |

## Details

### eval-71
- New neural net with 32 king buckets, trained for 348B positions.
- STC +2.07 ± 1.45 (https://furybench.com/test/5025/), LTC +2.78 ± 1.98 (https://furybench.com/test/5029/).
- Caissa 1.25 net; added to this repository on 2026-06-28.

### eval-75
- Dataset 20.4B positions, trained in four sessions of 193B, 254B, 202B and 235B positions; LR 1e-5 -> 1.25e-6.
- Data changes: +32M positions from selfplay on the `endgames_cdb95105.epd` book (endgame positions, ~50% draws),
  + positions from selfplay on fortress positions, - queens-only games (7 queens + 8 pawns start position).
- STC +2.62 ± 1.68 (https://furybench.com/test/6277/), LTC +2.79 ± 1.98 (https://furybench.com/test/6282/),
  endgames +47.77 ± 1.77 (https://furybench.com/test/6285/).
- First net downloaded from this repository (Caissa `fc60c19`).

### eval-82-383B
- SCReLU activation. 383B positions on a 20.5B-position dataset in four LR phases: 138B at 1e-4 (validation error
  0.12860), 250B at 3e-5 (0.12795), 278B at 1e-5 (0.12770), 383B at 2.5e-6 (0.12768).
- STC +7.17 ± 3.01, LTC +3.93 ± 2.78, endgames +22.52 ± 5.18.
- Caissa 1.26 net.

### eval-84-144B
- Fine-tune of eval-82-383B with a cosine LR from 2e-5 to 2e-6 over 120B positions; dataset 20.5B positions.
- UHO STC +1.35 ± 1.06 (https://furybench.com/test/7208/), endgames +8.26 ± 3.10
  (https://furybench.com/test/7209/).
- Last single-layer net.

### eval-ml-5-200B-permuted
- First multilayer net. Trainer commit `d48750f`.
- Stage 1 (eval-ml-3): `trainCudaNetwork --lr 1e-3 --lambda 0.9 --seed 12345 --iterations 71600 --name eval-ml-3`;
  FT weight decay 0.01, subnet decay 0.01, FT clamp 4.0, factorizer on.
- Stage 2 (eval-ml-5): `trainCudaNetwork --net eval-ml-3-150B.pnn --lr 1e-4 --lambda 0.1 --seed 12345
  --trainingLength 200000000000 --iterations 95400 --name eval-ml-5`; FT weight decay 0.005, subnet decay 0,
  factorizer off (the stage-1 factorizer is folded into the king buckets of the packed net).
- Permuted with `permuteNet --net eval-ml-5-200B.pnn sprt10.epd.bin sprt15.bin sprt16.bin --maxPositions 500000`.
- UHO STC +18.21 ± 4.88 (https://furybench.com/test/7401/), UHO LTC +20.58 ± 6.13
  (https://furybench.com/test/7403/), DFRC STC +4.95 ± 3.02 (https://furybench.com/test/7404/), endgames
  +53.67 ± 7.28 (https://furybench.com/test/7405/), UHO 10k nodes +29.93 ± 7.14 (https://furybench.com/test/7402/).

### eval-ml-6-160B
- Head-only fine-tune of eval-ml-5-200B-permuted with the feature transformer frozen:
  `trainCudaNetwork --net eval-ml-5-200B-permuted.pnn --lr 5e-5 --lambda 0.05 --trainingLength 200000000000
  --freezeFeatureTransformer --reviveDeadNeurons --bucketLeak 0.5 --name eval-ml-6`; FT range 8.0, end LR 2.5e-6.
- STC +3.30 ± 1.92 (https://furybench.com/test/7408/), 25k nodes +1.91 ± 1.46.

### eval-ml-8-152B-permuted
- First 1536-wide net, 75.7 MB. Trainer commit `6dcd9d7`.
- Stage 1 (eval-ml-7): `trainCudaNetwork --LR 1e-3 --endLR 2.5e-6 --startLambda 0.9 --endLambda 0.5 --seed 12345
  --trainingLength 160 --bucketLeak 0.5 --name eval-ml-7`; FT weight decay 0.005, subnet decay 0, FT clamp 4.0,
  factorizer on; data without the adversarial0/1 sets.
- Stage 2 (eval-ml-8): `trainCudaNetwork --net eval-ml-7-160B.pnn --LR 1e-4 --endLR 2.5e-6 --startLambda 0.0
  --endLambda 0.0 --seed 12345 --trainingLength 150 --bucketLeak 0.5 --name eval-ml-8`; FT weight decay 0.0025,
  FT clamp 6.0, factorizer on, subnet decay 0.
- STC +1.58 ± 1.19 (https://furybench.com/test/7427/), LTC +7.32 ± 3.54 (https://furybench.com/test/7428/),
  25k nodes +22.73 ± 5.65.

### eval-ml-8-152B-spsa
- eval-ml-8-152B-permuted with the last layer (32 weights + bias per output bucket, buckets 1-7, 231 parameters)
  tuned with SPSA for 6769 iterations, 108,304 games at 6+0.06 (https://furybench.com/tune/7435/), written with
  Caissa's `scripts/patchNetLastLayer.py`.
- STC (8+0.08) +5.97 ± 2.73 (https://furybench.com/test/7436/), LTC (40+0.4) +6.09 ± 3.21
  (https://furybench.com/test/7437/).
- Caissa 2.0 net.

### eval-ml-11a-100B-spsa-transplant
- Stage 3 (eval-ml-11a), a fine-tune of eval-ml-8-152B-permuted (the net before the SPSA tune):
  `trainCudaNetwork --net eval-ml-8-152B-permuted.pnn --LR 5e-5 --endLR 2e-6 --startLambda 0.0 --endLambda 0.0
  --trainingLength 40 --bucketLeak 0.5 --name eval-ml-11a`. Fresh Adam moments, 50-iteration warmup, cosine LR
  over 40B then constant 2e-6, stopped at 100B; seed 12345, batch 32768, AdamW 0.9 / 0.999, FT weight decay
  0.0025, subnet decay 0, FT clamp 6.0, factorizer on. Trainer `feaa5ff` plus the loader filters later committed
  in `71f4344`, all off.
- Data: 2849 files, 21.0B positions. Since eval-ml-8: +16 files (0.73B positions: DFRC_UHO_15kn x4,
  UHO_Lichess_4852_v1_15kn x3, DFRC_16kn, sprt17, 7queens / 7bishops / 7knights / 7rooks missing-pawn books,
  fortresses_torch, weird_100k, 8moves_v3), -10 old 2023 selfplay files.
- Validation RMSE 0.12251 (0-10B) -> 0.12190 at 40B -> 0.12189 at 100B. Permuted with `utils permuteNet`.
- eval-ml-11a-100B-permuted vs eval-ml-8-152B-permuted: UHO STC +4.97 ± 2.44, DFRC STC +2.34 ± 1.58,
  UHO 25k nodes +2.04 ± 2.35.
- SPSA transplant: eval-ml-8-152B-spsa differs from its base only in the last layer, and eval-ml-11a's last two
  layers correlate 0.995-0.9999 with that base per bucket, so the hidden neurons still correspond. The
  per-weight difference was added: L3(eval-ml-11a) + L3(eval-ml-8-152B-spsa) - L3(eval-ml-8-152B-permuted),
  buckets 1-7, with `scripts/patchNetLastLayer.py`. No re-tuning.
- vs eval-ml-8-152B-spsa: STC +3.47 ± 2.00 (https://furybench.com/test/7644/), LTC +0.98 ± 1.25 with
  non-regression bounds, stopped at 70K games (https://furybench.com/test/7645/).
