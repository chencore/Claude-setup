---
name: tapestry
description: 统一的内容提取和行动计划。当用户说"tapestry <URL>"、"weave <URL>"、"帮我计划 <URL>"、"提取并计划 <URL>"、"使这可操作 <URL>"或类似短语表明他们想要提取内容并创建行动计划时使用。自动检测内容类型（YouTube 视频、文章、PDF）并相应处理。
---

# Tapestry：统一内容提取 + 行动计划

这是**主技能**，协调整个 Tapestry 工作流：
1. 从 URL 检测内容类型
2. 使用适当的技能提取内容
3. 自动创建 Ship-Learn-Next 行动计划

## 何时使用此技能

当用户：
- 说"tapestry [URL]"
- 说"weave [URL]"
- 说"帮我计划 [URL]"
- 说"提取并计划 [URL]"
- 说"使这可操作 [URL]"
- 说"将 [URL] 转化为计划"
- 提供 URL 并要求"从中学习和实施"
- 想要完整的 Tapestry 工作流（提取 → 计划）

**需要注意的关键词**：tapestry, weave, plan, actionable, extract and plan, make a plan, turn into action

## 它如何工作

### 完整工作流：
1. **检测 URL 类型**（YouTube、文章、PDF）
2. **使用适当的技能提取内容**：
   - YouTube → youtube-transcript 技能
   - 文章 → article-extractor 技能
   - PDF → 下载并提取文本
3. **使用 ship-learn-next 技能创建行动计划**
4. **保存**内容文件和计划文件
5. **向用户呈现**摘要

## URL 检测逻辑

### YouTube 视频

**检测模式**：
- `youtube.com/watch?v=`
- `youtu.be/`
- `youtube.com/shorts/`
- `m.youtube.com/watch?v=`

**操作**：使用 youtube-transcript 技能

### 网络文章/博客文章

**检测模式**：
- `http://` 或 `https://`
- 不是 YouTube，不是 PDF
- 常见域名：medium.com、substack.com、dev.to 等
- 任何 HTML 页面

**操作**：使用 article-extractor 技能

### PDF 文档

**检测模式**：
- URL 以 `.pdf` 结尾
- URL 返回 `Content-Type: application/pdf`

**操作**：下载并提取文本

### 其他内容

**回退**：
- 尝试 article-extractor（适用于大多数 HTML）
- 如果失败，通知用户不支持的类型

## 逐步工作流

### 步骤 1：检测内容类型

```bash
URL="$1"

# 检查 YouTube
if [[ "$URL" =~ youtube\.com/watch || "$URL" =~ youtu\.be/ || "$URL" =~ youtube\.com/shorts ]]; then
    CONTENT_TYPE="youtube"

# 检查 PDF
elif [[ "$URL" =~ \.pdf$ ]]; then
    CONTENT_TYPE="pdf"

# 检查 URL 是否返回 PDF
elif curl -sI "$URL" | grep -i "Content-Type: application/pdf" > /dev/null; then
    CONTENT_TYPE="pdf"

# 默认为文章
else
    CONTENT_TYPE="article"
fi

echo "📍 检测到：$CONTENT_TYPE"
```

### 步骤 2：按类型提取内容

#### YouTube 视频

```bash
# 使用 youtube-transcript 技能工作流
echo "📺 提取 YouTube 讲稿..."

# 1. 检查 yt-dlp
if ! command -v yt-dlp &> /dev/null; then
    echo "正在安装 yt-dlp..."
    brew install yt-dlp
fi

# 2. 获取视频标题
VIDEO_TITLE=$(yt-dlp --print "%(title)s" "$URL" | tr '/' '_' | tr ':' '-' | tr '?' '' | tr '"' '')

# 3. 下载讲稿
yt-dlp --write-auto-sub --skip-download --sub-langs en --output "temp_transcript" "$URL"

# 4. 转换为干净文本（去重）
python3 -c "
import sys, re
seen = set()
vtt_file = 'temp_transcript.en.vtt'
try:
    with open(vtt_file, 'r') as f:
        for line in f:
            line = line.strip()
            if line and not line.startswith('WEBVTT') and not line.startswith('Kind:') and not line.startswith('Language:') and '-->' not in line:
                clean = re.sub('<[^>]*>', '', line)
                clean = clean.replace('&amp;', '&').replace('&gt;', '>').replace('&lt;', '<')
                if clean and clean not in seen:
                    print(clean)
                    seen.add(clean)
except FileNotFoundError:
    print('错误：找不到讲稿文件', file=sys.stderr)
    sys.exit(1)
" > "${VIDEO_TITLE}.txt"

# 5. 清理
rm -f temp_transcript.en.vtt

CONTENT_FILE="${VIDEO_TITLE}.txt"
echo "✓ 已保存讲稿：$CONTENT_FILE"
```

#### 文章/博客文章

```bash
# 使用 article-extractor 技能工作流
echo "📄 提取文章内容..."

# 1. 检查提取工具
if command -v reader &> /dev/null; then
    TOOL="reader"
elif command -v trafilatura &> /dev/null; then
    TOOL="trafilatura"
else
    TOOL="fallback"
fi

echo "使用：$TOOL"

# 2. 根据工具提取
case $TOOL in
    reader)
        reader "$URL" > temp_article.txt
        ARTICLE_TITLE=$(head -n 1 temp_article.txt | sed 's/^# //')
        ;;

    trafilatura)
        METADATA=$(trafilatura --URL "$URL" --json)
        ARTICLE_TITLE=$(echo "$METADATA" | python3 -c "import json, sys; print(json.load(sys.stdin).get('title', 'Article'))")
        trafilatura --URL "$URL" --output-format txt --no-comments > temp_article.txt
        ;;

    fallback)
        ARTICLE_TITLE=$(curl -s "$URL" | grep -oP '<title>\K[^<]+' | head -n 1)
        ARTICLE_TITLE=${ARTICLE_TITLE%% - *}
        curl -s "$URL" | python3 -c "
from html.parser import HTMLParser
import sys

class ArticleExtractor(HTMLParser):
    def __init__(self):
        super().__init__()
        self.content = []
        self.skip_tags = {'script', 'style', 'nav', 'header', 'footer', 'aside', 'form'}
        self.in_content = False

    def handle_starttag(self, tag, attrs):
        if tag not in self.skip_tags and tag in {'p', 'article', 'main'}:
            self.in_content = True

    def handle_data(self, data):
        if self.in_content and data.strip():
            self.content.append(data.strip())

    def get_content(self):
        return '\n\n'.join(self.content)

parser = ArticleExtractor()
parser.feed(sys.stdin.read())
print(parser.get_content())
" > temp_article.txt
        ;;
esac

# 3. 清理文件名
FILENAME=$(echo "$ARTICLE_TITLE" | tr '/' '-' | tr ':' '-' | tr '?' '' | tr '"' '' | cut -c 1-80 | sed 's/ *$//')
CONTENT_FILE="${FILENAME}.txt"
mv temp_article.txt "$CONTENT_FILE"

echo "✓ 已保存文章：$CONTENT_FILE"
```

#### PDF 文档

```bash
# 下载并提取 PDF
echo "📑 下载 PDF..."

# 1. 下载 PDF
PDF_FILENAME=$(basename "$URL")
curl -L -o "$PDF_FILENAME" "$URL"

# 2. 使用 pdftotext 提取文本（如果可用）
if command -v pdftotext &> /dev/null; then
    pdftotext "$PDF_FILENAME" temp_pdf.txt
    CONTENT_FILE="${PDF_FILENAME%.pdf}.txt"
    mv temp_pdf.txt "$CONTENT_FILE"
    echo "✓ 从 PDF 提取文本：$CONTENT_FILE"

    # 可选：保留 PDF
    echo "保留原始 PDF？(y/n)"
    read -r KEEP_PDF
    if [[ ! "$KEEP_PDF" =~ ^[Yy]$ ]]; then
        rm "$PDF_FILENAME"
    fi
else
    # 没有 pdftotext
    echo "⚠️  未找到 pdftotext。已下载 PDF 但未提取。"
    echo "   使用以下命令安装：brew install poppler"
    CONTENT_FILE="$PDF_FILENAME"
fi
```

### 步骤 3：创建 Ship-Learn-Next 行动计划

**重要**：提取内容后始终创建行动计划。

```bash
# 读取提取的内容
CONTENT_FILE="[来自上一步]"

# 调用 ship-learn-next 技能逻辑：
# 1. 读取内容文件
# 2. 提取核心可操作课程
# 3. 创建 5 次练习进展计划
# 4. 保存为：Ship-Learn-Next Plan - [任务标题].md

# 完整细节请参阅 ship-learn-next/SKILL.md
```

**计划创建的关键点**：
- 提取可操作的课程（不只是摘要）
- 定义特定的 4-8 周任务
- 创建练习 1（本周可发布）
- 设计练习 2-5（渐进式迭代）
- 将计划保存为 markdown 文件
- 使用格式：`Ship-Learn-Next Plan - [简要任务标题].md`

### 步骤 4：呈现结果

向用户显示：
```
✅ Tapestry 工作流完成！

📥 内容已提取：
   ✓ [内容类型]：[标题]
   ✓ 已保存到：[filename.txt]
   ✓ 提取了 [X] 个单词

📋 行动计划已创建：
   ✓ 任务：[任务标题]
   ✓ 已保存到：Ship-Learn-Next Plan - [标题].md

🎯 您的任务：[一行摘要]

📍 练习 1（本周）：[练习 1 目标]

您什么时候会发布练习 1？
```

## 完整的 Tapestry 工作流脚本

```bash
#!/bin/bash

# Tapestry：提取内容 + 创建行动计划
# 使用方法：tapestry <URL>

URL="$1"

if [ -z "$URL" ]; then
    echo "用法：tapestry <URL>"
    exit 1
fi

echo "🧵 Tapestry 工作流启动中..."
echo "URL: $URL"
echo ""

# 步骤 1：检测内容类型
if [[ "$URL" =~ youtube\.com/watch || "$URL" =~ youtu\.be/ || "$URL" =~ youtube\.com/shorts ]]; then
    CONTENT_TYPE="youtube"
elif [[ "$URL" =~ \.pdf$ ]] || curl -sI "$URL" | grep -iq "Content-Type: application/pdf"; then
    CONTENT_TYPE="pdf"
else
    CONTENT_TYPE="article"
fi

echo "📍 检测到：$CONTENT_TYPE"
echo ""

# 步骤 2：提取内容
case $CONTENT_TYPE in
    youtube)
        echo "📺 提取 YouTube 讲稿..."
        # [上面的 YouTube 提取代码]
        ;;

    article)
        echo "📄 提取文章..."
        # [上面的文章提取代码]
        ;;

    pdf)
        echo "📑 下载 PDF..."
        # [上面的 PDF 提取代码]
        ;;
esac

echo ""

# 步骤 3：创建行动计划
echo "🚀 创建 Ship-Learn-Next 行动计划..."
# [使用 ship-learn-next 技能的计划创建]

echo ""
echo "✅ Tapestry 工作流完成！"
echo ""
echo "📥 内容：$CONTENT_FILE"
echo "📋 计划：Ship-Learn-Next Plan - [标题].md"
echo ""
echo "🎯 下一步：查看您的行动计划并发布练习 1！"
```

## 错误处理

### 常见问题：

**1. 不支持的 URL 类型**
- 尝试文章提取作为回退
- 如果失败："无法从此 URL 类型提取内容"

**2. 未提取内容**
- 检查 URL 是否可访问
- 尝试 alternate 提取方法
- 通知用户："提取失败。URL 可能需要身份验证。"

**3. 工具未安装**
- 尽可能自动安装（yt-dlp、reader、trafilatura）
- 如果自动安装失败，提供安装说明
- 在可用时使用回退方法

**4. 空或无效的内容**
- 在创建计划前验证文件有内容
- 如果提取失败不要创建计划
- 在计划前向用户显示预览

## 最佳实践

- ✅ 始终显示检测到什么（"📍 检测到：youtube"）
- ✅ 为每个步骤显示进度
- ✅ 同时保存内容文件和计划文件
- ✅ 显示提取内容的预览（前 10 行）
- ✅ 自动创建计划（不要询问）
- ✅ 在结束时显示清晰的摘要
- ✅ 询问承诺问题："您什么时候会发布练习 1？"

## 使用示例

### 示例 1：YouTube 视频（使用"tapestry"）

```
用户：tapestry https://www.youtube.com/watch?v=dQw4w9WgXcQ

Claude：
🧵 Tapestry 工作流启动中...
📍 检测到：youtube
📺 提取 YouTube 讲稿...
✓ 已保存讲稿：Never Gonna Give You Up.txt

🚀 创建行动计划...
✓ 任务：掌握视频制作
✓ 已保存计划：Ship-Learn-Next Plan - 掌握视频制作.md

✅ 完成！您什么时候会发布练习 1？
```

### 示例 2：文章（使用"weave"）

```
用户：weave https://example.com/how-to-build-saas

Claude：
🧵 Tapestry 工作流启动中...
📍 检测到：article
📄 提取文章...
✓ 使用 reader（Mozilla Readability）
✓ 已保存文章：How to Build a SaaS.txt

🚀 创建行动计划...
✓ 任务：构建 SaaS MVP
✓ 已保存计划：Ship-Learn-Next Plan - 构建 SaaS MVP.md

✅ 完成！您什么时候会发布练习 1？
```

### 示例 3：PDF（使用"帮我计划"）

```
用户：帮我计划 https://example.com/research-paper.pdf

Claude：
🧵 Tapestry 工作流启动中...
📍 检测到：pdf
📑 下载 PDF...
✓ 已下载：research-paper.pdf
✓ 已提取文本：research-paper.txt

🚀 创建行动计划...
✓ 任务：应用研究发现
✓ 已保存计划：Ship-Learn-Next Plan - 应用研究发现.md

✅ 完成！您什么时候会发布练习 1？
```

## 依赖关系

此技能协调整个其他技能，因此需要：

**对于 YouTube：**
- yt-dlp（自动安装）
- Python 3（用于去重）

**对于文章：**
- reader（npm）或 trafilatura（pip）
- 如果两者都不可用，则回退到基本 curl

**对于 PDF：**
- curl（内置）
- pdftotext（可选 - 来自 poppler 包）
  - 安装：`brew install poppler`（macOS）
  - 安装：`apt install poppler-utils`（Linux）

**对于计划：**
- 无额外要求（使用内置工具）

## 理念

**Tapestry 将学习内容编织成行动。**

统一的工作流确保您不只是消费内容 - 您总是创建一个实施计划。这使被动学习转变为主动构建。

提取 → 计划 → 发布 → 学习 → 下一个。

这就是 Tapestry 的方式。