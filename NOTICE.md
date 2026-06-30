# Third-party notices / 第三方致謝

This project bundles and builds upon the following works.
本專案打包並建立於以下作品之上。

---

## 1. verlet-js — Sub Protocol (bundled / 已打包)

The file `verlet-1.0.0.js` is the verlet-js physics engine, used verbatim.
`verlet-1.0.0.js` 為 verlet-js 物理引擎，原樣使用。

- Source: https://github.com/subprotocol/verlet-js
- The 8-legged spider builder and the `crawl()` routine in `index.html` are adapted from the
  project's `examples/spiderweb.html` (only draw colors and a `scale` factor were changed).
- License: **MIT**

```
Copyright 2013 Sub Protocol and other contributors
http://subprotocol.com/

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
"Software"), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
```

---

## 2. spider-webs-js — Greg Kepler (algorithm credit / 演算法致謝)

The `organicWeb()` generator in `index.html` is a **clean-room re-implementation** of the
web-generation algorithm from Greg Kepler's *spider-webs-js* (anchors → suspension frame →
rays → bars with organic gaps / Y-strands / reinforcement). **No source code was copied** —
the original is written on Processing.js + toxiclibs, whereas this is rebuilt on verlet-js.

`index.html` 中的 `organicWeb()` 是 Greg Kepler *spider-webs-js* 結網演算法的**淨室重寫**
（錨點 → 懸吊框 → 輻射絲 → 帶有機空隙／Y 形絲／補強的 bars）。**未複製任何原始碼**——原作建立
於 Processing.js + toxiclibs，本專案則重建於 verlet-js 之上。

- Source: https://github.com/gregkepler/spider-webs-js
- License: the original repository carries **no license file**. Algorithms/ideas are not
  copyrightable, so only the concept was reused; full credit goes to the original author.
  原 repo **無授權檔**；演算法／構想不受著作權保護，故僅借用概念，並完整致謝原作者。
