# Word文档

## 招标类型与替换逻辑

### 新增国际公开类型及替换字段逻辑
标签：#Word文档 #后端 #国际公开 #替换字段 #工作流

当前项目需要新增一个类型，即国际公开，请你根据其他项目类型新增这个类型，其中需要注意的内容如下：首先是工作流的设置，目前backend\graphs\base_graph.py中能否根据当前类型进行适当的修改，我再纠结是否可以复用backend\graphs\base_graph.py这个图，因为国际公开类型我想要的工作流是，word_operations_subgraph在国际公开类型时只有一两个节点了，即delete_tender_param和get_replacements，而replace_content移动到了StandardTenderWorkflowGraph中的update_word节点之后，即update_word后串行执行replace_content，然后就是需要参考backend\nodes\xjcg_word_nodes这种节点，创建一个新的关于国际公开的gjgk_word_nodes，其中放的模块就是gjgk_replace_replacements.py的获取逻辑，需要提取替换对的逻辑如图所示，请你提取出project_name即如图的"project_name"，project_number即如图"254DSITC2512",buyer_name即如图的"上海市第六人民医院"，project_content即如图的"实时细胞代谢分析仪		壹套
（项目预算：人民币210万元，可以采购进口产品）",project_zbr_xbr即如图的"陶文杰、刘韵",zbr_xbr_tel即如图的"8618、8606",zbr_pinyin即如图的"liuyun",bzj_rule即图的"项目预算的2%",然后就是参考backend\states\gngk_tender_state.py的GngkTenderGraphState新增一个GjgkTenderGraphState。国际公开的专属字段是fund_lx即如图的"财政资金"，还有专属字段是tender_nvitation即如图的"项目名称：实时细胞代谢分析仪，招标编号：0811-254DSITC2512"，还有一个专属字段是dlivery_location即如图的"上海市第六人民医院临港院区"，因此gjgk_get_replacements.py也要加上上面这三个专属字段的提取，我主要说明一下新增字段在后续backend\nodes\common_word_nodes\replace_content.py中的替换逻辑，在国际公开gjgk类型中，replace_content.py在替换上面新增的专属字段是要尤其注意，fund_lx需要根据当前项目在后端接口fetch_tender_data获取项目信息中的fund_lx决定内容，即0是自筹资金，1是财政资金，只有这两个值去替换，tender_nvitation也可以从fetch_tender_data中进行拼接，需要的字段是project_name，project_number，如果拼接逻辑不清楚你可以再问我，然后就是dlivery_location这个字段是从AI生成的内容中提取，即内容中要有“交货地点：上海市中医院临港院区”，然后需要用“上海市中医院临港院区”替换前面gjgk_get_replacements.py获取的“上海市第六人民医院临港院区”

### 国内公开服务自筹替换字段抽取与公共逻辑解耦
标签：#Word文档 #后端 #国内公开 #替换字段 #解耦

当前项目需要参考backend\nodes\gjgk_word_nodes\gjgk_get_replacements.py，在backend\nodes\gngk_word_nodes中创建新的gngk_fw_zc_replace_replacements.py的获取逻辑，然后新增提取替换对的逻辑如图所示，请你提取出project_name即如图一的"信息系统开发运维服务"，project_number即如图二的"253677",buyer_name即如图二的"上海市皮肤病医院"，project_content即如图三的"项目名称：信息系统开发运维服务          壹套（项目预算：人民币50万元）",project_zbr_xbr即如图四的"史倩倩、陈雯婷",zbr_xbr_tel即如图的"8607、8619",zbr_pinyin即如图四的"shiqianqian",bzj_rule即图五的"项目预算的2%",platform即图六的““中国采购与招标网”（https://www.chinabidding.cn/）”，请你核实在国内公开服务自筹的类型中，我上面说的需要替换的字段跟backend\nodes\gngk_word_nodes\gngk_hw_zc_get_replacements.py中已经有的替换字段逻辑是否一致，请把一致的内容提取出来并生成新的backend\nodes\gngk_word_nodes\gngk_get_replacements.py，然后在hw_zc和fw_zc中导入，不一致的提取逻辑则分别放在各自的backend\nodes\gngk_word_nodes\gngk_xx_xx_get_replacements.py文件中

### 重命名 gjgk Word 节点并抽取公共 replacement 逻辑
标签：#Word文档 #gjgk #命名规范 #重构 #公共逻辑

有两个方面需要你修改，首先backend\nodes\gjgk_word_nodes中的删除节点和更新节点都需要重命名，改成gjgk_xxxx_xxx的格式，这是我发现的目前不符合命名规则的地方，当然你发现别的地方也有这个字问题可以修改，并且我希望这种命名规则可以写到AGENTS.md中，注意所有的调用地方都需要修改，第二个修改就是请你查看当前三个类型对应的xxx_get_replacements节点，请你提取完全相同的提取字段的函数，然后解耦到backend\nodes\common_word_nodes中，创建一个get_repalcement，然后在三个类型的xxx_get_replacements节点中从公共的get_repalcement调用，

## 修改生成内容流程

### 新增生成内容修改按钮与 rewrite_graph 工作流
标签：#Word文档 #前端 #后端 #rewrite #下载文件

我想要在当前项目上新增新的逻辑和功能，请你分析该功能实现的可能性并进行功能代码设计和计划，如图，这是前端的用户聊天界面的输入框，该前端组件似乎没有接入后端任何接口，现在我想要实现如下的功能，首先在前端用户输入框的组件中，在与模型选择组件并列增加一个相同样式的按钮类似“修改生成内容或者重新生成”，我不知道该怎么起名，功能逻辑是用户点击这个按钮后，首先要判断当前对话中有没有生成完成的文档，如果没有生成完毕的文档，点击按钮时要提示用户需要先生成文档才能进行操作，还需要注意在文档生成过程是不可以点击的，应该灰掉，如果当前用户聊天交互页面中有生成完成的文件，如图出现了可以下载文件的消息，则此时点击该按钮将被选中，然后用户输入问题，这个问题必须跟AI生成内容的问题相关，比如AI生成的内容中，用户发现出现了不需要的内容比如“四、服服务条款期限”，用户可以说“去掉‘四、服服务条款期’这段表述”，那么最终实现结果是ai根据用户的要求调用大模型重新生成了一段内容，在前端页面上体现是机器人先发出一条进度日志的消息，然后再发出了一条AI生成的内容，注意是该消息内容也需要是流式生成，生成完成后机器人应该再发出了一个下载文件的消息，整个过程有如下多个需要注意的事项和设计思路，首先后端一定需要增加一个MessageList去记录机器人和用户的交互聊天记录，记录的触发条件应该是后端graph完整的运行结束，中间不能取消中断，并且前端发送了下载文件的消息，需要记录Message内容是应该包含当前对话的项目类型和下载文件的路径和AI生成的内容，比如当前对话的任务完成后机器人发出了下载文件的消息，此时MessageList应该插入一条AIMessage(content=rewrite_state(tender_type='xjcg'，prepared_doc_path='xxxxx'，polished_text='xxxxx'))，注意不管在何时调用MessageList都只取最新的并且最多六条message，，当用户选中了该按钮，并发送修改提示词点击发送时候，此时应该调用后端，逻辑应该也是一个graph，参考当前的base_graph.py另写一个叫rewirte_graph，该图的工作流应该包含如下节点，首先是判断节点，该节点根据MessageList和用户输入的内容输出应该是修改第几条rewrite_state的数据，比如MessageList当前长度为3，则ai应该生成2，然后获取MessageList[2]的值作为返回值rewrite_state的值。然后从这个节点需要扇出两个并行执行的节点，一个是重写节点，该节点应该构建一个SystemMessage，该系统消息包含修改生成内容提示词，包含了rewrite_state中需要修改的polished_text，然后拼接最新一条的用户消息HumanMessage，然后组成message发给模型invoke，另外一个并行节点是删除节点，是根据状态中的tender_type拿到对应类型的锚点和prepared_doc_path拿到对应的文件路径，然后根据路径读取文件并根据锚点删除文件段落，请判断是否可用直接复用delete_tender_param.py，这两个节点并行进行，然后这两个节点汇入一个更新节点，即将重写的AI生成的内容插入更新到文件中，该更新节点请判断是否可以直接复用generate_polished_text.py，然后这个rewirte_graph工作流全都完成该功能结束，前端在用户聊天交互页面的设计体现则是当用户点击按钮并发送修改消息后，后端在进行生成的这个过程中，前端在聊天交互界面的输入框中，首先发送的向上箭头应该变成可以暂停的样式，如果用户点击这个暂停按钮，就相当于取消生成了，当用户点击发送时机器人应该马上发出一个进度日志的消息，参考目前的前端逻辑该日志记录了rewirte_graph中所有的节点进度日志，然后当后端rewirte_graph进行到重写节点时候发出一条AI生成内容的消息，也参考当前前端逻辑，当rewirte_graph结束以后，机器人发出一条下载文件的消息，这三条消息应该完全参考目前前端机器人发出的三条消息，只不过调用的后端逻辑是不一样的

## 批注与修订痕迹

### 输出专家批注 JSON 数组的文档审查提示词
标签：#Word文档 #批注 #JSON #专家审查 #提示词

1、仅输出一个纯净的 JSON 数组， 格式：`[{"reference_text": "...", "comment_text": "..."}]`，其中reference_text是批注范围，不得少于五个字符串，comment_text有三种写法，参考批注内容的逻辑思维想要提示内容是否有问题用“建议提示：xxxx”，参考删除线批注的逻辑思维想要删除多余语句时用“建议删除：xxxx”，参考非黑色字体批注的逻辑思维想要新增严谨语句时用“建议新增：xxxx”
2、应该学习专家的逻辑思维（例如：看到“等”字就考虑删除，看到参数就质疑是否具有指向性），去发现文档中专家还没点出来的其他类似问题
3、 AI 在修订时，如果遇到不确定的缩写，就参考批注中进行质疑，格式为[{"reference_text": "DAR", "comment_text": "建议提示：专业术语？"}]

### rewrite_graph 修改后保留原批注
标签：#Word文档 #批注 #rewrite #回填 #修复

现在有个问题需要进行修复，当前backend\graphs\rewrite_graph.py中由于删除节点会把锚点之间的所有内容删除，包括批注，这是没有问题的逻辑，但是我想要实现的效果是修改节点再插入新的生成的内容后，之前的批注仍然保留，因此需要加入参考backend\nodes\common_word_nodes\get_comments.py这个节点，参考里面的方法重新写一个同样的节点，叫get_rewrite_comments，用这个节点在修改前提取需要修改文档中的锚点之间的批注，然后存入到同样polished_comments状态中，进行复用，然后在插入文档节点后再根据对应的批注字典重写写入批注，当然如果因为修改后的内容导致批注找不到了，就按当前的逻辑是会显示添加失败这是正确的逻辑，

### 探讨提取并回填 Word XML 修订属性
标签：#Word文档 #XML #删除线 #字体颜色 #相似度回填

如图，在edit这个skill中的图中，包括涉及的相关节点，不知道能否有技术做到提取word锚点中xml的内容，我记得xml内容包含了这个字段的各种属性，是否可以在helper中增加一个业务功能相关的函数，专门用于提取在锚点之间文本的xml的属性，比如是否有删除线，比如该文字的字体颜色，这个函数可以放在extract_edit_context中提取文字的时候一起调用，同时还可以在helper中增加一个函数，把上面提取的这些xml元素，回填到当前的word中，这个函数可以在dispatch_tender_aware_update_word中导入，目的是把这些xml的元素回填到ai生成的内容中，当然ai生成的内容会有一些不同，比如ai会修改内容中的序号之类的，因此回填不是机械的一定要内容一致才把这个xml元素回填到对应的内容中，可以设置一个相似度，比如相似度在90%就可以把相似的内容进行回填，比如”1、标注“★”的项目为关键技术参数。“这句话有删除线和字体为红色， 如果ai生成的内容是“2、标注“★”的项目为关键技术参数。”，这个相似度肯定很高，那么是否可以做到在word中将“2、标注“★”的项目为关键技术参数。”这段增加删除线和字体变红，我不清楚有没有这样的技术实现，需要和你探讨

## 文档插入与回填

### 创建 update_gjgk_word 并保证同页插入与表格顺序
标签：#Word文档 #gjgk #update_word #表格 #测试

请你参考backend\nodes\common_word_nodes\update_word.py的代码逻辑，创建一个update_gjgk_word.py用来专门处理gjgk类型的文件，测试可以参直接在update_gjgk_word.py构建main函数，写一个main我可以直接运行测试，测试文件路径是backend\test_doc\254DSITC2512-招标文件-发售稿-财政模板.doc，然后需要插入的文本内容是“第1包：细胞电转仪
一、项目概述
1、设备名称及数量：
| 序号 | 设备名称 | 数量 | 是否按照医疗器械管理 |
| --- | --- | --- | --- |
| 1 | 细胞电转仪 | 壹套 | 是 |
2、交付日期：合同签订后30天内
3、交付地点：采购人指定地点
4、付款方式：货到验收合格（出具合同验收单或验收报告）且采购人收到其发票后三个月内，支付全部货款（100%）。
二、技术需求
须提供详细技术需求。
三、售后要求
1、★质保期：验收合格后整机免费质保≥3年。
2、★售后服务：提供报价设备均需提供原厂（制造商）售后，并出具相关证明文件。
3、医疗设备必须符合 IHE 医疗信息系统集成规范，并免费提供信息系统接口，医学影像设备须提供 DICOM软硬件接口，数字化医疗设备须提供HL7软硬件接口，并由供应商承担相应信息系统联机费用。
四、每套配置要求
| 序号 | 内容 | 数量 |
| --- | --- | --- |
| 1 | 主机 | 1台 |
| 2 | 腔室数 | ≥6个 |
| 3 | 温度控制 | 1个 |
| 4 | 软件 | 1套 |
| 5 | 校正 | 1套 |
注：供应商按上述配置要求自行提供响应设备的配置清单。
”我希望你仔细参考backend\nodes\common_word_nodes\update_word.py节点是怎么插入内容，你只需要注意插入内容是和锚点在同一页，并且插入内容是没有加密保护的内容，可以直接插入，就这么简单的逻辑，你要注意容易出现的两个bug，首先是插入的文本不应该是在前置锚点的下一页，这里需要应该就接着前置锚点后的第一个可插入位置插入文本，然后就是生成的表格和文本内容顺序乱套以及插入的会每行都换行

### 仅针对 gjgk 修复 update_word 每行换行问题
标签：#Word文档 #gjgk #update_word #换行 #修复

请你修改backend\nodes\common_word_nodes\update_word.py的代码逻辑，如图是我运行main函数后update_word后的结果，首先我不理解为上面会每行都换行，因为我自己用word使用插入一个长的文本直接就能插入，后面的锚点会自动往后瞬移到后面的页码，这是一个很自然的过程，我要提醒你一点，上面的改动应该只针对gjgk这个类型，其他类型的代码逻辑应该不变还是之前的逻辑，我觉得逻辑应该很简单，如果你觉得update_word需要兼容之前的代码，可以删除gjjgk的代码，创建一个update_gjgk_word.py用来专门处理gjgk类型的文件，测试可以参考update_word的main函数，

### 新增国内公开服务自筹专属删除与更新节点
标签：#Word文档 #后端 #国内公开 #服务自筹 #保护字段

我现在需要增加新的后端分支，当前这个后端的graph只是在国内公开这个类型中生效，当用户在国内公开选择服务，自筹这个类型时，在后端走的是backend\graphs\gngk_fw_zc_tender_graph.py，需要修改的注意是这个graph中的节点，这个图目前导入的节点逻辑基本不变，需要修改的节点主要是替换调共享的节点，改成国内公开服务自筹的特有节点，即在backend\nodes\gngk_word_nodes中新增多个节点文件，在backend\nodes\gngk_word_nodes中首先需要修改backend\nodes\gngk_word_nodes\gngk_get_replacements.py的名称，改成gngk_hw_zc_get_replacements，同时创建一个gngk_fw_zc_get_replacements，这个文件代码逻辑和gngk_hw_zc_get_replacements目前一致即可，还有需要新增的节点文件有gngk_fw_zc_delete_tender_param、gngk_fw_zc_update_word，需要重点注意是这两个新增的节点文件完全可以参考backend\nodes\common_word_nodes\delete_tender_param.py和backend\nodes\common_word_nodes\update_word.py，因为基本逻辑应该是一模一样的，但是国内公开服务自筹的加密保护的内容由两个变成了三个，分别是"服务地点："、"服务期限："、"付款方式："，请你根据这个点进行参考修改，创建新的节点文件

### 修复服务自筹删除更新节点的保护字段与空白页问题
标签：#Word文档 #后端 #服务自筹 #保护字段 #空白页

在国内公开中选择服务自筹路由到graph，出现了很大的问题，肯定是backend\nodes\gngk_word_nodes\gngk_fw_zc_delete_tender_param.py和backend\nodes\gngk_word_nodes\gngk_fw_zc_update_word.py这两个节点的问题，首先如图，我怀疑在删除节点就没有把这三个保护字段的可编辑部分删除干净，然后就是ai生成的内容并不像这个backend\nodes\common_word_nodes\update_word.py一样根据这三个字段分块插入到这三个保护字段的区间中，其次第二个问题就是锚点结束字段后面的内容应该如图所示，但是当前出现的bug是在第四章 合同条款后直接出现了一个空白页，后面的内容到了下一页，最后当年修改好代码后再帮我直接写一个测试直接在backend\nodes\gngk_word_nodes\gngk_fw_zc_update_word.py构建main函数，写一个main我可以直接运行测试，测试文件路径是backend\test_doc\复旦大学附属华山医院虹桥院区VRV空调和分体式空调维护保养251498-招标文件-发售稿.doc，然后需要插入的文本内容是"一、项目概述
1、项目名称：复旦大学附属华山医院院本部1号楼急诊改扩建区域风机盘管空气过滤器更换及设备保养
2、服务地点：复旦大学附属华山医院院本部1号楼急诊改扩建区域
3、服务期限：2026年6月16日-2029年6月15日
4、付款方式：
11)前三季度维保费用的支付方式遵循季度结算原则。即甲方需按每季度（每三个月）一次的频率向乙方支付相应的维保服务费用。
2)最后一季度付款：年度维保工作结束后第三个月，经过甲方的考核评估，甲方向乙方支付相应的维保服务费用。
3)每次付款前，乙方的服务质量须经过甲方的考核评估。只有在满足合同约定的服务标准时，甲方方才承担支付当期维保费用的责任。
4)在考核过程中，若发现乙方存在违约或未达到约定服务标准的情形，甲方有权根据合同中规定的考核说明，对应付款项进行相应扣减。
5)扣款的具体数额及条件应详细列明于合同中的《维保项目管理考核表》章节中的考核说明，并以此为依据执行。任何扣款必须符合该等条款的规定，并且乙方应提前得到通知，明确扣款的原因及金额。
6)所有扣款处理应在当季度维保费用支付前完成，以确保支付金额的准确性。
7)甲方应于每次考核合格确认后的30个工作日内，将扣除相应扣款后的维保费用支付给乙方。
四、服务内容及要求
1、投标人需提供具体维保方案、维修方案、应急预案
2、基础维保内容包括但不限于：
| 序号 | 检 查 项 目 |
| --- | --- |
| 01 | 检查或清洗室内机盘管翅片 |
| 02 | 测量风速、温度 |
| 03 | 检查空调电脑板、除尘 |
| 04 | 检查各接线柱螺栓是否收紧 |
| 05 | 检查各接线插片、插头、插座是否牢固 |
| 06 | 检查空调机组运行高、低压力 |
| 07 | 检测控制箱，除垢处理 |
| 08 | 检查空调机组的运行电流 |
| 09 | 检查风机盘管的电动执行阀机构状况 |
| 10 | 检查风机盘管的水过滤器通过性 |
| 11 | 检查风机盘管的冷凝水排水是否畅通 |
| 12 | 检插或更换空调各易损件 |
| 13 | 温度传感器性能定期巡检及功能检查 |
| 14 | 检查线控器、遥控器 |"
我希望你仔细参考backend\nodes\common_word_nodes\delete_tender_param.py和backend\nodes\common_word_nodes\update_word.py代码逻辑

### 在 main 中串联删除节点和更新节点复现并修复空白页 bug
标签：#Word文档 #后端 #服务自筹 #测试 #修复

 能否修改main，加上backend\nodes\gngk_word_nodes\gngk_fw_zc_delete_tender_param.py的逻辑，即word先经过backend\nodes\gngk_word_nodes\gngk_fw_zc_delete_tender_param.py然后再经过backend\nodes\gngk_word_nodes\gngk_fw_zc_update_word.py，这也是当前出了bug的地方，如图一和图二，这分别是运行backend\nodes\gngk_word_nodes\gngk_fw_zc_update_word.py的main函数和直接e2e测试经过完整的backend\graphs\gngk_fw_zc_tender_graph.py的流程得到word内容，首先是锚点结束字段后面的内容两边效果不一样，e2e的结果锚点结束是第四章合同条款，后面直接出现了一个空白页，后面的内容到了下一页，正常应该是图一的结果才对，因此我需要你修改main，测试先经过删除节点再经过更新节点才能知道问题出在哪里，并且请你修复我上面的说经过删除节点后出现的bug
 

### 修复 edit 生成日志缺失与保护字段模糊匹配漏洞
标签：#Word文档 #后端 #edit #日志 #保护字段

 现在有两个bug需要修复，首先是触发edit这个skill时，ai生成的内容和提示词没有记录到backend\prompts_log\generate_log中，第二个当前ai生成的部分内容是“一、项目概述
1、设备名称及数量：大功率电场发生装置等设备/壹批
| 序号 | 产品名称 | 数量 | 交付日期 |
| --- | --- | --- | --- |
| （一） | 大功率电场发生装置 | 壹套 | 合同签订后60日内 |
| （二） | 直流电流负载 | 贰台 | 合同签订后120日内 |
| （三） | 精密测量直流电源 | 陆台 | 合同签订后120日内 |★供应商请按上述设备明确分项报价（共3项），并包含于响应总价中。
2、交付日期：详见上表。
3、交付地点：采购人指定地点。
4、付款方式：分期付款：第一笔付款-预付款（60%）：合同签订后支付预付款；第二笔付款-付款(30%)：货物完全到达买方指定地点后支付第二笔合同款；第三笔付款-最终验收付款(10%)：货物安装、调试、计量、验收合格后，采购人于最终验收合格之日起 30 日内支付剩余合同款。”
但是在回填时，代码把表格中的“交付日期”错误的匹配到了密码保护的字段即“交付日期：”，注意密码保护的字段是“交付日期：”，但是代码却把交付日期匹配过去了，如图所示，把表格中的交付日期匹配了密码保护字段，这是明显的错误问题，请你仔细审查当前所有类型的节点，在回填ai生成的内容时候，处理密码保护的字段是否有这种模糊匹配的错误和漏洞

## 上传文件修改流程

### 新增上传文件修改 edit skill 与文档回填流程
标签：#Word文档 #前端 #后端 #edit #上传文件

我需要新增一个功能，首先是前端设计，如图一能否修改我当前聊天输入框，如图二在模型选择旁边增加一个加号，点击加号出现功能选择，当前只设计一个功能即上传文件修改，用户点击该功能应该弹出文件夹可以选择文件上传，点击文件后即出现类似图三的样式，然后就是后端功能的设计，在调用后端时，首先要传的内容肯定是根据当前用户在前端页面选择的三个类型之一，以及自筹或者财政，货物或者服务，以及高级设置里的前后锚点内容，我拿国内公开的前端做举例，当在国内公开类型中选择自筹货物时，上传了需要修改的文件，调用后端的逻辑需要参考backend\skills\rewrite的逻辑，新增一个skill，该skill命名为edit，我大致说一下后端核心逻辑设计，你不清楚可以询问我进行补充，参考rewrite肯定需要一个graph，该graph第一个节点就是选择文件作为模板，该文件其实就是用户上传的文件，然后接下去一个节点是否可以参考backend\nodes\common_word_nodes\extract_tender_params.py，创建一个在锚点之间即提取内容又提取批注的节点，提取出来的内容仍然放到状态中的origin_tender_params，提取出来的批注则放到状态中的polished_comments，然后用接下来的节点顺序依次就是删除锚点之间的内容，ai生成修改后的内容，回填更新到锚点之间，其中删除节点和更新节点需要根据类型、自筹财政、货物服务去路由不同的delete节点和update节点，目前已知xjcg和gngk的自筹货物都路由到相同的delete_tender_param和update_word，而gjgk的自筹货物和财政货物都路由到backend\nodes\gjgk_word_nodes中删除和更新节点，其他我没有提交的组合都是默认delete_tender_param和update_word节点，ai生成修改后的内容节点可以参考rewrite_text，命名edit_text，该节点的用户提示词则是用户输入的修改要求以及锚点中提取的origin_tender_params，系统提示词则是要引导ai根据用户的修改要求和origin_tender_params生成符合的内容，然后就是更新节点回填到用户上传的文本，前端生成可以下载的文件，整个前后端流程基本可以参考rewrite的逻辑，前端也需要用卡片显示进度和ai流式生成的内容，

## Word 通用 Helper

### 抽取 Word 删除插入与批注能力到 helper 小包
标签：#Word文档 #后端 #Helper #批注 #重构

请你仔细审查backend\nodes\common_word_nodes\delete_tender_param.py、backend\nodes\common_word_nodes\update_word.py、backend\nodes\gjgk_word_nodes\gjgk_delete_tender_param.py、backend\nodes\gjgk_word_nodes\gjgk_update_word.py、backend\nodes\gngk_word_nodes\gngk_fw_zc_delete_tender_param.py、backend\nodes\gngk_word_nodes\gngk_fw_zc_update_word.py这些文件的代码逻辑，尤其是在寻找锚点，删除锚点之间的内容和在锚点之间插入内容的代码逻辑， 是否有相同重复的代理逻辑可以提取到共享的helper中，helper就直接创建在backend\helper中，然后在helper下新建 word_helper/ 小包，而不是单文件，更加功能语义拆分，保留 comment 无关的通用 Word 能力，也同时请你仔细审查comment_writeback 这一类的文件的代码逻辑，是否也能逐步并入这个helper 收口体系，如果可以也做成comment_helper/ 小包，承接 Word 侧 comment 能力

