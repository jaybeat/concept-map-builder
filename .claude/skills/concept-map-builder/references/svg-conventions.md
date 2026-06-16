# 概念图 SVG 渲染约定(Claude Code 自包含版)

本版把概念图写成一个**自包含 `.svg` 文件**:所有颜色、字体、节点/连线样式都内嵌在 `<style>` 里,用**具体值**而非 CSS 变量,不依赖任何运行时类。这样文件用浏览器直接打开就有完整样式。布局/连接词/交叉连接的**原则**与通用版一致,本文件只给出落地用的模板与具体取值。

## 文件骨架(直接套用)

```svg
<?xml version="1.0" encoding="UTF-8"?>
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 720 440" width="720" font-family="system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif">
  <style>
    .bg   { fill:#ffffff; }
    .th   { font-size:14px; font-weight:600; }
    .ts   { font-size:11px; fill:#475569; }
    .arr  { stroke:#64748b; stroke-width:1.5; fill:none; }
    .lbl  { fill:#ffffff; }                 /* 连接词底色块,遮断线 */
    .xlink{ stroke:#D85A30; stroke-width:1.5; fill:none; }  /* 交叉连接 */
    .xtext{ fill:#9a3412; font-size:11px; }                 /* 交叉连接文字 */
    /* 节点配色:按语义分,不按顺序 */
    .c-purple rect{ fill:#f3e8ff; stroke:#9333ea; } .c-purple .th{ fill:#6b21a8; }
    .c-blue   rect{ fill:#e0f2fe; stroke:#0284c7; } .c-blue   .th{ fill:#075985; }
    .c-teal   rect{ fill:#ccfbf1; stroke:#0d9488; } .c-teal   .th{ fill:#115e59; }
    .c-amber  rect{ fill:#fef3c7; stroke:#d97706; } .c-amber  .th{ fill:#92400e; }
    .c-gray   rect{ fill:#f1f5f9; stroke:#64748b; } .c-gray   .th{ fill:#334155; }
  </style>
  <rect class="bg" x="0" y="0" width="720" height="440"/>
  <defs>
    <marker id="arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="context-stroke" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
  </defs>
  <!-- 焦点问题写在最顶部 -->
  <text class="ts" x="40" y="20">焦点问题：……？</text>
  <!-- 连线、连接词、节点…… -->
</svg>
```

要点:`<rect class="bg">` 给白底(标签遮块也用白 `.lbl`,与背景一致);`viewBox`/`width` 按内容调整;节点放在 `<g class="node c-XXX">` 里。

## 节点

- 圆角矩形 `rx="8"`、`stroke-width="0.5"`(描边色由 `.c-XXX rect` 给)。
- 框内:概念名用 `class="th"`,下面可加一行简短**属性/限定**子标题 `class="ts"`,如"哈希表 / 均摊 O(1)·无序"。子标题要**准确**,别为简洁而失真。
- 文字垂直居中:`dominant-baseline="central"`;两行时标题在框中心上方约 9px、子标题下方约 9px。
- 终端环境无点击交互,**不要**用 `onclick`/`sendPrompt`。

## 连接词(命题的关键)

- 每条线都要有连接词,放线上,**底下垫一个白色小矩形 `class="lbl"`** 把线遮断再写字——这是把"连线"变"命题"的标准画法。
- 连接词短、具体、有方向("取决于""导致""由…构成""需有序时选"),禁用"相关/有关/包括"。
- 读出来必须是通顺且为真的一句话。**命题方向要对**:别把度量/属性概念当被度量物的父节点(如把数据结构挂在"时间复杂度"下,会被读成"复杂度包含它们"——应让结构作终点、复杂度作其属性挂出)。

## 层级与配色

- 自上而下:概括在顶、具体在底;焦点问题用一行 `class="ts"` 写在最顶。
- **配色按语义/分支分,不按顺序循环**(如目标=紫、分析维度=蓝、度量尺子=橙、候选/实例=绿、中性=灰),控制在 3–4 个色系内。

## 交叉连接(权重最高,主动找)

- 跨分支连线用 `class="xlink"`(珊瑚色 `#D85A30`)、带箭头,与普通层级连线区分;文字用 `class="xtext"`。
- 主动找"看似无关的两块其实强相关"的连接——这是把图从"树"变成"网"的关键,也是评估里分值最高的一项。

## 防交叉布局(很重要)

线交叉是可读性头号杀手,按优先级:
1. **排好节点顺序**:常一起连的概念相邻,减少长程线。
2. **从单点扇出不自交**:父节点扇向多子节点时扇形不自交;把连接词放在**靠子节点那端**(各线已分开),别堆在汇聚处。
3. **交叉连接走边距**:跨版面的线用 L 形路径沿空白边距走(`<path d="M.. H.. V.. H..">`),绕开框。
4. **可接受少量线—线交叉**:网状图本就会有交叉,只要交叉点落空白处、不压框和标签即可,别把连接词放交叉点上。
5. **超载就拆图**:精确连线超过约五六条、怎么排都打架时,停手,按 `focus-question.md` 拆成互链子图。

## 标签防重叠与字宽

- 每个连接词配 `.lbl` 白底块,尺寸刚好盖住文字与其下的线。
- 中日韩字符宽度约等于字号(`th` 14px、`ts` 11px),据此估算宽度,确保文字不溢出框/标签;标签之间留足间距,放线的清爽段,避开箭头与交叉点。

## 写文件与交付

- 把完整 SVG 写到 `concept-map-<英文短名>.svg`(默认当前工作目录,或用户指定路径)。
- 修订时可覆盖同名文件,或写 `-v2.svg` 保留历史。
- 完成后报告**绝对路径**,并说一句"用浏览器打开即可"。不要假设有 `open`/浏览器。
- 想要单文件可分享的话,SVG 已自包含;若用户更想要可嵌网页,可改写成内嵌同样 `<style>` 的 `.html`。

## 渐进呈现(可选,利于教学)

需要展示"怎么一步步搭"时,可先写一张**骨架图**(只有框和无字连线)文件,再写一张**成图**文件(补全连接词、交叉连接、实例),让用户对比看清连接词与交叉连接带来的差别。
