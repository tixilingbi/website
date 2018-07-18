+++
title = "标记"
description = "标记是一个特定提交的引用，Git 可以给历史中的某一个提交打上标签，以示重要。"
weight = 3
+++

# 标记

通过系统内标记模块为分支创建的标记属于轻量级标记，其实是一个独立的分支，指向特定提交对象的引用。通常开发者会为比较重要的代码版本创建一个标记，例如在新的版本发布的时候，会给这个版本打上标记，方便代码版本管理。

- **菜单层次**：项目层
- **菜单路径**：开发流水线 > 标记
- **默认角色**：项目所有者、项目成员

## 标记列表

标记模块提供一个**项目所有者**和**项目成员**可查看的列表，通过**选择应用**选择器，可以选择查看不同应用下的所有标记。

- 应用选择器，切换应用并查看该应用下拥有的标记
- 标记名称，使用[语义化版本](https://semver.org/lang/zh-CN/)规范
- 提交编码，创建标记的来源分支上最新一次提交的编码，也是标记所引用的特定提交
- 提交描述，创建标记的来源分支上最新一次提交的描述信息
- 提交时间，创建标记的来源分支上最新一次提交的时间
- 删除操作，点击`删除按钮`→ ![删除按钮](/docs/user-guide/development-pipeline/image/del_net_button.png)，可以删除该标记
  <blockquote class="note">
    只有项目所有者可以进行标记删除操作。
  </blockquote>

## 标记名称的命名规则
标记名称格式如：`0.8.0`，其中三个数字含义如下：

1. **主版本号**：当你做了不兼容的 API 修改
2. **次版本号**：当你做了向下兼容的功能性新增
3. **修订号**：当你做了向下兼容的问题修正

## 创建标记

1. **前置条件** - 应用中存在至少一个分支，否则无法创建标记。
2. **创建标记**

  ![创建标记](/docs/user-guide/development-pipeline/image/create-tag.png)

- 标记名称，必须符合[语义化版本](https://semver.org/lang/zh-CN/)规范
- 标记来源，可以是该应用下任意的分支
  <blockquote class="warning">
    如果所选应用下没有任何分支，则无法创建标记，需要先去分支模块创建分支。
  </blockquote>

## 删除标记

项目所有者可以删除标记，删除标记成功后，代码仓库中不再存在该标记，并且线上应用无法再使用该标记。

标记列表每一行的最后都提供点击`删除按钮`→ ![删除按钮](/docs/user-guide/development-pipeline/image/del_net_button.png)，点击该按钮，会弹出一个确认删除的弹框，点击确认，该标记被删除。