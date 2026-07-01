# 蜘蛛網 — 生成演算法 × verlet 物理

*[English README](./README.md)*

![黑寡婦蜘蛛在程序化生成的蛛網中心進食，網上各深度都黏著獵物](./docs/screenshot.jpg)

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
- 🕷 **黑寡婦蜘蛛**——頭朝下垂絲降落，以「至少 4 腳著地」的步態在網上爬行，拉遠可把牠扯離網面。
- 🐛 **即時狩獵模擬**——生成昆蟲（蒼蠅／蛾／蚊子／蟑螂），飛入並黏在網上任意深度；蜘蛛會狩獵：
  接近 → 撕咬並纏成白繭 → 搬回中心 → 吸食 → 吸乾的殼掉落消失。附真實的獵物優先權邏輯。
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
| **🐛 放昆蟲** 按鈕 | 生成一隻隨機昆蟲飛到網上 |
| **拖曳絲／蜘蛛** | 拉動；把蜘蛛拉遠會扯離網面（之後會自己爬回） |
| **拖曳昆蟲** | 把黏在網上的昆蟲拔起（放開後掉落消失） |
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

## 狩獵模擬

昆蟲是輕量物件（非 verlet 粒子），從畫面邊緣飛入網上的隨機**深度**，黏在最近的絲上並掙扎
（掙扎會搖動網），振幅逐漸衰減。蜘蛛跑一個小狀態機：

```
下降 → 回中心 → 等待 → 接近 → 纏繞 → 搬運 → 吸食 →（掉落）→ 等待
```

纏繭時間依獵物大小（**4–8 秒**）；吸食為計時（不萎縮）。獵物優先權依真實行為建模：

- 永遠被**最新**黏住的昆蟲吸引；
- 中斷纏繞時**保留該獵物的半纏進度**（之後回來纏完）；
- **搬繭回中心不可中斷**；
- **吸食可被中斷**——蜘蛛丟下繭去抓新獵物，之後再回來把繭吸完（抓獵永遠優先於吸食）。

## 進度與路線圖

**已完成**

- verlet-js 上的程序化結網、依生物學調校的蛛絲階層、自由區、強化螺旋、純黑呈現。
- 黑寡婦蜘蛛：頭朝下垂絲降落 → 回中心 → 等待。
- 完整獵物生命週期（飛入 → 黏網 → 掙扎 → 接近 → 纏繞 → 搬運 → 吸食 → 掉落）。
- 多獵物優先權／中斷邏輯（最新優先、保留半纏、搬運不可中斷、吸食可續）。
- 滑鼠互動：拖絲／蜘蛛、拔昆蟲、把蜘蛛扯離網並可自行爬回。
- 穩定的「至少 4 腳著地」步態；抓握更軟，爬行時較不扯網。

**未來優化方向**

- **真正的交替四足步態**——可見的「4 抬 4 落」擺動循環，讓走路更擬真（目前保證 ≥4 腳著地，
  但未動畫化擺動相）。
- **可見的吐絲**——纏繞時從紡器畫出環繞獵物的絲。
- **降低搬運造成的網變形**——進一步調整抓握剛性／身體耦合。
- **網的破損與修補**——大型獵物扯斷絲，蜘蛛重新織補。
- **效能**——為 `nearestWebParticle`／`nearestWebTo` 加空間雜湊（目前每次線性掃描約 500 粒子），
  以支援更大更密的網與更多昆蟲。
- **獵物多樣性**——會掙脫的獵物、各蟲種不同速度與大小，以及畫面上的控制面板（密度／重力／生成率等滑桿）。
- **觸控／行動裝置支援**，以及可選的露珠／發光美術風格。

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
