# Bloodborne bbport 0.4（简体中文）

本次发布基于 [deadinside28/bloodborne_pc](https://github.com/deadinside28/bloodborne_pc) 的 0.4 版打包，附带已汉化的启动器界面；游戏本体内容、版本号和改动与原版 0.4 完全一致，仅为该版本附加了简体中文翻译。

bbport 是专为《血源诅咒》（Bloodborne，CUSA03173，v1.09）这一款游戏打造的 Wine + DXVK 对应物，运行在 Linux x86-64 平台上。游戏的 x86-64 代码直接在 CPU 上运行，一个为这款游戏专门编写的运行时取代了 PS4 系统库，图形指令被转换为 Vulkan。**不包含任何游戏文件**——你需要自备游戏转储（dump）。面向桌面和 Steam Deck 提供 AppImage：数据保存在 `~/.local/share/bbport`，`--play` 参数可在不打开启动器窗口的情况下直接启动游戏。

关于本项目的问题请前往 Discord：https://discord.gg/KYZRKk9CB——请勿前往 shadPS4 的服务器提问。

## 相较于 0.3 的更新内容

**用一个 DLL、一个按钮搞定 FSR 4.1.1**
- 启动器：*超分辨率 → 用你自己的 AMD DLL 构建 FSR 4.1.1 → 选择 DLL…*，选择版本为 4.1.x 的 `amd_fidelityfx_upscaler_dx12.dll`（OptiScaler 的 `FSR4_LATEST` 文件夹，或来自某个支持 FSR 4.1 的游戏）——只需要这一个文件，不需要加载器 DLL。程序会先检查该 DLL（版本、模型），然后在 Proton 下进行录制并转换；进度会显示在该行以及「日志」标签页中，完成后自动选用 FSR 4.1.1。无窗口命令行方式：`--build-fsr411 <DLL>`。
- AppImage 自带所需工具：除了 Proton 本身，不需要额外安装任何东西。在 NixOS 上，AppImage 现在也能完成构建：录制过程通过用户自己的 systemd 在系统本身上运行，并使用来自 `nix-shell` 的 umu-launcher。会自动选择可用的 Proton 构建版本（GE-Proton 10+、Proton-CachyOS、Proton Experimental；Steam 自带的 Proton 11.0 和 GE-Proton 9 均无法启动 FSR 4.1），运行在它所需的 Steam 运行时环境中，或通过 umu-launcher 运行。
- 无法工作的 DLL 会立刻被拒绝并给出说明：包括 AMD 官方仅限 RDNA4 的 FSR 4.0.x，以及社区的 INT8 构建版本（4.0.2b/c）。
- 修复：从 AppImage 内第二次构建会报 `Permission denied` 失败的问题。
- **RDNA4（RX 9000）：FP8 版本的 FSR 4.1.1。** 在这类显卡上，该 DLL 会运行另一种基于 FP8 矩阵的渲染通道变体。构建过程同样会录制这一变体，游戏会在具备 FP8 矩阵的显卡上自动使用它，其余显卡则使用 INT8。已通过在 RX 7800 XT 上进行的 FP8 模拟验证：与 DLL 逐位一致。**尚未在真正的 RDNA4 硬件上运行过**，欢迎反馈（设置 `BB_FSR411_VARIANT=int8` 可强制回退到 INT8）。

**弱处理器下速度更快**
- `WRITE_DATA` 和 DMA 不再等待所有待处理的主机复制操作，只等待涉及其自身范围的那些（每帧等待次数从 37 次降到 2 次）；画面呈现现在记录在录制命令的线程上。在 4 核 / 8 线程配置下（与 Steam Deck 相同）：站在亚哈古尔（Yahar'gul）时帧率从 158 提升到 167 FPS，在猎人之梦中从 143 提升到 146 FPS。

**新内存与指令转译模型——仍为实验性功能，默认关闭**
- 启动器：*工作模式 → 新内存与指令转译模型*。该模式下，显卡会原地使用游戏的内存，PS4 命令处理器的工作被转译为 Vulkan 命令。仅支持 AMD 显卡（在其他显卡上该开关不可用）。不开启时，游戏使用 0.3 版的内存模型，并包含下方列出的所有修复。

**修复内容**
- 景深效果：使用任意超分辨率方案时，远景模糊（天空、雾气）中出现黑色矩形、条纹以及翻转的场景重影问题（#44）。
- 损坏的着色器缓存（断电、写入时崩溃）不再导致游戏在启动时卡住（#28、#38、#39）；缓存现在以原子方式写入。
- 镜头运动矢量：垂直方向的符号现在从每一帧的 G-buffer 中读取，而不是写死不变——0.2 和 0.3 版本都各自在不同位置出现过错误（#41、#42、#47、#18）。
- 最小化窗口（Win+D）不再导致游戏退出（#31）。
- 不支持 AVX/BMI1/MOVBE/LZCNT/POPCNT/SSE4.2 的处理器（Haswell 之前的 Intel 处理器）现在会收到清晰的提示信息，而不是 `Guest fault (signal 4)`（#26）。
- `BB_READBACKS=2`（精确回读）不再导致启动时卡死。
- 当 Steam 库中没有 GE-Proton 时，FSR 4.1.1 的构建过程不再无声失败（#40）。

**操作与游戏内菜单**
- 手势菜单重新可以通过触摸板打开了（此前完全无法打开）：触摸事件现在按照 DualShock 4 的方式上报（每次触摸分配新 id，并记录按住时长）。左半部分为手势，右半部分为重要物品；Tab / Backspace 以及 Back/Select 的行为保持不变。
- 键盘现在可以和已连接的手柄同时工作（Steam Deck 上手柄始终处于连接状态）：此前只有 Tab 和 Backspace 可用。
- 在游戏内菜单中更改设置，不再清除在启动器中设置的键盘和手柄按键绑定。
- 游戏内菜单（Insert 或 L3+R3）会记住上次关闭时的位置，重启后依然保留，并会显示鼠标光标。
- 游戏内菜单支持英语或俄语（菜单中的 *Language* 选项）；启动器支持巴西葡萄牙语，本次打包额外加入了启动器的简体中文翻译。

## 说明
- 在 Steam Deck 上请使用 **FSR 3** 并将输出分辨率设为 1280×720。
- **不包含** FSR 4.1.1 模型文件（它们来自 AMD 的 DLL）：请使用启动器中的按钮自行构建。
- NVIDIA 显卡：请使用 FSR 3；新内存模型暂不可用。
- 尚未测试：真实 RDNA4 硬件上的 FP8、Steam Deck 以及 NixOS 以外发行版上的 FSR 4.1.1 构建按钮、手柄连接状态下使用键盘游玩。

详细改动记录（俄文）：[docs/CHANGES_0.4.md](https://github.com/deadinside28/bloodborne_pc/blob/master/docs/CHANGES_0.4.md)。

本次简体中文翻译及打包由本仓库维护，与原作者 deadinside28 无关；如有问题，建议同时参考英文/俄文原版说明。
