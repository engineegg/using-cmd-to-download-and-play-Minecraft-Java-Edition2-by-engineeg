# Minecraft Java Edition 一键下载启动脚本

> 一个**单文件、零依赖安装、纯 Windows 命令行**的 Minecraft Java 版下载与启动脚本。
> 双击运行 → 自动装 Java（OpenJDK 27）→ 走国内镜像下载游戏 → 离线账号启动。不需要任何第三方启动器。

`Windows 10 1803+` · `纯 cmd` · `国内镜像优先` · `断点续传/秒级重开` · `离线 / 正版登录` · `可选 Fabric`

---

## 这是什么

一个 `.txt` 文本文件，里面是完整的 Windows 批处理命令（cmd 脚本）。把后缀改成 `.bat` 双击，它就会：

1. 检查本机有没有 Java，没有就自动装 OpenJDK 27；
2. 优先从国内高速镜像下载游戏文件，镜像不可用时自动回退 Mojang 官方源；
3. 下载完直接用离线账号启动游戏；
4. 下次再运行同一个版本时跳过下载，几秒内进游戏。

**它默认是什么**：纯原版。想要 mod 只需把 `MOD_LOADER` 改成 `fabric`，脚本会自动装好 Fabric 加载器并建好 `MC\game\mods`，把 mod 的 jar 丢进去就能玩（Forge / NeoForge 不支持）。
离线账号（默认）进不了正版验证服务器；要用正版身份玩、要显示自己的皮肤，把 `LOGIN` 改成 `msa`，用你自己的微软账号登录即可。

---

## 特性

| 特性 | 说明 |
|---|---|
| 自动装 Java | 用 Adoptium Temurin 的**便携版** OpenJDK 27，解压到 `MC\java`，**不改系统环境变量、不写注册表、不需要管理员权限** |
| 国内镜像优先 | 游戏资源走 **BMCLAPI**，JDK 走 **清华镜像 → 华为云 → GitHub 加速站**，全部失败才回退官方源 |
| 文件完整性校验 | 每个文件按 JSON 里的**精确字节数**校验；JDK 压缩包额外用 `tar -tf` 验证完整性，截断/HTML 错误页会被自动丢弃并换源 |
| 断点续传式重开 | 单个文件已有的跳过，缺的补下；版本级标记 `versions\<版本>\.ok` 保证已装好的版本直接启动 |
| 资源全版本共享 | `libraries` / `assets` 只下一份，切换游戏版本时只补该版本独有的文件 |
| 离线账号 | 传 `--username` + 由 `OfflinePlayer:<用户名>` 计算出的离线 UUID，`--accessToken 0`，完全离线启动 |
| 正版登录 | `LOGIN=msa` 时走微软官方 device code 流程（浏览器输 8 位代码），令牌缓存在 `MC\tmp`，过期自动用刷新令牌续期，不用反复登录 |
| 可选 Fabric | `MOD_LOADER=fabric` 时从 fabric-meta 取加载器 profile（BMCLAPI 镜像），自动补下载加载器库、切换主类为 `KnotClient`，并建好 `MC\game\mods` |
| 版本隔离 | 原版和 Fabric 各有独立完成标记（`.ok` / `.ok-fabric-<版本>`），来回切换只补差量，不会重复下几百 MB |
| 支持任意版本 | 版本号填 `1.21.4` / `1.20.1` / `26.3` …，留空自动用最新正式版；自带 923 个版本可查 |
| 纯绿色 | 所有文件都在脚本所在目录，删掉文件夹即彻底卸载 |

---

## 环境要求

| 项目 | 要求 |
|---|---|
| 系统 | Windows 10 1803 及以上 / Windows 11（需要系统自带 `curl.exe`、`tar.exe`） |
| 组件 | `curl.exe`、`tar.exe`、`java.exe`、`powershell.exe`（**全部是系统自带**，无需下载安装） |
| 网络 | 能访问 BMCLAPI 或 Mojang 服务器 |
| 磁盘 | 一个游戏版本约 **600 MB ~ 900 MB**（每个额外版本再加约 50~150 MB） |

> `powershell` 只用于解析 Mojang 的 JSON 文件；下载、解压、启动全部由 `curl` / `tar` / `java` 完成。

---

## 快速开始

1. 把 `Minecraft一键下载启动.txt` **复制**一份，重命名为 `Minecraft.bat`；
2. 双击 `Minecraft.bat`（想改用户名/版本先用记事本编辑配置区，见下文）；
3. 等它下载完，游戏窗口自动弹出。

也可以在 cmd 里一步完成复制并运行：

```bat
copy "Minecraft一键下载启动.txt" Minecraft.bat && Minecraft.bat
```

游戏会安装到 **脚本所在目录下的 `MC\` 文件夹**。例如脚本放在 `D:\脚本\`，游戏就在 `D:\脚本\MC\`。

---

## 配置项

用记事本打开脚本，改**文件顶部配置区**（当前在第 33 行附近）等号右边的值。注意：**不要加空格、不要再套引号**。

```bat
REM 游戏版本：留空 = 最新正式版；可填 1.21.4 / 1.20.1 / 26.3 …
set "MC_VERSION="
REM 离线用户名：3-16 位字母、数字或下划线，不要中文和空格
set "PLAYER_NAME=Steve"
REM 最大内存：2G / 6G / 8G
set "MEMORY=4G"
REM 1 = 国内镜像优先；0 = 只用官方源
set "USE_MIRROR=1"
REM 没装 Java 时要安装的 OpenJDK 主版本
set "JAVA_MAJOR=27"
REM 登录方式：offline = 离线账号（默认）；msa = 正版微软账号
set "LOGIN=offline"
REM 模组加载器：留空 = 纯原版；填 fabric = 安装 Fabric 加载器
set "MOD_LOADER="
REM Fabric 加载器版本：留空 = 自动选该游戏版本的最新稳定版
set "FABRIC_LOADER="
```

改了 txt 之后，记得**重新复制覆盖**成 `Minecraft.bat`（或者直接改那份 bat）。

### 查有哪些版本号

在 cmd 里粘这一行（走国内镜像，列出最新版和前 30 个版本）：

```bat
curl -s -L "https://bmclapi2.bangbang93.com/mc/game/version_manifest_v2.json" -o "%TEMP%\v.json" && powershell -NoProfile -Command "$m=Get-Content -Raw $env:TEMP\v.json | ConvertFrom-Json; $m.latest; $m.versions | Select-Object -First 30 id,type | Format-Table"
```

---

## 运行流程

```
[1/7] 检查 Java 运行环境     → 内置 java ？系统 java ？都没有 → 装 OpenJDK 27
[2/7] 获取版本清单           → BMCLAPI 拿 version_manifest_v2.json，确定目标版本
[3/7] 解析版本文件           → 算出 Windows 平台需要的库、classpath、所需 Java 版本
[3.5/7] 准备 Fabric（可选）  → 取 loader profile、合并 classpath、建 mods 目录
[4/7] 检查 Java 版本         → 版本要求高于当前 Java 时补装 OpenJDK 27
[4.5/7] 登录账号（仅 msa）   → 微软 device code 登录，换 XSTS 令牌，再换 Minecraft 令牌
[5/7] 下载核心文件           → 依赖库 + 客户端 jar + 资源索引 + 日志配置
[6/7] 下载资源文件           → 数千个贴图/音效/语言文件，curl 并行 24 线程
[7/7] 生成启动参数并启动     → 写入 tmp\args.txt，用 java @argfile 启动（避开 cmd 8191 字符上限）
```

任何一步失败都会打印 `[错误]` 原因并停下（不会卡住）。下载阶段失败的文件会打印 `[换源]` 并自动换下一个地址。

---

## 下载源与回退顺序

| 内容 | 国内镜像（优先） | 官方源（回退） |
|---|---|---|
| 版本清单 | `bmclapi2.bangbang93.com` | `piston-meta.mojang.com` |
| 版本 JSON | `bmclapi2.bangbang93.com/version/<版本>/json` | `piston-meta.mojang.com` |
| 依赖库 | `bmclapi2.bangbang93.com/maven/` | `libraries.minecraft.net` |
| 客户端 jar | `bmclapi2.bangbang93.com/version/<版本>/client` | `piston-data.mojang.com` |
| 资源索引 / 资源对象 | `bmclapi2.bangbang93.com/assets/` | `resources.download.minecraft.net` |
| Fabric 加载器配置 | `bmclapi2.bangbang93.com/fabric-meta/` | `meta.fabricmc.net` |
| Fabric 加载器库 | `bmclapi2.bangbang93.com/maven/` | `maven.fabricmc.net` |
| OpenJDK 27 | 清华 Adoptium → 华为云 → `ghfast.top` → `gh-proxy.com` → `ghproxy.net` | `api.adoptium.net` |

`USE_MIRROR=0` 时跳过所有国内镜像，只用官方源。

---

## 目录结构

```
脚本所在目录\
├─ Minecraft一键下载启动.txt   ← 本脚本（复制改名为 .bat 运行）
├─ README.md
└─ MC\                         ← 运行后自动生成
   ├─ java\                    OpenJDK 27 便携版（游戏用这个启动）
   ├─ versions\<版本>\         版本 json、客户端 jar、.ok 完成标记
   ├─ libraries\               依赖库（全版本共享）
   ├─ assets\                  资源文件（objects 按哈希散列存放）
   ├─ natives\                 原生库临时目录（java / jna / lwjgl / netty）
   ├─ game\                    游戏数据：saves 存档、options.txt、resourcepacks
   │  └─ mods\                 Fabric 模组放这里（MOD_LOADER=fabric 时自动创建）
   ├─ logs\latest.log          游戏日志（启动失败先看这里）
   └─ tmp\                     下载清单、启动参数 args.txt 等临时文件
```

> 存档在 `MC\game\saves`，想备份整个游戏直接拷 `MC` 文件夹即可。

---

## 常见问题（FAQ）

**Q：双击了没反应 / 打开了记事本？**
双击 `.txt` 只会打开记事本。必须**重命名成 `.bat`** 再双击。Win11 改扩展名可能要先在资源管理器里打开「文件扩展名」显示。

**Q：为什么给的是 `.txt` 而不是现成的 `.bat`？**
按需求要求交付文本形式的命令（不直接下发可执行文件），所以首次使用时自己复制改名成 `.bat` 即可，内容与 bat 完全一致，没有任何功能差别。

**Q：提示「没有找到 curl.exe」？**
系统太老（低于 Win10 1803，老版本 Windows 不自带 curl）。升级系统，或自己下一个 `curl.exe` 放到系统 PATH 覆盖的目录里。

**Q：提示「版本 xxx 不存在或下载失败」？**
版本号写错了（要和官方 id 完全一致，快照要写全名），或者网络不通。用上面那行命令查正确 id。

**Q：黑框里全是 `<log4j:Event ...>` 这种 XML？**
正常。Minecraft 新版自带的日志配置就是用 XML 格式往控制台输出的，游戏本身不受影响。想看人话版日志就看 `MC\logs\latest.log`。

**Q：下载到一半关了窗口，要重下吗？**
不用。重新运行会按文件大小校验，已完整的跳过，只补缺的。

**Q：想重新下载某个版本？**
删掉 `MC\versions\<版本号>\.ok`（或整个版本文件夹）再运行。

**Q：怎么用正版账号登录？**
把配置区的 `set "LOGIN=offline"` 改成 `set "LOGIN=msa"`，重新运行。窗口里会出现 `https://www.microsoft.com/link` 和一个 8 位代码，用浏览器打开、输入代码、登录微软账号并授权，回到窗口等几秒即可。前提是这个微软账号**已经购买 Minecraft: Java Edition**。登录成功后令牌会缓存，之后每次启动自动续期，约 24 小时有效期。

**Q：正版登录安全吗？令牌会不会外泄？**
不会上传到任何地方，只写在本机的 `MC\tmp\msa_profile.txt`（访问令牌）和 `MC\tmp\msa_refresh.txt`（刷新令牌）。**这两个文件等于你账号的登录凭证，不要发给别人、不要传到网盘**。想彻底退出登录，删掉这两个文件即可。

**Q：正版登录报错怎么办？**
- 提示「没有购买 Minecraft Java 版」→ 这个微软账号确实没有 Java 版所有权；
- 提示「还没有 Xbox 档案，请先登录 xbox.com」→ 新账号需要先去 xbox.com 登录一次生成档案；
- 提示「未成年账号需要家长批准」→ 在微软家庭组里让家长批准；
- 提示「登录代码已过期」→ 900 秒没输完，重新运行本脚本即可。

**Q：能不能进正版服务器？**
用 `LOGIN=msa` 登录了自己的正版账号后可以进正版验证服务器（如 Hypixel）；用默认的离线账号只能进离线服务器和自建内网服务器。

**Q：怎么装 mod？**
把配置区 `set "MOD_LOADER="` 改成 `set "MOD_LOADER=fabric"`，重新运行。脚本会装好 Fabric 加载器并创建 `MC\game\mods`，之后把 mod 的 `.jar` 直接丢进这个目录（不用解压），启动游戏即可。三个注意点：mod 必须和**游戏版本 + Fabric 版本**匹配；Fabric 版 mod 不能和 Forge 版混用；光影/整合包一般是「Fabric + Sodium/Iris」这类组合，也是把 jar 一起丢进 mods。

**Q：为什么装完 mod 进游戏看不到 mod 的按键/物品？**
先在标题画面看左下角有没有 `Fabric (Modded)` 标记，再看 `MC\logs\latest.log` 里有没有 mod 加载失败的报错——最常见原因是 mod 版本和游戏版本不匹配（比如给 1.20 的 mod 丢进 1.21.4）。

**Q：支持 Forge / NeoForge 吗？**
不支持。它们需要运行官方 installer 改写客户端 jar、生成新版本 profile，塞进纯 cmd 脚本里很脆弱。要玩 Forge 生态请用 HMCL / PCL2 / Prism。

**Q：想要老版本（1.12.2、1.16.5）？**
这两个版本**必须要 Java 8**，而脚本只会装/使用 OpenJDK 27（或系统已有 Java）。可以填版本号下载，但启动大概率崩溃，需要自己装 JDK 8。下表供参考：

| 游戏版本 | 需要的 Java |
|---|---|
| 1.12.2 及更早、1.16.5 | Java 8 |
| 1.17.x | Java 16+ |
| 1.18 – 1.20.4 | Java 17+ |
| 1.20.5 – 1.21.x | Java 21+ |
| 26.x（当前最新） | Java 25+ |

**Q：卡在某个文件反复「[换源]」？**
镜像临时抽风，脚本会自动换下一个源，最终会回退到官方源。网络太差就挂个代理重试。

**Q：会不会污染系统 / 装了 Java 会影响别的软件？**
不会。JDK 解压在 `MC\java`，不改 `PATH`、不写注册表、不碰系统自带的 Java，也不使用 `%APPDATA%\.minecraft`。

**Q：怎么彻底卸载？**
删掉脚本旁边的 `MC` 文件夹。

**Q：路径里有中文可以吗？**
可以（已实测中文路径）。但**路径里不要有单引号 `'`**，脚本会直接提示并退出。

---

## 已知限制

- 只支持 **Windows**（依赖 cmd / curl / tar / Java 的 Windows 路径）。
- mod 只支持 **Fabric**（脚本自动安装）；**Forge / NeoForge / OptiFine** 需要官方 installer，不在本脚本范围内。
- 离线模式（默认）没有皮肤、没有 Realms、进不了正版服务器；正版模式（`LOGIN=msa`）则和官方启动器一样正常。
- 正版登录依赖微软的 device code 流程，若微软调整策略可能失效，届时用官方启动器或 HMCL 登录更稳妥。
- 老版本（≤1.16.5）需要 Java 8，脚本当前不会自动安装 Java 8。
- 版本建议 **1.13 及以上**：1.12 及更早用的是旧版启动参数格式（`minecraftArguments`），脚本只做了兼容处理，未实测。
- 游戏控制台日志是 XML 格式（Minecraft 官方行为），不是脚本问题。

---

## 实测记录

在 Windows 10 22H2（19045）、系统已装 Java 21 的机器上完整跑通：

- 自动识别到版本 26.3 需要 Java 25+，从 GitHub 加速站下载 **OpenJDK 27.0.1**（142 MB）并解压到 `MC\java`；
- 下载核心文件 77 个 + 资源文件 **5149 个**（约 487 MB），BMCLAPI 有 30 个资源连接被重置，脚本自动回退官方源补齐；
- 启动日志出现 `Setting user: Steve`、`Backend library: LWJGL version 3.4.3+4`、`Sound engine started`，正常进入主菜单；
- 第二次运行打印 `[跳过] 版本 26.3 之前已经下载完成，本次直接启动`，约 2 秒进游戏；
- 故意填错版本号 `9.9.9` 时，正确提示「版本 9.9.9 不存在或下载失败」并退出（exit code 1）。
- 正版登录（`LOGIN=msa`）已验证到「弹出 device code 提示 + 轮询等待」这一步；真正的账号授权、XSTS / Minecraft 令牌交换需要真实微软账号，未在本机实测。
- Fabric 模式完整跑通：1.21.4 + Fabric Loader 0.20.1，自动从 BMCLAPI 取 profile、补下 8 个加载器库、切换主类为 `KnotClient`；把 Fabric API 丢进 `MC\game\mods` 后启动日志显示 `Loading 52 mods`，正常进主菜单（离线账号 Steve、LWJGL 3.3.3、Sound engine started）。
- 期间修掉一个真实的坑：原版 1.21.4 自带 `asm 9.6`，Fabric 要 `asm 9.10.1`，两者同时在 classpath 上会让 Fabric 直接拒绝启动（`duplicate ASM classes found on classpath`）；现在按 Maven 坐标自动去重，保留 Fabric 的版本。

---

## 免责声明

- 本脚本仅用于个人学习、离线试玩与教学演示。**Minecraft 是 Mojang / Microsoft 的商业软件，请购买正版**。
- 脚本只从 Mojang 官方源、BMCLAPI 镜像、Adoptium 官方源下载公开文件，不包含、不打包、不修改任何游戏文件；不执行任何第三方安装程序。
- 离线账号不提供任何绕过正版验证的能力（客户端本身仍会以离线模式运行），请勿用于侵权用途。
- `LOGIN=msa` 使用的是**你自己的**微软账号，登录凭证只保存在本机 `MC\tmp`，脚本不会上传或外传任何账号信息。

---

## 文件说明

| 文件 | 作用 |
|---|---|
| [`Minecraft一键下载启动.txt`](Minecraft一键下载启动.txt) | 主脚本，复制改名为 `.bat` 后运行 |
| [`要求.txt`](要求.txt) | 最初的需求说明（脚本的功能来源） |
| [`README.md`](README.md) | 本文件 |
