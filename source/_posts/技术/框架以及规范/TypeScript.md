---
layout: default
title: \{#\{ site.name }}
---
# TypeScript
团队的邹瑞同学分享了typescript相关的东西，自己做一下记录。

可以直接写js代码，比coffee友好多了。

类型的查错在编译的那一层做好了。

支持ES6的大部分的语法。

想使用的话，就直接安装就是npm install一下，然后就会有tic命令行工具。项目里需要一个tsconfig.json进行配置。后缀名都换成.ts就行了。

可以定接口，然互传参时进行对接口的检验，居然还有implements这种东西(其实只是种很弱的检测，并没有真正检测，只是进行了属性的检查)

居然还可以定义private，public，protected关键字

拥有命名空间，不过这个东西与module共存其实挺奇怪的，看上去好像是为了单个JS里面申明多个namespace存在的。

声明可以自动合并，接口，命名空间，但是也有很多是没法合并的
