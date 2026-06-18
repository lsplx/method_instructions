# 已复现方法统一清单

本文档汇总当前 `/home/public/minzhi` 下已经完成复现的各方法，按统一格式说明：

- 工具
- 前置条件
- 输入
- 输出
- 操作步骤

当前盘点到 7 个已有复现文档。其中前 5 个是主要复现方法；后 2 个是后续补充复现的方法。

## 1. AuthChain

### 工具

- 方法/仓库：`AuthChain`
- 目录：`/home/public/minzhi/AuthChain`
- 主要脚本：
  - `extract_information.py`
  - `AuthChain.py`
  - `model_client.py`
  - `run_reproduction.sh`
- 默认模型：`deepseek-v4-flash`
- API base URL：`https://antchat.alipay.com`

### 前置条件

- 安装依赖：

```bash
cd /home/public/minzhi/AuthChain
pip install -r requirements.txt
```

- 配置 API key，推荐环境变量：

```bash
export ANTCHAT_API_KEY="YOUR_REAL_KEY"
```

- `AuthChain.py` 中只保留脱敏占位：

```python
api_key = "XXXXX"
```

### 输入

- 样例问题：

```text
data/reproduction/sample_questions.json
```

- 信息抽取阶段输入为自然语言问题。
- AuthChain 生成阶段输入为抽取出的 `Intent / Entities / Relations`。

### 输出

```text
data/reproduction/sample_extract_info.real.json
data/reproduction/sample_authchain.real.json
```

### 操作步骤

一键运行：

```bash
cd /home/public/minzhi/AuthChain
./run_reproduction.sh
```

分步运行：

```bash
cd /home/public/minzhi/AuthChain

python3 extract_information.py \
  --input data/reproduction/sample_questions.json \
  --output data/reproduction/sample_extract_info.real.json \
  --overwrite \
  --limit 1 \
  --model deepseek-v4-flash

python3 AuthChain.py \
  --input data/reproduction/sample_extract_info.real.json \
  --output data/reproduction/sample_authchain.real.json \
  --overwrite \
  --limit 1 \
  --model deepseek-v4-flash \
  --include-diagnostics \
  --max-revisions 0 \
  --max-retries 5
```

## 2. IJBR / Puzzler

### 工具

- 方法/仓库：`IJBR`
- 方法名：`Puzzler`
- 目录：`/home/public/minzhi/IJBR`
- 主要脚本：
  - `OMG.py`
  - `jailbreak.py`
  - `model_client.py`
  - `run_reproduction.sh`
- 默认模型：`deepseek-v4-flash`
- 说明：当前复现脚本会约束输出为非操作性、脱敏研究结果，不输出可操作伤害细节。

### 前置条件

```bash
cd /home/public/minzhi/IJBR
pip install -r requirements.txt
export ANTCHAT_API_KEY="YOUR_REAL_KEY"
```

### 输入

```text
data/harmful_behaviors_custom.xlsx
```

输入是一条研究样例 query。复现流程会抽取核心内容，生成防御措施、相关线索，再进行第三阶段间接推断。

### 输出

```text
data/reproduction/offense.real.json
data/reproduction/jailbreak.real.json
```

### 操作步骤

一键运行：

```bash
cd /home/public/minzhi/IJBR
./run_reproduction.sh
```

分步运行：

```bash
cd /home/public/minzhi/IJBR

python3 OMG.py \
  --input data/harmful_behaviors_custom.xlsx \
  --output data/reproduction/offense.real.json \
  --overwrite \
  --limit 1 \
  --max-defenses 1 \
  --model deepseek-v4-flash

python3 jailbreak.py \
  --input data/reproduction/offense.real.json \
  --output data/reproduction/jailbreak.real.json \
  --overwrite \
  --limit 1 \
  --model deepseek-v4-flash
```

## 3. VEtesting / VEglue

### 工具

- 方法/仓库：`VEtesting`
- 方法名：`VEglue`
- 目录：`/home/public/minzhi/VEtesting`
- 主要脚本：
  - `extractentity_VE.py`
  - `VG.py`
  - `MR.py`
  - `model_client.py`
  - `run_reproduction.sh`
- 默认模型：`deepseek-v4-flash`
- 任务：为 Visual Entailment 生成 metamorphic relation 测试用例。

### 前置条件

```bash
cd /home/public/minzhi/VEtesting
pip install -r requirements.txt
export ANTCHAT_API_KEY="YOUR_REAL_KEY"
```

仓库已带有 bbox grounding 文件，当前复现不需要下载额外视觉 grounding 模型。

### 输入

```text
data/snli_ve_test.xlsx
data/snli_ve_test_bbox_entailment.xlsx
data/images/flickr30k/
```

- `snli_ve_test.xlsx`：图文蕴含样本。
- `snli_ve_test_bbox_entailment.xlsx`：已有对象-bbox 对齐结果。
- `data/images/flickr30k/`：原始图片。

### 输出

```text
data/reproduction/snli_test_entity.real.json
data/reproduction/snli_ve_test_bbox_entailment.real.json
data/reproduction/veglue_test_cases.real.json
data/reproduction/images/
```

### 操作步骤

一键运行：

```bash
cd /home/public/minzhi/VEtesting
./run_reproduction.sh
```

分步运行：

```bash
cd /home/public/minzhi/VEtesting

python3 extractentity_VE.py \
  --input data/snli_ve_test.xlsx \
  --output data/reproduction/snli_test_entity.real.json \
  --overwrite \
  --limit 1 \
  --model deepseek-v4-flash

python3 VG.py \
  --entities data/reproduction/snli_test_entity.real.json \
  --bbox data/snli_ve_test_bbox_entailment.xlsx \
  --output data/reproduction/snli_ve_test_bbox_entailment.real.json \
  --overwrite \
  --limit 1

python3 MR.py \
  --input data/reproduction/snli_ve_test_bbox_entailment.real.json \
  --output data/reproduction/veglue_test_cases.real.json \
  --overwrite \
  --limit 1
```

## 4. InstruCoT-LLM

### 工具

- 方法/仓库：`InstruCoT-LLM`
- 目录：`/home/public/minzhi/InstruCoT-LLM`
- 主要脚本：
  - `PI_generation.py`
  - `CoT_generation.py`
  - `train_sft.py`
  - `infer_finetuned.py`
  - `run_reproduction.sh`
- 远程生成模型：`deepseek-v4-flash`
- SFT 基座模型：`Qwen/Qwen3.5-0.8B`
- 训练方式：LoRA SFT

### 前置条件

```bash
cd /home/public/minzhi/InstruCoT-LLM
/home/public/.local/bin/python3.10 -m venv .venv
.venv/bin/pip install -r requirements.txt
export ANTCHAT_API_KEY="YOUR_REAL_KEY"
export HF_HUB_DISABLE_XET=1
```

推荐提前下载 Qwen3.5：

```bash
cd /home/public/minzhi/InstruCoT-LLM
export HF_HUB_DISABLE_XET=1
.venv/bin/python -c "from huggingface_hub import snapshot_download; print(snapshot_download('Qwen/Qwen3.5-0.8B'))"
```

### 输入

```text
data/base_system_prompts.json
```

该文件提供基础系统角色/任务提示。复现流程会生成约 500 条 prompt injection 样本，再生成 InstruCoT SFT 数据。

### 输出

```text
data/reproduction/pi_samples.json
data/reproduction/instrucot_sft.json
data/reproduction/instrucot_sft.jsonl
outputs/qwen3_5_instrucot_lora/
outputs/qwen3_5_instrucot_lora/reproduction_train_state.json
outputs/inference_test_outputs.json
```

### 操作步骤

一键运行：

```bash
cd /home/public/minzhi/InstruCoT-LLM
chmod +x run_reproduction.sh
./run_reproduction.sh
```

分步运行：

```bash
cd /home/public/minzhi/InstruCoT-LLM

.venv/bin/python PI_generation.py \
  --input data/base_system_prompts.json \
  --output data/reproduction/pi_samples.json \
  --target-count 500 \
  --batch-size 10 \
  --model deepseek-v4-flash \
  --overwrite

.venv/bin/python CoT_generation.py \
  --input data/reproduction/pi_samples.json \
  --output data/reproduction/instrucot_sft.json \
  --jsonl-output data/reproduction/instrucot_sft.jsonl \
  --target-count 500 \
  --batch-size 8 \
  --model deepseek-v4-flash \
  --overwrite

.venv/bin/python train_sft.py \
  --train-file data/reproduction/instrucot_sft.jsonl \
  --output-dir outputs/qwen3_5_instrucot_lora \
  --model-name-or-path Qwen/Qwen3.5-0.8B \
  --device cuda:0 \
  --max-length 512 \
  --num-train-epochs 1 \
  --max-steps 100 \
  --per-device-train-batch-size 1 \
  --gradient-accumulation-steps 1

.venv/bin/python infer_finetuned.py \
  --base-model Qwen/Qwen3.5-0.8B \
  --adapter outputs/qwen3_5_instrucot_lora \
  --output outputs/inference_test_outputs.json \
  --device cuda:0
```

## 5. Patcher

### 工具

- 方法/仓库：`patcher`
- 方法名：`Patcher`
- 目录：`/home/public/minzhi/patcher`
- 主要脚本：
  - `reproduce_patcher.py`
  - `model_client.py`
- 默认 T2I：`segmind/tiny-sd`
- 默认 CLIP：`openai/clip-vit-base-patch32`
- 注意力解释工具：`DAAM`
- 大模型：`deepseek-v4-flash`

### 前置条件

```bash
cd /home/public/minzhi/patcher
/home/public/.local/bin/python3.10 -m venv .venv
.venv/bin/pip install -U pip setuptools wheel
.venv/bin/pip install --extra-index-url https://download.pytorch.org/whl/cu121 torch==2.5.1+cu121 torchvision==0.20.1+cu121
.venv/bin/pip install -r requirements_reproduction.txt
.venv/bin/python -m spacy download en_core_web_trf
export ANTCHAT_API_KEY="YOUR_REAL_KEY"
```

首次运行会下载：

```text
segmind/tiny-sd
openai/clip-vit-base-patch32
WordNet
```

DAAM 通过 pip 安装：

```bash
.venv/bin/pip install daam==0.2.0
```

### 输入

默认 prompt：

```text
a bicycle is parked beside the car.
```

也可通过 `--prompt` 或 `--input-json` 替换。

### 输出

```text
outputs/reproduction_real/00_original.png
outputs/reproduction_real/01_explicit_color_*.png
outputs/reproduction_real/02_explicit_shape_*.png
outputs/reproduction_real/03_explicit_combined_*.png
outputs/reproduction_real/04_implicit_hyponym_*.png
outputs/reproduction_real/result.json
```

### 操作步骤

```bash
cd /home/public/minzhi/patcher
CUDA_VISIBLE_DEVICES=0 \
NO_PROXY=antchat.alipay.com,antchat-gray.alipay.com \
HF_HOME=/home/public/minzhi/patcher/.hf_cache \
MPLCONFIGDIR=/home/public/minzhi/patcher/.mpl_cache \
HF_HUB_DISABLE_XET=1 \
.venv/bin/python reproduce_patcher.py \
  --output-dir outputs/reproduction_real \
  --prompt "a bicycle is parked beside the car." \
  --clip-threshold 0.28 \
  --num-inference-steps 8 \
  --max-explicit-per-type 1 \
  --max-hyponym-candidates 1 \
  --width 256 \
  --height 256 \
  --seed 2026
```

