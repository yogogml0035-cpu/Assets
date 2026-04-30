# 已看 区分agent framework与agent harness

## 视频来源

https://www.bilibili.com/video/BV1Mzwtz7EYQ?p=1

![](https://pic.aihaoji.com/user_3412cc21-72bc-d9dd-d569-1069dd8b6c83/img/20260314/d9d431c0-2ffe-adb2-c984-0f30c051b02a/9b3d1e37-599c-434f-b590-99f9d1fb5b5d.webp)

很多人学AI开发的第一步是去学**LangChain**，学完之后发现还有**LangGraph**，学完了又出来一个**DeepAgents**，就像打地鼠层出不穷。但真正重要的问题，根本不在这里。没有人告诉你，这些工具压根不是同一类东西。有的是**Framework**，有的是**Harness**，搞不清楚这个区别，你永远在追工具，永远追不完。

今天我们精读前**crewAI**工程师的一篇短评，帮你建立一个更底层的认知。以后看到任何Agent工具，都能一眼判断它是什么。先画一条光谱，最左边是没有任何封装的纯代码，直接调大模型API手动管理所有状态。



![](https://pic.aihaoji.com/user_3412cc21-72bc-d9dd-d569-1069dd8b6c83/img/20260314/d9d431c0-2ffe-adb2-c984-0f30c051b02a/9ce33e33-4d04-49b2-affc-7675247294d9.webp)

灵活性最大，但所有麻烦都得自己扛：断网重连、上下文截段全靠自己写。**光谱中间是Agent Frameworks**——LangChain、crewAI都是这一类。Framework给你封装好的组件工具接口、任务调度、角色分工，帮你屏蔽底层通信的麻烦。但系统怎么设计还是你说了算，用什么模型、怎么存记忆全由你决定。就像去宜家买家具，板材和螺丝钉都给你了，但拼成什么样子是你的事儿。Framework是给喜欢自己搭系统的工程师准备的**。光谱最右边是Agent Harnesses**，它不给你零件，直接给你一套完整的系统。最近爆火的OpenClaw就是典型：填1个API Key，它就能直接跑。怎么存、出错怎么重试全都替你决定好了，就像买了一辆原厂精调好的跑车，给油就走。代价是你不能动它的内部逻辑。Harness是用控制权换上限速度。



![](https://pic.aihaoji.com/user_3412cc21-72bc-d9dd-d569-1069dd8b6c83/img/20260314/d9d431c0-2ffe-adb2-c984-0f30c051b02a/d70dcffd-aabf-4fd3-a42b-72954139582c.webp)

有意思的是，**LangChain**自己也在向右扩张。他们把技术栈分成了三层：最底层的LangChain是Framework层，中间的LangGraph是执行引擎，负责状态管理和持久化。最外层的Deep Agents就是一个开箱即用的Harness。一家公司把整条光谱都吃下来了。你现在追的Agent Frameworks，其实是LangChain Agent生态里的Harness层。

很多人会有一个误区：Harness是开源的，我去改改底层代码，它不就变成Framework了吗？理论上可以，但代价很大。Framework本来就留着插孔让你接组件，而Harness的内部是高度耦合的。改起来就像拿电锯拆承重墙，一旦遇到企业内网特殊鉴权这类需求，很容易在生产环境崩掉。这时候不如直接退回Framework层，从核心组件重新拼装。



![](https://pic.aihaoji.com/user_3412cc21-72bc-d9dd-d569-1069dd8b6c83/img/20260314/d9d431c0-2ffe-adb2-c984-0f30c051b02a/5c691540-b5a4-4fcd-a55d-5165ac3a1990.webp)

所以选工具之前先问自己一个问题：你是要长期掌控每一行代码，还是要赶进度，拿现成套件直接交付？答案决定了你站在光谱的哪个位置。

最后说一个很多工程师忽略的真相。**Anthropic在《Building Effective Agents》里明确说过，做Agent要慎用复杂框架**。框架的黑盒一旦出错，调试会很痛苦，而且调用太方便容易让团队陷入过度工程化。很多时候直接写几行代码，连大模型API反而更快更稳。

好了，本期就到这里，这里是**慢学AI**，我们下期见。



