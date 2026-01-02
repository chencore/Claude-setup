---
name: modern-frontend-design
description: 用于创建独特、生产级前端界面的全面前端设计系统，避免通用的 AI 美学。当用户请求 Web 组件、页面、应用程序或任何前端界面时使用。提供设计工作流程、美学指南、代码模式、动画库、排版系统、色彩理论和反模式，以创建令人难忘的、特定于上下文的设计，感觉真正经过精心设计而不是生成的
---

# 现代前端设计系统

此技能提供了创建独特、生产级前端界面的全面系统，超越通用的 AI 美学。每个设计都应该感觉为其特定上下文有意设计。

## 核心哲学

**每个界面都在讲述一个故事。** 设计不是应用到功能上的装饰 - 它是目的、情感和交互合成为连贯体验的合成。

在编写任何代码之前，建立：
1. **上下文**：这解决什么问题？谁使用它？应该唤起什么情感？
2. **概念**：驱动所有设计决策的核心隐喻或想法是什么？
3. **承诺**：选择一个大胆的方向并精确地执行。

## 设计工作流程

### 第 1 阶段：发现和概念（总是从这里开始）

深入了解请求：
- 字面要求是什么？
- 根本需求是什么？
- 这应该唤起什么情感反应？
- 这与一切其他有什么不同？

从以下选择一个主要美学方向：
- **新野蛮主义**：原始混凝土纹理，粗体排版， harsh 对比
- **柔和极简主义**：柔和的调色板，慷慨的留白，微妙的交互
- **复古未来主义**：CRT 效果，扫描线，霓虹发光，赛博朋克元素
- **编辑/杂志**：动态网格，混合媒体，粗体文字处理
- **有机/自然**：流动形状，自然灵感调色板，纸张纹理
- **玻璃形态**：半透明层，背景滤镜，通过透明度深度
- **混乱的最大主义**：信息密度，拼贴美学，受控的混乱
- **装饰艺术**：几何图案，金色点缀，复古奢华
- **孟菲斯设计**：粗体图案，原色，游戏性几何
- **瑞士设计**：网格系统，无衬线字体，功能美
- **黑暗学院**：丰富的纹理，衬线排版，学术氛围
- **Y2K 复兴**：渐变，金属质感，早期网络怀旧
- **混合自定义**：结合 2-3 个方向创建独特内容

### 第 2 阶段：设计系统定义

在编码前定义您的设计令牌：
```css
/* 示例：新野蛮主义系统 */
:root {
  /* 排版比例 */
  --font-display: 'Archivo Black', sans-serif;  /* 永远不要使用 Inter/Roboto */
  --font-body: 'Work Sans', sans-serif;
  --scale-base: clamp(1rem, 2vw, 1.125rem);
  --scale-ratio: 1.333;  /* 完全四度 */

  /* 空间系统 */
  --space-unit: 0.5rem;
  --grid-columns: 12;
  --container-max: 1440px;

  /* 色彩理念 */
  --color-ink: #0A0A0A;
  --color-paper: #F7F3F0;
  --color-accent: #FF3E00;
  --color-bruise: #7B68EE;

  /* 动画时间 */
  --ease-out-expo: cubic-bezier(0.19, 1, 0.22, 1);
  --duration-base: 200ms;
  --stagger-delay: 50ms;
}
```

### 第 3 阶段：实现模式

#### 排版层次结构

永远不要使用默认字体堆栈。总是有意地配对字体：
```css
/* 坏 - 通用 AI 糟粕 */
font-family: Inter, system-ui, sans-serif;

/* 好 - 有意配对 */
font-family: 'Instrument Serif', 'Crimson Pro', serif;  /* 编辑 */
font-family: 'Space Mono', 'JetBrains Mono', monospace;  /* 技术 */
font-family: 'Bebas Neue', 'Oswald', sans-serif;  /* 粗体显示 */
font-family: 'Playfair Display', 'Libre Baskerville', serif;  /* 奢华 */
```

#### 色彩使用

避免可预测的渐变。有意地使用色彩：
```css
/* 坏 - 过度使用的紫色渐变 */
background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);

/* 好 - 上下文特定方法 */
/* Risograph 启发的双色 */
background:
  linear-gradient(45deg, #FF6B6B 0%, transparent 70%),
  linear-gradient(-45deg, #4ECDC4 0%, transparent 70%),
  #F7FFF7;

/* 噪声纹理覆盖 */
background:
  url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www[.]w3[.]org/2000/svg'%3E%3Cfilter id='noiseFilter'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.65' numOctaves='3' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noiseFilter)' opacity='0.02'/%3E%3C/svg%3E"),
  linear-gradient(180deg, #0A0E27 0%, #151B3D 100%);
```

#### 布局策略

有意地打破网格：
```css
/* 动态非对称网格 */
.container {
  display: grid;
  grid-template-columns: 2fr 1fr 1fr 3fr 1fr;
  grid-template-rows: auto 1fr auto;
  gap: clamp(1rem, 3vw, 2rem);
}

.hero-content {
  grid-column: 1 / span 3;
  grid-row: 2;
  z-index: 2;
}

.hero-visual {
  grid-column: 3 / -1;
  grid-row: 1 / span 2;
  margin-top: -10vh;  /* 打破容器边界 */
}
```

### 第 4 阶段：动画和交互

#### 入场动画

一个编排的入场胜过分散的微交互：
```css
[@]keyframes revealUp {
  from {
    opacity: 0;
    transform: translateY(30px) scale(0.98);
  }
}

.hero > * {
  animation: revealUp 800ms var(--ease-out-expo) both;
}

.hero > *:nth-child(1) { animation-delay: 0ms; }
.hero > *:nth-child(2) { animation-delay: 80ms; }
.hero > *:nth-child(3) { animation-delay: 160ms; }
```

#### 滚动触发效果
```javascript
// 带 Intersection Observer 的视差效果
const parallaxElements = document.querySelectorAll('[data-parallax]');
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const scrolled = window.pageYOffset;
      const rate = scrolled * entry[.]target.dataset.parallax;
      entry[.]target[.]style.transform = `translateY(${rate}px)`;
    }
  });
});
parallaxElements.forEach(el => observer.observe(el));
```

#### 悬停状态令人惊讶
```css
.card {
  transition: transform 200ms var(--ease-out-expo);
}

.card:hover {
  transform: perspective(1000px) rotateX(5deg) scale(1.02);
}

.card:hover::before {
  /* 揭示隐藏层 */
  opacity: 1;
  transform: translate(-5px, -5px);
}
```

## 必须避免的关键反模式

### "AI 外观"检查清单

永远不要一起做所有这些：
- ❌ 紫色/蓝色渐变背景
- ❌ Inter 或系统字体
- ❌ 居中的英雄区域与副标题
- ❌ 3 列功能卡片
- ❌ 万物圆角
- ❌ 所有卡片上的阴影
- ❌ #6366F1 作为主色
- ❌ 16px 边框半径
- ❌ "现代"、"干净"、"简单"作为唯一描述符

### 常见陷阱

1. **过度动画**：不是一切都需要移动。选择时刻。
2. **胆怯选择**：完全承诺您的美学方向。
3. **复杂性不匹配**：极简设计需要完美的细节，而不是复杂的代码。
4. **上下文忽视**：银行应用不应该看起来像音乐节网站。
5. **追逐趋势**：玻璃形态到处都是新的紫色渐变。

## 框架特定指导

### React 组件

对于 React，强调组合和状态管理：
```jsx
// 为复杂 UI 使用复合组件
const Card = ({ children }) => {
  const [isExpanded, setIsExpanded] = useState(false);
  return (
    <CardContext.Provider value={{ isExpanded, setIsExpanded }}>
      <article className="card" data-expanded={isExpanded}>
        {children}
      </article>
    </CardContext.Provider>
  );
};

Card.Header = ({ children }) => {
  const { setIsExpanded } = useContext(CardContext);
  return (
    <header onClick={() => setIsExpanded(prev => !prev)}>
      {children}
    </header>
  );
};
```

### Vue 组合

对于 Vue，利用响应式设计：
```vue
<script setup>
import { ref, computed } from 'vue'

const theme = ref('dark')
const themeClasses = computed(() => ({
  'theme-dark': theme.value === 'dark',
  'theme-light': theme.value === 'light'
}))
</script>
```

## 质量检查清单

在交付任何前端之前：

### 视觉冲击
- [ ] 它有明确的观点吗？
- [ ] 有人明天会记住这个吗？
- [ ] 它避免所有通用的 AI 模式吗？

### 技术卓越
- [ ] 所有断点都响应式？
- [ ] 可访问（ARIA 标签，键盘导航）？
- [ ] 性能优化（延迟加载，代码分割）？
- [ ] 跨浏览器测试？

### 对细节的关注
- [ ] 定义了自定义焦点状态？
- [ ] 设计了加载和错误状态？
- [ ] 微交互增强了可用性？
- [ ] 排版层次结构一致？

## 资源使用

### 脚本
- **generate-palette[.]py**：从基础颜色创建和谐的色彩系统
- **optimize-animations[.]py**：将 CSS 动画转换为 GPU 加速的变换
- **accessibility-check[.]py**：验证 WCAG 合规性

### 参考
- **aesthetic-systems[.]md**：每个设计方向的深度探索与示例
- **typography-pairings[.]md**：按心情和目的策划的字体组合
- **animation-curves[.]md**：自定义缓动函数和时间模式
- **color-psychology[.]md**：色彩选择的情感影响

### 资产
- **reset-styles/**：现代 CSS 重置变体
- **grid-systems/**：灵活的网格模板
- **icon-sets/**：自定义 SVG 图标库
- **texture-library/**：背景图案和噪声纹理

## 最终提醒

您不是在生成"前端" - 您在创造体验。每个选择都应该为概念服务。每个细节都应该强化故事。当用户看到它时，应该感受到一些东西。

使其令人难忘。使其独特。使其感觉被设计，而不是生成。