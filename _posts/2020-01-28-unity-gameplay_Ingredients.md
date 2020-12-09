---
layout: post
title: "Unity Gameplay Ingredients框架"
featured-img: shane-rounce-205187
categories: [unity,框架]
---
## <center>Gameplay Ingredients</center>

Gameplay Ingredients是针对unity的一套运行时和编辑器工具，一系列的脚本让你轻松的创建游戏或者构建游戏原型。

</br>

#### Runtime Tools

Events、Logic and Action:根据逻辑和事件执行操作的原子组件。

Callables: Gameplay Ingredients框架原理的编程核心。

Managers:处理低级游戏逻辑的单例。

Rigs:动画、绑定和连接对象在一起的组件。

State Machines:执行简单状态行为的抽象。

Factories:基于蓝图创建对象并且管理其生命周期的组件。

Timers:跟踪时间的基于时间的组件。

Global Variables:一个可以读、写和获得逻辑的值的黑板。

interactive:角色互动系统。

Gameplay Ingredients Settings:项目的配置资产。

</br>

#### Editor Tools

Welcome Screen:为您的项目提供提示以及初始化设置的窗口。

Play From Here：从当前场景视图位置播放。

Advanced Hierarchy View:在 Hierarchy Window显示更多的信息。

Link Game View:在你的关卡中导航视图预先录制的点。

Scence Setups:在编辑器中一次性打开多个场景。

New Scence From Template:根据一个已经存在的Scence场景创建新的Scence场景。

Find and Replace：查找游戏物体并且使用预制件代替查找的游戏物体的一个工具。

Callable Tree Explorer:记录你所有的Events、Logic、Actions。

Folders:文件夹的层次结构。

Discover:使用浏览器导航并且记录项目。

Other Tools:其他的一些工具。

</br>

#### Requirements

- Uinty 2019.4/2019.3/2019.2/2019.1/2018.3
- PackageManager UI

</br>

### How to install

1. #### Manual Version

   - 克隆仓库都任何你喜欢的地方
   - 在你的项目中，打开Window/Package Manager窗口，使用 + 按钮选择 Add Package from disk... 选项
   - 定位到你的仓库的文件夹，并且选择package.json文件
   - 这样，仓库就会被添加到你的项目中

2. #### OpenUPM

   - 该包在openUPM注册表中可用，建议通过openupm_cli来安装openUPM

   - ```shell
     openupm add net.peeweek.gameplay-ingredients
     ```

   - 在unity关闭的情况下，通过文本编辑器编辑Packages/manifest.json

   - 在denpendencies下添加

     ```
     "net.peeweek.gameplay-ingredients": "https://github.com/peeweek/net.peeweek.gameplay-ingredients.git#2018.3.0",
     ```

   - 你可以通过查看项目窗口来检查包是否被导入，在Packages/Hierarchy结构下，应该有一个Gameplay Ingredients层次结构

</br>

</br>

### Events,Logic  and Actions

Events,Logic and Action是连接彼此执行关卡脚本逻辑的脚本。这些脚本使用Callable列表来执行调用的链接。

- Events：由游戏事件触发并且执行calls。
- Logic：由calls触发，并且会根据条件和逻辑触发其他的calls。
- Actions：由calls触发，并且将触发游戏改变。

<center>![](../assets/img/resources/events-logic-actions.png)</center>

























































































































































