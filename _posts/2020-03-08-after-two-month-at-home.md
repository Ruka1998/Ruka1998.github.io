---
title: 呆在家里的两个月
tags: 随记
key: after-two-month-at-home
show: false
---

想一想这两个月，其实还挺奇妙的，因为人在疫情的最中心，结果在家一直关禁闭。一开始玩了好几天，后来毕设开始开题，就一股脑的研究题目去了。

这个研究还有点点波折，因为申请了校外毕设，公司的导师又是美术出身，所以对渲染器底层没有太多可以提供的意见，我的毕设就变成了孤军奋战。题目自己想，首先得做个渲染器，然后又问了问公司的图程朋友，于是想试着研究一下Vulkan。本来对Google的Filament已经研究了了一段时间，结果学校的老师又说应用背景不清晰，得换题目。不得已只能将目光放到老项目上，看看能不能给老项目的丝绸类布料重做个PBR模型。

在换题目之前，我翻译了《Real-Time Rendering 4th》的第九章，想对PBR大框架有个全面的认识。翻译下来花了一个月（其实中间还有很多摸鱼的时间），虽然latex用的熟练了，脑子里混沌的知识却越来越多。当时怪难受的，本来是想把以前的东西翻出来看看休息休息，却一眼看到filament的文档，有一种茅塞顿开的感觉。现在想一想，可能就是理论和应用结合在一起之后，对于每样东西有了初步的认识的原因。这里不得不说，filament的文档真的做得很好，每个模型及其后面的原理都有说法，在多个模型之间如何选择也有道理，而且因为主要针对安卓平台所以涉及到一些优化的内容。我觉着如果想研究PBR，这份文档是很好的材料。

说到Filament，不由得就想起来在Filament同好群里面认识的图程姑娘，在北京上班，正好比我大三届，刚研究生毕业。和这个姑娘讨论一些问题之后，我又长了不少见识。深感我还有太多的东西要学习。

写这篇文章的念头在完成了开题报告之后愈发强烈，因为这两个月我的脑子里似乎发生了不小的变化。从最开始我朝着TA努力，到我成为了TA，到知识逐渐清晰的过程中，我看到了许多问题的解决方法，登上山顶，本来以为下面是好风光，结果发现脚下的仅仅是个小山丘，前面是更陡的高山。有点像Journey最后登雪山时候的无措感，好在环境并没有那样恶劣，目标也还算是明确，我的身边也有一个鼓舞人心的同行人。

我的思维又发散到了Journey这款游戏上，本来之前PS平台独占的游戏最近被移植到PC和手机，我也很荣幸地可以亲手玩一玩。玩了一次，通关之后，欲罢不能，于是又通关了两三次达成全收集的目标，拿了白袍，紧接着用白袍又通关了好几次。除了第一次碰到了其他玩家，后面几次都是我自己走过了整个旅程。痴迷于其治愈人心的游戏性和独特的画面效果，我找到了GDC上Journey开发者的技术宣讲，决心要实现一下里面的沙丘渲染，正巧，一个博客最近的一系列文章正是针对模拟Journey沙丘渲染的。目前正在一点一点做，碰到了一些问题，还需要继续研究。

不知道疫情什么时候会结束，希望毕业的夏天可以快乐。