# 实验报告：基于 AI 的歌曲翻唱系统（简版）

## 1. Task Definition（任务定义）
- 目标：把中文 MTV 视频中的歌曲，自动转换为英文翻唱版本。
- 方案：提取音频 → 分离人声与伴奏 → 中文识别 → 英文翻译 → 英文歌声合成 → 时间对齐与混音 → 导出视频/音频。
- 评价标准：流程完整、结果清晰、代码关键环节可复现。

## 2. Method（方法）
- 流程图：`[视频输入] -> [音频提取] -> [人声分离(Demucs)] -> [语音识别(Whisper)] -> [文本翻译(M2M100)] -> [语音合成(Bark)] -> [对齐与混音] -> [输出]
  

- 简要说明：
  - 使用 FFmpeg 从视频里提取高质量音频；
  - 用 Demucs 把音频分成人声和伴奏两条轨；
  - Whisper 识别中文歌词并标注时间戳，过滤“作词/作曲”等非歌词；
  - M2M100 把中文翻译成英文，作为 TTS 的输入；
  - Bark 根据英文文本合成英文歌声；
  - 利用时间伸缩对齐到原曲时间轴，再与伴奏混音输出。

### 2.2 Every Module（模块详解）
- Video & Audio Extraction（视频/音频提取）
  - 输入：MP4 MTV 视频；输出：WAV（44.1kHz，16-bit PCM）。
- Source Separation（人声分离）
  - 工具：Demucs；输出：`vocals.wav` 与 `other.wav`。
- ASR & Translation（识别与翻译）
  - Whisper 识别中文，保留时间戳；关键词过滤；M2M100 翻译为英文。
- TTS & Mixing（合成与混音）
  - Bark 生成英文歌声；librosa 做时间伸缩（对齐）；与伴奏混音导出。

## 3. Codes（关键代码）

### 3.1 Code Block 1：Bark 语音合成
```python
# 将英文文本编码并生成音频
inputs = processor(
    text=[en_text],
    return_tensors="pt"
)
speech_values = model.generate(
    **inputs,
    do_sample=True
)
```
**Denotion（说明，约30词）：**先用处理器编码英文文本，再用 Bark 生成音频波形。开启采样提升表现力。结合时间戳逐段生成，便于后续与伴奏精准对齐。

### 3.2 Code Block 2：节奏对齐（Rhythm Alignment）
```python
# 根据目标时长计算变速比例
rate = current_duration_ms / target_duration_ms
rate = max(0.6, min(rate, 1.6))

# 超出阈值时进行时间伸缩
if abs(rate - 1.0) > 0.05:
    y_stretched = librosa.effects.time_stretch(y.astype(np.float32), rate=rate)
```
**Denotion（说明，约30词）：**用生成片段与目标时长的比例计算语速，设定上下限避免失真。必要时进行时间伸缩，使英文歌声与原曲节奏严格匹配。

## 4. Optimization（优化）

 具体思路如下：首先利用 YumeKey 将 Demucs 分离出的原始中文人声 WAV 转化为带有音高信息的 MIDI 文件。随后将该 MIDI 连同 M2M100 翻译生成的英文歌词导入 Synthesizer V Studio Pro。在该软件中，利用基于神经网络的 AI 歌声合成引擎，对导出的旋律进行精细化的微调（如调整音高曲线、颤音与力度）。最后导出专业级英文歌声。这种“自动识别 + 手工微调”的方案，解决了 AI 朗读没有起伏的痛点，实现了真正意义上的跨语言旋律翻唱，使作品具备了商业级的表现力。  


## 5. Results & Reproducibility（结果与复现）
- 中间文件：`output_project/translated_segments.json`（翻译结果）、`generated_audio/segment_*.wav`（合成片段）。
- 运行顺序：按 `001.ipynb` 的 Cell 1→2→3→4→5→6 执行，即可得到最终音频/视频。
- 依赖环境：`demucs`、`whisper`、`transformers`、`librosa`、`scipy` 等，已在笔记本中展示。
