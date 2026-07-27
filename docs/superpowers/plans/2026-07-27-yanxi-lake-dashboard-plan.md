# 严西湖水质管理仪表盘：轮播图接入与 veFaaS 部署实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 将 `轮播图/` 下全部图片按文件名顺序接入仪表盘轮播，并通过火山引擎 veFaaS 部署为可公网访问的静态站点。

**Architecture:** 单文件 HTML 大屏仪表盘，轮播图改为从 `CAROUSEL_FILES` 数组读取 `轮播图/<filename>`；使用 `@volcengine/vefaas-cli` 创建新应用并绑定新网关，上传根目录作为静态站点。

**Tech Stack:** HTML/CSS/JS (ECharts), PowerShell, @volcengine/vefaas-cli

---

### 文件结构

| 文件 | 责任 |
|------|------|
| `index.v4.0.html` | 现有仪表盘源码，将被修改并复制为 `index.html` |
| `index.html` | 静态站点入口，包含轮播图清单 |
| `轮播图/` | 53 张原始轮播图片 |
| `.vefaas/config.json` | veFaaS CLI 生成的应用绑定配置 |

---

### Task 1: 安装并检查 veFaaS CLI

**Files:** 无

- [ ] **Step 1: 全局安装最新版 veFaaS CLI**

```powershell
npm i -g @volcengine/vefaas-cli@latest
```

- [ ] **Step 2: 确认 CLI 版本 ≥ 0.2.7**

```powershell
vefaas --version
```

**Expected:** 输出版本号，例如 `0.2.7` 或更高。

- [ ] **Step 3: 检查当前登录状态**

```powershell
vefaas login --check
```

**Expected:** 显示已登录账号；若未登录，使用已配置的 AK/SK 登录：

```powershell
vefaas login --accessKey $env:VOLC_ACCESS_KEY_ID --secretKey $env:VOLC_SECRET_ACCESS_KEY
```

---

### Task 2: 生成轮播图文件名清单

**Files:**
- 修改：`index.v4.0.html`

- [ ] **Step 1: 在 PowerShell 中生成排序后的文件名数组**

```powershell
$files = Get-ChildItem -Path "e:\TraeCode\严西湖水质管理仪表盘\轮播图" -File |
  Where-Object { $_.Extension -imatch '\.(jpg|jpeg|png|gif|webp)$' } |
  Sort-Object Name |
  ForEach-Object { '"' + $_.Name + '"' }
"const CAROUSEL_FILES = [" + ($files -join ",`n  ") + "];"
```

**Expected:** 输出形如 `const CAROUSEL_FILES = ["15.jpg", "16.jpg", ...];`，共 53 项。

- [ ] **Step 2: 将清单插入 `index.v4.0.html`**

在 `<script>` 区内、轮播初始化代码之前（约第 709 行）添加：

```javascript
const CAROUSEL_FILES = [
  "“科普区”展示获领导大力表扬.png",
  "15.jpg",
  "16.jpg",
  "202202生态研学社团巡湖1.png",
  "20230401红领巾志愿：王子涵、霍前隆1.jpg",
  "20250417美化严西湖.jpg",
  "20250529湖北省科技馆生态实践活动1.jpg",
  "8.jpg",
  "出发.jpg",
  "出发3.jpg",
  "合影1.jpg",
  "合影2.jpg",
  "合影3.jpg",
  "合影4.jpg",
  "合影5.jpg",
  "合影7.jpg",
  "合照2.jpg",
  "捡起湖边垃圾.jpg",
  "讲解安全事项.jpg",
  "教师授课.jpg",
  "取水回实验室.jpg",
  "取严西湖水样2.jpg",
  "荣誉合影2.png",
  "荣誉合影4.JPG",
  "森林.jpg",
  "生态区 (5).jpg",
  "生态宣讲2.jpg",
  "生态宣讲4.jpg",
  "水质检测.jpg",
  "水质检测实验1.jpg",
  "水质检测实验4.jpg",
  "水质检测实验5.jpg",
  "外场展示.jpg",
  "武汉电视台：学生采访1.jpg",
  "现场颁奖.jpg",
  "向长江日报记者介绍我校巡湖事迹.JPG",
  "宣讲5.jpg",
  "悬挂环保标语.jpg",
  "悬挂环保标语(1).jpg",
  "学生1.jpg",
  "学生3.jpg",
  "学生完成河湖长知识拼图.jpg",
  "展台2.jpg",
  "IMG_20241024_155745.jpg",
  "IMG_20241210_095723.jpg",
  "IMG_20250415_165444.jpg",
  "IMG_20250415_170603.jpg",
  "IMG_20250415_171053.jpg",
  "IMG_20250420_155757.jpg",
  "IMG_20250420_155907.jpg",
  "IMG_20250430_154436.jpg",
  "IMG_20250430_154648.jpg",
  "IMG_20250430_162308.jpg"
];
```

- [ ] **Step 3: 替换轮播初始化逻辑**

将原代码块：

```javascript
(function(){
  const TOTAL=52;
  const track=document.getElementById('caroTrack');
  const dots=document.getElementById('caroDots');
  const tag=document.getElementById('caroTag');
  const box=document.getElementById('caro');
  let cur=0;
  // 构建图片
  for(let i=1;i<=TOTAL;i++){
    const img=document.createElement('img');
    img.src='assets/photos/patrol_'+String(i).padStart(2,'0')+'.jpg';
    img.loading='lazy';
    img.alt='';
    track.appendChild(img);
  }
  // 构建圆点
  for(let i=0;i<TOTAL;i++){
    const d=document.createElement('i');
    d.onclick=()=>{go(i);reset();};
    dots.appendChild(d);
  }
  const dotEls=[...dots.children];
  function render(){
    track.style.transform='translateX(-'+(cur*100)+'%)';
    dotEls.forEach((d,i)=>d.classList.toggle('on',i===cur));
    tag.textContent=String(cur+1).padStart(2,'0')+' / '+TOTAL;
  }
  function go(i){cur=(i+TOTAL)%TOTAL;render();}
  let timer=null;
  function play(){timer=setInterval(()=>go(cur+1),3500);}
  function reset(){clearInterval(timer);play();}
  document.getElementById('caroPrev').onclick=()=>{go(cur-1);reset();};
  document.getElementById('caroNext').onclick=()=>{go(cur+1);reset();};
  box.addEventListener('mouseenter',()=>clearInterval(timer));
  box.addEventListener('mouseleave',play);
  render();play();
})();
```

替换为：

```javascript
(function(){
  const files=CAROUSEL_FILES;
  const TOTAL=files.length;
  const track=document.getElementById('caroTrack');
  const dots=document.getElementById('caroDots');
  const tag=document.getElementById('caroTag');
  const box=document.getElementById('caro');
  let cur=0;
  files.forEach(name=>{
    const img=document.createElement('img');
    img.src='轮播图/'+encodeURIComponent(name);
    img.loading='lazy';
    img.alt='';
    track.appendChild(img);
  });
  for(let i=0;i<TOTAL;i++){
    const d=document.createElement('i');
    d.onclick=()=>{go(i);reset();};
    dots.appendChild(d);
  }
  const dotEls=[...dots.children];
  function render(){
    track.style.transform='translateX(-'+(cur*100)+'%)';
    dotEls.forEach((d,i)=>d.classList.toggle('on',i===cur));
    tag.textContent=String(cur+1).padStart(2,'0')+' / '+TOTAL;
  }
  function go(i){cur=(i+TOTAL)%TOTAL;render();}
  let timer=null;
  function play(){timer=setInterval(()=>go(cur+1),3500);}
  function reset(){clearInterval(timer);play();}
  document.getElementById('caroPrev').onclick=()=>{go(cur-1);reset();};
  document.getElementById('caroNext').onclick=()=>{go(cur+1);reset();};
  box.addEventListener('mouseenter',()=>clearInterval(timer));
  box.addEventListener('mouseleave',play);
  render();play();
})();
```

- [ ] **Step 4: 本地打开验证轮播**

```powershell
Start-Process "e:\TraeCode\严西湖水质管理仪表盘\index.v4.0.html"
```

**Expected:** 浏览器打开后，轮播图区域显示图片并能自动切换。

---

### Task 3: 准备静态站点入口

**Files:**
- 创建：`index.html`

- [ ] **Step 1: 复制 `index.v4.0.html` 为 `index.html`**

```powershell
Copy-Item "e:\TraeCode\严西湖水质管理仪表盘\index.v4.0.html" "e:\TraeCode\严西湖水质管理仪表盘\index.html" -Force
```

**Expected:** 项目根目录出现 `index.html`，内容与修改后的 `index.v4.0.html` 一致。

---

### Task 4: 创建并部署 veFaaS 应用

**Files:**
- 生成：`.vefaas/config.json`

- [ ] **Step 1: 进入项目目录**

```powershell
Set-Location "e:\TraeCode\严西湖水质管理仪表盘"
```

- [ ] **Step 2: 检测项目框架与配置**

```powershell
vefaas inspect
```

**Expected:** CLI 识别为静态站点（Static Site），输出 build command / output path / start command / port 等建议。

- [ ] **Step 3: 获取可用网关**

```powershell
vefaas gateway list --first
```

**Expected:** 返回一个 gateway name；若为空，停止并提示用户先到火山引擎控制台创建 APIG 网关。

- [ ] **Step 4: 创建并部署新应用**

```powershell
vefaas deploy --newApp --name yanxi-lake-dashboard --gateway <gateway-name> --yes
```

将 `<gateway-name>` 替换为 Step 3 返回的名称。

**Expected:** CLI 完成打包、上传、创建函数/应用/触发器，最终显示部署成功。

- [ ] **Step 5: 查看访问域名**

```powershell
vefaas domains
```

**Expected:** 返回一个或多个公网 URL，例如 `https://yanxi-lake-dashboard-xxx.apigw.volcengineapi.com`。

---

### Task 5: 线上验证

**Files:** 无

- [ ] **Step 1: 浏览器访问返回的 URL**

```powershell
Start-Process "<deployed-url>"
```

- [ ] **Step 2: 人工确认以下检查项**

- [ ] 页面标题显示“严西湖水智眸智慧监测平台”。
- [ ] 轮播图自动播放，左右箭头与底部圆点可切换。
- [ ] 趋势图、热力图、雷达图、WQI 仪表盘正常渲染。
- [ ] 浏览器 DevTools Network 中无 404（尤其 `轮播图/` 下图片）。
- [ ] 页面底部页脚正常显示。

---

### 回滚方案

- 若部署后页面异常，可执行 `vefaas app release --id <app-id>` 查看历史版本并按 veFaaS 流程回滚。
- 本地保留 `index.v4.0.html` 作为源码副本，`index.html` 为部署入口。
