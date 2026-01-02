---
name: gemini-imagegen
description: 使用 Gemini API（Nano Banana）生成和编辑图像。当用户想要从文本提示创建图像、编辑现有图像、应用风格迁移、生成带文本的标志、创建贴纸、产品模型或任何图像生成/处理任务时使用此技能。支持文本到图像、图像编辑、多轮改进和从多个参考图像组合
---

# Gemini 图像生成

使用 Google 的 Gemini API 生成和编辑图像。需要 `GEMINI_API_KEY` 环境变量。

## 默认输出和日志

当用户未指定位置时，将图像保存到：
```
/Users/samarthgupta/Documents/generated images/
```

每个生成的图像都有一个配套的 `.md` 文件，其中包含使用的提示（例如，`logo.png` → `logo.md`）。

当收集参数（宽高比、分辨率）时，提供指定自定义输出位置的选项。

---

## 核心提示原则

**叙述性地描述场景，不要列出关键词。** Gemini 具有深入的语言理解能力 - 像散文而不是标签一样编写提示。

```
❌ "猫，巫师帽，魔法，幻想，4k，详细"

✓ "一只毛茸茸的橙色虎斑猫优雅地坐在天鹅绒垫子上，
   戴着一顶装饰着银星的华丽紫色巫师帽。
   柔和的烛光从左侧照亮场景。
   氛围是异想天开的但庄严的。"
```

### 公式

```
[主题 + 形容词] 在 [位置/上下文] 中做 [动作]。
[构图/相机]。[光照/氛围]。[风格/媒介]。[约束]。
```

不是每个提示都需要每个元素 - 将细节与意图相匹配。

### 规定性 vs 开放式提示

**规定性**（用户有特定愿景）：详细描述，精确规格
**开放式**（探索/希望模型创造力）：一般方向，让模型决定细节

两者都有效。如果不清楚，询问用户的意图。

---

## 能力模式

### 照片级写实场景
像摄影师一样思考：描述镜头、光线、时刻。
- 指定相机（85mm 人像，24mm 广角），光圈（f/1.8 背景模糊，f/11 整体清晰）
- 描述光照方向和质量（黄金时刻从相机左侧，三点柔光箱）
- 包含氛围和格式（宁静，垂直人像）

### 产品摄影
- **隔离**：干净的白色背景，均匀柔和的光照，电子商务就绪
- **生活方式**：使用环境中的产品，自然设置，有抱负但真实
- **英雄镜头**：电影般的构图，戏剧性光照，文本叠加的空间

### 标志和文本（使用专业模型）
- 将文本放在引号中：`'Morning Brew Coffee Co'`
- 描述字体："干净粗体无衬线字体，字母间距宽敞"
- 指定配色方案、形状约束、设计意图
- 使用多轮聊天进行迭代改进

### 风格化插图
- 命名风格："kawaii 风格贴纸"，"动漫影响"，"复古旅行海报"
- 描述设计语言："粗线条，平色，cel 着色"
- 包含格式约束："白色背景"，"模切贴纸格式"

### 编辑图像
- **确认主体**："使用我提供的猫的图像..."
- **明确保留**："保持所有内容不变，除了..."
- **真实集成**："应该看起来像自然地印在布料上"

模式：确认 → 指定更改 → 描述集成 → 保留其余

### 多图像组合（专业模型）
- 首先声明输出目标
- 分配元素："从第一个图像取 X，从第二个图像取 Y"
- 描述集成要求（光照匹配，真实阴影）
- 支持最多 14 个参考图像

### 角色一致性
- 使用多轮聊天会话获取多个视图
- 在后续中明确引用独特特征
- 包含"完全相同的角色"或"保持所有设计细节"
- 成功设计保存为将来提示的参考

---

## 通过命名唤起美学

名称唤起美学。模型学习了胶片、相机、工作室、艺术家和风格的关联。不要描述特征，而是直接引用名称。

```
"黄金时刻人像，使用柯达 Portra 400 拍摄"
→ 温暖的肤色，柔和的高光，细腻的颗粒感

"吉卜力工作室森林场景"
→ 茂密的自然，柔和的光照，异想天开的氛围

"时尚编辑，哈苏中画幅"
→ 异常的细节，浅景深，那种中画幅外观
```

这适用于摄影、动画、插图、游戏艺术、平面设计、美术—任何具有可识别视觉身份的东西。

**有关胶片、相机、工作室、艺术家和风格的全面词典，请参阅 [STYLE_REFERENCE.md](STYLE_REFERENCE.md)。**

---

## 模型

| 模型 | 最适用于 |
|-------|----------|
| `gemini-2.5-flash-image` | 速度，迭代，简单生成（1024px 固定） |
| `gemini-3-pro-image-preview` | 文本渲染，复杂指令，高分辨率（高达 4K），多图像组合，Google Search 基础 |

**默认值**：专业模型使用 1K 分辨率，1:1 宽高比。更改前与用户确认。

### 图像配置（仅专业模型）

**宽高比**：1:1, 2:3, 3:2, 3:4, 4:3, 4:5, 5:4, 9:16, 16:9, 21:9
**分辨率**：1K（约 1024px），2K（约 2048px），4K（约 4096px）

---

## 高级功能

### Google Search 基础（仅专业模型）
使用 `--grounding` 标志启用实时数据有帮助时：
- 天气可视化
- 当前事件信息图
- 真实世界数据图表

### 多轮改进
使用聊天进行迭代编辑，而不是在一次尝试中完善提示：
```
→ "为 Acme Corp 创建标志"
→ "使文本更粗"
→ "添加蓝色渐变背景"
```

### 语义遮罩
不需要手动遮罩。对话式地描述更改：
- "将沙发改为红色皮革"
- "用日落海滩替换背景"
- "从天空移除电线"

---

## 脚本

```bash
# 从提示生成
python scripts/generate_image.py "prompt" output.png [--model MODEL] [--aspect RATIO] [--size SIZE] [--grounding]

# 编辑现有图像
python scripts/edit_image.py input.png "instruction" output.png [--model MODEL] [--aspect RATIO] [--size SIZE]

# 组合多个图像
python scripts/compose_images.py "instruction" output.png img1.png [img2.png ...] [--model MODEL] [--aspect RATIO] [--size SIZE]

# 交互式多轮聊天
python scripts/multi_turn_chat.py [--model MODEL] [--output-dir DIR]
```

模型：`gemini-2.5-flash-image`（默认），`gemini-3-pro-image-preview`

---

## 核心 API 模式

```python
from google import genai
from google.genai import types

client = genai.Client(api_key=os.environ["GEMINI_API_KEY"])

response = client.models.generate_content(
    model="gemini-2.5-flash-image",
    contents=["您的叙述性提示"],
    config=types.GenerateContentConfig(response_modalities=["TEXT", "IMAGE"])
)

for part in response.parts:
    if part.inline_data:
        # 从 part.inline_data.data 保存图像
```

对于带配置的专业模型：
```python
config=types.GenerateContentConfig(
    response_modalities=['TEXT', 'IMAGE'],
    image_config=types.ImageConfig(aspectRatio="16:9", imageSize="2K"),
    tools=[{"google_search": {}}]  # 可选基础
)
```

---

## 快速检查

生成前：
- [ ] 叙述性描述（不是关键词列表）？
- [ ] 照片级写实的相机/光线细节？
- [ ] 文本在引号中，字体样式描述？
- [ ] 适合任务的模式（专业模式用于文本/复杂）？
- [ ] 宽高比适合用例？
- [ ] 用户偏好：规定性还是开放式？