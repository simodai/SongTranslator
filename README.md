<div align="center">

# 🎵 SongTranslator

### AI-Powered Cross-Language Song Cover Pipeline

[![Python](https://img.shields.io/badge/Python-3.8+-blue?logo=python&logoColor=white)](https://www.python.org/)
[![License](https://img.shields.io/badge/License-MIT-green)](./LICENSE)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange?logo=jupyter&logoColor=white)](https://jupyter.org/)

</div>

---

## 📖 项目简介

SongTranslator 是一套**全自动 AI 歌曲翻唱流水线**，能将一首中文歌曲（或 MTV 视频）自动转换为多种语言的翻唱版本。

核心链路：**音频提取 → 人声分离 → 语音识别 → 机器翻译 → 语音合成 → 节奏对齐 → 混音输出**

<p align="center">
  <img src="https://img.shields.io/badge/ASR-Whisper-68BC71?logo=openai&logoColor=white" />
  <img src="https://img.shields.io/badge/Separation-Demucs-0066FF?logo=meta&logoColor=white" />
  <img src="https://img.shields.io/badge/Translation-M2M100-E31E52?logo=huggingface&logoColor=white" />
  <img src="https://img.shields.io/badge/TTS-Suno%20Bark-FFD21E?logo=sunomusic&logoColor=black" />
  <img src="https://img.shields.io/badge/Mixing-librosa%20%2B%20pydub-7B42BC" />
</p>

---

## 🏗️ 系统架构

```
                      ┌──────────────────────────────────────┐
                      │           🎬 Video (eg. MTV)         │
                      └────────────────┬─────────────────────┘
                                       │  extract audio
                                       ▼
                      ┌──────────────────────────────────────┐
                      │              🔊 Audio M              │
                      └────────────────┬─────────────────────┘
                                       │  Demucs → vocals + backing
                                       ▼
                      ┌──────────────────────────────────────┐
                      │         🎤 Music2Text (Whisper)      │
                      │             Text T                   │
                      └────────────────┬─────────────────────┘
                                       │  M2M100 translation
                                       ▼
                      ┌──────────────────────────────────────┐
                      │        🌐 Machine Translation        │
                      │             Text T*                  │
                      └────────────────┬─────────────────────┘
                                       │  Suno Bark synthesis
                                       ▼
                      ┌──────────────────────────────────────┐
                      │       🎶 Text2Music (Suno Bark)      │
                      │             Audio M*                 │
                      └────────────────┬─────────────────────┘
                                       │  time-stretch + mix with backing
                                       ▼
                      ┌──────────────────────────────────────┐
                      │   🎥 Insert M* → New Target Video    │
                      └────────────────┬─────────────────────┘
                                       │  optimization
                                       ▼
                      ┌──────────────────────────────────────┐
                      │      ⚡ Optimization Algorithms      │
                      │  rhythm alignment · clarity enhance  │
                      └──────────────────────────────────────┘
```

---

## 🌍 语言支持

| Flag | 源语言 | 目标语言 | Code |
|:---:|:---:|:---:|:---:|
| 🇨🇳 → 🇬🇧 | 中文 | English | `zh→en` |
| 🇨🇳 → 🇩🇪 | 中文 | Deutsch | `zh→de` |
| 🇨🇳 → 🇫🇷 | 中文 | Français | `zh→fr` |
| 🇨🇳 → 🇯🇵 | 中文 | 日本語 | `zh→ja` |
| 🇨🇳 → 🇰🇷 | 中文 | 한국어 | `zh→ko` |
| 🇨🇳 → 🇮🇳 | 中文 | Indian | `zh→hi` |

---

## 🔧 核心流程 (6 Steps)

| Step | 模块 | 工具 | 说明 |
|:---:|:---|:---|:---|
| **1** | 🎬 音频提取 | `FFmpeg` | 从视频/音频文件中提取高质量原始音频 |
| **2** | 🎤 语音识别 | `Whisper` | 识别中文歌词，输出带时间戳的文本 |
| **3** | 🌐 机器翻译 | `M2M100-418M` | 逐句翻译为目标语言 |
| **4** | 🎶 语音合成 | `Suno Bark` | 生成统一音色的目标语言歌声 |
| **5** | ⏱️ 节奏对齐 | `librosa` | 时间伸缩匹配原曲节奏 |
| **6** | 🎛️ 混音输出 | `pydub` | 新歌声 + 原伴奏合成最终文件 |

---

## ⚙️ 环境配置

| 要求 | 说明 |
|:---|:---|
| **Python** | `≥ 3.8` |
| **GPU** | CUDA 可选，有 GPU 速度更快 |
| **OS** | Windows / macOS / Linux |

```bash
pip install demucs openai-whisper transformers suno-bark pydub librosa soundfile torch torchaudio
```

---

## 🤖 模型下载

模型通过 HuggingFace 自动下载，也可离线使用。将模型放置于以下目录：

```
models/
├── m2m100_418M/          # facebook/m2m100_418M  (翻译)
└── sunobark-small/        # ylacombe/bark-small   (TTS + speaker embeddings)
```

```python
# 自动下载
from transformers import M2M100ForConditionalGeneration, M2M100Tokenizer
model = M2M100ForConditionalGeneration.from_pretrained("facebook/m2m100_418M")

from transformers import BarkModel
model = BarkModel.from_pretrained("suno/bark-small")
```

---

## 🚀 快速开始

按顺序运行 `basic.ipynb` 中的 Cell 即可：

| Cell | 操作 | 输出 |
|:---:|:---|:---|
| 1 | ⚙️ 配置路径 | — |
| 2 | 🎵 人声伴奏分离 | `output_project/htdemucs/<song>/vocals.wav` · `other.wav` |
| 3 | 📝 语音转写 | `output_project/transcript.txt` |
| 4 | 🌐 机器翻译 | `output_project/translated_segments.json` |
| 5 | 🎤 英文合成 | `output_project/bark_outputs/segment_*.wav` |
| 6 | 🎛️ 对齐混音 | `output_project/final_mix_output.wav` |

---

## 🎨 可调参数

| 参数 | 位置 | 说明 |
|:---|:---|:---|
| 🎙️ **音色** | Cell 5 | 切换 `speaker_embeddings/v2/` 下不同 `en_speaker_*` 的 NPY 组合 |
| 🔊 **音量平衡** | Cell 6 末尾 | `final_output = (backing - 5).overlay(final_mix + 2)` |
| ⏩ **变速范围** | Cell 6 | `rate = max(0.7, min(rate, 1.5))` 控制时间伸缩强度 |

---

## 📂 输出文件一览

```
output_project/
├── htdemucs/<song>/
│   ├── vocals.wav              # 🎤 分离出的人声
│   └── other.wav               # 🎸 伴奏
├── transcript.txt              # 📝 中文歌词 + 时间戳
├── translated_segments.json    # 🌐 英文翻译
├── bark_outputs/
│   └── segment_*.wav           # 🎶 TTS 片段
└── final_mix_output.wav        # ✅ 最终成品
```

---

## 📄 License

MIT © [simodai](https://github.com/simodai)
