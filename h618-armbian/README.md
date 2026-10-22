# h618-armbian

全志 H618 机顶盒（D7BE_H618_V1.2，4G / 128GB eMMC）的 Armbian 自建镜像仓库。

本仓库**不含 Armbian 源码**，只放自定义补丁与编译流水线。CI 每次运行时实时拉取官方
`armbian/build` 最新代码，把本仓库的 `userpatches/` 合入后编译。好处：上游更新可直接跟进，
不会因为我们改了源码而产生冲突。

---

## M0 阶段目标

**只验证流水线能跑通、产物格式正确，不烧机。**

用官方 `orangepizero3` 板型（同为 H618）编译，得到可下载的 `.img`。
这一步不动板子，纯管道验证。

---

## 上传步骤（网页拖拽，不需要 token）

1. GitHub 右上角 `+` → **New repository**
2. 仓库名填 `h618-armbian`，选 **Public**（公开仓库跑 Actions 不消耗额度）
3. 不要勾选 Initialize with README
4. 进入空仓库页面，点 **uploading an existing file**
5. 把本地 `h618-armbian` **整个文件夹**拖进浏览器上传区
6. 提交（Commit changes）

### 如果 `.github` 目录没上传成功

有些浏览器会跳过点号开头的目录。检查方式：仓库首页能看到 `.github` 条目。

看不到的话，手工建一次：

1. 仓库页 → **Add file** → **Create new file**
2. 文件名填 `.github/workflows/build.yml`（斜杠会自动建目录）
3. 把本地同路径文件的内容整段粘进去 → Commit

---

## 触发编译

1. 仓库页 → **Actions** 标签（首次会提示启用，点 enable）
2. 左侧选 **build-armbian-h618** → 右侧 **Run workflow**
3. 参数保持默认：
   - board: `orangepizero3`
   - branch: `current`
   - release: `bookworm`
   - minimal: `yes`
4. 点绿色 **Run workflow** 按钮

预计耗时 60～120 分钟。跑完在该次运行页面底部 **Artifacts** 里下载
`h618-...zip`，解压得到 `.img`。

---

## 关键参数说明

| 参数 | 值 | 原因 |
|---|---|---|
| `BOARD` | `orangepizero3` | 官方 H618 基线，U-Boot/DRAM 均为主线上游代码 |
| `BUILD_MINIMAL` | `yes` | 调试期不要桌面环境，省 30 分钟并减小镜像 |
| `KERNEL_CONFIGURE` | `no` | 用 Armbian 默认内核配置，不弹交互菜单 |
| `ARTIFACT_IGNORE_CACHE` | `yes` | 每次重新打包，避免拿到旧产物 |

---

## 已知限制

- **编译时长** 60～120 分钟（runner 4 核）。属正常，不要中途取消。
- **磁盘** 官方 runner 初始可用约 14 GB，不够。流水线第一步会释放约 30 GB，`df -h /` 会打印结果。
- **产物保留 7 天**，过期自动删除，需要就当时下。
- **默认参数编出的镜像不适用于本机顶盒**，仅验证管道。适配见 M1。

---

## 路线

| 阶段 | 内容 | 状态 |
|---|---|---|
| M0 | 流水线打通，拿到官方 Zero3 镜像 | 进行中 |
| M1 | 新增板型 `h618-d7be`：本机 DTS（千兆 PHY / eMMC / 2×USB / 无无线） | 待 M0 |
| M2 | PhoenixSuit `BurnByAddress` 烧写 raw 镜像 | 待 M1 |
| M3 | 首启验证，串口/HDMI 看日志 | 待 M2 |
| M4 | 部署智慧菜园 + 私有云 | 待 M3 |
