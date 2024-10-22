TL1 和 TL2 的代码生成
------------------------

codegen_tl1.py 和 codegen_tl2.py 使用参数为不同设备生成内核代码，以实现 TL1 和 TL2 的最快性能。

我们将权重切割成多个计算块，以充分利用硬件能力。

### 示例
bitnet_b1_58-large:

- 确保矩阵乘法内核形状\
例如，bitnet_b1_58-large 的矩阵乘法内核形状为：\
[1536, 4096]\
[1536, 1536]\
[4096, 1536]

- 确保每个内核的 BM、BK、bm 满足以下要求
- 生成代码\
例如，对于 bitnet_b1_58-large，我们可以这样生成代码：

```bash
# 对于 TL1
python utils/codegen_tl1.py --model bitnet_b1_58-large --BM 256,128,256 --BK 128,64,128 --bm 32,64,32

# 对于 TL2
python utils/codegen_tl2.py --model bitnet_b1_58-large --BM 256,128,256 --BK 96,192,96 --bm 32,32,32
```

### TL1:
![TL1](../assets/tl1.png)

对于 TL1，我们将权重切割成 M / BM 权重，每个权重形状为 (BM, K)。然后我们将权重切割成 K / BK 权重，每个权重形状为 (BM, BK)。对于 (BM, BK) 权重，我们以相同的方式将其切割成 (bm, compute_num / bm) 计算块，并在其中完成计算。

因此，我们需要确保
- M % BM == 0
- K % BK == 0
- BM % bm == 0
- bm 选择在 \[32, 64\]

### TL2:
![TL2](../assets/tl2.png)

对于 TL2，事情稍微复杂一些。由于 TL2 需要 BK % 6 == 0，我们需要将 K 分割成 threeK 和 twoK，在其中计算 TL2 的 (M, threeK)，计算 TL1 的 (M, two_K)。

因此，我们需要确保
- M % BM == 0
- K % BK % 32 == 0
- BM % bm == 0
- bm 选择在 \[32\]
