# SATINT 方案B 实施计划

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** 将 satint-demo 重构为"眼见为实"叙事：态势分析（卫星真相层）→ 信实链（验证层）→ 智能预测（AI预测层），对标安总 openclaw demo 的功能深度。

**Architecture:** App.jsx 调整路由顺序，SitePackage 升级为三栏态势分析，mockData 新增 ACI/DCI/status/dailyData/facilities 字段，信实链迁移路由并增强联动，报告改名并强化叙事标签。

**Tech Stack:** React 18 + Vite + React Router DOM + Leaflet（动态import）+ 纯 SVG 绘图（无新依赖）

---

## Task 1: 路由 + 导航栏重构

**Files:**
- Modify: `src/App.jsx`

**Step 1: 修改 NAV_ITEMS 和 Routes**

将 `src/App.jsx` 中 NAV_ITEMS 替换为：

```jsx
const NAV_ITEMS = [
  { path: '/',        label: '态势分析', sub: 'SITUATION MAP',  icon: '◎' },
  { path: '/chain',   label: '信实链',   sub: 'EVENT CHAIN',    icon: '⬡' },
  { path: '/report',  label: '智能预测', sub: 'INTEL REPORT',   icon: '▣' },
]
```

Logo 区域 `SATINT` 右侧加数据流副标题，在 `<span>SATINT</span>` 后新增：

```jsx
<span style={{
  fontSize: '9px', color: '#1e3a5f', marginLeft: '8px',
  letterSpacing: '0.08em', borderLeft: '1px solid #1a2d45', paddingLeft: '8px',
}}>
  卫星影像 → 信实验证 → 智能预测
</span>
```

将 Routes 改为：

```jsx
<Routes>
  <Route path="/"            element={<SitePackage />} />
  <Route path="/site/:siteId" element={<SitePackage />} />
  <Route path="/chain"       element={<EventChain />} />
  <Route path="/report"      element={<Report />} />
</Routes>
```

import 顺序不变，组件名不改。

**Step 2: 验证**

```bash
npm run dev
```

访问 http://localhost:5173 — 应直接进入地图页，导航栏显示「态势分析 / 信实链 / 智能预测」，副标题可见。

**Step 3: Commit**

```bash
git add src/App.jsx
git commit -m "feat: 路由重构 - 态势分析为首页，信实链迁移至 /chain"
```

---

## Task 2: mockData.js 数据模型升级

**Files:**
- Modify: `src/data/mockData.js`

**Step 1: 为每个 SITE 添加新字段**

在 SITES 数组中，每个对象添加以下字段（按基地逐一填写）：

**S1 塔尔阿夫尔空军基地**（空袭出发基地，状态良好）
```js
status: 'operational',
strategicValue: 'S',
aci: 88,
dci: 75,
dailyData: {
  dates: ['10/28','10/29','10/30','10/31','11/01','11/02','11/03'],
  aci:   [82, 83, 85, 86, 87, 88, 88],
  dci:   [70, 71, 72, 73, 74, 75, 75],
},
facilities: [
  { name: '主跑道', damage: 0,  verified: true },
  { name: '加固机库', damage: 0,  verified: true },
  { name: 'F-16停机坪', damage: 0,  verified: true },
  { name: 'SAM阵地', damage: 0,  verified: true },
],
```

**S2 侦察机前进基地**（低战略价值，已部署）
```js
status: 'operational',
strategicValue: 'B',
aci: 40,
dci: 30,
dailyData: {
  dates: ['10/28','10/29','10/30','10/31','11/01','11/02','11/03'],
  aci:   [38, 39, 40, 40, 40, 40, 40],
  dci:   [28, 29, 30, 30, 30, 30, 30],
},
facilities: [
  { name: '临时跑道', damage: 0, verified: true },
  { name: '侦察设备区', damage: 0, verified: true },
],
```

**S3 胡拉玛军事设施**（受打击，严重损毁）
```js
status: 'destroyed',
strategicValue: 'A',
aci: 18,
dci: 15,
dailyData: {
  dates: ['10/28','10/29','10/30','10/31','11/01','11/02','11/03'],
  aci:   [75, 76, 77, 77, 75, 20, 18],
  dci:   [68, 68, 69, 70, 68, 18, 15],
},
facilities: [
  { name: '主机库(1#)', damage: 0,   verified: true },
  { name: '机库(2#受损)', damage: 95,  verified: true },
  { name: '主跑道', damage: 40,  verified: true },
  { name: '指挥中心', damage: 60,  verified: false },
],
```

**S4 霍尔木兹海峡监控区**（航母编队，已撤离）
```js
status: 'operational',
strategicValue: 'S',
aci: 92,
dci: 85,
dailyData: {
  dates: ['10/28','10/29','10/30','10/31','11/01','11/02','11/03'],
  aci:   [80, 82, 85, 88, 92, 92, 92],
  dci:   [72, 74, 78, 82, 85, 85, 85],
},
facilities: [
  { name: 'CVN-77 甲板', damage: 0, verified: true },
  { name: '护卫编队', damage: 0, verified: false },
],
```

**S5 岸基导弹阵地**（激活状态）
```js
status: 'damaged',
strategicValue: 'A',
aci: 55,
dci: 70,
dailyData: {
  dates: ['10/28','10/29','10/30','10/31','11/01','11/02','11/03'],
  aci:   [50, 51, 52, 53, 55, 55, 55],
  dci:   [62, 64, 65, 68, 70, 70, 70],
},
facilities: [
  { name: '发射车遮蔽棚', damage: 0,  verified: true },
  { name: '雷达站', damage: 20, verified: false },
],
```

**S6 某扩建空军基地**（施工中）
```js
status: 'damaged',
strategicValue: 'A',
aci: 62,
dci: 58,
dailyData: {
  dates: ['10/15','10/20','10/25','10/28','10/30','11/01','11/03'],
  aci:   [50, 53, 55, 58, 60, 61, 62],
  dci:   [48, 50, 52, 54, 56, 57, 58],
},
facilities: [
  { name: '跑道(延伸中)', damage: 0,  verified: true },
  { name: '新型雷达阵地', damage: 0,  verified: false },
  { name: '工程施工区', damage: 0,  verified: true },
],
```

**Step 2: 升级 REPORT.sources，添加 sourceType 字段**

```js
sources: [
  { label: 'OpenSky 飞行轨迹数据',     confidence: 0.95, verified: true,  sourceType: 'satellite' },
  { label: '卫星存档影像 (过境#4872)', confidence: 0.97, verified: true,  sourceType: 'satellite' },
  { label: '社交媒体爆炸报道',          confidence: 0.55, verified: false, sourceType: 'osint' },
  { label: '官方通报',                  confidence: 0.62, verified: false, sourceType: 'osint' },
  { label: 'AIS 舰船动态',             confidence: 0.93, verified: true,  sourceType: 'chain' },
],
```

**Step 3: 验证**

```bash
npm run dev
```

打开浏览器控制台，确认无 import 报错。切换各基地，右侧面板应仍能正常渲染（目前新字段尚未用到，不影响现有展示）。

**Step 4: Commit**

```bash
git add src/data/mockData.js
git commit -m "feat(data): 新增 ACI/DCI/status/strategicValue/dailyData/facilities 字段"
```

---

## Task 3: 态势分析 — 三栏布局 + 左侧基地列表

**Files:**
- Modify: `src/pages/SitePackage/index.jsx`

**Step 1: 新增左侧面板组件 `BaseList`**

在文件顶部（SiteMap 定义之前）添加：

```jsx
// ── 状态颜色 ─────────────────────────────────────────────────
function statusColor(status) {
  if (status === 'operational') return '#22c55e'
  if (status === 'damaged')     return '#f59e0b'
  return '#ef4444'  // destroyed
}

function statusLabel(status) {
  if (status === 'operational') return '运行中'
  if (status === 'damaged')     return '受损'
  return '被摧毁'
}

// ── 左侧基地列表 ──────────────────────────────────────────────
function BaseList({ sites, selectedId, onSelect }) {
  const [filter, setFilter] = useState('all')
  const filters = [
    { key: 'all',         label: '全部' },
    { key: 'operational', label: '运行中' },
    { key: 'damaged',     label: '受损' },
    { key: 'destroyed',   label: '被摧毁' },
  ]
  const visible = filter === 'all' ? sites : sites.filter(s => s.status === filter)

  return (
    <div style={{
      width: '220px', flexShrink: 0,
      background: '#040810', borderRight: '1px solid #1a2d45',
      display: 'flex', flexDirection: 'column', overflow: 'hidden',
    }}>
      {/* 标题 */}
      <div style={{ padding: '12px 14px', borderBottom: '1px solid #1a2d45' }}>
        <div style={{ fontSize: '9px', color: '#64748b', letterSpacing: '0.1em', marginBottom: '8px' }}>
          SITUATION MAP · 监控点位
        </div>
        {/* 筛选按钮 */}
        <div style={{ display: 'flex', gap: '4px', flexWrap: 'wrap' }}>
          {filters.map(f => (
            <button key={f.key} onClick={() => setFilter(f.key)}
              style={{
                padding: '3px 8px', borderRadius: '2px', cursor: 'pointer',
                fontSize: '9px', letterSpacing: '0.06em',
                background: filter === f.key ? '#f59e0b22' : 'transparent',
                border: `1px solid ${filter === f.key ? '#f59e0b' : '#1a2d45'}`,
                color: filter === f.key ? '#f59e0b' : '#64748b',
              }}
            >{f.label}</button>
          ))}
        </div>
      </div>

      {/* 基地列表 */}
      <div style={{ flex: 1, overflowY: 'auto' }}>
        {visible.map(site => {
          const isSelected = site.id === selectedId
          const sc = statusColor(site.status)
          return (
            <div key={site.id} onClick={() => onSelect(site.id)}
              style={{
                padding: '10px 14px', cursor: 'pointer',
                borderBottom: '1px solid #0d1a2e',
                borderLeft: `2px solid ${isSelected ? sc : 'transparent'}`,
                background: isSelected ? `${sc}0d` : 'transparent',
                transition: 'all 0.15s',
              }}
            >
              <div style={{ display: 'flex', alignItems: 'center', gap: '8px', marginBottom: '3px' }}>
                <div style={{
                  width: '7px', height: '7px', borderRadius: '50%',
                  background: sc, boxShadow: `0 0 4px ${sc}`,
                  flexShrink: 0,
                }} />
                <div style={{ fontSize: '11px', color: isSelected ? '#e2e8f0' : '#94a3b8', fontWeight: isSelected ? 600 : 400, lineHeight: 1.3 }}>
                  {site.name}
                </div>
              </div>
              <div style={{ display: 'flex', justifyContent: 'space-between', paddingLeft: '15px' }}>
                <span style={{ fontSize: '9px', color: '#334155' }}>{site.country}</span>
                <span style={{ fontSize: '9px', color: sc }}>{statusLabel(site.status)}</span>
              </div>
            </div>
          )
        })}
      </div>

      {/* 底部数据流提示 */}
      <div style={{ padding: '10px 14px', borderTop: '1px solid #1a2d45', fontSize: '9px', color: '#1e3a5f', lineHeight: 1.6 }}>
        ◉ 卫星影像 → 事实基础层<br/>
        点位数据锚定信实链节点
      </div>
    </div>
  )
}
```

**Step 2: 修改主组件布局为三栏**

将 `SitePackage` 主组件的 return 最外层从：
```jsx
<div style={{ display: 'flex', height: '100%', overflow: 'hidden' }}>
```
改为：
```jsx
<div style={{ display: 'flex', height: '100%', overflow: 'hidden' }}>
  <BaseList sites={SITES} selectedId={selectedId} onSelect={id => { setSelectedId(id); setImgIdx(0) }} />
  {/* 原有的地图+右侧详情 div 保持不变，包在一个 flex:1 的容器里 */}
  <div style={{ flex: 1, display: 'flex', overflow: 'hidden' }}>
    {/* 以下为原来的地图区 + 右侧详情，完整保留 */}
```

并在文件末尾对应位置加上闭合标签 `</div></div>`。

**Step 3: 验证**

```bash
npm run dev
```

左侧应出现基地列表，点击条目右侧详情切换，筛选按钮生效。整体三栏布局正确。

**Step 4: Commit**

```bash
git add src/pages/SitePackage/index.jsx
git commit -m "feat(situation): 三栏布局 + 左侧基地列表 + 状态筛选"
```

---

## Task 4: 地图标记三态色 + flyTo

**Files:**
- Modify: `src/pages/SitePackage/index.jsx`（SiteMap 组件）

**Step 1: 标记颜色改为 statusColor**

在 `SiteMap` 的 `sites.forEach` 中，将：
```jsx
const color = scoreColor(site.combatScore)
```
改为：
```jsx
const color = statusColor(site.status)
```

标记 HTML 内的数值从 `site.combatScore` 改为 `site.aci`（显示 ACI）。

**Step 2: 选中时 flyTo**

在 `SiteMap` 组件中新增 selectedId 响应：

```jsx
// 在 useEffect([], []) 之后添加：
useEffect(() => {
  if (!mapInstance.current || !selectedId) return
  const site = sites.find(s => s.id === selectedId)
  if (site) mapInstance.current.flyTo([site.lat, site.lng], 7, { duration: 1 })
}, [selectedId])
```

同时将 `SiteMap` 的 props 签名中已有的 `selectedId` 确认传入（现有代码已有此 prop，只是未使用）。

**Step 3: 图例更新**

将地图左上角图例的"战斗力评分"替换为"基地状态"：

```jsx
<div style={{ fontSize: '9px', color: '#64748b', marginBottom: '6px', letterSpacing: '0.1em' }}>基地状态</div>
{[
  ['operational', '#22c55e', '运行中'],
  ['damaged',     '#f59e0b', '受损'],
  ['destroyed',   '#ef4444', '被摧毁'],
].map(([s, c, l]) => (
  <div key={s} style={{ display: 'flex', alignItems: 'center', gap: '8px', marginBottom: '3px', fontSize: '10px' }}>
    <div style={{ width: '8px', height: '8px', borderRadius: '50%', background: c }} />
    <span style={{ color: c }}>{l}</span>
  </div>
))}
```

**Step 4: 验证**

运行 dev，切换基地时地图平滑飞行。三个状态的标记颜色分别为绿/黄/红。

**Step 5: Commit**

```bash
git add src/pages/SitePackage/index.jsx
git commit -m "feat(map): 标记三态色(operational/damaged/destroyed) + flyTo联动"
```

---

## Task 5: 右侧详情面板完整升级

**Files:**
- Modify: `src/pages/SitePackage/index.jsx`

这是工作量最大的 Task，逐步替换右侧面板内容。

**Step 1: 新增 `DualScore` 组件**（ACI + DCI 并排）

```jsx
function DualScore({ aci, dci }) {
  const aciColor = aci >= 80 ? '#22c55e' : aci >= 60 ? '#f59e0b' : '#ef4444'
  const dciColor = dci >= 80 ? '#22c55e' : dci >= 60 ? '#f59e0b' : '#ef4444'
  return (
    <div style={{ display: 'flex', gap: '10px', marginBottom: '14px' }}>
      {[['ACI', '攻击能力指数', aci, aciColor], ['DCI', '防御能力指数', dci, dciColor]].map(([key, label, val, color]) => (
        <div key={key} style={{
          flex: 1, padding: '12px', borderRadius: '4px',
          background: `${color}11`, border: `1px solid ${color}44`,
          textAlign: 'center',
        }}>
          <div style={{ fontSize: '8px', color: '#64748b', letterSpacing: '0.1em', marginBottom: '4px' }}>{key}</div>
          <div style={{ fontSize: '28px', fontWeight: 800, color, fontFamily: 'var(--font-display)', lineHeight: 1 }}>{val}</div>
          <div style={{ fontSize: '8px', color: '#334155', marginTop: '4px' }}>{label}</div>
        </div>
      ))}
    </div>
  )
}
```

**Step 2: 新增 `DualTrendChart` 组件**（ACI/DCI 双线 SVG）

```jsx
function DualTrendChart({ dailyData }) {
  const W = 280, H = 60
  const { dates, aci, dci } = dailyData
  const n = dates.length

  function toPath(values) {
    return values.map((v, i) => {
      const x = (i / (n - 1)) * W
      const y = H - (v / 100) * H
      return `${i === 0 ? 'M' : 'L'}${x.toFixed(1)},${y.toFixed(1)}`
    }).join(' ')
  }

  const aciPath = toPath(aci)
  const dciPath = toPath(dci)
  const lastDate = dates[n - 1]
  const firstDate = dates[0]

  return (
    <div>
      <div style={{ display: 'flex', justifyContent: 'space-between', fontSize: '8px', color: '#334155', marginBottom: '3px' }}>
        <span>{firstDate}</span>
        <span style={{ display: 'flex', gap: '10px' }}>
          <span style={{ color: '#22c55e' }}>— ACI</span>
          <span style={{ color: '#0ea5e9' }}>— DCI</span>
        </span>
        <span>{lastDate}</span>
      </div>
      <svg width={W} height={H} style={{ overflow: 'visible', display: 'block' }}>
        {/* 网格线 */}
        {[0.25, 0.5, 0.75, 1].map(r => (
          <line key={r} x1={0} y1={H * (1 - r)} x2={W} y2={H * (1 - r)}
            stroke="#0d1a2e" strokeWidth={0.5} />
        ))}
        {/* DCI 线（蓝） */}
        <path d={dciPath} fill="none" stroke="#0ea5e9" strokeWidth={1.5} />
        {/* ACI 线（绿） */}
        <path d={aciPath} fill="none" stroke="#22c55e" strokeWidth={1.5} />
        {/* 最后一个点高亮 */}
        <circle cx={W} cy={H - (aci[n-1]/100)*H} r={3} fill="#22c55e" />
        <circle cx={W} cy={H - (dci[n-1]/100)*H} r={3} fill="#0ea5e9" />
      </svg>
    </div>
  )
}
```

**Step 3: 新增 `DamageStatus` 组件**

```jsx
function DamageStatus({ status }) {
  const map = {
    operational: { label: '完好无损', color: '#22c55e', icon: '✓' },
    damaged:     { label: '受损',     color: '#f59e0b', icon: '⚠' },
    destroyed:   { label: '严重损毁', color: '#ef4444', icon: '✕' },
  }
  const { label, color, icon } = map[status] || map.operational
  return (
    <div style={{
      display: 'inline-flex', alignItems: 'center', gap: '6px',
      padding: '4px 10px', borderRadius: '2px',
      background: `${color}15`, border: `1px solid ${color}44`,
      fontSize: '10px', color,
    }}>
      <span>{icon}</span><span>{label}</span>
    </div>
  )
}
```

**Step 4: 新增 `FacilitiesList` 组件**

```jsx
function FacilitiesList({ facilities }) {
  return (
    <div>
      {facilities.map((f, i) => (
        <div key={i} style={{ marginBottom: '8px' }}>
          <div style={{ display: 'flex', justifyContent: 'space-between', marginBottom: '3px', fontSize: '10px' }}>
            <span style={{ color: '#94a3b8' }}>{f.name}</span>
            <span style={{ display: 'flex', alignItems: 'center', gap: '6px' }}>
              {f.verified
                ? <span style={{ color: '#22c55e', fontSize: '9px' }}>✓ 卫星验证</span>
                : <span style={{ color: '#334155', fontSize: '9px' }}>◌ 待核实</span>}
              <span style={{ color: f.damage > 0 ? '#ef4444' : '#22c55e', fontWeight: 700 }}>
                {f.damage > 0 ? `损毁 ${f.damage}%` : '完好'}
              </span>
            </span>
          </div>
          <div style={{ height: '3px', background: '#0d1a2e', borderRadius: '2px' }}>
            <div style={{
              width: `${f.damage}%`, height: '100%', borderRadius: '2px',
              background: f.damage > 60 ? '#ef4444' : f.damage > 20 ? '#f59e0b' : '#22c55e',
              transition: 'width 0.4s ease',
            }} />
          </div>
        </div>
      ))}
    </div>
  )
}
```

**Step 5: 新增 `RelatedEvents` 组件**（关联信实链事件）

在文件顶部 import 处添加 `EVENTS`：
```jsx
import { SITES, EVENTS } from '../../data/mockData'
```

组件：
```jsx
function RelatedEvents({ siteId, onGoToChain }) {
  const related = EVENTS.filter(e => e.siteId === siteId && e.verified)
  if (!related.length) return null
  return (
    <div>
      <div style={{ fontSize: '9px', color: '#64748b', letterSpacing: '0.1em', marginBottom: '8px' }}>
        信实链关联事件 · 卫星已锚定
      </div>
      {related.map(e => (
        <div key={e.id} style={{
          padding: '8px 10px', marginBottom: '4px',
          background: '#080f1e', borderRadius: '3px',
          border: '1px solid #1e3a5f',
          display: 'flex', justifyContent: 'space-between', alignItems: 'center',
        }}>
          <div>
            <div style={{ fontSize: '10px', color: '#e2e8f0' }}>{e.label.replace('\n', ' ')}</div>
            <div style={{ fontSize: '9px', color: '#64748b', marginTop: '2px' }}>{e.date} · {e.source}</div>
          </div>
          <button onClick={() => onGoToChain(e.id)}
            style={{
              padding: '3px 8px', fontSize: '9px',
              background: '#22c55e15', border: '1px solid #22c55e44',
              color: '#22c55e', borderRadius: '2px', cursor: 'pointer',
            }}>
            → 链
          </button>
        </div>
      ))}
    </div>
  )
}
```

**Step 6: 主组件引入 useNavigate，更新右侧面板**

在组件顶部添加：
```jsx
import { useParams, useNavigate } from 'react-router-dom'
// ...
const navigate = useNavigate()
```

将右侧详情面板（`width: '340px'` 那个 div）内容替换为：

```jsx
{/* 基地头部 */}
<div style={{ marginBottom: '14px', paddingBottom: '14px', borderBottom: '1px solid #1a2d45' }}>
  <div style={{ display: 'flex', justifyContent: 'space-between', alignItems: 'flex-start', marginBottom: '6px' }}>
    <div style={{ fontSize: '9px', color: '#64748b', letterSpacing: '0.1em' }}>
      {site.country} · {site.type}
    </div>
    {/* 战略价值徽章 */}
    <div style={{
      padding: '2px 8px', borderRadius: '2px', fontSize: '10px', fontWeight: 700,
      background: site.strategicValue === 'S' ? '#ef444422' : site.strategicValue === 'A' ? '#f59e0b22' : '#22c55e22',
      border: `1px solid ${site.strategicValue === 'S' ? '#ef4444' : site.strategicValue === 'A' ? '#f59e0b' : '#22c55e'}`,
      color: site.strategicValue === 'S' ? '#ef4444' : site.strategicValue === 'A' ? '#f59e0b' : '#22c55e',
    }}>
      {site.strategicValue} 级
    </div>
  </div>
  <div style={{ fontSize: '14px', fontWeight: 700, color: '#e2e8f0', marginBottom: '10px' }}>{site.name}</div>
  <DamageStatus status={site.status} />
</div>

{/* ACI / DCI 双评分 */}
<DualScore aci={site.aci} dci={site.dci} />

{/* 趋势图 */}
<div style={{ marginBottom: '14px', paddingBottom: '14px', borderBottom: '1px solid #1a2d45' }}>
  <div style={{ fontSize: '9px', color: '#64748b', letterSpacing: '0.1em', marginBottom: '8px' }}>能力趋势（7日）</div>
  <DualTrendChart dailyData={site.dailyData} />
</div>

{/* 综合能力雷达图 */}
<div style={{ marginBottom: '14px', paddingBottom: '14px', borderBottom: '1px solid #1a2d45' }}>
  <div style={{ fontSize: '9px', color: '#64748b', letterSpacing: '0.1em', marginBottom: '8px' }}>综合能力评估</div>
  <div style={{ display: 'flex', justifyContent: 'center' }}>
    <RadarChart site={site} />
  </div>
</div>

{/* 卫星影像（保留现有） */}
<div style={{ marginBottom: '14px', paddingBottom: '14px', borderBottom: '1px solid #1a2d45' }}>
  <div style={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center', marginBottom: '8px' }}>
    <div style={{ fontSize: '9px', color: '#64748b', letterSpacing: '0.1em' }}>点位影像</div>
    <div style={{ fontSize: '10px', color: statusColor(site.status) }}>
      验证置信度 {(site.imagery[imgIdx]?.score * 100).toFixed(0)}%
    </div>
  </div>
  <div style={{
    width: '100%', height: '140px', borderRadius: '4px',
    background: 'linear-gradient(135deg, #0d1a2e 0%, #1a2d45 50%, #0d1a2e 100%)',
    border: '1px solid #1a2d45',
    display: 'flex', alignItems: 'center', justifyContent: 'center',
    position: 'relative', overflow: 'hidden',
  }}>
    <div style={{ fontSize: '10px', color: '#334155', textAlign: 'center' }}>
      <div style={{ fontSize: '20px', marginBottom: '4px', opacity: 0.3 }}>◉</div>
      <div>WMTS 影像加载点</div>
      <div style={{ fontSize: '9px', marginTop: '2px', color: '#1a2d45' }}>{site.name}</div>
      <div style={{ fontSize: '9px', color: '#1a2d45' }}>{site.imagery[imgIdx]?.date}</div>
    </div>
    <div style={{
      position: 'absolute', top: 0, left: 0, right: 0, height: '2px',
      background: `linear-gradient(to right, transparent, ${statusColor(site.status)}88, transparent)`,
      animation: 'scanline 3s linear infinite',
    }} />
    <style>{`@keyframes scanline { 0%{top:0} 100%{top:100%} }`}</style>
  </div>
  <div style={{ marginTop: '6px', fontSize: '10px', color: '#94a3b8', lineHeight: 1.5 }}>
    <span style={{ color: '#64748b' }}>识别结果：</span>{site.imagery[imgIdx]?.desc}
  </div>
</div>

{/* 设施损毁清单 */}
<div style={{ marginBottom: '14px', paddingBottom: '14px', borderBottom: '1px solid #1a2d45' }}>
  <div style={{ fontSize: '9px', color: '#64748b', letterSpacing: '0.1em', marginBottom: '8px' }}>设施损毁评估</div>
  <FacilitiesList facilities={site.facilities} />
</div>

{/* 信实链关联事件 */}
<RelatedEvents
  siteId={site.id}
  onGoToChain={() => navigate('/chain')}
/>
```

注意：时间轴 div（底部影像选择器）保持在地图区域，不在右侧面板内，无需修改。

**Step 7: 验证**

```bash
npm run dev
```

右侧面板从上到下依次展示：战略价值徽章 / 基地名 / 受损状态标签 / ACI+DCI双分 / 7日双线趋势图 / 雷达图 / 影像 / 设施损毁 / 关联事件。点击「→ 链」跳转到 /chain。

**Step 8: Commit**

```bash
git add src/pages/SitePackage/index.jsx
git commit -m "feat(situation): 右侧面板完整升级 ACI/DCI/趋势图/设施损毁/关联事件"
```

---

## Task 6: 信实链 — 路由适配 + 节点脉冲 + 联动

**Files:**
- Modify: `src/pages/EventChain/index.jsx`

**Step 1: 路由已在 Task 1 迁移，确认 NavLink 的 `end` 属性**

App.jsx 中信实链的 NavLink `path="/chain"` 不需要 `end` 属性（非根路由），已正确。

**Step 2: 验证节点脉冲动画**

在 EventChain 的 SVG 渲染中，找到绘制节点的代码（verified 节点的外圈）。
在组件顶部或 return 前添加全局 style：

```jsx
<style>{`
  @keyframes pulse-ring {
    0%   { opacity: 0.6; r: 18; }
    50%  { opacity: 0.2; r: 22; }
    100% { opacity: 0.6; r: 18; }
  }
  .verified-pulse { animation: pulse-ring 2s ease-in-out infinite; }
`}</style>
```

找到渲染已验证节点外圈的 `<circle>` 元素（有 `opacity: 0.3` 或类似的外环），添加 `className="verified-pulse"`。

**Step 3: 详情面板加关联基地跳转**

在节点详情面板（右侧滑出面板）中，找到渲染 `selected.siteId` 的区域，将现有的「查看点位包影像」按钮文字改为「查看态势分析 →」，跳转路径保持 `/site/${selected.siteId}` 不变（App.jsx 已保留 `/site/:siteId` 路由）。

**Step 4: 左侧链信息区加数据流说明**

在左侧链概览面板底部，加一行提示：

```jsx
<div style={{
  marginTop: '12px', padding: '8px 10px',
  background: '#22c55e08', border: '1px solid #22c55e22',
  borderRadius: '3px', fontSize: '9px', color: '#1e3a5f', lineHeight: 1.6,
}}>
  ✓ 卫星影像锚定节点（绿色）<br/>
  将提升整链置信度加权评分
</div>
```

**Step 5: 验证**

```bash
npm run dev
```

在 /chain 页面，已验证节点（绿色）外圈应有脉冲呼吸动画。详情面板按钮文字更新。

**Step 6: Commit**

```bash
git add src/pages/EventChain/index.jsx
git commit -m "feat(chain): 验证节点脉冲动画 + 态势分析联动跳转 + 数据流说明"
```

---

## Task 7: 智能预测 — 改名 + 信源标签 + 反哺动效

**Files:**
- Modify: `src/pages/Report/index.jsx`

**Step 1: 信源列表加来源类型标签**

在 Report 组件中，找到渲染 `REPORT.sources` 的 map，在每条信源的 `verified` 标签旁边添加 sourceType 标签：

```jsx
{s.verified && (
  <div style={{ display: 'flex', gap: '6px', marginTop: '3px' }}>
    <span style={{ fontSize: '9px', color: '#22c55e' }}>✓ 卫星验证</span>
    <span style={{
      fontSize: '9px', padding: '1px 5px', borderRadius: '2px',
      background: s.sourceType === 'satellite' ? '#22c55e15' : s.sourceType === 'chain' ? '#0ea5e915' : '#64748b15',
      color:      s.sourceType === 'satellite' ? '#22c55e'   : s.sourceType === 'chain' ? '#0ea5e9'   : '#64748b',
      border:     `1px solid ${s.sourceType === 'satellite' ? '#22c55e33' : s.sourceType === 'chain' ? '#0ea5e933' : '#64748b33'}`,
    }}>
      {s.sourceType === 'satellite' ? '卫星影像' : s.sourceType === 'chain' ? '信实链节点' : 'OSINT'}
    </span>
  </div>
)}
```

对 `!s.verified` 的信源也加一个 OSINT 标签：
```jsx
{!s.verified && (
  <div style={{ fontSize: '9px', color: '#334155', marginTop: '3px' }}>
    ◌ 未验证 · OSINT
  </div>
)}
```

**Step 2: 置信度说明小字**

在报告右侧面板顶部（AUTO REPORT 标题下方）加一行：

```jsx
<div style={{ fontSize: '9px', color: '#1e3a5f', marginBottom: '12px', lineHeight: 1.5 }}>
  置信度由信实链加权算法生成<br/>
  <span style={{ color: '#22c55e' }}>✓ 卫星验证</span> 节点权重×1.4
</div>
```

**Step 3: 「验证结果反哺 Agent」按钮动效**

找到该按钮，添加点击状态：

```jsx
const [feedbackActive, setFeedbackActive] = useState(false)

// 按钮：
<button
  onClick={() => { setFeedbackActive(true); setTimeout(() => setFeedbackActive(false), 2000) }}
  style={{
    flex: 1, padding: '10px',
    background: feedbackActive ? '#22c55e33' : '#22c55e22',
    border: `1px solid ${feedbackActive ? '#22c55e' : '#22c55e66'}`,
    color: '#22c55e', borderRadius: '3px', cursor: 'pointer',
    fontSize: '11px', letterSpacing: '0.08em', fontFamily: 'var(--font-mono)',
    transition: 'all 0.3s',
    boxShadow: feedbackActive ? '0 0 20px #22c55e44' : 'none',
  }}
>
  {feedbackActive ? '↻ 已发送至 Agent · 验证中...' : '↻ 验证结果反哺 Agent'}
</button>
```

**Step 4: 验证**

```bash
npm run dev
```

在 /report，信源列表每条信源有来源类型标签。点击反哺按钮出现绿色发光效果并显示「验证中...」2秒后恢复。

**Step 5: Commit**

```bash
git add src/pages/Report/index.jsx
git commit -m "feat(report): 信源类型标签 + 置信度说明 + 反哺按钮动效"
```

---

## Task 8: 最终收尾 + Push

**Step 1: 全流程回归测试**

```bash
npm run dev
```

逐项验证：
- [ ] 首页进入态势分析（地图）
- [ ] 左侧基地列表 + 状态筛选正常
- [ ] 点击基地 → 地图 flyTo + 右侧面板切换
- [ ] 右侧面板：战略价值/ACI/DCI/双线图/雷达图/影像/设施/关联事件
- [ ] 底部时间轴切换影像正常
- [ ] 「→ 链」按钮跳转 /chain
- [ ] /chain 页面：验证节点脉冲动画 + 「查看态势分析」按钮
- [ ] /report 页面：信源标签 + 反哺动效
- [ ] 导航副标题「卫星影像 → 信实验证 → 智能预测」可见

**Step 2: Push**

```bash
git push origin master
```
