---
layout: post
title: Solution to mermaid won't render in jekyll
categories: [Geek]
tags: [mermaid,jekyll]
---

# Mermaid在jekyll生成的页面中不渲染的问题解决方法

---

## 背景

`Mermaid`本身不被`Github Pages`支持，需要用插件或手动加入`javascript`脚本来支持。

## 问题

从`mermaid`官方文档的[getting-started](https://mermaid.js.org/intro/getting-started.html#examples)页面说明来看，
将`mermaid.min.js`脚本加载进页面后使用`mermaid.initialize(config)`方法就可以开启渲染`mermaid`。

```html
<html lang="en">
  <head>
    <meta charset="utf-8" />
  </head>
  <body>
    <pre class="mermaid">
            graph LR
            A --- B
            B-->C[fa:fa-ban forbidden]
            B-->D(fa:fa-spinner);
    </pre>
    <pre class="mermaid">
            graph TD
            A[Client] --> B[Load Balancer]
            B --> C[Server1]
            B --> D[Server2]
    </pre>
    <script type="module">
      import mermaid from 'The/Path/In/Your/Package/mermaid.esm.mjs';
      mermaid.initialize({ startOnLoad: true });
    </script>
  </body>
</html>

```

本代码使用`Github Pages`生成发布网页，在`chrome`和`safari`浏览器上，只有当清空缓存第一次打开的时候可以渲染`mermaid`图象，后续打开只会显示代码。

## 解决方法

无意中看到`jekyll`的`mode-toogle.html`代码中的`updateMermaid()`方法，当显示模式变化后，会改变`confg`再次使用`mermaid.initialize(config)`方法，后面紧根`mermaid.init()`方法刷新。

```javascript
updateMermaid() {
    if (typeof mermaid !== "undefined") {
        let expectedTheme = (this.modeStatus === ModeToggle.DARK_MODE? "dark" : "default");
        let config = { theme: expectedTheme };

        /* re-render the SVG › <https://github.com/mermaid-js/mermaid/issues/311#issuecomment-332557344> */
        $(".mermaid").each(function() {
                let svgCode = $(this).prev().children().html();
                $(this).removeAttr("data-processed");
                $(this).html(svgCode);
                });

        mermaid.initialize(config);
        mermaid.init(undefined, ".mermaid"); // **刷新渲染mermaid一次**
    }
}

```
于是把`mermaid.init(undefined, ".mermaid");`这句代码加到初始化代码的`mermaid.initialize(config)`后，测试每次都成功渲染`mermaid`。

虽然问题解决，但原因未明，究其原因可能是浏览器在有缓存的情况下调用`mermaid.initialize(config)`时并没有真正去渲染。

