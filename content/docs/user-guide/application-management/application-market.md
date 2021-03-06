+++
title = "应用市场"
description = "是应用的集市，包含本项目应用发布的发布层级为本组织或者其他组织项目下的应用发布层级为全平台的应用，应用可以只发布某些版本，供其他项目预览和使用。通过应用市场的导入可以将一个应用的一个或多个版本导入，导出功能可以导出一个应用的多个版本到项目下。"
weight = 5
+++

# 应用市场

集合本项目可访问的已发布应用，供用户浏览，可部署所需的应用版本至项目中。并且能进行应用市场内应用的导入导出。

- **菜单层次**：项目层
- **菜单路径**：应用市场 > 应用市场
- **默认角色**：项目所有者、项目成员、部署管理员

<h2>应用市场</h2>

查看应用市场，可以以卡片或列表查看项目可访问应用的基础信息，包括应用名称、分类、贡献者、描述以及应用图标。  

点击卡片或列表最右的详情查看应用的详细信息。


<h2> 应用详情 </h2>

在详情界面可查看应用市场中应用的详细信息，包含不同版本的说明文件，应用的应用名称、分类、贡献者、描述以及应用图标。

通过点击版本可以进行不同版本的切换以查看不同版本的说明文件。可选的对不同版本进行部署到当前项目的操作。

<h2> 应用导出 </h2>

通过点击应用市场界面的`导出`按钮，可以进行导出操作。

1. 在列表中选择需要导出的一个或多个应用。

2. 在下一步选择应用需要导出的一个或多个版本，此处默认选定导出最新发布版本。

3. 在下一步查看已选择导出应用的总览信息，点击导出，下载应用及其版本文件压缩包。

4. 导出完成。

<h2> 应用导入 </h2>

通过点击应用市场界面的`导入`按钮，可以进行导入操作。

1. 点击选取或拖拽上传导出操作生成的压缩包。

2. 在下一步中，选择导入应用是否要发布，以及发布层级。

3. 在下一步中，确认导入应用的清单，点击导入，完成导入。

<blockquote class="note">
    若项目下已有相同应用编码的应用，导入操作则会更新应用的相关信息。
    若应用下已有相同版本号的版本，导入操作会更新相关的版本信息。
    若导入的应用已经发布，则不会对发布层级产生影响。
    目前只支持.zip包的导入。
</blockquote>
