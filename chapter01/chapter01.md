# 第一章 操作系统概述
## 1.什么是操作系统

### 1.1 操作系统的定义

<table style="width:100%;border:none;">
  <tr style="border:none;">
    <!-- 左侧文字区域 -->
    <td width="45%" style="border:none;vertical-align:top;">


- 操作系统（OS）是运行计算机硬件的核心程序，同时管理软件并协调用户与硬件之间的交互。它就像计算机的大脑，确保各项功能正常运作，使用户能够通过图形界面或命令与计算机沟通，而无需使用计算机本身的语言。没有操作系统，计算机将无法运行，是最重要的软件之一。  

- 操作系统的核心任务是管理计算机的所有软件和硬件。软件是指指导计算机完成特定任务的程序，如浏览网页或播放视频；硬件则是计算机的物理组成部分，包括CPU、内存、硬盘等。操作系统协调二者运行，确保程序高效使用硬件资源，使计算机顺利完成用户指令。  
    </td>
    <td width="55%" style="border:none;text-align:center;border:none;">
<img src="./picture/os1.png" width="250">
    </td>
  </tr>
</table>

### 1.2 操作系统的功能

<table style="width:100%;border:none;border-collapse:collapse;">
  <tr style="border:none;">
    <td width="48%" style="border:none;vertical-align:top;background:transparent;">

- 命令解释：翻译用户输入的指令；  
- 通信管理：协调软件资源；
- 设备管理：管理各种外部设备；
- 文件管理：管理文件的组织、命名、读取、共享和保护；
- 输入/输出管理：处理用户的输入和计算机的输出；
- 作业计费：记录每项任务或用户所耗费的时间和资源；
    </td>
    <td width="48%" style="border:none;vertical-align:top;background:transparent;margin-left:4%;">
- 内存管理：负责内存分配与释放；
- 网络管理：使不共享硬件或内存的处理器之间能够通信；
- 处理器管理：负责进程的创建、删除、同步与通信；
- 二级存储管理：管理数据在硬盘等长期存储设备中的存放；
- 安全管理：保护计算机中的数据不受恶意软件或非法访问的威胁。
    </td>
  </tr>
</table>




## 2.操作系统的发展历程

操作系统的进化是伴随硬件技术飞跃的，大致经历了六个时代：
1. 手工操作阶段（20世纪40年代末-50年代中期）

    状态：无操作系统。程序员通过插拔导线、拨动开关来运行程序。

    痛点：人机速度矛盾极大，计算机大部分时间在等待人工操作，效率极低。

2. 批处理系统阶段（20世纪50年代末-60年代中期）

    状态：出现单道批处理（一次一个任务）和多道批处理（内存中同时驻留多个程序，CPU在它们之间切换）。

    里程碑：解决了人工慢的问题，实现了作业的自动化“成批”处理。但此时交互性很差，用户无法干预运行中的程序。

3. 分时与实时系统阶段（20世纪60年代中期-70年代）

    分时系统：将CPU时间切成极短的“时间片”轮流转给多个终端用户。代表系统是Multics，它提出了“多用户、多任务”的现代操作系统雏形。

    实时系统：用于火箭、工业控制，要求系统在绝对规定时间内响应外部事件。

    重大事件：1969年，Unix操作系统在贝尔实验室诞生，它奠定了现代操作系统在安全、多任务、网络等方面的基石。

4. 个人电脑时代（20世纪80年代-90年代）

    状态：硬件成本降低，微处理器普及。

    标志性事件：

      - DOS（磁盘操作系统）成为早期IBM PC的标准。

      - Mac OS（1984年）率先推出成熟的图形用户界面（GUI），让电脑走向大众。

      - Windows 3.0/95（1990年代）凭借图形界面和兼容性，统治了个人桌面市场。

      - Linux内核（1991年，林纳斯·托瓦兹发布）诞生，作为开源的类Unix系统，迅速吸引全球开发者，成为日后国产系统的基石。

5. 网络化与移动时代（21世纪初-2010年代）

    状态：互联网和智能手机爆发。

    标志：

      - 服务器端Linux（如Red Hat、CentOS）成为Web服务器绝对主流。

      - 移动端iOS（2007年）和Android（2008年，基于Linux内核）重新定义了操作系统形态，使移动应用生态空前繁荣。

6. 万物互联与国产化崛起（2020年代至今）

    状态：云计算、AI大模型、物联网（IoT）和自主可控成为核心诉求。

    趋势：

      - 操作系统走向“分布式”和“轻量化”（如华为鸿蒙HarmonyOS）。

      - 国产操作系统迎来黄金期，正是在这个阶段，你之前提到的欧拉（服务器）、统信UOS/银河麒麟（桌面） 开始大规模替代CentOS和Windows，并深度适配国产芯片（龙芯、飞腾、鲲鹏）。


## 3.操作系统的类别
<!--常见系统介绍-->
日常最常接触到的操作系统：Microsoft Windows、macOS 和 Linux。

<div style="text-align:center;margin-bottom:50px;">
<img src="./picture/macos-linux-windows.png" width="500" style="border-radius:12px;box-shadow:0 3px 12px #cccccc;">
</div>

<!-- windows介绍 -->
<table style="width:100%;border:none;margin-bottom:40px;">
  <tr>
    <!-- 左侧文字 -->
    <td width="45%" style="border:none;vertical-align:top;">

- Microsoft Windows：诞生于 1980 年代中期，是最常见的操作系统，有多个版本可供使用。最近的版本是 2021 年发布的 Windows11。几乎所有的个人计算机（PC）都预装了 Windows，当然也有例外，最著名的就是苹果产品。Windows的功能相当多样化，并不断通过更新来提升安全性。
    </td>
    <!-- 右侧图片 -->
    <td width="55%" style="border:none;text-align:center;">
<img src="./picture/win11.jpg" style="width:400px;height:200px;object-fit:cover;">
    </td>
  </tr>
</table>

<!-- macos介绍 -->
<table style="width:100%;border:none;margin-bottom:40px;">
  <tr>
    <!-- 左侧文字区域 -->
    <td width="45%" style="border:none;vertical-align:top;">

- Mac OS：这是安装在苹果电脑上的操作系统，全球使用率不到10%。它的界面看起来更为简洁时尚，但通常只能使用专有软件和硬件。总体而言，在这三种系统中，要充分发挥 macOS 的功能通常成本最高。macOS 的使用和自定义性相对受限，许多常见的程序、游戏和软件都不兼容该系统。不过，由于 macOS 被攻击的频率较低，因此安全性可能更好。
    </td>
    <td width="55%" style="border:none;text-align:center;">
<img src="./picture/macos.png" style="width:400px;height:200px;object-fit:cover;">
    </td>
  </tr>
</table>

<!--Linux介绍-->
<table style="width:100%;border:none;">
  <tr>
    <!-- 左侧文字区域 -->
    <td width="45%" style="border:none;vertical-align:top;">

- Linux：与前两者不同，Linux 是开源的，任何人都可以对其进行修改并免费分发。这使得 Linux 在三者中限制最少，然而它在全球系统中的使用率不足 2% 。尽管如此，由于其高度的灵活性和可定制性，大多数服务器仍选择使用 Linux。
    </td>
    <td width="55%" style="border:none;text-align:center;">
<img src="./picture/linux.jpg" style="width:400px;height:200px;object-fit:cover;">
    </td>
  </tr>
</table>


## 4.总结

如果你把计算机比作一个现代化大工厂。在其中，操作系统拥有整个计算机系统的最高管理权限，扮演着三个核心角色：

  - 对硬件（向下管理）：是“资源大总管”。负责把有限的CPU时间、内存空间、磁盘容量，公平且高效地分配给多个竞争的程序。

  - 对软件（向上服务）：是“标准接口提供者”。为开发者屏蔽了不同品牌、不同型号硬件的底层差异（比如你是Intel还是AMD芯片），让软件只需调用系统API即可运行，无需关心底层电路。

  - 对用户（人机交互）：是“翻译官”。把晦涩的二进制指令翻译成你熟悉的桌面图标、文件夹和鼠标点击。