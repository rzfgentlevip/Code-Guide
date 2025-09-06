---
title: 前言
createTime: 2025/02/19 15:10:06
permalink: /begin/
---

<Layout/>

您好，亲爱的朋友。这是一本和计算机、项目建设、开放公共服务、大学生活等相关的手册，由一些老师、同学以及朋友共同编写，也是我们对整个大学生涯的一个纪念。

这同时也是一份献给重医学弟学妹们的礼物。如果这本书能对你们的本科生涯有哪怕一丝一毫的帮助，都是对我们极大的鼓励和慰藉。

如果你觉得我们写得还不错，请在[::line-md:github-twotone:: Github](https://github.com/PGuideDev/PGuide-Docs)
上给我们点一个大大的::fxemoji:star:: star，这是对我们最大的鼓励。

---

项导文档目前包括了以下部分，如果你有其他好的建议，或者想加入贡献者的行列，欢迎 [邮件联系](mailto:rand777@pguide.studio)
阅读 [贡献指南](/contribute/) ，在下方评论区提问：

<CardGrid>

<LinkCard
icon="/icon/note.svg"
title="学习笔记"
href="/learning-notes/">
学长学姐的学习笔记，带你一起走过大学的每一个阶段～

- 😊课程笔记
- 😊自学心得
- 😊在线问答

我们已经帮你踩过很多坑啦，就等你来走了!
</LinkCard>

<LinkCard
icon="/icon/project.svg"
title="实战项目"
href="/project-docs/">
历届优质项目全记录！快来看看我们都做了些什么项目吧~

- 💡 包含项目创意说明
- 💡 具体实现教程
- 💡 团队协作经验

学了知识怎么应用？想做项目却不知从何下手？快来看这里～>
</LinkCard>

<LinkCard
icon="/icon/code.svg"
title="CS自学指南"
href="/cs-diy/">
一本关于计算机各方向的自学指南，今天也在努力敲代码啊～

- 🌟 零基础入门路径
- 🌟 热门方向学习路线
- 🌟 精选学习资源推荐及下载

自学找不到方向？这本指南就是你的导航！
</LinkCard>

<LinkCard
icon="/icon/public-service.svg"
title="公共服务"
href="/public-service/">
学长学姐为你准备的校园生存工具箱，助力有目标的大学生活～

* ✅ 各家模型免费用
* ✅ 常用软件下载
* ✅ 生活服务入口

指尖上的便利校园生活一键开启，点击直达
</LinkCard>

<LinkCard
icon="/icon/wiki.svg"
title="大学百科"
href="/campus-wiki/">
大学通关秘籍看这里！

* ❓ 如何平衡学习与社团？
* ❓ 选课怎样不踩雷？
* ❓ 老师们的一些建议

新人疑问一站式解答！
</LinkCard>

<LinkCard
icon="/icon/link.svg"
title="友情链接"
href="/friends/persons/">

优质资源直连通道，共建学习生态圈～

* 🌐 合作组织官网
* 🌐 开源项目社区
* 🌐 技术交流论坛

点击探索更多合作伙伴，构建你的资源网络！
</LinkCard>

</CardGrid>

## 为什么写这本书

在查找各种资料的时候，尤其是计算机类的，我时常会因为资源获取的麻烦（要么是百度网盘，要么是微信关注公众号）、CSDN 等平台的收费、文档的时效性等等问题而感到苦恼，而我在这种苦恼中度过了课程最多的大一。

这一切的转机发生在发生在大二的时候，我无意中看到了 [CS-DIY](https://csdiy.wiki/)
这本书，当时的内容已经相当丰富，基本覆盖了计科、智能系、软工系的绝大多数课程。起初，我抱着怀疑的态度阅读了其中比较简单的一篇- [CS61A](https://csdiy.wiki/%E7%BC%96%E7%A8%8B%E5%85%A5%E9%97%A8/Python/CS61A/)
，哇，那时候我才感受到原来 Python 或者说编程可以这么简单而有趣。

:::details 下面这段话是复制的 [Yinmin Zhong](/friends/persons/) 的，也算是我的心里话：
> 我一口气 3 个星期上完了这门课，它让我第一次感觉到原来 CS 可以学得如此充实而有趣，原来这世上竟有如此精华的课程。为避免有崇洋媚外之嫌，我单纯从一个学生的视角来讲讲自学
> CS61A 的体验：
>
> - 独立搭建的课程网站: 一个网站将所有课程资源整合一体，条理分明的课程 schedule、所有 slides, homework,
    discussion的文件链接、详细明确的课程给分说明、历年的考试题与答案。这样一个网站抛开美观程度不谈，既方便学生，也让资源公正透明。
> - 课程教授亲自编写的教材：CS61A 这门课的开课老师将 MIT 的经典教材 Structure and Interpretation of Computer Programs (
    SICP) 用Python这门语言进行改编（原教材基于 Scheme
    语言），保证了课堂内容与教材内容的一致性，同时补充了更多细节，可以说诚意满满。而且[全书开源](https://www.composingprograms.com/)
    ，可以直接线上阅读。
> - 丰富到让人眼花缭乱的课程作业：14 个 lab 巩固随堂知识点，10 个 homework，还有 4 个代码量均上千行的 project。与大家熟悉的
    OJ 和 Word 文档式的作业不同，所有作业均有完善的代码框架，保姆级的作业说明。每个 Project 都有详尽的 handout 文档、全自动的评分脚本。
> - CS61A 甚至专门开发了一个[自动化的作业提交评分系统](https://okpy.org/)（据说还发了论文）。当然，有人会说“一个 project
    几千行代码大部分都是助教帮你写好的，你还能学到啥？”。此言差矣，作为一个刚刚接触计算机，连安装 Python
    都磕磕绊绊的小白来说，这样完善的代码框架既可以让你专注于巩固课堂上学习到的核心知识点，又能有“我才学了一个月就能做一个小游戏了！”的成就感，还能有机会阅读学习别人高质量的代码，从而为自己所用。我觉得在低年级，这种代码框架可以说百利而无一害。就是苦了老师和助教，因为开发这样的作业可想而知需要相当大的时间投入和多年的迭代积累。
> - 每周 Discussion 讨论课：助教会讲解知识难点和考试例题，习题全部用 LaTeX
    撰写，相当规范并且会给出详细的解答，让学生及时查漏补缺巩固知识点。这样的课程，你完全不需要任何计算机的基础，你只需要努力、认真、花时间就够了。此前那种有劲没处使的感觉，那种付出再多时间却得不到回报的感觉，从此烟消云散。这太适合我了，我从此爱上了自学。试想如果有人能把艰深的知识点嚼碎嚼烂，用生动直白的方式呈现给你，还有那么多听起来就很
    fancy，种类繁多的 project 来巩固你的理论知识，你会觉得他们真的是在倾尽全力想方设法地让你完全掌握这门课，你会觉得不学好它简直是对这些课程建设者的侮辱。如果你觉得我在夸大其词，那么不妨从
    CS61A 开始，因为它是我的梦开始的地方。
> - 这样的课程，你完全不需要任何计算机的基础，你只需要努力、认真、花时间就够了。此前那种有劲没处使的感觉，那种付出再多时间却得不到回报的感觉，从此烟消云散。这太适合我了，我从此爱上了自学。
> - 试想如果有人能把艰深的知识点嚼碎嚼烂，用生动直白的方式呈现给你，还有那么多听起来就很 fancy，种类繁多的 project
    来巩固你的理论知识，你会觉得他们真的是在倾尽全力想方设法地让你完全掌握这门课，你会觉得不学好它简直是对这些课程建设者的侮辱。
> - 如果你觉得我在夸大其词，那么不妨从 [CS61A](https://cs61a.org/) 开始，因为它是我的梦开始的地方。

:::

大二这一年，我参加了许多比赛，也收获了一些奖项。但一种无名的空虚感侵蚀着我，望着一张张冰冷的“奖状”，我不禁反思：这是我想要的大学生活吗？这些对我真的有帮助吗？

的确，名义上这些代表我，或者说我的团队在过去做出的努力，在各种面试中也起到了一定作用。但我也清楚地知道，这些身外之物说明不了什么，想要真正成长，我需要实际的、系统性的解决问题的能力，并且将这些如 CSDIY 一样无偿地分享给其他人。

于是，在大三上，我整理了所有参加过的比赛、学习过的知识，将其放入了语雀文档站里。最初，我找到了几位熟悉的老师交流了下想法，他们都十分支持这种做法。有一位老师非常担心写这个会占用我们的时间，认为对我们成长没太大用处，表示在现在这个信息十分容易获取的时代意义不大，发了长达200秒的语音劝我们慎重考虑。

但我内心十分清楚计算机类学科的特殊性，正是因为信息爆炸，现在获取到所需要的资料变得更为困难，尤其是生成式人工智能盛行的今天，对于稍微困难点的问题，我们需要花费更多时间来鉴别资料的时效性、可用性、真实性等，很多无意义的时间便浪费在了上面。再者，我们十分需要一个交流的平台，来减少信息的极度不对等性。

在更多地与同学、老师、朋友们交流后，我决定开始建设项导文档站，现在项导文档包含的内容已相对丰富。当然，作为一个还未毕业的本科生，我深感自己没有能力也没有权利去宣扬这种方式，古人荀子《劝学》中所述说明有人积极的同时也有人消极，我只是希望这份资料能让那些同样有自学之心和毅力朋友可以少走些弯路，收获更丰富、更多样、更满足的学习体验。

## 关于为什么自行开发与维护

### 语雀的缺陷

项导文档之前在语雀也有存档，随着用户体量的增大，也暴露出一些问题：

1. 内容审核严格：语雀会对发布到互联网的团队文档进行严格审核，一旦检测到敏感词就会风控，极为不便；
2. 自定义程度低：不支持图标、 markdown 拓展等，组件库几乎等于没有；
3. 域名不可自定义：对 SEO 不友好，而且文档永久链接是Nano加密的字符组（像` ag1xgaxa `这样），不直观；
4. 需要注册和邀请：语雀需要每一位成员都进行注册和管理员审核协作，对于双方都不方便；
5. 太过简单：不需要开发，对于 markdown 协作和前端开发未必是好事；
6. 不可控风险：语雀曾在[宕机后长时间后才恢复](https://36kr.com/p/2487990260602760)的过往，虽然可能并非故意为之，但确实会影响到文档的维护和企业形象。

## 你也想加入到贡献者的行列

一个人的力量终究是有限的，项导文档欢迎各位加入建设，目前我们针对同学们的情况编写了部分文档，涵盖的内容可能不齐全或存在可优化之处。如果有大佬想在其他领域分享自己的自学经历与资源，或有项目想要分享的，欢迎加入文档建设。请参考[贡献指南](contribute.md)
或也欢迎[和我电子邮件](mailto:losmosa@foxmail.com)联系。

## 关于我们

_项导文档，人人为我，我为人人。不必在意我们的名字，因为你就是下一个故事。_

> _PGuide Docs: Each exists for all, and all for each. Our names are but fleeting echoes, for within you lies the seed
of the next tale._

### 贡献者名单

所有贡献者名单：

- [个人](friends-persons.md)
- [组织](friends-organizations.md)

### 名字的由来

> 工作室名为 PGuide Studio ，中文为 项导工作室。P代表 Project ，Guide 代表引导或指南。我们希望能为大家提供一个编程类项目学习的指南，将这些项目代代相传。

## 怎么在评论区互动

1. 转到[登录界面](https://comment.pguide.studio/ui/login?lng=zh-CN)
2. 使用 Markdown 格式提问
3. 邮件收到回复

## 一起学习

项导文档的设计思想是通过新同学的阅读中产生的疑惑来完善文档，将所学知识应用到真实的开发场景中。我们虽然名义上是医科大的同学，但同样对计算机充满好奇与热情。

从现实条件上讲，每个人的精力和经验有限，计算机领域知识相关性强、迭代速度快，文档维护只能尽力而为。如果您对某篇文档有更好的建议，可以参考贡献指南来加入文档的建设。

好在互联网上，不乏有更优秀的前辈们探索的道路与智慧的结晶。如果您也是类似的开源文档、项目开发者，欢迎[添加友情链接](/friends/quotes/#我该怎样添加社会链接)，互相交流学习。下面是项导文档中参考和引用的优秀开源项目：

<CardGrid>
<RepoCard repo="Ac-Wiki/Ac-Wiki"></RepoCard>
<RepoCard repo="chenyme/LinuxDoWiki"></RepoCard>
<RepoCard repo="ustclug/Linux101-docs"></RepoCard>
<RepoCard repo="ustclug/Linux201-docs"></RepoCard>
<RepoCard repo="criwits/missing-web/"></RepoCard>
<RepoCard repo="pkuflyingpig/cs-self-learning/"></RepoCard>
<RepoCard repo="forthespada/developer-roadmap-zh-CN"></RepoCard>
<RepoCard repo="OI-wiki/OI-wiki"></RepoCard>
<RepoCard repo="ProbiusOfficial/Hello-CTF"></RepoCard>
<RepoCard repo="ctf-wiki/ctf-wiki"></RepoCard>
<RepoCard repo="CTF-Archives/.github"></RepoCard>
<RepoCard repo="forthespada/CS-Books"></RepoCard>
<RepoCard repo="forthespada/Awsome-Courses"></RepoCard>
<RepoCard repo="Snailclimb/JavaGuide"></RepoCard>
<RepoCard repo="/CollegesChat/university-information/"></RepoCard>
</CardGrid>

较为完整的目录：

- Ac Wiki
- Linux Do Wiki
- Linux 101
- Linux 201
- 你缺失的那门计算机课
- CSDIY
- 编程导航
- Code RoadMap
- OI Wiki
- CTF Wiki
- Hello CTF
- 电脑开荒网
- Python PEP
- CTF Archive
- CS Books
- Awesome Courses
- Java Guide
- 大学生活指北

> 以上，对互联网开源精神致敬，对世界的美好致谢。

## 致谢

自项导文档站开设已来，已累计获得15万+的阅读量，共有90+位成员参与建设，感谢每一位同学、老师、大佬们的支持与贡献。我们会继续努力，争取在未来的日子里为大家提供更好的服务和文档支持。

### 里程碑

::: timeline between card

- 项导文档语雀站
  time=2024-05-26 icon=/icon/yuque.svg

  rand777 在语雀建设第一版项导文档站，邀请了部分同学加入。

- 建设里程碑01
  time=2024-11-20 icon=tdesign:cooperate-filled type=important

  [::ri:yuque-fill::语雀文档站](https://www.yuque.com/pguide/public)字数达10W，共有21位成员，此时并未在互联网开放，阅读量达4000。

- 项导文档站开放
  time=2025-01-22 type=success icon=/icon/logo.svg

  项导文档站在互联网公开访问，秉承着人人为我，我为人人的精神，吸引了许多优秀的同学加入建设。
- 入选Plume选集
  time=2025-03-09 type=success icon=https://theme-plume.vuejs.press/plume.png

  通过 PullRequest 入选[Vuepress Plume主题选集](https://theme-plume.vuejs.press/demos/)，感谢Plume团队的支持与认可。

- 建设里程碑02
  time=2025-03-11 type=important icon=tdesign:cooperate-filled

  项导文档站总字数达8W，阅读量达3W+，共计50+位成员参与建设，感谢！
- 建设里程碑03
  time=2025-04-27 type=important icon=tdesign:cooperate-filled

  项导文档站总字数达12W，阅读量达4W+!
- 建设里程碑04
  time=2025-05-20 type=important  icon=tdesign:cooperate-filled

  项导文档站总字数达15W，阅读量达8W，共有77位成员、18个组织加入建设。
- 获得奖项01
  time=2025-05-24 type=warning  icon=healthicons:award-trophy

  项导文档获[::https://2025.jsjds.com.cn/assets/logo2-1fc42965.png::中国计算机设计大赛](/campus-wiki/competition/4c/)省级一等奖，创造==我校最好成绩=={.tip}。<br>同时，祝贺工作室项目[::token-branded:wow::Oh my API](/project-docs/oh-my-api/)获得省级三等奖。 
- 建设里程碑05
  time=2025-06-11 type=important  icon=tdesign:cooperate-filled

  项导文档站总字数达16W，阅读量达10W，共有82位成员、25个组织加入建设。
- 基础设施建设01
  time=2025-06-15 type=success icon=qlementine-icons:build-16

  支持了在线编辑、评论区、搜索、服务器状态查询等多种功能。各种网络优化下大陆地区访问速度大幅提升，[国际主站点](https://ping.chinaz.com/docs.pguide.studio)平均延迟低于100ms，[国内镜像站点平均延迟](https://ping.chinaz.com/docs.pguide.cloud)低于20ms，优化了用户体验。

- 获得奖项02
  time=2025-07-31 type=warning  icon=healthicons:award-trophy

  项导文档获[::https://2025.jsjds.com.cn/assets/logo2-1fc42965.png::中国计算机设计大赛](/campus-wiki/competition/4c/)全国三等奖，继续加油！

- 建设里程碑06
  time=2025-08-06 type=important  icon=tdesign:cooperate-filled

  项导文档站总字数达17W，阅读量达15W，共有92位成员、28个组织加入建设。

- 未来
  time=2077-07-07 type=tip  icon=token:future

  我们的故事还在继续，欢迎加入建设！
:::


