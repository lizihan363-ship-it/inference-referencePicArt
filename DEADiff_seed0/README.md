# DEADiff：data1200 生成结果与七项指标

本目录保存 **60 张参考图 × 20 条目标提示词，共 1200 张 DEADiff 生成图**。所有生成任务使用 `seed=0`，图片尺寸为 `512×512`。

2026-11-07 更新了 **E21、E30** 两张参考图，并按原配置重跑其各自的 20 条提示词，共替换 **40 张**结果。其余 **1160 张**生成图的 SHA256 保持不变。当前参考图、生成结果和配对清单均对应更新后的版本。

## 目录与配对

```text
DEADiff_seed0/
├── images/                         # 1200 张生成图
├── references/                     # 60 张当前参考图
├── prompts_selected_20.json        # 20 条原始目标提示词
├── portable_manifest.json          # 1200 个参考图—提示词—生成图配对及哈希
├── generation_config.json          # 生图配置、软件版本及模型/源码哈希
├── rerun_update_20261107.json       # E21、E30 的更新范围与校验摘要
├── SHA256SUMS                      # 原始存档负载文件的 SHA256
├── archive_content_manifest.json   # 原始存档文件清单、大小及哈希
└── metrics/                        # 七项指标、逐图明细及评估配置/验证记录
```

[配对清单](portable_manifest.json) 使用相对路径显式关联图片，不依赖目录排序。生成图命名为 `参考图ID__p提示词编号.png`，例如 `images/E21__p01.png`；对应参考图及完整目标文本以清单为准。

## 本次生图配置

完整配置见 [generation_config.json](generation_config.json)。

| 参数 | 本次取值 |
|---|---|
| 方法 | DEADiff 官方生成接口 `DEADiff.generate` |
| 随机种子 | `0`，每个任务相同 |
| 采样器 | `DDIM` |
| 采样步数 | `50` |
| CFG / `scale` | `8.0` |
| `image_weight` / `img_weight` | `1.0` |
| `subject_text` | `style` |
| Batch size | `1` |
| 输出尺寸 | `512×512` |
| 参考图预处理 | 官方接口：转换 RGB，缩放至 `224×224`，再按官方实现归一化 |
| 保存结果 | 官方返回列表的第一张生成图 |

原始负面提示词保持如下：

```text
over-exposure, under-exposure, saturated, duplicate, out of frame, lowres, cropped, worst quality, low quality, jpeg artifacts, morbid, mutilated, out of frame, ugly, bad anatomy, bad proportions, deformed, blurry, duplicate
```

生图环境记录的主要软件版本：

| 软件包 | 版本 |
|---|---|
| torch | `2.0.0` |
| torchvision | `0.15.0` |
| pytorch-lightning | `1.8.6` |
| diffusers | `0.16.0` |
| transformers | `4.28.0` |
| salesforce-lavis | `1.0.1` |
| xformers | `0.0.17` |
| Pillow | `9.5.0` |

DEADiff 源码版本：`df3238da32037d0ac5054f55df8388cc08021bcf`。

| 文件身份 | SHA256 |
|---|---|
| 模型检查点 | `b485088b9e0134a53ea686929e385e78c22a16a965bdd4f78a94d1077eb828cf` |
| 官方生成接口源码 | `95fbb5e1fd2c66e362004179018ae18254c3c0e59059eff089cd5e95c9dde857` |
| 推理模型配置 | `8cd26af581f44e60c46b40f7218ce95979e37ced6645ee61d82f4004b7d110d5` |
| 批处理运行脚本 | `e20f3b38a8e16ab601be5b02fe7ca1489a128db1e54b06e1fa28a69ea34266d4` |
| 20 条提示词文件 | `78dc93e82b899b8485a41cc5a92ee2e15134c9e0e158e04e502bf43b77028312` |

## 七项指标

指标对全部 **1200 个配对等权汇总**，参考图采用当前 60 张版本。英文参考内容描述逐张绑定参考图 SHA256，与已有方法评估使用的描述保持一致。

| 指标 | 均值 | 读数含义 |
|---|---:|---|
| CSD 风格余弦 | `0.298367731988` | 越高表示所用风格特征越接近 |
| Gram 风格距离 | `6.235839463898e-05` | 越低表示所定义纹理统计距离越小 |
| DreamSim 距离 | `0.644298497240` | 越低表示感知特征越接近 |
| CLIP-T 目标文本 | `0.310178521872` | 越高表示与目标提示词越对齐 |
| Leakage | `0.221090321679` | 越低为较少参考内容泄漏的诊断证据 |
| CLIP-T 参考内容 | `0.062966926366` | 参考内容相似度诊断 |
| DINO 参考图余弦 | `0.154116565707` | 越高表示 DINO 特征越相似 |

[指标汇总 CSV](metrics/指标汇总.csv) 包含均值、样本标准差和数量；[逐图指标 CSV](metrics/逐图指标.csv) 包含 1200 张逐图值；[完整汇总 JSON](metrics/combined_summary.json) 保留完整数值精度。标准差采用 `ddof=1`。

### 固定评估配置

评估环境与生图环境分别配置，固定使用以下设置：

- Python `3.12`，torch `2.8.0+cu128`，torchvision `0.23.0+cu128`。
- CUDA、FP32、关闭 TF32 与自动混合精度，batch size `8`，CPU 线程 `8`。
- 评估随机种子 `0`；Gram 输入 `512×512`，采用固定的 VGG19 五层定义。
- 模型、源码、公式与图像预处理均按固定协议和完整 SHA256 校验；正式计算只读取本地固定权重。
- DINO 使用 **DINOv3 ViT-L/16、LVD1689M、1024 维全局 CLS**，比较生成图与其原参考图的归一化特征点积；不乘 100，不转换成 `1−cos`。
- Leakage 使用固定分段公式：若 `s(g,c_ref) < s(g,target)`，取 `s(g,c_ref) / s(ref,c_ref)`，否则取 `1`；不额外截断或添加 epsilon。

DINO 是整体特征相似度，不能单独区分风格保留与参考内容复制。生图 seed 和评估 seed 分别记录。

六项评估内核 SHA256：`0d21c71c654a084bf3b93f0e1d5cdc0901ec7667b7aecd07c7e3ec578cab4373`。更完整的软件、模型与预处理记录保存在 [metrics](metrics/) 中。

## 校验记录

生成图和参考图均重新核对 SHA256。七项指标均覆盖 1200 张，计算包的总体严格验证、六项验证及 DINO 验证全部通过。独立复核还重新检查配对、更新范围、Leakage 公式、Gram 的 FP32 层均值，并从逐图值复算均值、样本标准差、中位数、最小值与最大值。

[SHA256SUMS](SHA256SUMS) 与 [archive_content_manifest.json](archive_content_manifest.json) 记录最初图片存档的负载文件，二者不包含自身；后续加入的中文说明与指标目录以对应交付/验证记录为准。目录中未包含大型模型检查点、环境缓存、日志或连接凭据。
