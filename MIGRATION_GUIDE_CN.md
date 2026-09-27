# 华夏轨道模拟运营游戏：新电脑迁移与继续开发说明

> 最后核对日期：2026-09-27  
> GitHub 仓库：<https://github.com/jwu64287-debug/train-network-game>  
> 在线游戏：<https://jwu64287-debug.github.io/train-network-game/>  
> 主分支：`main`  
> 当前基准提交：`b904e36`（后续提交以 GitHub 上 `main` 的最新版本为准）

## 1. 迁移目标与最重要的结论

这个项目是一个不依赖后端、数据库或安装包的纯静态网页游戏。运行所需内容已经提交到 GitHub；在新电脑上克隆仓库后即可本地运行，推送到 `main` 后 GitHub Pages 会自动更新线上版本。

迁移时要把三类内容分开处理：

1. **游戏代码与可运行地图**：已经在 GitHub 仓库中，可通过克隆完整恢复。
2. **浏览器游戏存档**：保存在旧电脑浏览器的 `localStorage` 中，不在 GitHub，必须单独导出和导入。
3. **原始 CorelDRAW 文件**：最初的文件名为 `_of_架空高铁.cdr`，历史路径是 `C:\Users\asus\Desktop\CorelDRAW\cdr\Backup\_of\_架空高铁.cdr`。该文件没有提交到 GitHub，而且截至本说明编写时旧电脑该路径下已不存在。游戏仍可运行，因为转换后的 `dist/cdr-network.svg` 已在仓库中；但若以后需要在 CorelDRAW 中编辑原稿，必须从旧硬盘、回收站、网盘或其他备份中另行找回并复制。

## 2. 新电脑上的最短恢复流程

### 2.1 安装基础工具

建议安装：

- Git：<https://git-scm.com/downloads>
- Visual Studio Code（可选）：<https://code.visualstudio.com/>
- Python 3 或 Node.js，二选一即可，用于启动本地静态服务器
- GitHub Desktop（不熟悉命令行时可选）：<https://desktop.github.com/>

### 2.2 克隆项目

在新电脑选定一个工作目录，然后执行：

```powershell
git clone https://github.com/jwu64287-debug/train-network-game.git
cd train-network-game
git switch main
git pull --ff-only
```

如果使用 GitHub Desktop，选择 **File → Clone repository**，仓库选 `jwu64287-debug/train-network-game`。

### 2.3 本地启动

不要直接双击 `dist/index.html`。浏览器对本地 `file://` 页面加载 SVG `<object>` 有安全限制，可能导致线网或站点无法载入。应启动本地 HTTP 服务。

Python 方式：

```powershell
python -m http.server 4173 --directory dist
```

然后打开：

```text
http://127.0.0.1:4173/
```

如果使用 Node.js，可使用任意静态服务器把 `dist` 作为网站根目录。端口不必固定为 4173。

### 2.4 验证恢复是否成功

打开页面后至少检查以下内容：

- 能看到完整 CDR 转换线网和具名站点，而不是一张假地图。
- 可以拖动和缩放地图，也可以点击站点。
- 选站只能沿真实轨道图连通，不连通的站点不能建立线路。
- 线路显示为半透明彩色实线，不再是流动虚线。
- 新建线路可以选择马卡龙、千里江山、青花瓷、敦煌矿彩、夜行霓虹等色系。
- 时间开始流逝后，列车点沿线路连续移动。
- 已有存档若已导入，可以正常读取、改名、修改、删除线路。

## 3. 浏览器存档迁移（必须单独做）

### 3.1 存档在哪里

游戏没有云端账户系统。所有进度保存在访问该游戏的浏览器、该网址对应的 `localStorage` 中。

主要键名：

- `huaxiaRailActiveSlot`：当前存档槽
- `huaxiaRailSave_slot_1` 至 `huaxiaRailSave_slot_6`：六个存档
- `huaxiaRailSave`：早期版本的兼容存档键，可能存在，也可能不存在

`127.0.0.1`、`localhost` 和 GitHub Pages 是三个不同的网站来源，它们的存档互不相通。在哪个网址游玩，就必须在哪个网址下导出。

### 3.2 在旧电脑导出

1. 用平时实际游玩的同一个浏览器打开原游戏网址。
2. 按 `F12`，打开开发者工具。
3. 进入 **Console/控制台**。
4. 粘贴并执行下面这段代码：

```javascript
copy(JSON.stringify(Object.fromEntries(
  Object.keys(localStorage)
    .filter(k => k === 'huaxiaRailActiveSlot' || k === 'huaxiaRailSave' || k.startsWith('huaxiaRailSave_slot_'))
    .map(k => [k, localStorage.getItem(k)])
)))
```

执行后，存档 JSON 已复制到剪贴板。立即粘贴到一个 UTF-8 文本文件，例如 `huaxia-rail-saves-2026-09-27.json`，并随项目一起带到新电脑。

如果 Chrome/Edge 阻止向控制台粘贴，按控制台提示手动输入允许粘贴的确认文字，再重新粘贴。不要把存档内容公开上传，因为里面包含你的游戏进度。

### 3.3 在新电脑导入

1. 在新电脑打开与旧电脑相同的游戏网址。若旧存档来自 GitHub Pages，就打开 GitHub Pages；若来自本地 `127.0.0.1:4173`，就在新电脑也打开该本地地址。
2. 按 `F12` 打开控制台。
3. 把导出的完整 JSON 放到下面代码的 `PASTE_JSON_HERE` 位置，并执行：

```javascript
const saves = JSON.parse(`PASTE_JSON_HERE`);
Object.entries(saves).forEach(([key, value]) => localStorage.setItem(key, value));
location.reload();
```

如果 JSON 中含反引号，建议不要直接嵌入上述模板字符串，而是先把 JSON 文件内容复制到剪贴板，再在支持异步剪贴板读取的控制台中执行：

```javascript
const saves = JSON.parse(await navigator.clipboard.readText());
Object.entries(saves).forEach(([key, value]) => localStorage.setItem(key, value));
location.reload();
```

导入后依次切换 1—6 号存档核对线路数、资金、日期和车辆设置。

## 4. 项目目录与文件职责

```text
train-network-game/
├─ .github/
│  └─ workflows/
│     └─ pages.yml          # 推送 main 后自动发布 dist 到 GitHub Pages
├─ .openai/
│  └─ hosting.json          # 早期网站工具留下的配置；当前正式发布仍以 GitHub Pages 为准
├─ dist/
│  ├─ index.html            # 游戏全部界面、样式、数据和逻辑；当前为单文件实现
│  ├─ cdr-network.svg       # 从 CDR 转换出的可交互矢量底图，是运行关键文件
│  └─ network-map.jpg       # 辅助图片资源，保留以兼容历史版本
└─ MIGRATION_GUIDE_CN.md    # 本说明
```

当前没有 `package.json`、构建步骤、数据库或服务器 API。GitHub Pages 直接发布 `dist` 目录。

## 5. 当前游戏已经实现的核心功能

### 地图与路网

- `cdr-network.svg` 是底图和原始线路数据来源，不是普通截图。
- 页面从 SVG 的路径、文字和站点符号中识别站点并构造可点击节点。
- 站点按原图符号尺寸区分普通站、区域枢纽和大型枢纽。
- 地图支持自由缩放、拖动、复位以及线网/编辑/比例辅助模式。
- 已修正部分站名乱码，包括空格、异常括号及若干错误字符。

### 线路规划

- 用户按顺序选择起点、任意数量经停站和终点。
- 路线只能沿从 CDR 路径构建的铁路图运行。
- 在真正相接的节点可以换线；无法通过轨道图连通的站点不能建线。
- 选线禁止重复经过同一轨道边，防止走回头路。
- 可以跳过中间站，但列车的实际几何路径仍沿原轨道走。
- 里程按整条轨道路径积分，并结合地理校准结果计算，不按起终点直线距离计算。

### 运营系统

- 每条线路可设置名称、颜色、车型、速度、编组、票价、首末班和每天 1—24 对班次。
- 已建线路可以重命名、重新配置、调整班次和票价，也可以删除。
- 支持 120、140、160、200、250、300、350、400、450、600 km/h 等车辆等级。
- 一条线路可同时有多辆车双向运行，列车位置按游戏时钟和速度连续插值，不是半小时跳一帧。
- 共线同方向列车会加入追踪间隔，避免同速同向完全重合。
- 可升级铁路区间容量与限速，也可升级站点承载量。
- 线路详情即时显示理论客流、实际承运量、收入、能源、人员、车辆、铁路和站务成本以及每日盈亏。

### 需求与票价

- 理论需求主要由票价、人口、经济、流动意向以及车型/有效速度影响。
- 班次、座位和轨道容量只限制实际承运量，不反向改变“有多少人想出行”。
- 票价意愿曲线使用分段三次 Hermite 插值并限制在 0%—100%，避免极高票价反弹到 100%。
- 已知锚点：0 元/人公里约 100%，0.45 元约 92%，0.55 元约 75%，1 元约 10%，1.25 元及以上归零；车型和速度会调整有效心理价格。

### 存档与性能

- 六个本机存档槽，可自定义新档启动资金。
- 当前存档结构版本为 `version: 6`。
- 存档只保存必要字段；载入后重新计算路径、里程、车辆和财务衍生数据。
- 列车动画缓存 SVG 路径及长度，避免每一帧重复扫描整张 SVG。
- 列车保持逐帧动画，统计界面约每秒刷新，自动存档约每 15 秒写入一次。

## 6. 关键实现约束：继续开发时不要破坏

1. **不要把运营路径退化成两点直线。** `pathFor(l)` 必须使用由铁路图生成的 `routeD`。
2. **不要允许无轨道连接的站点建线。** 所有线路必须通过 `plannedRailRoute`/`shortestRail` 验证。
3. **不要恢复按同一颜色或同一原始 SVG path 判定换线。** 换线依据是铁路图的真实相交连接。
4. **不要限制经停站数量。** 只禁止回头和重复轨道边。
5. **不要把理论需求和班次数量绑定。** 班次只影响可承运量与成本。
6. **不要按起终点直线计算运营里程。** 必须沿 `routeStations`/轨道采样点计算。
7. **不要让平移手势吞掉站点点击。** 点击站点是选站；拖动地图空白区域才是平移。
8. **站点和列车标记必须随地图坐标系一起缩放和平移。** 不要叠加屏幕固定坐标的假点。
9. **保留旧存档迁移。** 修改存档格式时提升 `version`，并为旧字段提供兼容逻辑。
10. **不要删除 `cdr-network.svg`。** 它是没有原 CDR 时唯一可继续使用的矢量源。
11. **不要重新引入流动虚线。** 目前线路为低眩光的半透明彩色实线。
12. **地图裁切范围要覆盖最西、最北站点。** 历史上曾出现西侧和绥化未显示的问题。

## 7. 关键代码入口

所有逻辑目前都在 `dist/index.html` 内，可用下面的函数名快速定位：

- `cleanCdrName`：清洗从 SVG 识别的站名
- `initCdrEditableMap`：识别 SVG 路径、站点符号和文字并生成交互站点
- `buildRailGraph`：把 CDR 轨道路径构造成图
- `shortestRail`：在铁路图上寻径
- `plannedRailRoute`：组合起点、经停站和终点，并检查回头路
- `routeKilometers`：沿轨道路径计算里程
- `routeData`：生成新线路的完整运营数据
- `drawLines`：绘制已建线路与列车
- `renderTrainPositions`：逐帧更新列车位置
- `ticketWillingness`：票价和车型速度对应的购票意愿
- `economicDemand`：人口、GDP、人均 GDP 和流动意向形成的理论需求
- `recalculateExistingLine`：重新计算线路客流、成本与利润
- `openLine` / `applyServicePlan`：线路详情和调整
- `persist` / `restore`：本机存档
- `normalizeSavedLines`：读取存档后按当前轨道重新生成路线和里程

## 8. 修改、检查和发布流程

### 8.1 修改前

```powershell
git switch main
git pull --ff-only
git status
```

如果 `git status` 显示用户自己的未提交修改，先备份或提交，不要覆盖、重置或删除。

### 8.2 修改后最低限度检查

1. 浏览器实际打开本地服务器页面。
2. 测试新建线路、经停站、无法连通的站点、回头路线拦截。
3. 测试时间流逝和多车动画。
4. 测试线路修改、重命名、删除和每日盈亏更新。
5. 测试保存、刷新、读取以及创建新存档。
6. 在窄屏或手机尺寸下检查地图和管理面板。
7. 检查控制台没有 JavaScript 运行错误。

可额外执行：

```powershell
git diff --check
```

### 8.3 发布

```powershell
git add dist/index.html dist/cdr-network.svg
git commit -m "简要说明本次修改"
git push origin main
```

推送后，`.github/workflows/pages.yml` 自动部署。到仓库 **Actions** 页面查看 `Deploy game to GitHub Pages` 是否显示绿色成功。发布地址始终是：

```text
https://jwu64287-debug.github.io/train-network-game/
```

如果浏览器仍显示旧资源，可在网址末尾临时添加新的查询参数，例如 `?v=39`，或强制刷新。若修改 `cdr-network.svg`，也应同步更新 `index.html` 中加载 SVG 的版本参数，避免旧缓存。

## 9. GitHub 登录与权限

- 仓库所有者账号是 `jwu64287-debug`。
- 新电脑需要在浏览器、Git Credential Manager 或 GitHub Desktop 中登录该账号，才能推送。
- 不要把 GitHub 密码、Personal Access Token 或浏览器 Cookie 写入仓库。
- 如果只需要运行游戏，公开仓库无需登录即可克隆。
- 如果以后更改仓库名或用户名，GitHub Pages 地址与 Git remote 都要同步修改。

## 10. 原始 CDR 的处理建议

当前仓库里的 `cdr-network.svg` 足以继续游戏开发，它保留了矢量路径和文字。但 SVG 不等于完整的 CorelDRAW 工程，可能没有原始图层命名、页面设置、Corel 专属效果和编辑历史。

如果在旧设备或备份中找到 `_of_架空高铁.cdr`：

1. 原样复制到新电脑的独立资料目录。
2. 再复制一份只读备份，不要直接覆盖唯一原件。
3. 不建议直接提交到公开 GitHub，除非确认其中没有版权或隐私问题，并能接受较大的二进制仓库体积。
4. 可使用 Git LFS 或私有云盘保存 CDR 原件。
5. 每次重新导出 SVG 后，必须比较站点坐标、文字编码、路径数量与连接关系；不能直接覆盖线上 `cdr-network.svg` 后发布。

## 11. 当前已知限制与后续改进方向

- 存档仍是浏览器本机存储，没有账号同步；后续最值得做的是游戏内“导出存档/导入存档”按钮。
- 代码集中在单个 HTML 文件，功能继续增长后应拆分为地图、寻径、经济、存档、UI 和动画模块。
- 站点识别依赖 SVG 的文字和符号位置；若重新导出 SVG，字体、坐标或对象结构变化可能导致识别数量变化。
- 经济数据目前是区域级模型和中心城市近似，不是逐城市完整统计库。
- GitHub Pages 是纯静态托管，不能直接提供云存档、多人联机或服务器权威模拟；这些功能需要后端。
- 目前自动化主要验证发布成功，缺少针对寻径、票价曲线、里程与存档迁移的单元测试。

## 12. 可直接交给新电脑 Codex 的项目上下文

把以下文字作为新任务的第一条消息，并让 Codex先阅读本文件：

```text
这是“华夏轨道模拟运营游戏”的继续开发任务。项目仓库是
https://github.com/jwu64287-debug/train-network-game
主分支 main，线上地址是
https://jwu64287-debug.github.io/train-network-game/

请先完整阅读仓库根目录 MIGRATION_GUIDE_CN.md，再检查 git status、最新提交、
dist/index.html、dist/cdr-network.svg 和 .github/workflows/pages.yml。

这是一个基于原 CDR 矢量铁路图转换而来的静态网页游戏。站点和线路必须来自真实 SVG 对象，
运营路线必须沿铁路图寻径，允许在真实相交节点换线，不连通则禁止建线，不得用两点直线代替。
经停站数量不限，但不得走回头路。里程必须沿实际轨道计算。地图点位、线路和列车必须在同一
坐标系内缩放和平移。理论客流与班次容量必须分离。旧存档必须保持兼容。

在修改前先说明你对现状和约束的理解；完成后必须本地运行验证，再提交并推送 main，确认
GitHub Pages Action 成功。不得删除或简化 cdr-network.svg，也不得覆盖用户未提交的修改。
```

## 13. 迁移完成清单

- [ ] 新电脑已克隆 GitHub 仓库
- [ ] `main` 已更新到 GitHub 最新提交
- [ ] 本地 HTTP 地址能完整显示 SVG 线网和站点
- [ ] 旧电脑所有使用过的游戏网址都已分别检查存档
- [ ] 六个存档已导出为 JSON 并备份
- [ ] 新电脑已在相同网址来源下导入存档
- [ ] GitHub 账号已登录并具备推送权限
- [ ] 做过一次小修改、提交、推送和 Pages 发布测试
- [ ] 已从旧硬盘/备份寻找原始 `_of_架空高铁.cdr`
- [ ] 原始 CDR 若找到，已有至少两份独立备份

完成以上项目后，新电脑就具备独立运行、继续开发、保存进度和发布新版本的完整条件。
