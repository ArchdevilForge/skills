# Dial definitions（档位 → 布局/动效语义）

（从主文件分流。设完三个 dial 之后，需要把数字翻译成 CSS 时再读。）

## DESIGN_VARIANCE (1–10)

* **1–3 Predictable:** 对称 CSS Grid（12-col、equal fr）、均等 padding、居中
* **4–7 Offset:** `margin-top: -2rem` 重叠、混用图片比例（4:3 旁 16:9）、左对齐标题配居中数据
* **8–10 Asymmetric:** Masonry、`grid-template-columns: 2fr 1fr 1fr`、大块留白（`padding-left: 20vw`）
* **MOBILE OVERRIDE:** 4–10 在 `md:` 以上的不对称，`< 768px` 必须收成单列（`w-full px-4 py-8`）

## MOTION_INTENSITY (1–10)

* **1–3 Static:** 无自动动画。只有 CSS `:hover` / `:active`
* **4–7 Fluid CSS:** `transition` + `cubic-bezier(0.16, 1, 0.3, 1)`，只动 `transform` / `opacity`
* **8–10 Choreography:** scroll-driven / GSAP ScrollTrigger / Motion hooks。`window.addEventListener('scroll')` 硬禁，替代见主文件 5.D

## VISUAL_DENSITY (1–10)

* **1–3 Art Gallery:** 大留白，section `py-32`–`py-48`
* **4–7 Daily App:** `py-16`–`py-24`
* **8–10 Cockpit:** 紧 padding，不用卡片盒；1px 线分隔数据；数字强制 `font-mono`
