### HarmonyOS调研报告

#### 目的

了解鸿蒙系统的相关知识, 评估是否能对项目或团队带来收益

#### 什么是鸿蒙

> HarmonyOS是一款“面向未来”、面向全场景（移动办公、运动健康、社交通信、媒体娱乐等）的分布式操作系统。在传统的单设备系统能力的基础上，HarmonyOS提出了基于同一套系统能力、适配多种终端形态的分布式理念，能够支持手机、平板、智能穿戴、智慧屏、车机等多种终端设备。
> 
> - 对消费者而言，HarmonyOS能够将生活场景中的各类终端进行能力整合，可以实现不同的终端设备之间的快速连接、能力互助、资源共享，匹配合适的设备、提供流畅的全场景体验。
> - <u>**对应用开发者而言，HarmonyOS采用了多种分布式技术，使得应用程序的开发实现与不同终端设备的形态差异无关。这能够让开发者聚焦上层业务逻辑，更加便捷、高效地开发应用。**</u>
> - 对设备开发者而言，HarmonyOS采用了组件化的设计方案，可以根据设备的资源能力和业务特征进行灵活裁剪，满足不同形态的终端设备对于操作系统的要求。
> 
> [系统定位视频介绍](https://mos-vod-drcn.dbankcdn.cn/P_VT/video_injection/A91343E9D/v3/9AB0A7921049102362779584128/MP4Mix_H.264_1920x1080_6000_HEAAC1_PVC_NoCut.mp4)
> 
> [系统架构视频介绍](https://mos-vod-drcn.dbankcdn.cn/P_VT/video_injection/2A1343EFA/v3/6CC21C811065945606293295744/MP4Mix_H.264_1920x1080_6000_HEAAC1_PVC_NoCut.mp4)

#### 技术架构

HarmonyOS整体遵从分层设计，从下向上依次为：内核层、系统服务层、框架层和应用层。系统功能按照“系统 > 子系统 > 功能/模块”逐级展开，在多设备部署场景下，支持根据实际需求裁剪某些非必要的子系统或功能/模块。HarmonyOS技术架构如下所示。

![img](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20210108173212.37805437498232424696659721812732:50520108113129:2800:A8719795C992A08D5ED70A7C3570B2A3EDFD98928DD91964A819EDC754D07367.png?needInitFileName=true?needInitFileName=true)

#### 鸿蒙历程及路标

![img](https://raw.githubusercontent.com/erleizh/erleizh.github.io/master/screenshots/HarmonyOS调研报告/164200_d26618a1_8276414.png)

#### 关于OpenHarmony

[OpenHarmony](https://gitee.com/openharmony)是HarmonyOS的开源版，由华为捐赠给开放原子开源基金会（OpenAtom Foundation）开源。第一个开源版本支持在128KB~128MB的设备上运行，欢迎参加开源社区一起持续演进。目前源代码托管在 [gitee](https://gitee.com/)

OpenHarmony截止2021-03，共有137个仓库，包含内核，驱动，等等，大部分仓库都使用的是[Apache-2.0](https://gitee.com/openharmony/security/blob/master/LICENSE) 许可，详细信息请看文档[OpenHarmony开发者文档](https://gitee.com/openharmony/docs)

OpenHarmony 和 HarmonyOS 的关系类似于 AOSP 和 Android 的关系

####HarmonyOS如何兼容Android

目前官网上没有关于如何兼容android的说明，但是查看源码可知

![image20210308203714998](https://raw.githubusercontent.com/erleizh/erleizh.github.io/master/screenshots/HarmonyOS调研报告/image-20210308203714998.png)

鸿蒙针对Android的四大组件有相应的继承类，想必应该是做了一个兼容层

知乎上有相关的分析

[如何看待 9 月 10 日华为发布的鸿蒙 OS 2.0 系统，应用前景如何？ - 年轻人啊不要熬夜的回答](https://www.zhihu.com/question/420404904/answer/1466006612)

[我的鸿蒙 2.0 App 到底是什么](https://zhuanlan.zhihu.com/p/234397497)

[我的第一个鸿蒙app，以及所见内容](https://zhuanlan.zhihu.com/p/338663467)

####如何开发HarmonyOS App

1. 开发工具
  
  华为基于**IntelliJ IDEA**开发了 **DevEco-Studio** , 用于开发鸿蒙 app。构建系统使用gradle
  
2. app结构
  
  HarmonyOS的应用软件包以**APP** **Pack**（Application Package）形式发布，它是由一个或多个**HAP**（HarmonyOS Ability Package）以及描述每个HAP属性的pack.info组成。HAP是[Ability](https://developer.harmonyos.com/cn/docs/documentation/doc-guides/basic-fundamentals-0000000000041611#ZH-CN_TOPIC_0000001063248002__section121002188527)的部署包，HarmonyOS应用代码围绕Ability组件展开。
  
  一个HAP是由代码、资源、第三方库及应用配置文件组成的模块包，可分为entry和feature两种模块类型，如图所示。
  
  - **entry**：应用的主模块。一个APP中，对于同一设备类型必须有且只有一个entry类型的HAP，可独立安装运行。
  - **feature**：应用的动态特性模块。一个APP可以包含一个或多个feature类型的HAP，也可以不含。只有包含Ability的HAP才能够独立运行。
  
  ![结构图](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20201215223213.31985069518549702064004286913516:50511215152811:2800:D3F7218CE424AA87186CCCE98444DFA4224834A8C36DB9BFFF8F1DB06B3A1D68.png?needInitFileName=true?needInitFileName=true)
  
  - Ability
    
    Ability是应用所具备的能力的抽象，一个应用可以包含一个或多个Ability。Ability分为两种类型：FA（Feature Ability）和PA（Particle Ability）。FA/PA是应用的基本组成单元，能够实现特定的业务功能。FA有UI界面，而PA无UI界面。
    
    - FA支持
      
      Page Ability
      
      Page模板是FA唯一支持的模板，用于提供与用户交互的能力。一个Page实例可以包含一组相关页面，每个页面用一个AbilitySlice实例表示。
      
    - PA支持
      
      Service Ability和Data Ability：
      
      - Service模板：用于提供后台运行任务的能力。
      - Data模板：用于对外部提供统一的数据访问抽象。
  - 库文件
    
    库文件是应用依赖的第三方代码（例如so、jar、bin、har等二进制文件），存放在libs目录。
    
  - 资源文件
    
    应用的资源文件（字符串、图片、音频等）存放于resources目录下，便于开发者使用和维护，详见[资源文件的分类](https://developer.harmonyos.com/cn/docs/documentation/doc-guides/basic-resource-file-categories-0000001052066099)。
    
  - 配置文件
    
    配置文件 (config.json) 是应用的Ability信息，用于声明应用的Ability，以及应用所需权限等信息，详见[应用配置文件](https://developer.harmonyos.com/cn/docs/documentation/doc-guides/basic-config-file-overview-0000000000011951)。
    
  - pack.info
    
    描述应用软件包中每个HAP的属性，由IDE编译生成，应用市场根据该文件进行拆包和HAP的分类存储。HAP的具体属性包括：
    
    - delivery-with-install: 表示该HAP是否支持随应用安装。“true”表示支持随应用安装；“false”表示不支持随应用安装。
    - name：HAP文件名。
    - module-type：模块类型，entry或feature。
    - device-type：表示支持该HAP运行的设备类型。
  - HAR
    
    HAR（HarmonyOS Ability Resources）可以提供构建应用所需的所有内容，包括源代码、资源文件和config.json文件。HAR不同于HAP，HAR不能独立安装运行在设备上，只能作为应用模块的依赖项被引用。
    
3. HMS Core
  
  ![image20210310163509210](https://raw.githubusercontent.com/erleizh/erleizh.github.io/master/screenshots/HarmonyOS调研报告/image-20210310163509210.png)![image20210310163634966](https://raw.githubusercontent.com/erleizh/erleizh.github.io/master/screenshots/HarmonyOS调研报告/image-20210310163634966.png)![image20210310163803584](https://raw.githubusercontent.com/erleizh/erleizh.github.io/master/screenshots/HarmonyOS调研报告/image-20210310163803584.png)
  

#### HarmonyOS先行者说

[美的与HarmonyOS共同布局智能家居产业升级](https://developer.huawei.com/consumer/cn/forum/topic/0204419392996130561?fid=0101303901040230869)

[老板电器与HarmonyOS共同布局智能厨电产业升级](https://developer.huawei.com/consumer/cn/forum/topic/0202493965552050250?fid=0101303901040230869)

[WPS携手HarmonyOS一起打造智慧办公超级终端](https://developer.huawei.com/consumer/cn/forum/topic/0201493972745850269?fid=0101303901040230869)

[Fit健身APP联手HarmonyOS,更好体验和更多入口双收](https://developer.huawei.com/consumer/cn/forum/topic/0201493982044610271?fid=0101303901040230869)

#### 存在的问题

- 布局预览有概率导致 DevEco Studio 崩溃卡死，遇见过3次
  
- 截止2021.03.16 还不支持webview，距论坛反馈beta版本还在联调测试中
  
- 关于真机调试，看官网介绍需要申请应用、证书。然后申请远程OTA升级系统为鸿蒙系统，嫌麻烦没有尝试
  
  [HarmonyOS 2.0手机开发者Beta公测招募](https://developer.huawei.com/consumer/cn/activity/301607581257578636)
  
  [HarmonyOS Beta版本回退到EMUI 11.0官方稳定版本指导](https://developer.huawei.com/consumer/cn/doc/10010101)
  
  支持的真机
  
  ![img](https://communityfile-drcn.op.hicloud.com/FileServer/getFile/cmtybbs/300/118/079/0890086000300118079.20201217154259.27687879087608642591978317654494:50520309061821:2800:FE7A6CD775CAD87BB0E6FBEFF59232E998F20E9291AC7B4BCACF40F5ED4DF4A1.png)
  
- 关于模拟器
  
  目前因为没有搭载HarmonyOS系统的手机，华为提供了远程模拟器，开发时可以连接到远程模拟器进行开发调试，但是每次使用模拟器有1小时的时长限制，体验极差。录音，播放音乐等设备功能无法调试。
  
  ![image20210306192558901](https://raw.githubusercontent.com/erleizh/erleizh.github.io/master/screenshots/HarmonyOS调研报告/image-20210306192558901.png)
  

####总结

**作为Android应用开发者，鸿蒙无论是开发工具，还是项目结构都与Android保持高度一致 。开发者学习成本低**

**但就目前来看，对没有多设备协同需求的应用开发者吸引力较小 ，基础功能尚不完善，保持适当关注即可**

####扩展一 、关于Fuchsia

1. Fuchsia简介
  
  Fuchsia 是Google 开发的开源操作系统，是基于Google开发的 Zircon 微内核，使用 C/C++编写，面向未来的全场景操作系统。
  
  > Fuchsia历史
  > 
  > 2016年8月，媒体报道了发布于GitHub上的神秘源码，显示Google正在开发一个名为“Fuchsia”的新操作系统，虽然官方没有正式公布，其源码检查显示其能够跨平台运行，包括“汽车的娱乐媒体系统和嵌入式设备，如红绿灯、数码手表、智能手机、平板电脑与个人电脑”。[[6\]](https://zh.wikipedia.org/zh-hans/Google_Fuchsia#cite_note-6)[[7\]](https://zh.wikipedia.org/zh-hans/Google_Fuchsia#cite_note-7)
  > 
  > 2017年5月，Ars Technica编写了关于Fuchsia的新[用户界面](https://zh.wikipedia.org/wiki/用户界面)的文章，从8月首次披露时的命令行界面上升级，以及开发人员表示“此项目不是玩具项目，不是20%时间项目，不是我们不再关心的死去的项目的垃圾场”，多家媒体写到“Fuchsia项目”和[Android](https://zh.wikipedia.org/wiki/Android)似乎有密切联系，有人猜测Fuchsia可能是“重做”[[8\]](https://zh.wikipedia.org/zh-hans/Google_Fuchsia#cite_note-8)或替换Android[[9\]](https://zh.wikipedia.org/zh-hans/Google_Fuchsia#cite_note-9)[[10\]](https://zh.wikipedia.org/zh-hans/Google_Fuchsia#cite_note-10)以在某种程度上修复该平台上的问题。
  > 
  > 2017年11月，对[Swift](https://zh.wikipedia.org/wiki/Swift_(程式語言))语言提供了初始支持。[[11\]](https://zh.wikipedia.org/zh-hans/Google_Fuchsia#cite_note-11)
  > 
  > 2018年1月3日，Google允许开发者以Google Pixelbook为目标设备，下载Fuchsia OS进行开发与测试[[12\]](https://zh.wikipedia.org/zh-hans/Google_Fuchsia#cite_note-12)[[13\]](https://zh.wikipedia.org/zh-hans/Google_Fuchsia#cite_note-13)[[14\]](https://zh.wikipedia.org/zh-hans/Google_Fuchsia#cite_note-14)。
  > 
  > 2018年4月，Fuchsia的原始码出现在[AOSP](https://zh.wikipedia.org/wiki/AOSP)的[ART](https://zh.wikipedia.org/wiki/ART)当中，疑似是AOSP已经开始将ART移植至Fuchsia上，但原始码仍处于被注释处理的状态。[[15\]](https://zh.wikipedia.org/zh-hans/Google_Fuchsia#cite_note-15)
  > 
  > 2019年6月28日，Fuchsia OS的开发者网站Fuchsia.dev上线。[[16\]](https://zh.wikipedia.org/zh-hans/Google_Fuchsia#cite_note-16)
  > 
  > 2020年12月8日，首度在Google Open Source 部落格亮相，呼吁开发者来做出贡献。[[17\]](https://zh.wikipedia.org/zh-hans/Google_Fuchsia#cite_note-17)
  
  Fuchsia 的 用户界面与应用使用 Flutter开发。
  
  目前，国内华为、小米、Oppo、Vivi 均被曝参与Fuchsia系统的开发 。
  
2. Fuchsia系统架构
  
  ![img](https://fuchsia-china.com/wp-content/uploads/2018/11/layer-fuchsia.jpg)
  
  > 第一层：也是最底下一层，是构建 Fuchsia OS 的基石，Zircon 内核，去年的新闻是叫 Magenta，但是后来改为了 Zircon 这个名字，这是一个由Google全新设计的新内核，主要处理硬件访问和软件之间的通信。
  > 
  > 对于不太了解内核作用的同学简而言之，Zircon之于Fuchsia，恰如Linux之余于Android。Linux内核驱动了多个操作系统，很多操作系统构建在它之上，比如 Ubuntu、Android、Manjaro、ArchLinux、Debian、Red Hat、SUSE 甚至 Chrome OS ，所以我们也可以大胆预测，如果未来Fuchsia OS 发展良好， Zircon 内核也被证明好用，那么很有可能有更多的操作系统采用这一新内核。
  > 
  > 第二层：也是直接构建在 Zircon 上的一层名叫 Garnet。 Garnet 包含各种操作系统所需的各种底层功能，包括硬件的驱动程序（网络，图形等）和软件安装。这一层最激动人心的事情是 Escher（图形渲染器），Amber（Fuchsia 的更新程序）和Xi Core，它是Xi文本和代码编辑器的底层引擎（今年早些时候已经发布了）。
  > 
  > 第三层：Peridot 是接下来的这一层，主要处理Fuchsia的模块化应用程序设计， Peridot的另外两个主要组件直接用于模块。 Ledger 可以跨设备保存您在应用/模块中的位置，并同步到您的Google帐户。Maxwell 是一个更复杂的主题，需要更多进一步的深入研究，但是 Maxwell 极有可能是让 Fuchsia 充分施展魔力的点睛之笔，可以提前透露的是，Maxwell 的厉害之处包括 Kronk，也是大家熟知的 Google Assistant。
  > 
  > 第四层：Topaz，是这个 Layer Cake 蛋糕的顶层，也是对开发者和用户直接影响最大的一层。Topaz 提供 Flutter 支持，而有了Flutter 的支持，各种华丽的应用程序，可以帮助充实地提供日常使用的功能齐全的应用程序。比如，现在最令人印象深刻的当然是 Armadillo UI，它是 Fuchsia 主要用户界面和主屏幕
  

#### 扩展二、关于鸿蒙和Flutter的结合

国内有美团在探索Flutter结合鸿蒙，[[让 Flutter 在鸿蒙系统上跑起来](https://tech.meituan.com/2021/01/22/flutter-in-harmonyos.html)

而github上也有人针对Flutter是否会支持鸿蒙发起了Issues [HarmonyOS Support](https://github.com/flutter/flutter/issues/38437)

![image20210308224028696](https://raw.githubusercontent.com/erleizh/erleizh.github.io/master/screenshots/HarmonyOS调研报告/image-20210308224028696.png)

#### 相关资源

1. [鸿蒙开发文档](https://developer.harmonyos.com/cn/docs/documentation/doc-guides/document-outline-0000001064589184)
  
2. [美团.让 Flutter 在鸿蒙系统上跑起来](https://tech.meituan.com/2021/01/22/flutter-in-harmonyos.html)
  
3. [如何看待 9 月 10 日华为发布的鸿蒙 OS 2.0 系统，应用前景如何？](https://www.zhihu.com/question/420404904)
  
4. [如何看待华为 2019 年 8 月 9 日正式发布的 HarmonyOS 鸿蒙系统？](https://www.zhihu.com/question/339567108)
  
5. [Fuchsia OS开发者网站](https://fuchsia.dev/)
  
6. [什么是微内核](https://www.zhihu.com/question/339638625)
  
7. [关于 AOSP，一些你需要了解的](https://www.xbxit.com/545)