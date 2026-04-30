---
title: "Navicat Premium 17"
source: "https://chatgpt.com/g/g-p-69eec58d1f748191867b4b235bb952be/c/69f19c03-9dcc-83e8-b5cb-d7bf7871e6a7"
author:
published:
created: 2026-04-29
description: "Navicat Premium 17"
tags:
  - "clippings"
---
## 你这张页面是什么：Navicat 的“查询编辑器 + 查询结果”工作台

从截图看，你现在打开的是 **Navicat Premium 17.0.8 的查询页面** ，正在连接 Oracle 服务器里的 `ds225` ，当前模式/Schema 是 `DONGSONG` ，SQL 编辑器里执行的是：

```markdown
select * from dongsong.upssh
```

结果区已经返回了 `UPSSH` 表的数据，底部显示大约 **14288 条记录** ，右下角显示运行时间约 **0.061s** 。你的截图是 Windows 风格界面；官方发布记录中，Navicat Premium Windows 17.0.8 是 2024-06-19 的版本，主要是修复模型工作区打开相关问题和小修复。 [Navicat](https://www.navicat.com/en/products/navicat-premium-release-note)

你可以先把这个页面理解成一句话：

> **左边选数据库对象，中间写 SQL，点运行，下面看结果，右边用代码模板辅助写 SQL。**

Navicat Premium 本质上是一个多数据库客户端，用来连接、管理、查询和分析数据库；官网当前产品页介绍它可在一个应用里管理多种数据库系统，你截图里实际使用的是 Oracle 连接。 [Navicat](https://www.navicat.com/en/products/navicat-premium?utm_source=chatgpt.com)

---

## 1\. 先建立数据库思维：这个页面里的几个核心概念

你作为刚接触项目的学生，先别急着点很多按钮，先记住 5 个词。

| 概念 | 在你截图里的样子 | 通俗解释 |
| --- | --- | --- |
| 连接 Connection | 左侧 `ds223` 、 `ds225` | 一台数据库服务器或一个数据库入口，像“连到某台服务器的门”。 |
| Schema / 用户 | `DONGSONG` 、 `DONGJIA` 、 `BUDGET` 等 | Oracle 里常见情况是“用户名 = Schema 名”，Schema 是装表、视图、函数等对象的容器。官方手册也说明，Oracle 中创建用户时会隐式创建同名 schema。 [Navicat](https://www.navicat.com/manual/pdf_manual/en/navicat_17/win_manual/navicat_en.pdf) |
| 表 Table | `dongsong.upssh` | 真正存数据的地方，像 Excel 表，但更严格，有字段类型、索引、权限。 |
| SQL | `select * from dongsong.upssh` | 你对数据库说的话。这里意思是：查询 `DONGSONG` 这个 Schema 下 `UPSSH` 表的所有列、所有行。 |
| 结果集 Result Set | 下方表格 | SQL 执行后返回的数据。 |

Oracle 里写 `DONGSONG.UPSSH` 的意思是：

```markdown
Schema 名 . 表名
```

所以：

```markdown
select * from dongsong.upssh
```

等价于：

> 从 DONGSONG 用户/Schema 下面的 UPSSH 表中查询全部数据。

---

## 2\. 页面整体分区：从上到下、从左到右认识

这个页面可以分成 7 个区域：

| 区域 | 位置 | 作用 |
| --- | --- | --- |
| 顶部菜单栏 | 最上方：文件、编辑、查看、查询、格式等 | 全局操作入口。 |
| 顶部功能区 | 连接、新建查询、表、视图、函数、用户、数据泵、查询等图标 | 快速进入常用功能。 |
| 左侧导航树 | `我的连接` 、 `ds223` 、 `ds225` 、一堆 Schema | 查看连接、Schema、表、视图、函数等对象。 |
| 中间 SQL 编辑区 | 写着 `select * from dongsong.upssh` 的地方 | 写 SQL、格式化 SQL、运行 SQL、解释执行计划。 |
| 下方结果区 | 显示 DHDH、PO、YDH、CHBM 等列的表格 | 查看查询结果、导出、固定、数据分析。 |
| 右侧信息/代码段面板 | CASE、SELECT Syntax、UPDATE Syntax 等 | 放代码模板、对象信息、DDL、标识符等。Navicat 的信息面板可显示对象信息、DDL、依赖关系、代码段、标识符等内容。 [Navicat](https://www.navicat.com/manual/pdf_manual/en/navicat_17/win_manual/navicat_en.pdf) |
| 底部状态栏 | `只读` 、 `第1条记录` 、 `Ln 1, Col 30` 、运行时间 | 告诉你当前状态、光标位置、记录数、执行耗时。 |

Navicat 官方手册也把主窗口描述为由多个工具栏和面板组成：主工具栏、导航面板、对象面板、信息面板、状态栏等。 [Navicat](https://www.navicat.com/manual/pdf_manual/en/navicat_17/win_manual/navicat_en.pdf)

---

## 3\. 顶部菜单栏：每个菜单是干什么的

你截图最上方有这些菜单：

```markdown
文件  编辑  查看  查询  格式  收藏夹  工具  窗口  帮助
```

### 文件

主要处理打开、保存、导入、导出、打印等。

在查询页面里，你最常用的是：

```markdown
保存查询
打开外部 SQL 文件
保存为外部 SQL 文件
```

Navicat 支持把打开的外部查询保存成 Navicat 查询，也支持把 Navicat 查询另存为外部文件。 [Navicat](https://www.navicat.com/manual/pdf_manual/en/navicat_17/win_manual/navicat_en.pdf)

你现在的查询标签叫 **“无标题 - 查询”** ，说明这条 SQL 还没有保存名字。保存后它会变成一个可复用的查询对象。

---

### 编辑

常用来做：

```markdown
复制、粘贴、撤销、查找、替换、代码补全
```

在 SQL 编辑器里， `Ctrl + F` 可以查找文本， `F3` 可以继续查找下一个。官方手册说明，Navicat 的编辑器支持查找、替换、高亮全部、区分大小写、正则、整词匹配等。 [Navicat](https://www.navicat.com/manual/pdf_manual/en/navicat_17/win_manual/navicat_en.pdf)

---

### 查看

控制页面布局，例如：

```markdown
显示/隐藏导航面板
显示/隐藏信息面板
调整结果区位置
自动换行
缩放 SQL 字体
```

例如你觉得右侧代码段太占位置，可以在“查看”里隐藏信息面板；如果你想让查询结果显示在新页面、底部或右侧，也可以通过“查看 -> 结果”相关设置调整。官方手册说明，查询结果位置可以放在整页、编辑器右侧或编辑器下方。 [Navicat](https://www.navicat.com/manual/pdf_manual/en/navicat_17/win_manual/navicat_en.pdf)

---

### 查询

和 SQL 执行直接相关，常用功能是：

```markdown
运行
运行当前语句
运行选中部分
停止
解释 Explain
```

Navicat 的查询设计器支持执行整个查询、执行当前光标所在语句、执行选中的 SQL，以及停止正在执行的查询。 [Navicat](https://www.navicat.com/manual/pdf_manual/en/navicat_17/win_manual/navicat_en.pdf)

---

### 格式

用来整理 SQL 代码。

常用功能：

```markdown
缩进
取消缩进
注释 / 取消注释
大小写转换
美化 SQL
压缩 SQL
```

比如你写了：

```markdown
select * from dongsong.upssh where dhdh='D25DS051489'
```

点击 **美化 SQL** 后，可能整理成：

```markdown
SELECT
  *
FROM
  dongsong.upssh
WHERE
  dhdh = 'D25DS051489';
```

官方手册说明，Navicat 支持缩进、注释、大小写转换、美化 SQL、压缩 SQL 等格式化能力。 [Navicat](https://www.navicat.com/manual/pdf_manual/en/navicat_17/win_manual/navicat_en.pdf)

---

### 收藏夹

把常用连接、表、查询加入收藏，之后不用在左边一层层找。

适合你以后常用这些对象时使用：

```markdown
DONGSONG.UPSSH
某些常用查询
某些项目表
```

---

### 工具

一般是高级工具入口，例如：

```markdown
选项设置
数据传输
数据同步
结构同步
数据生成
导入导出
服务器监控
```

你刚学项目时，不建议一开始乱点“同步”“传输”“生成数据”，这些功能可能会改动数据库结构或数据。

---

### 窗口

管理打开的多个标签页。

比如你同时打开：

```markdown
查询 1
查询 2
表 UPSSH
某个视图
某个函数
```

可以用窗口菜单切换或排列。

---

### 帮助

查看帮助文档、注册信息、检查更新等。

---

## 4\. 顶部功能区：截图里每个大图标的含义

截图上方这一排图标是 Navicat 的快速入口。

## 连接

用来新建或管理数据库连接。

你左边现在有：

```markdown
ds223
ds225
```

这两个就是连接。右键连接通常可以编辑连接、测试连接、断开连接、重连、删除连接等。

新手最容易犯的错是：

> 明明想查测试库，却连到了生产库；明明想查 `ds225` ，结果 SQL 跑在 `ds223` 上。

所以运行 SQL 前，必须看中间工具栏的连接下拉框，现在你的截图显示的是：

```markdown
ds225
```

---

## 新建查询

打开一个新的 SQL 编辑窗口。

Navicat 官方手册说明，可以通过主工具栏的 **New Query** 创建新查询，即使没有先打开连接也能新建查询窗口。 [Navicat](https://www.navicat.com/manual/pdf_manual/en/navicat_17/win_manual/navicat_en.pdf)

快捷键通常是：

```markdown
Ctrl + Q
```

---

## 表

管理数据库表。

表是保存真实数据的地方。Navicat 手册说明，表由行和列组成，表设计器可以创建、编辑字段、索引、外键等；打开表后，表查看器会以网格或表单方式显示数据。 [Navicat](https://www.navicat.com/manual/pdf_manual/en/navicat_17/win_manual/navicat_en.pdf)

你现在查询的：

```markdown
dongsong.upssh
```

就是一张表。

表相关操作一般包括：

```markdown
查看数据
新建表
修改表结构
查看字段
查看索引
导入导出数据
```

新手注意：  
**不要随便右键删除表、清空表、截断表。**

---

## 视图

视图可以理解成“保存好的查询结果定义”。

它不一定真的保存数据，而是把一段 SQL 包装成一个可查询对象。手册也说明，视图允许用户像访问单表一样访问一组表的数据，也可用于限制用户访问某些行。 [Navicat](https://www.navicat.com/manual/pdf_manual/en/navicat_17/win_manual/navicat_en.pdf)

例如公司可能有一个视图：

```markdown
V_UPSSH_VALID
```

里面只展示有效记录，你直接查视图比直接查原始表更安全。

---

## 实体化视图

实体化视图可以理解成：

> 预先算好并保存起来的查询结果。

普通视图像“公式”，每次查都现算；实体化视图像“提前算好的缓存表”，适合汇总、复制、分发数据。Navicat 手册说明，Oracle 的 Materialized View 用于汇总、计算、复制、分发数据，并可以刷新。 [Navicat](https://www.navicat.com/manual/pdf_manual/en/navicat_17/win_manual/navicat_en.pdf)

---

## 函数

管理数据库里的函数、过程。

Oracle 项目里常见对象有：

```markdown
Function 函数
Procedure 存储过程
Package 包
Trigger 触发器
```

手册说明，Navicat 的函数设计器可用于编辑过程/函数定义，也支持执行函数/过程，出错时会显示错误信息；在 Premium/Enterprise/Standard 版本中，Oracle 对象还支持调试相关能力。 [Navicat](https://www.navicat.com/manual/pdf_manual/en/navicat_17/win_manual/navicat_en.pdf)

你刚入门时先会查表即可，函数和过程后面再学。

---

## 用户

管理数据库用户、角色、权限。

比如：

```markdown
谁能查表
谁能改表
谁能执行函数
谁能创建对象
```

项目开发中，这部分通常由 DBA 或后端负责人管理。你作为学习者，先了解即可，不建议随便改权限。

---

## 其它

Oracle 下这里会放很多其他对象，比如：

```markdown
Database Link
Index
Sequence
Synonym
Trigger
Package
Type
Tablespace
Public Synonym
```

Navicat 手册列出了 Oracle 的其他对象，包括 Database Link、Index、Sequence、Synonym、Trigger、Type、Tablespace 等。 [Navicat](https://www.navicat.com/manual/pdf_manual/en/navicat_17/win_manual/navicat_en.pdf)

简单理解：

| 对象 | 通俗解释 |
| --- | --- |
| Index 索引 | 加快查询的数据结构。 |
| Sequence 序列 | 自动生成编号。 |
| Trigger 触发器 | 数据变化时自动执行的逻辑。 |
| Package 包 | Oracle 里把函数、过程封装在一起。 |
| Synonym 同义词 | 给对象起别名。 |
| Database Link | 连接另一个数据库的通道。 |
| Tablespace 表空间 | Oracle 存储空间管理单位。 |

---

## 数据泵

这是 Oracle Data Pump，用于导出、导入数据和元数据。

官方手册说明，Oracle Data Pump 包括 Export 和 Import 两个工具：Export 把数据和元数据卸载到 dump 文件集，Import 把 dump 文件加载到目标系统。 [Navicat](https://www.navicat.com/manual/pdf_manual/en/navicat_17/win_manual/navicat_en.pdf)

新手注意：

> 数据泵是重量级功能，可能涉及整库/整 Schema 导入导出，不要在生产环境随便执行。

---

## 查询

进入保存的查询列表。

你现在所在页面就是查询编辑器。Navicat 的 Query Editor 允许创建和编辑 SQL 文本，准备并执行选中查询，也可以在一个查询窗口里写多条 SQL。 [Navicat](https://www.navicat.com/manual/pdf_manual/en/navicat_17/win_manual/navicat_en.pdf)

---

## 自动运行

用来创建批处理任务和计划任务。

例如：

```markdown
每天凌晨自动导出某个查询
每周自动备份
每天自动生成报表
```

官方手册说明，Automation 可以按指定时间或周期执行 Query、Backup、Data Transfer、Import、Export、BI、Model 等任务，并可使用 Windows Task Scheduler 计划运行。 [Navicat](https://www.navicat.com/manual/pdf_manual/en/navicat_17/win_manual/navicat_en.pdf)

新手不要一开始配置自动任务，先把手工查询学会。

---

## 模型

数据库建模工具。

可以用来画 ER 图、反向工程数据库结构、生成数据字典、比较模型等。Navicat 手册说明，模型功能可创建概念、逻辑、物理模型，支持反向工程、正向工程、模型比较、生成数据字典等。 [Navicat](https://www.navicat.com/manual/pdf_manual/en/navicat_17/win_manual/navicat_en.pdf)

你以后理解项目表关系时很有用。

---

## BI

用来做图表、仪表盘、数据分析。

例如把查询结果做成：

```markdown
柱状图
折线图
饼图
数据看板
报表页面
```

你刚学项目时，先用查询结果表格看数据；后面再学 BI。

---

## 5\. 左侧导航树：怎么读懂 ds225 下面那一堆名字

左侧显示：

```markdown
我的连接
  ds223
  ds225
    ANONYMOUS
    APEX_030200
    BD
    BUDGET
    DONGJIA
    DONGSONG
    ...
```

这里不是“一个个普通文件夹”，而是数据库连接和 Oracle Schema/用户。

你现在连接的是：

```markdown
ds225
```

下面那一堆：

```markdown
ANONYMOUS
APEX_030200
BD
DONGSONG
...
```

多数是 Oracle 用户/Schema。Oracle 中 Schema 是数据库对象的逻辑容器，通常与用户名同名。 [Navicat](https://www.navicat.com/manual/pdf_manual/en/navicat_17/win_manual/navicat_en.pdf)

### 新手应该怎么用左侧导航树？

第一步，确认连接：

```markdown
ds225
```

第二步，找到项目相关 Schema：

```markdown
DONGSONG
```

第三步，展开后找：

```markdown
表
视图
函数
包
触发器
索引
```

第四步，右键对象：

```markdown
打开表
设计表
查看 DDL
复制名称
新建查询
```

### 左下角搜索框

底部有“搜索”，可以搜索对象名称。

当项目表很多时，不要肉眼找，可以直接搜：

```markdown
UPSSH
```

---

## 6\. 中间 SQL 编辑器：这是你每天最常用的地方

你截图中间区域主要有这些按钮：

```markdown
保存
查询创建工具
美化 SQL
代码段
连接下拉框 ds225
Schema 下拉框 DONGSONG
运行
停止
解释
```

## 保存

保存当前 SQL。

建议你把常用查询保存成有意义的名字，比如：

```markdown
查询_UPSSH_按单号查询
查询_UPSSH_最近更新记录
查询_UPSSH_统计记录数
```

这样下次不用重新写。

---

## 查询创建工具

也叫 Query Builder，可视化构建 SELECT 查询。

它适合不会写复杂 SQL 的初学者，比如你可以用鼠标拖表、选字段、设置条件、建立连接关系。

不过要记住：  
**Query Builder 主要支持 SELECT。复杂的 INSERT、UPDATE、DELETE 还是要在 SQL 编辑器里写。** 官方手册也说明 Query Builder 支持 SELECT 语句，其他复杂查询应使用 Query Editor。 [Navicat](https://www.navicat.com/manual/pdf_manual/en/navicat_17/win_manual/navicat_en.pdf)

---

## 美化 SQL

把 SQL 整理得更好看。

比如：

```markdown
select dhdh,po,ydh from dongsong.upssh where dhdh='D25DS051489'
```

美化后更适合阅读：

```markdown
SELECT
  dhdh,
  po,
  ydh
FROM
  dongsong.upssh
WHERE
  dhdh = 'D25DS051489';
```

美化 SQL 不改变查询逻辑，只改变排版。

---

## 代码段

打开或关闭右侧的代码模板列表。

你右边看到的：

```markdown
CASE
COMMENTS
FOR
IF...ELSE...
INSERT Syntax
SELECT Syntax
UPDATE Syntax
Runtime Parameter
```

就是代码段。

官方手册说明，代码段可以把可复用代码插入编辑器；可以从右侧代码段库拖拽进去，也可以通过智能补全插入。 [Navicat](https://www.navicat.com/manual/pdf_manual/en/navicat_17/win_manual/navicat_en.pdf)

---

## 连接下拉框：ds225

这是最重要的位置之一。

它决定 SQL 跑在哪个连接上。

你必须养成习惯：

> 每次点运行前，看一眼连接是不是正确。

你的截图现在是：

```markdown
ds225
```

---

## Schema 下拉框：DONGSONG

它决定当前默认 Schema。

如果你的 SQL 写成：

```markdown
select * from upssh;
```

那么数据库会优先在当前 Schema 里找 `UPSSH` 。

但你现在写的是：

```markdown
select * from dongsong.upssh;
```

这叫“全限定名”，更清楚，意思是明确查询 `DONGSONG` 下面的 `UPSSH` 。

新手建议尽量写全：

```markdown
select * from dongsong.upssh;
```

这样不容易查错 Schema。

---

## 运行

运行 SQL。

Navicat 支持：

```markdown
运行整个查询
运行当前语句
运行选中的 SQL
```

官方手册说明，选中一部分 SQL 后可以运行选中部分；也可以让光标停在某条 SQL 上，然后运行当前语句。 [Navicat](https://www.navicat.com/manual/pdf_manual/en/navicat_17/win_manual/navicat_en.pdf)

### 新手正确运行方式

如果编辑器里有多条 SQL，比如：

```markdown
select count(*) from dongsong.upssh;

select * from dongsong.upssh where rownum <= 10;
```

不要直接点“运行全部”，而是：

1. 鼠标选中你要执行的那一条；
2. 点击“运行”旁边的小箭头；
3. 选择“运行选中”。

这样不容易误执行。

---

## 停止

如果 SQL 查太久、卡住、返回数据太大，可以点 **停止** 。

比如你写了：

```markdown
select * from very_big_table;
```

然后数据库一直跑，赶紧停止。

---

## 解释

解释执行计划，也就是看数据库准备怎么执行你的 SQL。

这对判断 SQL 慢不慢很有帮助。官方手册说明，点击 Explain 时，Navicat 会向数据库发送 EXPLAIN 语句，数据库生成执行计划，然后显示在 Explain 标签中；也可以只解释选中的 SQL。 [Navicat](https://www.navicat.com/manual/pdf_manual/en/navicat_17/win_manual/navicat_en.pdf)

你截图下方已经有：

```markdown
解释 1
```

说明你可能已经点过“解释”。

---

## 7\. 认真拆解你现在这句 SQL

你的 SQL 是：

```markdown
select * from dongsong.upssh
```

逐词解释：

| 部分 | 意思 |
| --- | --- |
| `select` | 查询数据。 |
| `*` | 查询所有字段。 |
| `from` | 从哪个表查。 |
| `dongsong` | Schema 名。 |
| `upssh` | 表名。 |

这条 SQL 的完整含义是：

> 查询 `DONGSONG` 这个 Schema 下 `UPSSH` 表的所有字段、所有记录。

---

## 为什么不建议长期使用 select \*

初学可以用：

```markdown
select * from dongsong.upssh;
```

但正式开发中不建议经常这样写，因为：

1. 字段太多，看不清重点；
2. 数据太大，容易慢；
3. 可能把敏感字段也查出来；
4. 表结构变化时，程序或导出结果容易受影响。

更好的写法是只查需要的字段：

```markdown
select
  dhdh,
  po,
  ydh,
  chbm,
  dhsl,
  updatetime
from
  dongsong.upssh
where
  rownum <= 50;
```

Oracle 里可以用 `ROWNUM` 限制返回行数，例如 `where rownum <= 50` 。Oracle 官方文档说明， `ROWNUM` 是返回行的序号，可用于限制查询返回的行数；如果要先排序再取前 N 条，应把 `ORDER BY` 放进子查询。 [Oracle Docs](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/ROWNUM-Pseudocolumn.html?utm_source=chatgpt.com)

---

## 8\. 你截图结果表里的字段怎么理解

你结果区看到这些列：

```markdown
DHDH
PO
YDH
CHBM
DHSL
UPDATETIME
SCDATE
SXDATE
XLH
YBSL
UDI
BZ
```

这些字段名大概率是业务缩写，不能完全靠猜，但可以先这样理解：

| 字段 | 可能含义 | 学习时怎么处理 |
| --- | --- | --- |
| `DHDH` | 到货单号 / 单据号一类 | 先当作业务主编号。 |
| `PO` | 采购订单号 | 可能来自采购系统。 |
| `YDH` | 原单号 / 运单号 / 业务单号 | 需要查项目文档确认。 |
| `CHBM` | 存货编码 / 产品编码 / 物料编码 | 可能是物料主数据。 |
| `DHSL` | 到货数量 | 数量字段。 |
| `UPDATETIME` | 更新时间 | 常用来查最近数据。 |
| `SCDATE` | 生产日期 / 生成日期 | 要结合业务确认。 |
| `SXDATE` | 失效日期 / 生效日期 | 要结合业务确认。 |
| `XLH` | 序列号 | 可能是设备或物料序列号。 |
| `YBSL` | 应报数量 / 已报数量 / 原包数量 | 需要业务确认。 |
| `UDI` | 唯一设备标识或外部编码 | 医疗/设备类项目中常见。 |
| `BZ` | 备注 | 备注字段。 |

学习数据库项目时，遇到缩写字段，不要只靠猜，要做三件事：

1. 查项目数据字典；
2. 查表注释、字段注释；
3. 问业务或项目负责人。

你也可以用 Oracle 数据字典查字段类型。Oracle 官方文档说明， `ALL_TAB_COLUMNS` 描述当前用户可访问的表、视图、簇的列信息。 [Oracle Docs](https://docs.oracle.com/en/database/oracle/oracle-database/19/refrn/ALL_TAB_COLUMNS.html?utm_source=chatgpt.com)

```markdown
select
  column_id,
  column_name,
  data_type,
  data_length,
  nullable
from
  all_tab_columns
where
  owner = 'DONGSONG'
  and table_name = 'UPSSH'
order by
  column_id;
```

---

## 9\. 结果区：你下方表格里的每个功能

下方有这些标签：

```markdown
消息
摘要
结果 1
解释 1
```

## 消息

显示执行日志和错误信息。

比如 SQL 写错时，可能出现：

```markdown
ORA-00942: table or view does not exist
ORA-00904: invalid identifier
ORA-01722: invalid number
```

你以后排错第一步就是看“消息”。

---

## 摘要

显示执行概要，例如运行时间、影响行数、执行状态等。

你截图底部也显示：

```markdown
运行时间: 0.061s
```

这个代表这次 SQL 执行很快。

---

## 结果 1

这是 SQL 返回的数据。

Navicat 手册说明，查询返回数据时会打开 Result 标签，并以网格显示结果；结果可以用 Grid View 或 Form View 查看。 [Navicat](https://www.navicat.com/manual/pdf_manual/en/navicat_17/win_manual/navicat_en.pdf)

你现在的 `结果 1` 显示了 `UPSSH` 表数据。

---

## 解释 1

显示执行计划。

它不是业务数据，而是数据库告诉你：

```markdown
我打算怎么查这张表
是否走索引
是否全表扫描
连接顺序是什么
成本大概多少
```

以后 SQL 慢时，你要学会看这里。

---

## 结果区上方的按钮

你截图中结果区右上方有：

```markdown
单元格编辑器
数据分析
导出
固定
```

### 单元格编辑器

打开单元格编辑窗口，方便查看长文本、大字段、复杂内容。

例如字段内容很长，表格里看不全，就用它。

---

### 数据分析

对当前结果做数据概览或剖析。

Navicat 查询结果工具栏中有 Data Profiling，可对查询结果做数据剖析。 [Navicat](https://www.navicat.com/manual/pdf_manual/en/navicat_17/win_manual/navicat_en.pdf)

适合查看：

```markdown
空值比例
不同值数量
最大值
最小值
分布情况
```

---

### 导出

把查询结果导出成文件。

常见格式可能包括：

```markdown
CSV
Excel
TXT
JSON
XML
```

正式项目里导出数据前要注意权限和敏感数据。

---

### 固定

把当前结果固定住。

官方手册说明，重新执行查询时，旧结果标签通常会被清理；固定结果标签后，它会保留在结果栏中。 [Navicat](https://www.navicat.com/manual/pdf_manual/en/navicat_17/win_manual/navicat_en.pdf)

这个功能很好用，比如你想比较两次查询结果：

1. 查第一版 SQL；
2. 固定结果；
3. 改 SQL；
4. 再查第二版；
5. 对比两个结果标签。

---

## 10\. 右侧代码段面板：帮你快速写 SQL

右侧显示的是代码段列表。

你看到：

```markdown
CASE
COMMENTS
FOR
IF...ELSE...
INSERT Syntax
LOOP
Runtime Parameter
SELECT Syntax
UPDATE Syntax
```

这些都是可插入的模板。

官方手册说明，代码段库包含内置和用户自定义代码段，可以通过标签筛选，也可以搜索；内置代码段不可编辑，自定义代码段可以编辑。 [Navicat](https://www.navicat.com/manual/pdf_manual/en/navicat_17/win_manual/navicat_en.pdf)

## 怎么用代码段

方法一：双击代码段。  
方法二：拖到 SQL 编辑器。  
方法三：在编辑器里输入代码段名称，让自动补全弹出来。

比如你双击：

```markdown
SELECT Syntax
```

它可能帮你插入 SELECT 模板。

---

## Runtime Parameter：运行时参数

这个对新手很实用。

你可以写：

```markdown
select
  *
from
  dongsong.upssh
where
  dhdh = [$dhdh];
```

运行时 Navicat 会弹出输入框，让你输入 `dhdh` 的值。

官方手册说明，查询参数可以写成以 `$` 开头、放在 `[]` 里的标识符，例如 `[$any_name]` ；执行时会弹出输入参数窗口。 [Navicat](https://www.navicat.com/manual/pdf_manual/en/navicat_17/win_manual/navicat_en.pdf)

这样你不用每次手动改 SQL 里的单号。

---

## 自定义结果标签名

你右侧有一个：

```markdown
Customize Result Tab Name
```

这个可以让结果标签更清楚。

例如：

```markdown
-- NAME:最近50条UPSSH
select
  dhdh,
  po,
  ydh,
  updatetime
from
  dongsong.upssh
where
  rownum <= 50;
```

执行后结果标签名会更易读。官方手册说明，在 SELECT 前添加 `-- NAME:tab_name` 或 `/*NAME:tab_name*/` 可以自定义结果标签名称。 [Navicat](https://www.navicat.com/manual/pdf_manual/en/navicat_17/win_manual/navicat_en.pdf)

---

## 11\. 底部状态栏：这些小字很重要

你底部能看到：

```markdown
只读
第 1 条记录（共 14288 条）
Ln 1, Col 30
运行时间: 0.061s
```

## 只读

说明当前结果集是只读状态。

常见原因：

```markdown
你没有修改权限
查询结果不是可编辑表
SQL 没有包含可唯一定位行的信息
结果来自复杂查询
Navicat 当前以只读方式打开
```

新手阶段看到“只读”是好事，说明不容易误改数据。

---

## 第 1 条记录（共 14288 条）

表示当前选中第 1 条，总结果大约 14288 条。

这也说明你的 SQL 没有加过滤条件，所以查出了很多数据。

---

## Ln 1, Col 30

表示光标在 SQL 编辑器的：

```markdown
第 1 行，第 30 列
```

出错时，很多数据库错误会提示行号、列号，你可以结合这里定位问题。

---

## 运行时间

```markdown
运行时间: 0.061s
```

表示这次查询执行耗时。

注意：运行时间短不代表 SQL 永远安全。现在只有 14288 条还可以，如果以后表有几百万条， `select *` 可能会很慢。

---

## 12\. 新手最容易不会的问题：逐个拆解

## 问题 1：为什么左边有很多名字，看不懂？

因为你连的是 Oracle。Oracle 里很多名字是用户/Schema，不一定是“数据库”。

你重点关注项目相关 Schema，例如：

```markdown
DONGSONG
```

不要随便动：

```markdown
SYS
SYSTEM
DBSNMP
OUTLN
APEX_*
```

这些通常是系统或平台相关用户。

---

## 问题 2：为什么我查表要写 dongsong.upssh？

因为这是完整对象名。

格式是：

```markdown
schema_name.table_name
```

写完整可以避免查错表。

推荐：

```markdown
select * from dongsong.upssh;
```

不推荐新手只写：

```markdown
select * from upssh;
```

除非你非常确定当前 Schema 就是 `DONGSONG` 。

---

## 问题 3：灰色的 (Null) 是什么意思？

`NULL` 不是字符串，也不是 0，也不是空格。

它表示：

> 数据库中这个字段没有值。

查询 NULL 要这样写：

```markdown
select
  *
from
  dongsong.upssh
where
  scdate is null;
```

不要写：

```markdown
where scdate = null
```

这是错误思路。

查非空：

```markdown
where scdate is not null
```

---

## 问题 4：我怎么只看前 20 条？

Oracle 中可以写：

```markdown
select
  *
from
  dongsong.upssh
where
  rownum <= 20;
```

更专业一点，只查必要字段：

```markdown
select
  dhdh,
  po,
  ydh,
  chbm,
  dhsl,
  updatetime
from
  dongsong.upssh
where
  rownum <= 20;
```

---

## 问题 5：我怎么查最近更新的 10 条？

如果你想先按更新时间倒序，再取前 10 条，Oracle 推荐写成子查询：

```markdown
select
  *
from
  (
    select
      dhdh,
      po,
      ydh,
      chbm,
      dhsl,
      updatetime
    from
      dongsong.upssh
    order by
      updatetime desc
  )
where
  rownum <= 10;
```

原因是 Oracle 的 `ROWNUM` 是在取行过程中产生的，排序和取前 N 条要注意顺序。Oracle 官方文档也说明，如果要让 `ROWNUM` 条件应用在排序之后，应把 `ORDER BY` 放入子查询，再在外层使用 `ROWNUM` 。 [Oracle Docs](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqlrf/ROWNUM-Pseudocolumn.html?utm_source=chatgpt.com)

---

## 问题 6：我怎么按单号查？

假设你要查 `DHDH = 'D25DS051489'` ：

```markdown
select
  *
from
  dongsong.upssh
where
  dhdh = 'D25DS051489';
```

字符串要加单引号。

数字通常不加：

```markdown
where dhsl = 2
```

日期要看字段类型。Oracle 日期常见写法：

```markdown
where updatetime >= date '2025-06-01'
```

---

## 问题 7：我怎么统计总条数？

```markdown
select
  count(*) as total_count
from
  dongsong.upssh;
```

这比 `select *` 更适合先了解表规模。

---

## 问题 8：我怎么知道某列有哪些值？

比如想看 `CHBM` 有哪些不同值：

```markdown
select distinct
  chbm
from
  dongsong.upssh
where
  chbm is not null;
```

如果想统计每个 `CHBM` 有多少条：

```markdown
select
  chbm,
  count(*) as cnt
from
  dongsong.upssh
where
  chbm is not null
group by
  chbm
order by
  cnt desc;
```

---

## 问题 9：我想导出结果怎么办？

推荐流程：

1. 先写带条件的 SQL，不要直接导出全表；
2. 点“运行”；
3. 确认结果没问题；
4. 点击结果区右上方“导出”；
5. 选择 CSV 或 Excel；
6. 保存文件。

正式项目中要注意数据权限，尤其是订单、客户、设备、人员、金额类数据。

---

## 问题 10：SQL 报错了怎么办？

按这个顺序排查：

### 第一步：看“消息”标签

错误一般会显示在那里。

### 第二步：看连接和 Schema

确认是不是：

```markdown
ds225
DONGSONG
```

### 第三步：看表名、字段名有没有拼错

比如：

```markdown
select dhdhh from dongsong.upssh;
```

如果表里没有 `DHDHH` ，就会报字段无效。

### 第四步：看字符串有没有加单引号

错误：

```markdown
where dhdh = D25DS051489
```

正确：

```markdown
where dhdh = 'D25DS051489'
```

### 第五步：看日期格式

日期字段不要乱写字符串，优先用：

```markdown
date '2025-06-01'
```

---

## 13\. 建议你按这个学习路线掌握 Navicat

## 第 1 阶段：会安全查询

你要掌握：

```markdown
select
from
where
order by
count(*)
distinct
is null
is not null
rownum
```

练习 SQL：

```markdown
select count(*) from dongsong.upssh;

select * from dongsong.upssh where rownum <= 10;

select dhdh, po, ydh, updatetime
from dongsong.upssh
where rownum <= 20;

select *
from dongsong.upssh
where dhdh = 'D25DS051489';
```

---

## 第 2 阶段：会看表结构

练习：

```markdown
select
  column_id,
  column_name,
  data_type,
  data_length,
  nullable
from
  all_tab_columns
where
  owner = 'DONGSONG'
  and table_name = 'UPSSH'
order by
  column_id;
```

你要能回答：

```markdown
UPSSH 有哪些字段？
每个字段是什么类型？
哪些字段允许为空？
哪个字段像主业务编号？
哪个字段是时间字段？
```

---

## 第 3 阶段：会分析结果

练习：

```markdown
select
  chbm,
  count(*) as cnt
from
  dongsong.upssh
where
  chbm is not null
group by
  chbm
order by
  cnt desc;
```

你要能看懂：

```markdown
哪个物料/编码最多？
有没有空值？
时间分布是否正常？
某个单号是否重复？
```

---

## 第 4 阶段：会用 Navicat 辅助功能

你要会：

```markdown
美化 SQL
运行选中 SQL
解释执行计划
固定结果
导出结果
使用代码段
使用运行时参数
查看字段信息
搜索对象
```

---

## 第 5 阶段：谨慎接触修改数据

修改数据前必须先会这些：

```markdown
select ... where ...
update ... set ... where ...
delete ... where ...
commit
rollback
```

但你现在刚学项目，先不要执行 `UPDATE` 和 `DELETE` 。

尤其不要执行这种危险 SQL：

```markdown
delete from dongsong.upssh;
```

或者：

```markdown
update dongsong.upssh set dhsl = 0;
```

因为没有 `where` 条件，会影响整张表。

---

## 14\. 给你一套“安全查询模板”

以后你可以直接套用。

## 查询前 20 条

```markdown
select
  *
from
  dongsong.upssh
where
  rownum <= 20;
```

## 查询指定字段

```markdown
select
  dhdh,
  po,
  ydh,
  chbm,
  dhsl,
  updatetime
from
  dongsong.upssh
where
  rownum <= 50;
```

## 按单号查询

```markdown
select
  *
from
  dongsong.upssh
where
  dhdh = 'D25DS051489';
```

## 查最近更新

```markdown
select
  *
from
  (
    select
      dhdh,
      po,
      ydh,
      chbm,
      dhsl,
      updatetime
    from
      dongsong.upssh
    order by
      updatetime desc
  )
where
  rownum <= 20;
```

## 查空生产日期

```markdown
select
  *
from
  dongsong.upssh
where
  scdate is null
  and rownum <= 50;
```

## 统计总数

```markdown
select
  count(*) as total_count
from
  dongsong.upssh;
```

## 按物料编码统计

```markdown
select
  chbm,
  count(*) as cnt
from
  dongsong.upssh
group by
  chbm
order by
  cnt desc;
```

## 使用运行时参数

```markdown
select
  *
from
  dongsong.upssh
where
  dhdh = [$dhdh];
```

---

## 15\. 你现在应该特别注意的 6 个安全习惯

1. **运行前看连接** ：确认是 `ds225` 还是 `ds223` 。
2. **运行前看 Schema** ：确认是 `DONGSONG` 。
3. **少用 `select *`** ：学习可以，正式分析尽量指定字段。
4. **先限制行数** ：Oracle 用 `rownum <= 20` 。
5. **不要乱点删除、清空、截断、同步、数据泵导入** 。
6. **修改数据前先截图、先备份、先让导师或负责人确认。**

---

## 16\. 最后验证：做完这些，基本说明你学会了这个页面

请你自己完成下面 8 个小任务。每个任务都能完成，说明你已经掌握这张页面的核心用法。

## 验证任务 1：指出当前连接和 Schema

回答：

```markdown
当前连接是：
当前 Schema 是：
当前查询的完整表名是：
```

标准答案应接近：

```markdown
当前连接是 ds225
当前 Schema 是 DONGSONG
当前查询的完整表名是 DONGSONG.UPSSH
```

---

## 验证任务 2：把全表查询改成只查 20 条

写出 SQL。

标准答案：

```markdown
select
  *
from
  dongsong.upssh
where
  rownum <= 20;
```

---

## 验证任务 3：只查询 6 个关键字段

字段：

```markdown
DHDH
PO
YDH
CHBM
DHSL
UPDATETIME
```

标准答案：

```markdown
select
  dhdh,
  po,
  ydh,
  chbm,
  dhsl,
  updatetime
from
  dongsong.upssh
where
  rownum <= 20;
```

---

## 验证任务 4：查某个单号

用截图里第一行的单号：

```markdown
D25DS051489
```

标准答案：

```markdown
select
  *
from
  dongsong.upssh
where
  dhdh = 'D25DS051489';
```

---

## 验证任务 5：查 SCDATE 为空的数据

标准答案：

```markdown
select
  *
from
  dongsong.upssh
where
  scdate is null
  and rownum <= 50;
```

---

## 验证任务 6：查最近更新时间前 10 条

标准答案：

```markdown
select
  *
from
  (
    select
      dhdh,
      po,
      ydh,
      chbm,
      dhsl,
      updatetime
    from
      dongsong.upssh
    order by
      updatetime desc
  )
where
  rownum <= 10;
```

---

## 验证任务 7：解释这些按钮

请你能说出下面按钮的作用：

```markdown
保存
查询创建工具
美化 SQL
代码段
运行
停止
解释
导出
固定
```

合格答案：

| 按钮 | 你应能说出的意思 |
| --- | --- |
| 保存 | 保存当前 SQL 查询。 |
| 查询创建工具 | 用图形化方式生成 SELECT 查询。 |
| 美化 SQL | 整理 SQL 排版。 |
| 代码段 | 插入常用 SQL 模板。 |
| 运行 | 执行 SQL。 |
| 停止 | 停止正在执行的 SQL。 |
| 解释 | 查看执行计划。 |
| 导出 | 导出查询结果。 |
| 固定 | 保留当前结果标签，方便对比。 |

---

## 验证任务 8：解释 (Null) 和 只读

你应能回答：

```markdown
(Null) 代表什么？
只读代表什么？
```

标准答案：

```markdown
(Null) 表示数据库中该字段没有值，不是字符串，也不是 0。
只读表示当前结果集不能直接编辑，可能是权限、查询形式或打开方式导致的。
```

---

## 评分标准

| 分数 | 掌握程度 |
| --- | --- |
| 0-3 题正确 | 还停留在看界面，需要继续练基本概念。 |
| 4-6 题正确 | 已经能安全做基础查询。 |
| 7-8 题正确 | 已经掌握这个查询页面的核心用法，可以开始学习表关系、视图、函数和执行计划。 |

完成后，把你写的 SQL 和运行结果截图发来，我会按这套标准帮你判断掌握程度。