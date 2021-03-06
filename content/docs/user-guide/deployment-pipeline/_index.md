﻿+++
title = "部署流水线"
description = "部署流水线主要描述部署的环境、实例、网络、域名、容器等，来告知用户如何部署和部署情况"
weight = 7
+++

## 使用 Choerodon 部署流水线

欢迎使用 Choerodon 猪齿鱼部署流水线用户手册，如果您是部署管理员，项目成员或者项目所有者，这些文档将对您有用。本节将围绕部署流水线各个功能的使用进行详细的介绍，包括 **环境流水线**、**应用部署**、**实例**、**网络管理**、**域名管理**，**容器管理**将涵盖您使用Choerodon猪齿鱼部署流水线过程中所有可能进行的操作。

- 部署管理员使用 **环境流水线** 创建agent环境客户端，用于后续操作实例，**应用部署**，**实例**, **网络管理**, **域名管理**,**容器管理**,部署管理员也可以在agent环境客户端版本落后时升级客户端，以及停用启用环境。

- 当使用 **开发流水线** 某个应用生成版本之后，部署管理员可以使用 **应用部署** 为该应用的应用版本，选择 **环境流水线** 创建的环境客户端，然后修改其要部署的配置信息，在环境中创建或升级一个实例。

- 当部署的实例的应用版本不是最新时，部署管理员可以通过 **实例** 的升级功能升级实例，当部署的实例创建是部署文件填写错误，部署管理员可以通过 **实例** 的修改配置文件功能修改配置文件，当某个实例失去价值需要删除时，可以酌情删除，停止，以及重启实例。部署管理员也可以查看实例的详情信息。

- 当某个应用在某个环境中有运行实例时，部署管理员可以使用 **网络管理** 为实例创建和编辑以及删除网络。用于同一集群内实例间的通信，当为网络设置外部ip时，所有可以访问集群该节点的机器都可以访问到该实例

- 当某个实例创建了网络之后， 部署管理员可以通过 **域名管理** 为该网络创建域名，所有可以访问外网的机器都可以通过创建域名时设置的域名地址访问该实例。

- 当实例运行之后，部署管理员可以通过 **容器管理** 查看实例容器的信息，包括容器的状态以及容器的日志

- 项目成员和项目所有者没有应用部署的权限，但是项目成员和项目所有者可以查看实例的详情，网络，域名，容器信息。

## 快速开始

 - 如果您是第一次使用接触部署流水线，在学习阅读本章节之前，请阅读[快速入门手册](../../quick-start/agile/)或观看对应[视频教程](../../quick-start/video-tutorial/)。
 - 如果您对部署流水线已经非常熟悉，请使用文档搜索功能，查找您需要了解的内容。

## 前置条件

 - Choerodon的部署流水线也是基于应用管理的，在使用Choerodon部署流水线之前，需要先了解和学习[Choerodon的应用管理](../application-management)。
 - Choerodon的部署流水线也是基于开发流水线的，在使用Choerodon部署流水线之前，需要先了解和学习[Choerodon的开发流水线](../development-pipeline)。
 - 系统用户必须具有`部署管理员`，或者`项目所有者`，或者`项目成员`角色，才能使用部署流水线。所以，在使用部署流水线之前，您需要为您的用户分配上述。关于权限管理，请参阅[项目角色分配](.././system-configuration/project/role-assignment/)。
 - 由于部署流水线环境流水线功能中需要创建agent环境客户端，请确保本机能够用ssh连接到系统所在的k8s集群且集群内安装好了helm客户端，有关[helm](https://docs.helm.sh/)和 [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/)的相关知识请自行去对应官网理解
 - 由于部署流水线应用部署部分用到了Helm repository,请确保在部署Choerodon平台的时候已经部署好helm,详情请见 [Chartmuseum部署](../../installation-configuration/steps/parts/base/chartmuseum)

## 权限说明
1. `部署管理员`拥有以下权限：

    模块|操作
    |---|---|
    环境流水线|创建，升级环境，停用，启用环境以及修改环境信息
    应用部署|选择应用和应用版本以及对应的环境，修改对应的values部署文件，分布创建或者替换实例
    实例|修改配置信息，升级，停止，重启实例，查询实例详情，其中包括实例下的k8s对象，目前平台支持查询的有(Pod,Servic,e,Ingress,Deployment,ReplicaSet)，实例部署时阶段信息（Job）以及上次部署的values的展示
    网络|为部署的应用的某个版本的实例在其部署的环境上创建网络，编辑网络
    域名|为部署的应用的某个版本的实例在其部署的环境上的网络创建域名，编辑域名
    容器|查询容器详情，以及查看容器的日志

2. `项目成员，项目所有者`拥有除了应用部署之外其它功能的查看权限

以下将围绕部署流水线各个功能的使用进行详细的介绍，包括 **环境流水线**、**应用部署**、**实例**、**网络**、**域名**、**容器**。

## 视频介绍

<div class="tutorial-img" id="tutorial-img">
    <div class="col-lg-6 col-md-6 col-xs-12 tutorial">
        <div class="tutorial-content" data-src="y0735zgqr1j">
            <div class="mask">
                <div class="head">
                    <div class="title ellipsis">部署流水线</div>
                    <a href="https://v.qq.com/x/page/y0735zgqr1j.html" target="_blank"><i class="iconfont icon-shijian" title="稍后观看"></i></a>
                </div>
                <div class="foot">
                    <div class="description">以部署并访问应用示例来介绍持续交付中部署流水线使用方法</div>
                    <div class="info">
                        <div class="begin">
                            <i class="iconfont icon-play-button"></i>
                            <div class="no-warp">开始</div>
                        </div>
                        <div class="time">6min</div>
                    </div>
                </div>
            </div>
            <img class="n-max " src="/docs/quick-start/video-tutorial/img/agile.jpg">
        </div>
    </div>
</div>
<div class="tutorial-video" id="tutorial-video">
    <div class="bg"></div>
    <iframe frameborder="0" src='' allowfullscreen="true" quality="high"></iframe>
    <div class="iconfont icon-guanbi"></div>
</div>

{{< docdir >}}
