# 蜘蛛網 — 生成演算法 × verlet 物理

*[English README](./README.md)*

一個單檔、幾乎零依賴、**可以用滑鼠拉扯**的蜘蛛網。它把兩個經典的蛛網 demo 合而為一：

- Greg Kepler 的 *spider-webs-js* 的**結網生成演算法**（有機放射錨點 → 懸吊框 → 輻射絲 →
  帶有隨機空隙、Y 形絲與補強絲的 bars），以及
- Sub Protocol 的 *verlet-js* 的**物理引擎、滑鼠拖曳，以及會走動的蜘蛛**。

原始的生成程式綁死在 Processing.js + toxiclibs 上，而且不能互動。這裡把生成邏輯
**從零用 verlet-js 的 `Particle` + `DistanceConstraint` 重寫**，所以整張網是一塊真正的
verlet 布料：會因重力下垂、會回彈、可被拖曳——還有一隻蜘蛛在上面爬。

蛛絲的粗細、中心的**強化螺旋（strengthening spiral）**，以及無黏絲的**自由區（free zone）**，
都依照真實圓蛛結網的方式去調校（見[生物學依據](#生物學依據)）。

---

## 特色

- 🕸 **程序化結網**——每張網都不同：隨機的錨點數量／半徑、輻射密度、有機空隙、Y 形絲與補強絲。
- 🧲 **真正的 verlet 物理**——用滑鼠拖任意一條絲或蜘蛛，網會變形再回復。
- 🕷 **自主蜘蛛**——一隻 8 腳 verlet 蜘蛛會抓住並在網上爬行（也可以被拉離網面）。
- 🔬 **依生物學設計**——蛛絲粗細階層、hub 強化螺旋、中心自由區，皆模擬真實圓蛛網。
- 📦 **自包含**——一個 `index.html` + 一個 vendored 的 `verlet-1.0.0.js`（53 KB）。無 build、無 CDN、無連線。

## 執行方式

此頁必須透過 HTTP 提供（不能直接用 `file://` 開啟）。

```bash
cd spiderweb-integrated
python3 -m http.server 8000
# 然後開 http://localhost:8000/
```

請讓分頁保持在前景——動畫由 `requestAnimationFrame` 驅動，瀏覽器會節流背景分頁。

## 操作

| 動作 | 效果 |
| --- | --- |
| **拖曳**（按住滑鼠移動） | 抓住最近的絲／蜘蛛並拉動 |
| **R** 鍵 ／ **重新結網** 按鈕 | 生成一張全新的隨機網 |
| **蜘蛛開關** 按鈕 | 切換蜘蛛有無 |
| 縮放視窗 | 網會重建以符合尺寸 |

## 運作原理

`VerletJS.prototype.organicWeb(origin, opts)` 分六個階段建網，把 Greg Kepler 的結構邏輯
移植到 verlet 基本元件上：

1. **錨點**——`pointNum` 個放射方向，每個半徑**隨機化**，使外形是不規則多邊形而非圓形。
2. **輻射絲 rays**——每段「錨點→錨點」弧上內插出 `raysPerSegment` 條 ray；每條 ray 是一串
   從外圍周界點往中心 hub 延伸的粒子。
3. **hub**——所有 ray 匯聚到中心一個 hub 粒子。
4. **外框 frame**——相鄰的外圍點連成周界環。
5. **懸吊線 suspension**——錨點的 ray 往畫面邊緣射出一條絲並**釘住（pin）**；這些加固線把整張網撐住。
6. **捕捉螺旋**——相鄰 ray 在每一徑向層用 bars 相連，帶有隨機空隙、Y 形絲與額外補強——並在 hub
   周圍留下無黏絲的**自由區**，而自由區本身再由 3 圈**強化螺旋**包覆。

蜘蛛的建構與 `crawl()` 取自 verlet-js 的 spiderweb 範例（只改了繪製顏色與一個 `scale` 縮放係數）。
滑鼠拖曳是 verlet-js 核心內建的（`nearestEntity()` + `frame()` 裡的拖曳步驟）。

## 生物學依據

真實圓蛛網針對不同功能使用不同的絲，粗細差異很大。本繪製依此階層（粗 → 細）：

| 部位 | 真實絲種 | 功能 | 相對粗細 |
| --- | --- | --- | --- |
| 錨絲／橋接絲 | 大壺狀腺 dragline，常加固／多股捆束 | 承載整張網的重量 | **最粗** |
| 外框（周界） | dragline，常雙股（主框＋次框） | 結構支撐 | 粗 |
| hub／強化螺旋 | 中心周圍密集的非黏性絲圈 | 加固蜘蛛停駐的 hub | 中粗 |
| 輻射絲 radii（輻條） | 單股 dragline，剛性約為捕絲的 3 個數量級 | 吸收大部分獵物動能 | 中 |
| 捕捉螺旋 | 鞭狀腺核心，僅 **1–5 µm**，可伸長 300%，串有黏珠 | 黏捕獵物 | **最細** |

兩個依文獻建模的結構細節：

- **自由區（free zone）**——hub 周圍一圈無黏絲、只有輻射絲的區域，蜘蛛坐在這裡。
- **強化螺旋（strengthening spiral）**——數圈緊密的非黏性絲加固 hub（所以中心較密，並非每一圈都粗）。

## 專案結構

```
spiderweb-integrated/
├── index.html          # 整個應用：organicWeb 生成器 + 蜘蛛 + 繪製迴圈
├── verlet-1.0.0.js     # vendored 的 verlet-js 引擎（Sub Protocol, MIT）
├── README.md           # 英文說明
├── README.zh-TW.md     # 本檔（繁體中文）
├── LICENSE             # MIT（本專案）
└── NOTICE.md           # 第三方致謝
```

## 致謝與參考資料

**來源專案**

- **Greg Kepler — spider-webs-js**——本專案重寫的結網*演算法*來源。
  https://github.com/gregkepler/spider-webs-js
  *（未複製任何程式碼；原作無授權檔，故僅借用演算法概念——見 [NOTICE.md](./NOTICE.md)。）*
- **Sub Protocol — verlet-js**——物理引擎（vendored）、拖曳互動與蜘蛛／crawl 程式。MIT 授權。
  https://github.com/subprotocol/verlet-js

**蜘蛛網生物學**

- The elaborate structure of spider silk — *PMC* — https://pmc.ncbi.nlm.nih.gov/articles/PMC2658765/
- Spider orb webs rely on radial threads to absorb prey kinetic energy — *PMC* — https://pmc.ncbi.nlm.nih.gov/articles/PMC3385755/
- The secondary frame in spider orb webs — *Scientific Reports* — https://www.nature.com/articles/srep31265
- The engineering logic behind spider web geometry — *Aptive* — https://aptivepestcontrol.com/pests/spiders/the-engineering-logic-behind-spider-web-geometry/

## 授權

[MIT](./LICENSE) © 2026 djchrisssssss。第三方元件保留各自授權（[NOTICE.md](./NOTICE.md)）。
