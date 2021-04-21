---
title: 使用 husky + lint-staged 防止💩的流入
date: 2019-12-21 21:05:05
categories:
  - 开发杂记
---

##### 写在前面

###### husky

husky 是一个用于给 `git` 相关操作添加钩子的工具，通过 husky 我们可以非常简单的给 git 相关操作添加[钩子](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)，最终我们会使用 husky 来给 git commit 操作挂上钩子, 来预防一些不良的 commit。

###### lint-staged

先看看看 lint-staged 的官方介绍，个人觉得还是挺有意思的

> &#x1f6ab;&#x1f4a9; lint-staged
> Run linters against staged git files and don't let &#x1f4a9; slip into your code base!

大概意思是，在你将提交暂存区的文件到仓库之前，可以通过 lint-staged 对暂存区的文件进行检查，检查通过才能提交到仓库，以防止&#x1f4a9;的流入。

<!--more-->

##### 使用 husky 和 lint-staged

1. 安装依赖库

```shell
yarn add --dev husky lint-staged
```

2. 在 package.json 中配置 husky

- 我们知道 husky 会给 git 相关操作添加钩子
- 所以我们需要做的就是，配置钩子触发以后要做的事情
- 在这里我们配置，在 `git commit` 之前，先调用 lint-staged 对暂存区的文件进行检查

```json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  }
}
```

3. 在 package.json 中配置 lint-staged

- 这一步我们将配置 lint-staged 的检查规则，lint-staged 会将与规则相匹配的`暂存区的文件`进行检查
- 第一个配置会把 src 目录下所有 .ts .tsx 文件进行 `eslint` 修复，`prettier` 格式化，之后将此文件再次提交到暂存区中(因为在进行检查之前 line-staged 会进行 `git stash` 相关操作，因此需要将文件重新提交到暂存区中)
- 第二个配置会把 src 目录下所有 .scss 文件进行 `stylelint` 修复，`prettier` 格式化，之后将此文件再次提交到暂存区中(因为在进行检查之前 line-staged 会进行 `git stash` 相关操作，因此需要将文件重新提交到暂存区中)

```json
{
  "lint-staged": {
    "src/**/*.{ts,tsx}": ["eslint --fix", "prettier --write", "git add"],
    "src/**/*.{scss}": ["stylelint --syntax scss --fix", "prettier --write", "git add"]
  }
}
```

##### husky 和 lint-staged 的大致工作流程

1.`git add .` 将所有改动的文件提交到暂存区

2.`git commit -m "chore: xxx"` 此操作会被 husky 所拦截, 之后调用 lint-staged 对文件进行检查

3.lint-staged 会先进行 `git stash` 相关操作, 之后会将与规则相匹配的`暂存区的文件`进行检查，`只有已经被提交到暂存区的文件才会被检查，不管规则怎么写`

4.等带 lint-staged 执行完成后，只要有一个文件没有通过检查，husky 将会阻止本次的 `git commt`，之后需要修改对应的文件，重新进行 `git add` 与 `git commit` 操作
