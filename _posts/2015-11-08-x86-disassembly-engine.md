---
layout: postlayout
title: 各种开源汇编、反汇编引擎的非专业比较
categories: [disassembly]
tags: [disassembly]
keywords: udis86,bochs,capstone,
---
　　由于平时业余兴趣和工作需要，研究过并使用过时下流行的各种开源的x86/64汇编和反汇编引擎。如果要对汇编指令进行分析和操作，要么自己研究Intel指令集
写一个，要么就用现成的开源引擎。自己写太浪费时间，又是苦力活，还容易出错，所以还是使用现成的好一点:laughing:。

这里对我曾使用过的比较流行的反汇编引擎做个比较，我使用过的反汇编引擎有：  

##1. Ollydbg的ODDisassm  
　　Ollydbg的[ODDisassm](http://www.ollydbg.de/)，这是我最早使用的一个开源的反汇编引擎，07年在[《加密解密》（三）](http://bbs.pediy.com/showthread.php?t=66210)
中我写的一个很简单的虚拟机就是使用的这个库，因为那个时候还没有那么多可选择。不过多亏有这样一个基础库，整个虚拟机从设计到开发完成只用了两个星期便开发完成
（当时对反汇编库的要求不高，只要求能用字符串文本做中间表示进行编码/解码）。  
　　这个反汇编库的优点是含有汇编接口（即文本解析，将文本字符串解析并编码成二进制），就拿这个特性来说在当时也算是独树一帜的了，到目前为止开源界在做这个工作的人也很少，
不过近年出现的调试器新秀x64dbg，也附带开发了开源的汇编库XEDParse，功能与OD的文本解析功能相似，并且支持的指令集更加完整，BUG更少，同时还支持X64，维护一直很强劲。  
但是ODDisassm的缺点也很多，比如：  
　　1. 指令集支持不全，由于Ollydbg年久失修，现在甚至连对MMX指令集都不全，而现在的INTEL/AMD的扩展指令集标准又更新了多个版本，什么SSE5/AVX/AES/XOP就更别提了，完全无法解析。  
　　2. 解码出来的结构不详细，比如指令前缀支持不够友好，这点从Ollydbg的反汇编窗口可以看出，除了movs/cmps等指令以外，repcc与其他指令组合时都是单独分开的；
	再比如寄存器无法表示`ah\bh\ch\dh`这种高8位寄存器。  
　　3. 作者一次性开源后便不再维护开源版本，对于反汇编上的BUG很难即时修复。  

　　不过这些也可以理解，因为在当时作者的开发目的是进行文本汇编\反汇编，所以没有为解码出的信息建立结构体以及接口。总的来说，如今再使用这个反汇编引擎，
已经落后于时代了。  

##2. BeaEngine  
　　[BeaEngine](https://github.com/BeaEngine/beaengine)是我用的第二个库，当时使用OD库已经不能满足我的需求了。
在做反编译器的时候，需要一个能够解码信息越多越好的库，于是我找到了`BeaEngine`，这个库我记得以前的版本不支持高8位寄存器识别，现在的版本也支持了。
在使用过程中基本上没有发现什么明显的缺点，不常用的新的扩展指令集也实现了不少。  
　　目前实现的扩展指令集有：
{% highlight c %}
FPU, MMX, SSE, SSE2, SSE3, SSSE3, SSE4.1, SSE4.2, VMX, CLMUL, AES, MPX
{% endhighlight %}
　　同时它也给不同种类的指令进行了分类，这在判断不同指令时很方便。它还有一个特点是可以解码出每一条指令所使用到和影响到的寄存器，
包括标志位寄存器，甚至精确到标志位寄存器的每一个位置。  
这个功能用来做优化器与混淆器再好不过了。  
　　但是个人认为`BeaEngine`的编码风格实在是不咋地，各种变量强制转换，各种命名风格，给人一种乱乱的感觉。
对我这种对编码有洁癖的人来说，实在是无法忍受:hushed:，所以后来又换了其他的库。如果你不在意这些，BeaEngine的性能还是比较不错的。  
还有一点，BeaEngine经常性的爆出一些BUG，所以使用它时要有心理准备。  

##3. udis86  
　　[`udis86`](https://github.com/vmt/udis86)应该是我最喜爱的反汇编引擎了。
udis86支持的X86扩展指令集有：
{% highlight c %}
MMX, FPU (x87), AMD 3DNow, SSE, SSE2, SSE3, SSSE3, SSE4.1, SSE4.2, AES, AMD-V, INTEL-VMX, SMX
{% endhighlight %}
　　udis86的代码风格十分简洁，功能函数短小，变量命名清楚又简洁，接口干净意思明白，操作灵活，如果你自己有需求维护一个自己的分支，花不了几十分钟的时间就能熟悉整个代码构架。  
　　说了这么多溢美之词，它到底有什么优点呢？  
　　udis86的优点就是接口很灵活，你可以选择对一条指令使用`ud_decode`只进行解码，再对解码后的结构使用`ud_translate_intel`转换成文本格式，  
或者你也可以直接使用`ud_disassemble`一次性完成整个操作。而且这些接口都只需要一行就能搞定。  
由于udis86的这种组合模式设计理念，他可以适应各种场景，如果你要开发一个IDA那样的反汇编器，它能做；
你要开发一个指令模拟器、分析器、优化器、混淆器，它也能做。  
　　这种理念直接使udis86在拥有了强大的适应能力的同时还兼顾了性能，我做过性能测试，
udis86是我用过的<font color='red'>解码细节能力相近</font>的情况下，解码速度最快的引擎了。  
　　至于缺点的话，目前还没发现，不过，udis86不支持BeaEngine那种的寄存器分析，算是点小遗憾。


##4. Capstone  
　　[`capstone`](https://github.com/aquynh/capstone)应该算是所有反汇编引擎中集大成者了，对于它我要多费点口水，因为我对他是又爱又恨:frowning:。
capstone是基于`LLVM`框架中的MC组件部分移植过来，所以`LLVM`支持的`CPU`构架，capstone也都支持。  
　　它支持的CPU构架有:`Arm, Arm64 (Armv8), M68K, Mips, PowerPC, Sparc, SystemZ, XCore & X86 (include X86_64)`。  
　　而且Capstone对X86构架的指令集支持是最全的，这一点是其他引擎都比不上的，其支持的X86扩展指令集有：
{% highlight c %}
3dnow, 3dnowa, x86_64, adx, aes, atom, avx, avx2, avx512cd, avx512er, avx512f, avx512pf, bmi, bmi2, fma, fma4, fsgsbase, lzcnt, mmx, sha, slm, sse, sse2, sse3, sse4.1, sse4.2, sse4a, ssse3, tbm, xop.
{% endhighlight %}
　　很强吧？
　　在目前移动端如此火热的背景下，支持ARM的反汇编库还是非常少的，如果要同时进行X86与ARM下的编译器方面的开发，能使用一个统一的接口那自然是更好。
另外capstone的`next`分支（`master`分支没有这个接口）中也支持BeaEngine那种解码时分析指令使用和影响到的寄存器这种炫酷的特技，有这样的基础库存在真的可以偷不少的懒:laughing:。  
　　仅从X86/64平台来看，无论是从解码能力还是指令集支持，`Capstone`可以称得上是一个完全超越了`BeaEngine`的存在。  
　　老套路，说完了好话，又该说缺点了。  
　　由于`capstone`是从`LLVM`移植过来，`capstone`是`C`语言的项目，而`LLVM`是`C++`项目，所以在移植过程中做了很多适配工作，显得很臃肿。  
　　举个栗子，`LLVM`中的`MCInst`是一个单条底层机制指令的描述类，由于`capstone`是C项目，
	移植时将这些类变成了结构，把成员函数变成了独立的C函数，比如`MCInst_Init`，`MCInst_setOpcode`等等。并且由于`LLVM`框架的复杂性和高度兼容性，
里面的所有的概念都做了高度抽象，并且Capstone又做了适配接口将其转换到自己的构架中，这会造成解码时中间层过多，性能下降。一条指令的解码过程用到的重要的中间层结构顺序是这样的：  
{% highlight c %}
MCInst => InternalInstruction => cs_insn
{% endhighlight %}
　　最基础的解码工作依靠LLVM的构架解码到Capstone的`InternalInstruction`，它是一个包含了解码过程中的所有细节的内部结构，
解码完成后再调用`update_pub_insn`将认为需要公开暴露出来的内容复制到`cs_insn`中。而其他反汇编引擎都是一次性解码到目标结构中的。  
　　Capstone的解码过程如此复杂，自然会对性能造成影响，我做过一个不太严格的性能测试，Capstone的性能消耗时间大概是udis86的5、6倍
（顺便吹一下，这里我给Capstone提交了一个小小的Pull Request,分别在[这里](https://github.com/aquynh/capstone/pull/484)和[这里](https://github.com/aquynh/capstone/pull/505)，
PR中附带了一个benchmark，经过测试性能提高了将近有20%），如果换种方式测试，udis86只使用`ud_decode`进行解码，而Capstone没有独立的解码接口，
需要做一些Hack，也让它不生成汇编文本，那么Capstone的消耗时间大概是udis86的2倍多，
由此可见Capstone的文本操作又比udis86慢更多。  
　　其次，Capstone的内存消耗很大，解码一条指令时你传入的指令结构`cs_insn`必须由动态分配函数来分配，并且还要分配两次，一次是cs_insn，一次是cs_detail。
这会造成巨量的内存碎片，另外，每一条指令的结构体都很大，有多大我不记得了，`sizeof(cs_insn)`+`sizeof(cs_detail)`好像至少在2K以上。
必须使用<font color='red'>动态内存</font>这一点是Capstone与其他反汇编引擎不一样的地方。
如果要使用Capstone做大量指令分析的话，那么得给它配一个固定对象内存分配器才行，那样能稍微缓解一下内存碎片情况，也能提高一点性能。  
　　可能是基于以上理由，x64dbg社区本来最开始是使用BeaEngine作为支撑基础的，但是BeaEngine总是爆出不少BUG，所以后来选择由Capstone替换，
但是也仅用Capstone来做GUI的文本反汇编，因为它解码速度虽然不行，但是BUG很少（毕竟LLVM有苹果那么大个公司做支撑），
而流图和指令分析（还不完善）目前仍然使用的BeaEngine，这也是没办法的事，毕竟性能也很重要。  
　　还有一个问题，如果你需要的是解码能力强的反汇编引擎，那么建议你在选择前先对比一下各引擎的解码结构，有没有你需要或者必须有的字段。  
　　因为Capstone有一个坑爹的地方，虽然其本身的解码能力其实很强的，但是Capstone把中间层封装了一遍，只暴露其认为需要暴露的字段，
并且其主要维护者有点固执（也可以说严谨），他坚持认为不太常用的字段没必要暴露出来，而接口是越简洁越好。  
　　比如说指令中立即数Immediate所在的偏移，内存操作数中Displacement所在的偏移，在内部结构`InternalInstruction`中本来是有的，但是复制到公开结构`cs_insn`结构中就丢弃了。
还有`REP`与`REPE`前缀，虽然都是同一个常量表示，但是配上不同指令功能还是不一样的，对于这个，capstone内部有一个`valid_repe`函数可以区别，
但是也没有暴露到公开结构中，都当做`REP`来识别。虽然这些都很冷门，但是做指令分析和变形，这些还是很有用处的。  
　　我个人认为capstone的接口用起来实在让人不爽，但是它的功能又实在太强大，如果研究下其源码的内部构造，会发现很多接口没提供，但是内部却有的好东西，
所以我自己维护了一个[分支](https://github.com/bughoho/capstone)，痛并快乐着的使用着。

##其他  
　　其实还有XDE，不过我没使用过，就不做评论了。  
　　另外，[blackbone](https://github.com/DarthTon/Blackbone)中含有一个长度反汇编引擎也值得一提，名字叫`ldasm`，其实它也算不上一个引擎，因为它只有一个函数，
作用只是计算一条指令的长度，在hook的时候重定位跳转指令时很有用处。[代码传送门](https://github.com/DarthTon/Blackbone/blob/master/src/BlackBone/Asm/LDasm.c)

##总结
　　这几款反汇编引擎各有长处（OD除外。。），但是又各自有那么一丁点儿的缺陷，这世上没有完美的事情，人家都开源了，有的用就不错了，自己总要做些事情不是？
挑一款好用的库，使用过程中发现BUG，给社区提交个Issue，或者做个解决方案再发个Pull Request，也算付了使用费了。  

| 特　	性      | 比较                                                                      |
|:-------------:|:--------------------------------------------------------------------------|
| 性能		    |udis86 > BeaEngine > capstone                                              |
| 解码能力	    |capstone > BeaEngine > udis86 (udis86不支持寄存器分析,其余解码能力是相近的)|
| 平台支持	    |capstone > (udis86 = BeaEngine)                                            |
| X86扩展指令集 |capstone > (udis86 ≈BeaEngine)                                            |


　　如果你需要的是一个X86/64下的性能又好同时解码能力又强的反汇编引擎，并且不需要寄存器分析这种特技的话，那么udis86合适你；
如果你还需要带寄存器分析功能的话，那么BeaEngine与capstone合适你；如果你还需要ARM构架支持的话，capstone应该会更适合你。  
　　每一款引擎各有优劣，使用起来仁者见仁智者见智，鞋子合不合适只有自己的脚才知道了。