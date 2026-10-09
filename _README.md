# 一、整体作用 

给身份证照片 / 扫描件添加 平铺式半透明水印 ，让图片在保持可读的同时，明确标注用途， 降低被挪作他用的风险。所有处理都在浏览器本地完成，图片不会上传到服务器。 

# 二、功能模块解析 

**1.** 场景模板（核心亮点） 

提供两套预设参数，点击即切换全套配置： 

场景 基准宽度 📄 A4 纸上的身份证 2480px 🪪 纯身份证照片 1000px 关键设计是 按比例自适应 ：参数以“图片宽度 / 基准宽度”的比值缩放。例如一张 1240px 宽的 A4 图，字号 = 40 × (1240/2480) = 20px ，这样不同分辨率的图都能得到视 觉一致的水印密度。 

**2.** 自动场景识别 

上传图片后根据宽高比自动判断场景： 

- <mark>比例 0.6~0.8 且宽度 ≥ 1200 → A4 （ A4 竖版约 0.707</mark> ） 

- <mark>比例 1.4~1.8 → 身份证横版（约 1.58</mark> ） 

- <mark>其他按尺寸：≥ 1500 判</mark> A <mark>4 ，否则判身份证 识别后在页面上显示提示文字。</mark> 

   **3.** 颜色模板 

预设 10 种颜色（浅蓝、浅红、浅灰、深蓝、深红、橙、绿、紫、黑、白），带名称、色 值和用途说明（推荐 / 低调 / 醒目 / 慎用等），点击圆形色块即可切换，并同步更新颜色选择 器和当前颜色显示。 

   **4.** 参数调节面板 

- <mark>文字大小（ 10~200 ，滑块</mark> + <mark>实时数值）</mark> 

- <mark>旋转角度（数字输入，默认 -30°</mark> ） 

- <mark>字体（微软雅黑 / 宋体 / 黑体 / 楷体 /Arial 等</mark> 8 <mark>种）</mark> 

- <mark>透明度（ 0~100 ，滑块）</mark> 

- <mark>水平间距、行间距（像素）</mark> 

- <mark>自定义颜色（ color 选择器）</mark> **<mark>5.</mark>** <mark>缩放与视图</mark> 

- <mark>缩放滑块（ 10%~200%</mark> ） 

- <mark>“适应屏幕”按钮：按容器宽高自动计算合适缩放比例</mark> **<mark>6.</mark>** <mark>实时预览与导出</mark> 

- <mark>多数参数（旋转、字体、透明度、间距、颜色、水印文字）修改后 立即重绘</mark> 

- <mark>点击“添加水印”正式生成</mark> 

- <mark>“下载图片”按钮将 canvas 导出为 PNG</mark> 

# 三、核心实现原理 

**1. Canvas** 绘制流程 复制代码 隐藏代码 <mark>redrawOriginalImage</mark> () // <mark>先清空并画原图</mark> ↓ <mark>ctx.save() + translate( 中心 ) + rotate( 角度</mark> ) ↓ <mark>双重循环平铺文字</mark> ↓ <mark>ctx.restore</mark> () **<mark>2.</mark>** <mark>水印平铺算法（关键）</mark> 复制代码 隐藏代码 

<mark>const diagonal = Math.sqrt(w² + h²);</mark> // <mark>画布对角线 const cols = ceil(diagonal / (textWidth + hSpacing));const rows = ceil(diagonal / vSpacing);for (i = -rows; i < rows; i++)</mark> 

<mark>for (j = -cols; j < cols; j++) fllText</mark> i <mark>(text, j*(textWidth+hSpacing), i*vSpacing);</mark> 

以画布中心为原点旋转后，用对角线长度确保旋转后的四个角也被覆盖， -rows ~ rows 的 正负范围保证左右上下都铺满。 

**3.** 透明度实现 

颜色选择器只给 RGB ，透明度单独用滑块控制，通过 hexToRgba() 转 成 rgba(r,g,b,alpha) 再赋给 fillStyle 。 

**4.** 状态管理 

- <mark>originalImage ：原图对象</mark> 

- <mark>lastAppliedWatermark ：上次水印参数快照，缩放时用它重绘</mark> 

- <mark>currentZoom</mark> / <mark>currentScene ：当前缩放和场景</mark> **<mark>5.</mark>** <mark>实时重绘的触发链</mark> 

- <mark>滑块 / 输入框 input 事件 → 调用 applyWatermark()</mark> 

- <mark>applyWatermark() 内部先 redrawOriginalImage() 清除旧水印，再重新绘制</mark> 

- <mark>缩放时 applyZoom() 改 canvas 的 CSS 尺寸（不改内部分辨率），再重绘，保证清晰</mark> 度 

总体是一个 纯前端、零依赖、开箱即用 的实用小工具，场景模板 + 自动识别 + 实时预览 的设计对普通用户很友好。 

