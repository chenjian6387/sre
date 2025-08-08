# 09-jenkins流水线pipeline

# 1.什么是流水线

学习jenkins的官网

> https://www.jenkins.io/zh/doc/book/pipeline/

Jenkins只是创建一个新的 Jenkins job 并不能构建一条流水线。

可以把 Jenkins 看做一个遥控器，在这里点击按钮即可。

当你点击按钮时会发生什么取决于遥控器要控制的内容。

Jenkins 为其他应用程序 API、软件库、构建工具等提供了一种插入 Jenkins 的方法，它可以执行并自动化任务。

Jenkins 本身不执行任何功能，但是随着其它的插件而变得越来越强大。

jenkins的流水线(pipeline)是一个单独的概念，指的是按顺序连接在一起的事件或作业组：

> “ *流水线(pipeline)*”是可以执行的一系列事件或作业。

理解流水线的最简单方法是可视化一系列阶段，如下所示：

![image-20210225100028272](http://book.bikongge.com/sre/2024-linux/image-20210225100028272.png)

在这里，你应该看到两个熟悉的概念： *阶段(Stage)*和 *步骤(Step)*。

- 阶段stage：一个包含一系列步骤的块。阶段块可以命名为任何名称；它用于展示可视化流水线过程。
  - `stage就是起个名字，比如叫做【构建】、【获取代码】、【部署代码】`
- 步骤：表明要做什么的任务。步骤定义在阶段块内。
  - 步骤step就是具体要执行的命令
  - 例如简单的，`echo 'hello,world'`

# 2.jenkins为什么用流水线

什么是Pipeline？

简单来说，就是一套运行于Jenkins上的工作流框架，将原本独立运行于单个或者多个节点的任务连接起来，实现单个任务难以完成的复杂发布流程（实用场景：将多个Jenkins构建任务轻松集成）。

咱们目前是基于一个单个的job，完成某一个构建任务。

Pipeline的实现方式是一套Groovy DSL，任何发布流程都可以表述为一段Groovy脚本，并且Jenkins支持从代码库直接读取脚本，从而实现了Pipeline as Code的理念。

```
本质上，jenkins是一个自动化引擎，它支持许多自动模式。

流水线向Jenkins添加了一组强大的工具，支持用例、简单的持续集成到全面的持续交付流水线。 通过对一系列的发布任务建立标准的模板，用户可以利用更多流水线的特性，比如：

代码化: 流水线是在代码中实现的，通常会存放到源代码控制，使团队具有编辑、审查和更新他们项目的交付流水线的能力。

耐用性：流水线可以从Jenkins的master节点重启后继续运行。

可暂停的：流水线可以由人功输入或批准继续执行流水线。

解决复杂发布： 支持复杂的交付流程。例如循环、并行执行。

可扩展性： 支持扩展DSL和其他插件集成。
```

**使用条件**

要使用Jenkins Pipeline，需要： Jenkins 2.x或更高版本、Pipeline插件

## 2.1 了解groovy

```
Groovy是一种基于JVM（Java虚拟机）的敏捷开发语言，它结合了Python、Ruby和Smalltalk的许多强大的特性，Groovy 代码能够与 Java 代码很好地结合，也能用于扩展现有代码。
由于其运行在 JVM 上的特性，Groovy 可以使用其他 Java 语言编写的库。


为什么要使用Pipeline？对于实践微服务的团队，产品有很多服务组成，传统的在Jenkins中集中进行job配置的方式会成为瓶颈，微服务团队会将CI Job的配置和服务的发布交给具体负责某个服务的团队，这正需要Pipeline as Code； 除此之外，一次产品的发布会涉及到多个服务的协同发布，用单个CI Job实现起来会十分困难，使用Pipeline可以很好的完成这个需求。
```

# 3.Pipeline的基本概念和Jenkinsfile

- Node：一个Node就是一个Jenkins节点，可以是Master，也可以是Slave，是Pipeline中具体Step的运行环境。
- Stage：一个Pipeline有多个Stage组成，每个Stage包含一组Step。注意一个Stage可以跨多个Node执行，即Stage实际上是Step的逻辑分组。
- Step：是最基本的运行单元，可以是创建一个目录、从代码库中checkout代码、执行一个shell命令、构建Docker镜像、将服务发布到Kubernetes集群中。Step由Jenkins和Jenkins各种插件提供。

将node、stage、step的Groovy DSL写在一个Jenkinsfile文件中，Jenkinsfile会被放到代码库的根目录下。

## 3.0 北京地铁和pipieline的理解

关于Jenkins流水线的运行我们可以抽象一下，例如：

- 可以把流水线(pipeline)想象成13号线地铁
- 把流水线的阶段(stage)想象成地铁的每一个站点
- 把流水线脚本(jenkinsfile)想象成地铁线路图

这就是流水线的多样性，每条线路都有不同的站点。

![images](http://book.bikongge.com/sre/2024-linux/25087d277f6965d2d30a94f8dce65dc4-20220714200406998.png)

```
北京地铁=======jenkins

地铁的运行路线图=============jenkinsfile

地图中要经过N个站=======stage
```

## 3.0.0 什么是pipeline/jenkinsfile

Pipeline就是写jenkinsfile脚本

```
Jenkins的Pipeline通过Jenkinsfile进行描述（类似于Dockerfile）（镜像通过dockerfile描述，pipeline通过jenkinsfile描述）

Jenkinsfile是Jenkins的特性（pipeline as code）

Pipeline是Jenkins的核心功能，提供一组可扩展的工具。

通过Pipeline 的DSL(领域特定语言)语法可以完成从简单到复杂的交付流水线实现。
```

**Jenkinsfile**

```
- Jenkinsfile使用两种语法进行编写，分别是声明式和脚本式。
- 声明式和脚本式的流水线从根本上是不同的。
- 声明式是jenkins流水线更友好的特性。
- 脚本式的流水线语法，提供更丰富的语法特性。
- 声明式流水线使编写和读取流水线代码更容易设计。
```

## 3.1 什么是jenkinsfile

```
jenkinsfile 就是一个脚本文件，放在项目的根目录下，能够被jenkins识别并自动触发pipeline流程

jenkinsfile的编写可以参考jenkins官网：https://www.jenkins.io/zh/doc/book/pipeline/syntax/)
```

![image-20210225101225848](http://book.bikongge.com/sre/2024-linux/image-20210225101225848.png)

# 4.安装blue ocean插件

用于可视化的展现jenkins Pipeline的执行过程

> https://www.jenkins.io/doc/book/blueocean/

该插件超哥已经给大家导入好了，要么就可以自己进行插件管理，在线安装

![image-20220714201405958](http://book.bikongge.com/sre/2024-linux/image-20220714201405958.png)

------

![image-20220714201418468](http://book.bikongge.com/sre/2024-linux/image-20220714201418468.png)