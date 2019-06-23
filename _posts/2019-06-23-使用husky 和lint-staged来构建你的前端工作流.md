---
layout: post
title: 使用 husky 和 lint-staged 来构建你的前端工作流
categories: 前端圈
description: 前端工作流构建
keywords: husky, lint-staged, ESLint
---

ESLint 是一个在前端工具链中被众人熟知的代码检查工具，它能够被开发者灵活的配置，使其能够达到我们提前制定好的代码规范的要求，并且在编码过程中实时检测输入的代码，对于不符合代码规范的代码警告或报错。不得不说，在有了 ESLint 这个工具之后，团队之间开发维护会舒服很多，因为在强制约束下，你只需要去理解代码本身的含义就可以了，对于风格的问题则少了很多麻烦。

既然 ESLint 这么好用，那我们为什么又要引入今天要介绍的这两种工具呢？因为在日常的团队工作中，自觉地同学会将 ESLint 的警告或报错修复了再提交代码，而总有一些不自觉的，对于代码风格不重视的同学，会不管报错，直接将代码风格不符合规范的代码提交到 git 仓库中，久而久之随之项目的代码数量越来越多，ESLint 报错的地方也会越来越多，想要一个一个修复实在是积重难返。于是当我在使用 Vant 这个前端开源组件库的时候，提交代码的过程中发现他们在 commit 之前会检查你提交的代码是否规范，当时就觉得这个非常实用，后来发现用 git 提供的一组 hook 可以实现这样的功能，`git commit` 是最常用的命令之一，它可以触发四个 hook ，分别是 `pre-commit`, `prepare-commit-msg`, `commit-msg` 和 `post-commit`。 从字面上猜测着四个 hook 分别对应了 “commit 之前”，“准备 commit log message 的时候”，“生成 commit log message 的时候”，“commit 之后”四个阶段。

而我们要解决我们的问题，其实只需要在 'pre-commit' 这个阶段去写一段脚本，就能解决我们检测代码的问题。

正当我准备写脚本来解决这个问题的时候，发现 github 上有一个已经被造好的轮子，有很多的 star，于是乎，本着不重复造轮子的精神，我去看了一下这两个工具的文档。

### husky

husky 这个库，老师说我看他的文档的时候看笑了，不为别的，就为了这个命名。我的理解是作者觉得这个库的作用是看(chai)家护院的二哈么？

> Husky can prevent bad git commit, git push and more 🐶 woof!

Husky 能够帮你阻挡住不好的代码提交和推送。一句话很精准的说明了这个库的意义，而配置也非常简单，

```js
// package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "npm test",
      "pre-push": "npm test",
      "...": "..."
    }
  }
}
```

就像这样，在我们的 `package.json` 中配置 husky，并且在对应的 git hook 阶段来执行对应的命令。于是乎，不用繁琐的去配置 git hook 阶段的脚本文件了，提供对应的 node 操作变好。

### lint-staged

lint-stage 这个库是这么解释它的存在的。

在你提交代码之前，Linting 的运行是更有意义的。当你这样去做了，那么就会有更少的错误进入你的代码库。

有多种方式能够配置 lint-staged，例如在 `package.json` 中添加对应的对象，例如使用 JSON 或者 YML 文件来配置，例如写一个 js 文件来配置等等。

而鉴于我们的 husky 使用了 package.json 的方式来配置，那么 lint-staged 也保持统一使用同样的方式好了。

```js
// in package.json
{
  "lint-staged": {
    "*.{js, vue, css}": [
      "eslint",
      "git add"
    ]
  }
}

```

当你这样配置完成，在你的 git commit 之前，会自动触发 eslint 检查，如果你的代码风格没问题，commit 会成功，否则提交会失败哦。

对于这样好的工具，闭着眼睛就能按 star 了，统一团队的代码风格，真的很重要。能够让错误防范于未然。
