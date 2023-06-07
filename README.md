基于代码链路分析的精准测试体系
===============

[软件质量报道](javascript:void(0);)

**软件质量报道** 

微信号 QualityReport

功能介绍 本公众号致力于健康、安全、绿色的软件生态，分享软件质量管理、软件测试的思想、方法、技术与优秀实践，追踪软件质量领域的热点，及时报道软件质量管理的成功案例或质量事故，以及分享深度思考、有温度的技术文章等，努力成为您工作中的朋友。

_2023-02-16 08:16_ _发表于上海_

收录于合集

以下文章来源于阿里巴巴技术质量 ，作者李华伟(正辰)

[

![](http://wx.qlogo.cn/mmhead/Q3auHgzwzM73ibG4VJE6jSEDtDvR2E5IOURYGb8b9bOqD73FKFOX1gQ/0)

**阿里巴巴技术质量** .

阿里巴巴QA官方技术号，呈现阿里巴巴质量领域最新的技术发展和创新。

](#)

> 导读：随着业务规模的增长，服务端的架构和功能趋近复杂化，代码变更上线也越来越快，面对需要快速上线的功能，测试资源无法参与每个变更的交付过程，**变更要不要测、要怎么测？**成为服务端交付质量和效率最关键问题，为此，优酷质量保障团队开始了服务端精准测试的探索，经过半年的技术沉淀和体系搭建，形成了具备**变更内容识别、变更影响分析、测试能力推荐、测试覆盖评估**的精准测试体系。

**1\. "要不要测、要怎么测？"引发的思考**
-------------------------

    随着优酷业务规模的增长，服务端的架构和功能趋近复杂化，代码变更上线也越来越快，面对需要快速上线的功能，测试资源无法参与每个变更的交付过程，**变更要不要测、要怎么测？**成为服务端交付质量和效率最关键问题：

  

**变更要不要测试？**

*   这次变更代码有多少是空行、多少是注释、多少是有效代码，是不是对原有代码逻辑产生了影响？
    
*   这次变更方法是新增还是修改，变更方法的圈复杂度高不高、代码耦合度高不高，是不是线上热度调用方法？
    
      
    

**变更要怎么测试？**

*   这次变更影响了哪些业务逻辑、代码链路，是否要做接口全量回归测试？
    
*   要能完整覆盖这次变更影响的业务逻辑、代码链路，需要执行哪些测试用例？
    
      
    

    面对这些问题，仅靠开发的提测文档和常规的代码分析，很难给出准确答案。为此，优酷质量保障团队开始了服务端精准测试的探索，经过半年的技术沉淀和体系搭建，形成了具备**变更内容识别、变更影响分析、测试能力推荐、测试覆盖评估**的精准测试体系。

**2\. 基于代码链路分析的精准测试体系**
-----------------------

    精准测试本质上是一种基于变更分析的测试决策和评估的过程，要做好这个过程中，需要能准确的回答下面这个问题：

**”****变更了什么内容、产生了什么影响、需要做哪些测试、变更覆盖够不够？****“**

从问题可以看到，变更内容、影响对象的定义是精准测试体系的基石，直接决定需要识别的变更对象和需要评估的影响对象，并基于识别、评估结论推荐相关测试用例，以及最终评估测试活动的覆盖率。通过参考业界已有方案，也结合优酷的业务特性，我们最终确定了优酷精准测试体系的探索方向：基于代码调用链路分析的精准测试体系。用一句话描述：

> 以应用Java方法为观测对象，通过静态分析识别变更的Java方法，通过动态采集获取线上Java方法调用链路，然后基于代码知识库的方法匹配，精准分析变更影响的Java方法调用链路，并基于影响的链路推荐测试流量，评估测试覆盖率的测试体系

    在此，特别说明一下应用Java方法调用链路的定义：**是指业务流量在应用内部处理过程中，产生的完整Java方法调用栈**。我们认为，相比外部子调用链路，Java方法调用链路更能代表业务场景和代码逻辑，作为变更影响、测试覆盖率评估对象，具有更高的业务价值。

### **2.1 实现方案**

    为了实现通过变更内容识别变更方法，通过变更方法分析影响代码链路，通过影响的代码链路推业务流量作为回归测试用例，既需要静态代码分析能力，能准确识别和分析变更内容。还需要能动态采集业务流量的请求参数、返回结果，代码调用链路。然后以方法为核心对象，准确建立**变更内容、代码方法、调用链路、业务入口、业务流量**的关联关系：

![图片](https://mmbiz.qpic.cn/mmbiz_png/DWQ5ap0dyHOS30cibgdoxNVC0paGqqUP4ruzZDLXgFeIGkDWxaOoVFYIT635aBHt9v84tMNnV5UMFobiaDAg7iaVg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

    基于不重复造轮子的原则，我们选择通过蚂蚁的Sparrow做静态代码分析，通过集团的Sandbox做动态流量采集，在此基础上通过构建数据中心、决策引擎实现精准测试体系的支撑能力，并且通过平台化提供给业务团队接入使用。方案的整体架构如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/DWQ5ap0dyHPL9WVVHpS11vBIgYzw5XaicBF8aTSGLph8yywwdbuS6OqGxEYkXu4Eib8CrhoE59gzmrtOD3OaoDaw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

###   

### **2.2 数据中心**

    主要是对静态代码分析数据、动态流量采集数据做结构化存储，包括主干代码知识库、变更代码知识库、流量知识库。

  

*   **主干代码知识库**
    

    对应用主干代码做静态分析，获取应用全量方法的结构化信息，包括：方法签名、方法修饰、方法注解、入参类型、返回类型、圈复杂度、耦合度。主干代码知识库定义了精准测试体系需要的全部观测对象（应用代码的Java方法），将作为后续变更方法匹配、变更方法风险评估、代码链路采集方法筛选的统一知识库

![图片](https://mmbiz.qpic.cn/mmbiz_png/DWQ5ap0dyHOS30cibgdoxNVC0paGqqUP4KpQ2e2YERoPxJSnc32MeDRia2Iu8OR5PZ2FhOibHVgowmz6JiaicW3OkXw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

*   **变更代码知识库**
    

    对变更分支代码做静态分析，获取变更代码的结构化信息，包括：变更文件、变更方法、变更代码语义。首先通过变更代码语义分析，能准确识别变更代码行是空行、注释、日志打印、逻辑判断，还是变量操作，从而计算有效变更代码行数。然后通过匹配主干代码知识库，获取变更方法在代码知识库的唯一ID，为后续变更影响分析和变更风险评估建立方法关联关系

![图片](https://mmbiz.qpic.cn/mmbiz_png/DWQ5ap0dyHOS30cibgdoxNVC0paGqqUP459zqjNkOqNZKrKknUuK2CUTSyxncdiceqSEQA1wuXQ5IZLPKWRLKEYw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

*   **流量知识库**
    

    动态采集业务入口的线上实时流量、线下测试流量，包括请求参数、返回结果、代码调用链路。为了获取业务入口完整的代码调用链路，需要给Sandbox配置需要采集的Java方法，为此，我们首先定义了应用核心方法模型，然后通过匹配主干代码知识库，可以实时分析出需要采集的核心方法，最后通过Sandbox对这些方法插桩和采集，从而获取到流量进入应用内部后的完整代码调用链路，如下图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/DWQ5ap0dyHOS30cibgdoxNVC0paGqqUP4Hmtk8o4KswVDHBcO1k3ezicjcPibZBuHibItdsLaD4OuJK4vibnC4wgFUw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

###   

### **2.3 决策引擎**

    决策引擎对代码知识库、流量知识库的做变更方法匹配和链路聚合分析，从而提供变更方法风险决策、变更链路测试用例推荐、测试覆盖率评估能力

*   **代码链路分析**
    

    通过聚合分析应用过去一周的线上流量，可以获取应用的全部代码调用链路，然后通过匹配主干代码知识库，获取链路上每个方法的知识库ID，最后以图形结构（点、边）对调用链路做结构化存储，从而可以实时计算每条链路的长度、深度、热度，以及从应用、入口、链路等维度，计算方法热度、调用热度

![图片](https://mmbiz.qpic.cn/mmbiz_png/DWQ5ap0dyHOS30cibgdoxNVC0paGqqUP4uHfw43amy6ACnqBUaVjznQPPQbB7j3FQgsfgQSRZH7Xp1Twce2bSzA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

*   **变更风险决策**
    

    以变更方法为分析对象，基于代码知识库，可以获取变更方法是否为有效代码变更，以及变更方法的圈复杂度、代码耦合度等静态分析数据。再基于代码链路分析结果，可以实时获取变更方法影响的代码链路、链路热度、方法热度、调用热度等动态采集数据。从而可以实现精准评估变更方法的风险等级和实际影响对象。为此，我们建立了以下变更风险评估模型：

![图片](https://mmbiz.qpic.cn/mmbiz_png/DWQ5ap0dyHOS30cibgdoxNVC0paGqqUP4e8mlbp5YNdvlYibkiajeoqBLoo593qroGLrCEwcZqNnM6kgIaPH0icraA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

其中：M(r)为方法变更风险值、M(c)为方法圈复杂度、M(e)为方法代码耦合度、M(c,h)为方法影响链路的热度、M(h)为变更方法热度

  

*   **测试用例推荐**
    

    基于线上代码链路聚合分析结果，可以建立业务流量和代码调用链路的关联关系，再基于变更方法影响的代码调用链路，就可以分析出变更影响链路对应的全部业务流量，从而实现基于代码链路影响的测试用例推荐

![图片](https://mmbiz.qpic.cn/mmbiz_png/DWQ5ap0dyHPL9WVVHpS11vBIgYzw5XaicwszS7JIGHK65GZib90O9Z6gy4gckOZBwEOib8LkFxG39d53bibCadT8cQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

（如上图所示，将为代码变更1推荐业务流量1、业务流量2，这些业务流量会被自动推荐给智能回放、自动化测试等回归测试任务）

### **2.4 平台能力**

    借助优酷服务端研发效能平台，我们将精准测试体系完备的基础能力和海量数据化繁为简，可以为业务同学提供更直观的变更分析、场景分析、链路分析，以及通过智能回放、自动化测试提供更高效精准的回归测试能力。以下对部分核心功能做简单说明和展示：

*   **变更分析**
    

对Aone线下部署流程的每次构建做增量代码分析，提供代码变更内容、代码变更影响面的展示和分析

![图片](https://mmbiz.qpic.cn/mmbiz_png/DWQ5ap0dyHOS30cibgdoxNVC0paGqqUP4dtw5ib4Nmg996rtBKRVLUFLAgcLNKSficu5VuXUAtZzkfTUpV0yA3yjw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)   ![图片](https://mmbiz.qpic.cn/mmbiz_png/DWQ5ap0dyHOS30cibgdoxNVC0paGqqUP4LvWzGQ0MORDwfVtj1REtffKeSF6vDNib80DmkLIDCxRbL7zONnuz3vA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)![图片](https://mmbiz.qpic.cn/mmbiz_png/DWQ5ap0dyHOS30cibgdoxNVC0paGqqUP4ya5Jl8YPiaetD9hMuu26icrP3pGEBC0bhaxk4nGWjPulBEFuicGZcHGuw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

*   **场景分析**
    

通过对业务入口的请求参数、返回结果建立业务特征，可以对采集到的全部业务流量做聚合分析，提供业务场景的特征组合结果、热度的展示和分析

![图片](https://mmbiz.qpic.cn/mmbiz_png/DWQ5ap0dyHOS30cibgdoxNVC0paGqqUP4XSU336Ck0OaBHSyWObS5ticVib3piayw2kMY528TaTUibamj7VmMeqgGTw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) ![图片](https://mmbiz.qpic.cn/mmbiz_png/DWQ5ap0dyHOS30cibgdoxNVC0paGqqUP4EzpibXpfLSnLTbGo6s2w9Ynl2fllHyEicEhD0iaIrdZtKPYfzUpMdqDgw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

*   **链路分析**
    

通过建立代码调用链路特征标识，可以对采集到的全部代码调用链路做聚合分析，提供代码调用链路热度、完整调用栈的展示和分析

![图片](https://mmbiz.qpic.cn/mmbiz_png/DWQ5ap0dyHOS30cibgdoxNVC0paGqqUP4vkicDuibY7vowLG9Bqqppr15icvoqS1YRaqbsVVGPFiccXhtUsKfu9Yz4A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)![图片](https://mmbiz.qpic.cn/mmbiz_png/DWQ5ap0dyHOS30cibgdoxNVC0paGqqUP4YIMjicvLlesicbKgyNtdicEAAJGXGHNIqibpkfC3aw50ib9gW7ZCEg5nlbA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

*   **智能回放、自动化测试**
    

将变更方法影响链路的业务流量推荐给智能回放、自动化测试等回归测试任务，提供精准回归测试能力，实现变更链路100%测试覆盖

![图片](https://mmbiz.qpic.cn/mmbiz_png/DWQ5ap0dyHOS30cibgdoxNVC0paGqqUP4fdK1sRs2FuIanVdSkJKIwY3909XG7dhHd2BOIuKx9iavY8ibhSaxMadA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1) ![图片](https://mmbiz.qpic.cn/mmbiz_png/DWQ5ap0dyHOS30cibgdoxNVC0paGqqUP4ZHLBia7Uia5HbkP8FO9NUUM3HEibYSBaiaicrlP662QQCcoUByibMzoewxJw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

**3\. 核心能力解决方案**
----------------

    借助Sandbox、Sparrow提供的基础能力，虽然可以快速实现静态代码分析和动态流量采集，但在实际落地过程中，还是遇到了很多基础能力之外的技术挑战，比如：

*   由于采集机异常下线、采集限流等问题，导致采集流量无法有效覆盖线上实际流量模型，导致应用代码调用链路不完备
    
*   由于人工配置采集方法的差异性，导致无业务意义方法(get\\set等)被采集，而关键链路上的方法没有采集，导致代码调用链路不完整
    
*   静态代码分析可以获取变更文件、类、方法、代码行信息，但不知道变更代码语义，无法准确识别变更有效性，导致变更分析不准确
    

    为此，在Sparrow、Sandbox基础能力之上，优酷自研了**流量采集智能调度、应用核心方法自动解析、变更代码语义精准识别**等创新解决方案，不仅保障了代码调用链路采集完备性、提升了变更方法分析准确性，也实现了在无需人工介入的情况下，优酷服务端大规模接入精准测试体系。

### **3.1 代码调用链路采集完备性**

    要获取应用完备的代码调用链路，需要具备2个前提：采集流量有效覆盖7\*24小时全部时间段、采集方法有效覆盖应用全部核心方法。

*   **流量采集智能调度**
    

    要能采集应用线上全部代码调用链路，就要保障采集流量的完备性，采集流量能有效覆盖业务全天流量模型。为此，我们首先构建了采集模块自动运维的能力，应用接入火影后，平台会实时监测采集机状态，一旦发现采集模块异常下线（应用弹性缩容、机器异常等原因），会再实时激活一台采集机，从而保障采集模块7\*24小时在线。然后还构建了采集配置智能调度能力，我们将一天分为48个采集调度窗口，每个调度窗口会根据入口当天入库流量数、当前窗口入库流量数，自动调整下个采集窗口的采集白名单和采样率，从而不仅可以保障采集流量能覆盖全天各个时间段，也有效解决了高QPS接口采集过量，低QPS接口采集不到流量的情况。整个过程完全由平台自动调度，不仅极大降低了业务团队采集模块、采集配置的维护成本，也提升了采集效率和质量。

![图片](https://mmbiz.qpic.cn/mmbiz_png/DWQ5ap0dyHPL9WVVHpS11vBIgYzw5XaicEGxhk7aKwibAcwu5D9jt7Zb2MJQILcSW04WM7jdU19HwZZyBn56KibRQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

*   **应用核心方法自动解析**
    

    Sandbox采集代码调用链路，需要配置采集方法列表，这个对业务团队接入提出了很大挑战，配置方法太多，会导致采集链路过于复杂、无业务意义的链路噪音比较多，极大增加了变更分析成本。配置方法太少，会导致采集链路过于简单，无法体现完备的代码逻辑和业务场景。这些问题都让精准测试体系变的不那么”精、准“。为此，我们通过静态代码分析，构建了应用代码全量方法知识库，结构化存储了类、方法的全部基础信息，然后通过建立核心方法模型：接口类的实现方法 + 圈复杂度大于5的方法 + 代码耦合度大于0方法，实现了应用核心方法自动解析能力。相比人工配置采集方法，代码调用链路完备性提升了50%（基于链路深度、链路长度做对比），变更方法采集率从30%提升到90%，保障了绝大多数变更方法都能分析出链路影响面。

### **3.2 变更代码语义精准识别**

    传统静态代码分析可以知道变更文件、类、方法、代码行等信息，基于这些信息分析出来的变更方法通常会比较多，但可能变更方法只是加了一些注解，或者减少了一些空行，这些方法变更并不影响代码业务逻辑，并不需要测试关注。因此，要能准确识别变更有效性和风险，除了需要知道变更多了行，还需要知道变更代码什么内容。为此我们借助Sparrow变更代码语义分析能力，通过获取变更代码完整的ast语义结构，能有效识别变更代码是否是空行、日志、注释、变量操作、逻辑判断等，从而实现了变更方法有效性评估。相比传统静态代码分析结果，有效变更方法数量降低了30%，同时也避免了无效代码改动对变更分析准确性的影响。

**4\. 落地效果和未来展望**
-----------------

### **4.1 落地效果**

    通过升级优酷应用部署流程，可以支持在"测试准入"阶段对构建产物实时分析，在”提交测试“阶段提供变更分析结果和测试用例推荐，从而在无需人工介入的情况下，实现应用无感接入优酷精准测试体系。

![图片](https://mmbiz.qpic.cn/mmbiz_png/DWQ5ap0dyHOS30cibgdoxNVC0paGqqUP4Prw63XFpb8BBgcmpQ8WmZKe35OkrwQhN1MkiauSYk3stsYzYib2C7XpQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

并且对接了服务端提测流程，在提测单中可以直观的看到提测变更的分析结论：

![图片](https://mmbiz.qpic.cn/mmbiz_png/DWQ5ap0dyHOS30cibgdoxNVC0paGqqUP4KlMzY9cNDOE0xqIr6TgWzH9RtHetiaLmm1AbpCibNkAtIibibg3JH7VDcQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  
