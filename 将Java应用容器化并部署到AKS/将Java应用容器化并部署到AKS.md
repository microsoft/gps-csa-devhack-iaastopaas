# 将 Java 应用容器化并部署到 Azure

**单元1: 简介**

-   3 分钟

假设你是一名 Java 开发人员，负责生成和部署将在本地服务器上运行的应用。
编排这些服务器、依赖项和规模等都是一个极具挑战性的过程。

与模拟硬件的虚拟机不同的是，容器直接在主机操作系统、内核和硬件上运行，本质上只是另一个进程。
因此，容器所需的系统资源更少，从而使得内存占用更小，开销明显减少，应用启动速度更快，并成为按需缩放的绝佳用例。

使用容器，可以确保应用和依赖项始终在容器映像中隔离，并准备好进行大规模部署。

在本模块中，你将首先容器化一个 Java 应用。
为方便起见，我们选择了一个现有的 Java
应用供你使用。 [[航空公司的航班预订系统]{.underline}](https://github.com/Azure-Samples/containerize-and-deploy-Java-app-to-Azure)来自开放的
Internet 并在 [[MIT
许可]{.underline}](https://github.com/git/git-scm.com/blob/main/MIT-LICENSE.txt)下共享，是一个完全响应式的、基于
Web 的航班预订系统示例，它以一个示例航空公司为基础，通过使用 Java
Servlets 和 Java Server Pages (JSP) 构建的模型视图控制器 (MVC)
体系结构生成。

然后，你将构造 Dockerfile 并编写生成容器映像所需的 Docker 指令。

然后在本地运行容器映像并测试应用。

接下来，将容器映像推送到 Azure 容器注册表，并部署到 Azure Kubernetes
服务。

在学习完本模块后，你将能够容器化 Java 应用，将容器映像推送到 Azure
容器注册表，然后部署到 Azure Kubernetes 服务。

你将使用你自己的 Azure
订阅（具有创建、更新和删除资源的访问权限）来部署本模块中的资源。
如果没有 Azure 订阅，请在开始之前创建一个[免费帐户]{.underline}。

** 重要**

为避免在 Azure 订阅中产生不必要的费用，请记得在完成此模块后取消预配
Azure 资源。

**学习目标**

学完本模块后，你将能够：

-   容器化 Java 应用

-   为 Java 应用生成容器映像

-   在本地运行容器映像

-   向 Azure 容器注册表推送容器映像

-   将容器映像部署到 Azure Kubernetes 服务

**先决条件**

作为一名 Java 开发人员，你已经熟悉了如何生成应用。
完成本模块中的练习后，你将使用个人 Azure 帐户。 请确保你拥有以下资源：

-   具有创建、更新和删除资源访问权限的 Azure 订阅。

-   在本地安装有 Docker CLI、Git CLI 和 Azure CLI（2.12 或更高版本）。

# 单元2:设置 Azure 环境

-   6 分钟

在本单元中，你将使用 Azure CLI 创建后面各单元所需的 Azure 资源。

## 使用 Azure CLI，执行以下步骤

** 备注**

为了节省时间，你将指示 Azure 先预配资源，然后继续下一个单元。 创建 Azure
Kubernetes 群集可能需要 10 分钟。
可以选择在后台运行，同时继续下一个单元。

### 向 Azure 资源管理器进行身份验证

登录：

Bash复制

az login

### 选择一个 Azure 订阅

Azure 订阅是用于在 Azure 中预配资源的逻辑容器。
你需要找到计划在本模块中使用的订阅 ID (SubscriptionId)。 列出 Azure
订阅：

Bash复制

az account list \--output table

确保你使用的是允许为本模块创建资源的 Azure 订阅，用你选择的订阅 ID
(SubscriptionId) 进行替换：

Bash复制

az account set \--subscription \"\<YOUR_SUBSCRIPTION_ID>\"

### 定义局部变量

为了简化将进一步执行的命令，请设置以下环境变量：

** 备注**

需要将 \<YOUR_AZURE_REGION> 替换为你选择的区域，例如：eastus

需要将 \<YOUR_CONTAINER_REGISTRY> 替换为一个唯一的值，因为它用于在创建
Azure 容器注册表时为其生成唯一的
FQDN（完全限定的域名），例如：someuniquevaluejavacontainerregistry

需要将 \<YOUR_UNIQUE_DNS_PREFIX_TO_ACCESS_YOUR_AKS_CLUSTER>
替换为一个唯一的值，因为它用于在创建 Azure Kubernetes
群集时为其生成唯一的
FQDN（完全限定的域名），例如：someuniquevaluejavacontainerizationdemoaks

Bash复制

AZ_RESOURCE_GROUP=javacontainerizationdemorg

AZ_CONTAINER_REGISTRY=\<YOUR_CONTAINER_REGISTRY>

AZ_KUBERNETES_CLUSTER=javacontainerizationdemoaks

AZ_LOCATION=\<YOUR_AZURE_REGION>

AZ_KUBERNETES_CLUSTER_DNS_PREFIX=\<YOUR_UNIQUE_DNS_PREFIX_TO_ACCESS_YOUR_AKS_CLUSTER>

### 创建 Azure 资源组

Azure 资源组是位于 Azure 订阅中的 Azure 容器，用于存放 Azure
解决方案的相关资源。 创建资源组：

Bash复制

az group create \\

\--name \$AZ_RESOURCE_GROUP \\

\--location \$AZ_LOCATION \\

\| jq

** 备注**

本模块使用 jq 工具，该工具默认安装在 [**Azure Cloud
Shell**](https://shell.azure.com/) 上，用于显示 JSON
数据并使其更具可读性。

如果不想使用 jq 工具，则可以安全地删除此模块中所有命令的 \| jq 部分。

### 创建 Azure 容器注册表

使用 Azure 容器注册表，可以生成、存储和管理容器映像，这些映像最终会存储
Java 应用的容器映像。 创建容器注册表：

Bash复制

az acr create \\

\--resource-group \$AZ_RESOURCE_GROUP \\

\--name \$AZ_CONTAINER_REGISTRY \\

\--sku Basic \\

\| jq

将 Azure CLI 配置为使用此新创建的 Azure 容器注册表：

Bash复制

az configure \\

\--defaults acr=\$AZ_CONTAINER_REGISTRY

向新创建的 Azure 容器注册表进行身份验证：

Bash复制

az acr login -n \$AZ_CONTAINER_REGISTRY

### 创建 Azure Kubernetes 群集

你需要一个 Azure Kubernetes 群集，用于在其中部署 Java 应用（容器映像）。
创建 AKS 群集：

Bash复制

az aks create \\

\--resource-group \$AZ_RESOURCE_GROUP \\

\--name \$AZ_KUBERNETES_CLUSTER \\

\--attach-acr \$AZ_CONTAINER_REGISTRY \\

\--dns-name-prefix=\$AZ_KUBERNETES_CLUSTER_DNS_PREFIX \\

\--generate-ssh-keys \\

\| jq

** 备注**

创建 Azure Kubernetes 群集可能需要 10
分钟，运行上述命令后，可以选择让它在 Azure CLI
选项卡中继续，然后继续下一单元。

**单元3:容器化 Java 应用**

-   10 分钟

在本单元中，你将容器化 Java 应用程序。

如前所述，容器直接在主机操作系统、内核和硬件上运行，本质上只是另一个系统进程。
容器所需的系统资源较少，因此内存占用更小、开销更少且应用程序启动速度更快。
这些是按需缩放的不错用例。

有 Windows 容器和 Linux 容器。 在本模块中，你将利用广泛使用的 Docker
运行时来生成 Linux 容器映像。 然后将 Linux
容器映像部署到本地计算机的主机操作系统。 最后，同样重要的是，将 Linux
容器映像部署到 Azure Kubernetes 服务。

**Docker 概述**

Docker 运行时用于生成、拉取、运行和推送容器映像。
下图描述了这些用例，接着是每个用例/Docker 命令的说明。

![Screenshot showing Docker
commands.](.//media/image1.png){width="5.768055555555556in"
height="2.8243055555555556in"}

  --------------------------------------------------------------------------------------------------------------------------------
  **Docker   **说明**
  命令**     
  ---------- ---------------------------------------------------------------------------------------------------------------------
  docker     生成容器映像，实质上是 Docker 最终从映像创建正在运行的容器所需的指令/层。 此命令的结果是生成一个映像。
  build      

  docker     容器从映像中初始化，这些映像拉取自注册表（例如 Azure 容器注册表），这也是拉取 Azure Kubernetes 服务的位置。
  pull       此命令的结果是将在 Azure 中对映像进行网络拉取。
             请注意，你可以选择在本地拉取映像，这在生成需要应用程序可能同时需要的依赖项/层（例如应用程序服务器）的映像时很常见。

  docker run 一个运行中的映像实例是一个容器，所有运行和与运行中的容器应用程序交互所需的层都是用这个命令执行的。
             此命令的结果是在主机操作系统上运行的应用程序进程。

  docker     Azure 容器注册表将存储映像，以便它们随时可用，并与 Azure 的部署和缩放密切相关。
  push       
  --------------------------------------------------------------------------------------------------------------------------------

**克隆 Java 应用程序**

首先，你要将航空公司的航班预订系统存储库和 cd 克隆到航空公司 Web
应用程序项目文件夹中。

** 备注**

如果在 CLI 选项卡中已成功创建 Azure Kubernetes
服务，请使用该项，否则，如果它仍在运行，请在要克隆航空公司的航班预订系统的位置打开一个新的选项卡和
cd。

（可选）如果已安装 Java 和 Maven，则可在 CLI
中运行以下命令，体验一下在没有 Docker 的情况下生成应用程序。
如果没有安装 Java 和 Maven，可以安全地跳转到标题为"构造 Docker
文件"的下一部分。在该部分中，你将使用 Docker 来提取 Java 和 Maven
来代表你执行生成。

在 CLI 中运行以下命令：

Bash复制

git clone
https://github.com/Azure-Samples/containerize-and-deploy-Java-app-to-Azure.git

在 CLI 中运行以下命令：

Bash复制

cd containerize-and-deploy-Java-app-to-Azure/Project/Airlines

（可选）如果已安装 Maven 和 JDK(8) 或更高版本，可以在 CLI
中运行以下命令：

Bash复制

mvn clean install

** 备注**

mvn clean install 命令用于阐明不使用 Docker
多阶段生成的操作难题，我们将在后面进行介绍。
同样，此步骤是可选的，无论采用哪种方式，你都可以安全地继续，无需执行
Maven 命令。

Maven 应已成功生成航空公司的航班预订系统 Web 应用程序存档项目
FlightBookingSystemSample-0.0.-SNAPSHOT.war，如下图所示：

Bash复制

\[INFO\] Building war:
/mnt/c/Users/chtrembl/dev/git/containerize-and-deploy-Java-app-to-Azure/Project/FlightBookingSystemSample/target/FlightBookingSystemSample-0.0.1-SNAPSHOT.war

\[INFO\]
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

\[INFO\] BUILD SUCCESS

\[INFO\]
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

\[INFO\] Total time: 17.698 s

\[INFO\] Finished at: 2021-09-28T15:18:07-04:00

\[INFO\]
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

假设你是一名 Java 开发人员，并且刚刚生成了此
FlightBookingSystemSample-0.0.1-SNAPSHOT.war，下一步就是与操作工程师协作，将此项目部署到本地服务器或虚拟机。
要使应用程序成功启动并运行，需要提供服务器和虚拟机，并配置所需的依赖项。
这是具有挑战性且非常耗时的工作，尤其是在应用程序负载增加时，更是如此。
有了容器，这些挑战就得到了缓解。

**构造 Docker 文件**

此时，你已准备好构造 Docker 文件。 Dockerfile
是一个文本文档，其中包含用户可在命令行上执行的所有命令，用于组装容器映像，其中每个都是层（可以缓存这些层以提高效率），它们基于彼此构建。

例如，航空公司的航班预订系统需要部署到应用程序服务器，并在其中运行。
应用程序服务器未打包在 FlightBookingSystemSample-0.0.1-SNAPSHOT.war
中，它是 FlightBookingSystemSample-0.0.1-SNAPSHOT.war 运行、侦听和处理
HTTP 请求、管理用户会话和帮助航班预订所需的外部依赖项。
如果这是一种传统的非容器化部署，则在向其部署
FlightBookingSystemSample-0.0.1-SNAPSHOT.war
之前，操作工程师将在一些物理服务器和/或虚拟机上安装和配置应用程序服务器。
这些操作工程师还需要确保计算机上使用的 JDK（mvn clean install
使用它来编译 war）实际上与应用程序服务器使用的同一 JRE 相对应。
管理这些依赖项非常困难且耗时。

通过
Dockerfile，你可以编写自动完成此操作所需的层（指令），通过在所需的步骤中分层来确保航空公司的航班预订系统拥有所有部署到
Docker 容器运行时所需的依赖项。
当你开始考虑在非计划的时间间隔内按需扩展时，这就非常引人注目了。
值得注意的是，每个层都利用了 Docker
缓存，其中包含每个指令里程碑的容器映像的状态，优化了计算时间和重复使用。
如果层不发生变化，则将使用缓存层。 缓存层的常见用例包括：Java
运行时、应用程序服务器和/或航空公司的航班预订系统 Web
应用程序的其他依赖项。
如果某个版本在以前的缓存层上发生更改，将会创建一个新的缓存条目。

下图描述了容器映像的各个层，你会注意到，所有顶层都是在以前的只读层基础上构建的读/写航空公司的航班预订系统
Web 应用程序层，所有这些层都是通过 Dockerfile 中的命令生成的。

![Screenshot showing the Docker
layers.](.//media/image2.png){width="5.768055555555556in"
height="4.257638888888889in"}

Docker
还具有多阶段生成的概念，借助该功能，可以创建具有更好的缓存和更小的安全内存占用的更小容器映像，使
Dockerfile 随着时间的推移得以持续优化和维护。 例如，可用于完成应用程序
FlightBookingSystemSample-0.0.1-SNAPSHOT.war
的编译和容器映像本身的构建的指令，使
FlightBookingSystemSample-0.0.1-SNAPSHOT.war
编译的剩余部分保持不变，从长远来看，这将减少内存占用，当你开始考虑这些在网络中传输的映像时，这将带来好处。
对于多阶段生成，可以在 Dockerfile 中使用多个 FROM 语句。 每个 FROM
指令都可以使用不同的基础映像，其中每个语句都以干净的盖板开始，删除缓存层中通常可能会缓存的所有不必要的文件。

必须确保应用程序是由与运行时将隔离在容器映像中的同一 JRE 对应的相同 JDK
生成的。 在下面的示例中，你将看到一个生成阶段，它利用特定版本的 Maven
和特定版本的 JDK 来编译 FlightBookingSystemSample-0.0.1-SNAPSHOT.war。
此阶段确保执行该阶段的任何 Docker 运行时都将获取 Dockerfile
作者指定的预期生成的字节代码（否则，操作工程师就必须将其 Java
和应用程序服务器运行时与开发人员的 Java
和应用程序服务器运行时交叉引用）。 然后，包阶段将使用特定版本的 Tomcat
和与生成阶段中的 JDK 相对应的 JRE。
同样，这样做是为了确保所有依赖项（Java 开发工具包 JDK、Java Runtime
Environment
JRE、应用程序服务器）都受到控制和隔离，以确保在运行此映像的所有计算机上都能看到预期行为。

同样值得注意的是，在此多阶段生成中，从技术上讲，无需在系统上安装 Maven
和 Java，Docker
会拉取它们用于生成应用程序和应用程序运行时，这样可以避免出现任何潜在的版本冲突和意外行为，当然，除非你是在
Docker 之外编译代码和生成项目。

下图根据 Dockerfile 中指定的命令描述了多阶段生成和每个阶段发生的情况。
在阶段 0，即生成阶段，你会注意到，这是编译航空公司的航班预订系统 Web
应用程序和生成 FlightBookingSystemSample-0.0.1-SNAPSHOT.war 的阶段。
此阶段在使用哪些版本的 Maven 和 Java 来编译此应用程序方面实现了一致性。
一旦创建完 FlightBookingSystemSample-0.0.1-SNAPSHOT.war，这就是阶段
1（运行时阶段）所需的唯一层，之前的所有层都可以被丢弃。 然后，Docker
将使用这个来自阶段 0 的 FlightBookingSystemSample-0.0.1-SNAPSHOT.war
层来构造运行时所需的剩余层，本示例中是配置应用程序服务器和启动应用程序。

![Screenshot showing the Docker multi-stage
build.](.//media/image3.png){width="5.768055555555556in"
height="5.768055555555556in"}

在项目的根目录
(containerize-and-deploy-Java-app-to-Azure/Project/Airlines)
中，创建一个名为 Dockerfile 的文件：

Bash复制

vi Dockerfile

将以下内容添加到 Dockerfile 中，然后保存并退出：

Dockerfile复制

\#

\# Build stage

\#

FROM maven:3.6.0-jdk-11-slim AS build

WORKDIR /build

COPY pom.xml .

COPY src ./src

COPY web ./web

RUN mvn clean package

\#

\# Package stage

\#

FROM tomcat:8.5.72-jre11-openjdk-slim

COPY tomcat-users.xml /usr/local/tomcat/conf

COPY \--from=build /build/target/\*.war
/usr/local/tomcat/webapps/FlightBookingSystemSample.war

EXPOSE 8080

CMD \[\"catalina.sh\", \"run\"\]

** 备注**

或者，你的项目根目录中的 Dockerfile_Solution 包含所需的内容。

如你所见，这个 Docker 文件的生成阶段包含六个指令。

  -------------------------------------------------------------------------------------
  **Docker   **说明**
  命令**     
  ---------- --------------------------------------------------------------------------
  FROM       FROM maven 将是生成 FlightBookingSystemSample-0.0.1-SNAPSHOT.war
             的基础层，它是特定版本的 Maven 和特定版本的
             JDK，用于确保在运行此生成的所有计算机上发生相同的字节代码编译。

  WORKDIR    WORKDIR
             用于在任何给定时间定义容器的工作目录，在本例中，编译的项目将位于此位置。

  复制       COPY 从 Docker 客户端的当前目录中添加文件。 设置 Maven
             编译所需的文件，Docker 上下文将需要 pom.xml

  复制       设置 Maven 编译所需的文件。 Docker 上下文需要包含航空公司的航班预订系统
             Web 应用程序的 src 文件夹

  复制       设置 Maven 编译所需的文件。 Docker 上下文需要包含航空公司的航班预订系统
             Web 应用程序依赖项的 Web 文件夹

  运行       RUN mvn clean package 指令用于在当前映像的基础上执行任何命令。
             在本例中，RUN 用于执行 Maven 生成，这将编译
             FlightBookingSystemSample-0.0.1-SNAPSHOT.war
  -------------------------------------------------------------------------------------

如你所见，此 Docker 文件的包阶段包含五个指令。

  -------------------------------------------------------------------------------------------------------------------------------
  **Docker   **说明**
  命令**     
  ---------- --------------------------------------------------------------------------------------------------------------------
  FROM       FROM tomcat 是此容器映像将基于其构建的基本层。 航空公司的航班预订系统容器映像将是基于 tomcat 映像生成的映像。 Docker
             运行时将尝试在本地查找 tomcat 映像，如果它没有此版本，它将从注册表中拉取一个。 如果你要检查此处引用的 tomcat
             映像，你将看到它是使用多个其他层生成的，所有这些层使其可以作为一个打包的应用程序服务器容器映像重用，以便用户在部署
             Java 应用程序时使用。 在模块中选择并测试了 tomcat:8.5.72-jre11-openjdk-slim。 请注意，当 Docker 识别出这第二个 FROM
             指令后，第一个生成阶段的所有先前层都会消失。

  复制       COPY tomcat-users.xml 将管理航空公司的航班预订系统用户的 tomcat-users.xml 文件（使用 Tomcat
             标识在源代码管理中进行管理，这通常发生在外部标识管理系统中）复制到 tomcat
             容器映像中，这样，每次创建容器映像时，它都存在于容器映像中

  ADD        ADD target/\*.war /usr/local/tomcat/webapps/FlightBookingSystemSample.war 将 maven 编译的
             FlightBookingSystemSample-0.0.1-SNAPSHOT.war 复制到 tomcat images webapps 文件夹，以确保在 Tomcat
             初始化时，它将实际查找要安装在应用程序服务器上的 FlightBookingSystemSample-0.0.1-SNAPSHOT.war。

  EXPOSE     需要 EXPOSE 8080，因为 Tomcat 被配置为侦听端口 8080 上的流量，这可确保 Docker 进程将侦听此端口。

  CMD        CMD 指令用于设置运行容器时要执行的命令。 在本例中，CMD \[\"catalina.sh\", \"run\"\] 指示 Docker 初始化 Tomcat
             应用程序服务器。
  -------------------------------------------------------------------------------------------------------------------------------

** 备注**

如果 FROM tomcat 行上没有版本标记，将应用最新的版本。
通常，你将需要利用版本标记（记住，将应用缓存，因此，如果层不断变化，会产生带宽、延迟、计算时间和/或未经测试的生成/层的副作用。）在本模块中，我们预先选择了特定的
Maven、Tomcat、Java JRE/JDK 标记，这些标记经过测试，可在运行时用于
FlightBookingSystemSample-0.0.1-SNAPSHOT.war。

有关 Dockerfile
构造的详细信息，请访问 [[https://docs.docker.com/engine/reference/builder/]{.underline}](https://docs.docker.com/engine/reference/builder/)

**单元4: 为 Java 应用生成并运行容器映像**

-   4 分钟

在本单元中，你将生成并运行容器映像。
如前文所述，正在运行的映像实例是一个容器。

**生成容器映像**

你已成功构造 Dockerfile，现在可以指示 Docker 为你生成容器映像。

** 备注**

确保将 Docker 运行时配置为生成 Linux 容器。 这一点非常重要，因为使用的
Dockerfile 会引用用于 Linux 体系结构的容器映像 (JDK/JRE)。

docker
build 是用于构建容器映像的命令。 -t 参数将用于指定容器标签，. 是用于查找
Dockerfile 的 Docker 位置。 在 CLI 中运行以下命令：

Bash复制

docker build -t flightbookingsystemsample .

你将看到如下内容：

Bash复制

docker build -t flightbookingsystemsample .

Sending build context to Docker daemon 101.3MB

Step 1/11 : FROM maven:3.6.0-jdk-11-slim AS build

3.6.0-jdk-11-slim: Pulling from library/maven

27833a3ba0a5: Pull complete

16d944e3d00d: Pull complete

6aaf465b8930: Pull complete

0684138f4cb6: Pull complete

67c4e741e688: Pull complete

933857515267: Pull complete

4f31e2918c2c: Pull complete

70a0a987b087: Pull complete

8369c7ef3731: Pull complete

7a73ce905393: Pull complete

c702b567a1e8: Pull complete

Digest:
sha256:4f0face24d2f79439a8fa394555b09be99c9ad537b9b19983fb8cc358818a42d

Status: Downloaded newer image for maven:3.6.0-jdk-11-slim

\-\--\> c7428be691f8

Step 2/11 : WORKDIR /build

\-\--\> Running in f1656ab15874

Removing intermediate container f1656ab15874

\-\--\> d9bfeb518c86

Step 3/11 : COPY pom.xml .

\-\--\> a60aab61d487

Step 4/11 : COPY src ./src

\-\--\> 049803b88a20

Step 5/11 : COPY web ./web

\-\--\> 8d98ddb1fbc3

Step 6/11 : RUN mvn clean package

\-\--\> Running in 71a462c5b3ad

\[INFO\] Scanning for projects\...

\...

\[INFO\] Packaging webapp

\[INFO\] Assembling webapp \[FlightBookingSystemSample\] in
\[/build/target/FlightBookingSystemSample-0.0.1-SNAPSHOT\]

\[INFO\] Processing war project

\[INFO\] Copying webapp webResources \[/build/web\] to
\[/build/target/FlightBookingSystemSample-0.0.1-SNAPSHOT\]

\[INFO\] Copying webapp resources \[/build/src/main/webapp\]

\[INFO\] Building war:
/build/target/FlightBookingSystemSample-0.0.1-SNAPSHOT.war

\[INFO\]
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

\[INFO\] BUILD SUCCESS

\[INFO\]
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

\[INFO\] Total time: 26.811 s

\[INFO\] Finished at: 2021-10-09T01:58:31Z

\[INFO\]
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

Removing intermediate container 71a462c5b3ad

\-\--\> 3d40f8a4f631

Step 7/11 : FROM tomcat:8.5.72-jre11-openjdk-slim

8.5.72-jre11-openjdk-slim: Pulling from library/tomcat

bd897bb914af: Pull complete

0cc7fec72146: Pull complete

14c358bab58a: Pull complete

c12f81e19ff2: Pull complete

89238a07057c: Pull complete

46896a83c0d8: Pull complete

15f6001bf0c8: Pull complete

34374d37175f: Pull complete

Digest:
sha256:807efbd54faa7342c1f75f1466889148fda1f299b81dc40404312352a2086a5f

Status: Downloaded newer image for tomcat:8.5.72-jre11-openjdk-slim

\-\--\> 77903b2cfd74

Step 8/11 : COPY tomcat-users.xml /usr/local/tomcat/conf

\-\--\> c540464832e7

Step 9/11 : COPY \--from=build /build/target/\*.war
/usr/local/tomcat/webapps/FlightBookingSystemSample.war

\-\--\> 6d1eb5d1568c

Step 10/11 : EXPOSE 8080

\-\--\> Running in 753385de3d77

Removing intermediate container 753385de3d77

\-\--\> f744382dd869

Step 11/11 : CMD \[\"catalina.sh\", \"run\"\]

\-\--\> Running in 58a822ee7a4a

Removing intermediate container 58a822ee7a4a

\-\--\> a0b73d3f3f91

Successfully built a0b73d3f3f91

Successfully tagged flightbookingsystemsample:latest

正如前面所看到的那样，Docker
已经执行了之前在前面的单元中编写的行中的指令。
每个指令都是按顺序排列的步骤。 再次重新运行 docker
build 命令，注意步骤中的差异，你会注意到未更改的层的 \-\--\> Using
cache。 如果你（在重新运行 docker
build 命令之前）没有更改应用，会注意到所有缓存层，因为二进制文件保持不变，并且可从
Docker 缓存派生）。
在优化容器映像以及与生成它们所花的时间相关的计算成本时，这是一个重要的启示。

Docker 还可以显示驻留的可用映像。 这有助于查看可运行的内容。 在 CLI
中运行以下命令：

Bash复制

docker image ls

你将看到如下内容：

Bash复制

docker image ls

REPOSITORY TAG IMAGE ID CREATED SIZE

flightbookingsystemsample latest cda4f5b459f1 About an hour ago 268MB

**运行容器映像**

现在，你已成功生成容器映像，接下来可以运行它。

docker run 是用于运行容器映像的命令。 -p ####:#### 参数将用于在运行时将
localhost
HTTP（冒号前的第一个端口）流量转发到容器（冒号后的第二个端口）。
请记住，在 Dockerfile 中，Tomcat 应用服务器侦听的是端口 8080 上的 HTTP
流量，因此这是需要公开的容器端口。
最后，需要映像标记 flightbookingsystemsample 来指示 Docker
运行哪个映像。 在 CLI 中运行以下命令：

Bash复制

docker run -p 8080:8080 flightbookingsystemsample

你将看到如下内容：

Bash复制

docker run -p 8080:8080 flightbookingsystemsample

NOTE: Picked up JDK_JAVA_OPTIONS:
\--add-opens=java.base/java.lang=ALL-UNNAMED
\--add-opens=java.base/java.io=ALL-UNNAMED
\--add-opens=java.base/java.util=ALL-UNNAMED
\--add-opens=java.base/java.util.concurrent=ALL-UNNAMED
\--add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED

02-Aug-2021 20:50:22.682 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Server version
name: Apache Tomcat/9.0.50

02-Aug-2021 20:50:22.687 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Server built: Jun
28 2021 08:46:44 UTC

02-Aug-2021 20:50:22.687 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Server version
number: 9.0.50.0

02-Aug-2021 20:50:22.688 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log OS Name: Linux

02-Aug-2021 20:50:22.688 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log OS Version:
5.10.16.3-microsoft-standard-WSL2

02-Aug-2021 20:50:22.689 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Architecture:
amd64

02-Aug-2021 20:50:22.690 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Java Home:
/usr/local/openjdk-11

02-Aug-2021 20:50:22.694 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log JVM Version:
11.0.12+7

02-Aug-2021 20:50:22.694 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log JVM Vendor: Oracle
Corporation

02-Aug-2021 20:50:22.695 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log CATALINA_BASE:
/usr/local/tomcat

02-Aug-2021 20:50:22.696 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log CATALINA_HOME:
/usr/local/tomcat

02-Aug-2021 20:50:22.729 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Command line
argument: \--add-opens=java.base/java.lang=ALL-UNNAMED

02-Aug-2021 20:50:22.730 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Command line
argument: \--add-opens=java.base/java.io=ALL-UNNAMED

02-Aug-2021 20:50:22.730 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Command line
argument: \--add-opens=java.base/java.util=ALL-UNNAMED

02-Aug-2021 20:50:22.730 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Command line
argument: \--add-opens=java.base/java.util.concurrent=ALL-UNNAMED

02-Aug-2021 20:50:22.731 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Command line
argument: \--add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED

02-Aug-2021 20:50:22.731 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Command line
argument:
-Djava.util.logging.config.file=/usr/local/tomcat/conf/logging.properties

02-Aug-2021 20:50:22.733 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Command line
argument:
-Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager

02-Aug-2021 20:50:22.735 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Command line
argument: -Djdk.tls.ephemeralDHKeySize=2048

02-Aug-2021 20:50:22.735 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Command line
argument: -Djava.protocol.handler.pkgs=org.apache.catalina.webresources

02-Aug-2021 20:50:22.735 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Command line
argument: -Dorg.apache.catalina.security.SecurityListener.UMASK=0027

02-Aug-2021 20:50:22.735 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Command line
argument: -Dignore.endorsed.dirs=

02-Aug-2021 20:50:22.735 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Command line
argument: -Dcatalina.base=/usr/local/tomcat

02-Aug-2021 20:50:22.736 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Command line
argument: -Dcatalina.home=/usr/local/tomcat

02-Aug-2021 20:50:22.736 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Command line
argument: -Djava.io.tmpdir=/usr/local/tomcat/temp

02-Aug-2021 20:50:22.762 INFO \[main\]
org.apache.catalina.core.AprLifecycleListener.lifecycleEvent Loaded
Apache Tomcat Native library \[1.2.30\] using APR version \[1.6.5\].

02-Aug-2021 20:50:22.763 INFO \[main\]
org.apache.catalina.core.AprLifecycleListener.lifecycleEvent APR
capabilities: IPv6 \[true\], sendfile \[true\], accept filters
\[false\], random \[true\], UDS \[true\].

02-Aug-2021 20:50:22.763 INFO \[main\]
org.apache.catalina.core.AprLifecycleListener.lifecycleEvent APR/OpenSSL
configuration: useAprConnector \[false\], useOpenSSL \[true\]

02-Aug-2021 20:50:22.776 INFO \[main\]
org.apache.catalina.core.AprLifecycleListener.initializeSSL OpenSSL
successfully initialized \[OpenSSL 1.1.1d 10 Sep 2019\]

02-Aug-2021 20:50:23.686 INFO \[main\]
org.apache.coyote.AbstractProtocol.init Initializing ProtocolHandler
\[\"http-nio-8080\"\]

02-Aug-2021 20:50:23.777 INFO \[main\]
org.apache.catalina.startup.Catalina.load Server initialization in
\[1697\] milliseconds

02-Aug-2021 20:50:23.977 INFO \[main\]
org.apache.catalina.core.StandardService.startInternal Starting service
\[Catalina\]

02-Aug-2021 20:50:23.978 INFO \[main\]
org.apache.catalina.core.StandardEngine.startInternal Starting Servlet
engine: \[Apache Tomcat/9.0.50\]

02-Aug-2021 20:50:24.039 INFO \[main\]
org.apache.catalina.startup.HostConfig.deployWAR Deploying web app
archive \[/usr/local/tomcat/webapps/FlightBookingSystemSample.war\]

02-Aug-2021 20:50:27.164 INFO \[main\]
org.apache.jasper.servlet.TldScanner.scanJars At least one JAR was
scanned for TLDs yet contained no TLDs. Enable debug logging for this
logger for a complete list of JARs that were scanned but no TLDs were
found in them. Skipping unneeded JARs during scanning can improve
startup time and JSP compilation time.

02-Aug-2021 20:50:30.887 INFO \[main\]
com.sun.xml.ws.server.MonitorBase.createRoot Metro monitoring rootname
successfully set to:
com.sun.metro:pp=/,type=WSEndpoint,name=/FlightBookingSystemSample-PriceAndSeats-PriceAndSeatsPort

02-Aug-2021 20:50:31.151 INFO \[main\]
com.sun.xml.ws.transport.http.servlet.WSServletDelegate.\<init>
WSSERVLET14: JAX-WS servlet initializing

02-Aug-2021 20:50:32.662 INFO \[main\]
com.sun.xml.ws.transport.http.servlet.WSServletContextListener.contextInitialized
WSSERVLET12: JAX-WS context listener initializing

02-Aug-2021 20:50:32.663 INFO \[main\]
com.sun.xml.ws.transport.http.servlet.WSServletContextListener.contextInitialized
WSSERVLET12: JAX-WS context listener initializing

02-Aug-2021 20:50:32.735 INFO \[main\]
org.apache.catalina.startup.HostConfig.deployWAR Deployment of web app
archive \[/usr/local/tomcat/webapps/FlightBookingSystemSample.war\] has
finished in \[8,695\] ms

02-Aug-2021 20:50:32.746 INFO \[main\]
org.apache.coyote.AbstractProtocol.start Starting ProtocolHandler
\[\"http-nio-8080\"\]

02-Aug-2021 20:50:32.768 INFO \[main\]
org.apache.catalina.startup.Catalina.start Server startup in \[8990\]
milliseconds

打开浏览器，并访问航空公司的航班预订系统的登陆页面，网址为 http://localhost:8080/FlightBookingSystemSample

应看到以下内容

![Screenshot showing the running
app.](.//media/image4.png){width="5.768055555555556in"
height="2.910416666666667in"}

你可以选择使用 tomcat-users.xml 中的任何用户登录，例如
someuser\@azure.com：密码

若要停止容器，请在 CLI 中按住 Ctrl + C。

**单元5: 向 Azure 容器注册表推送容器映像**

-   4 分钟

在本单元中，你将向 Azure 容器注册表推送容器映像。

使用 Azure
容器注册表可以在专用注册表中为所有类型的容器部署生成、存储和管理容器映像与项目。
将 Azure 容器注册表与现有的容器开发和部署管道配合使用。

** 备注**

如果会话已经空闲，则在另一个时间点和/或从其他 CLI
执行此步骤时，可能需要重新初始化环境变量，并使用以下 CLI
命令重新进行身份验证。

AZ_RESOURCE_GROUP=javacontainerizationdemorg

AZ_CONTAINER_REGISTRY=\<YOUR_CONTAINER_REGISTRY>

AZ_KUBERNETES_CLUSTER=javacontainerizationdemoaks

AZ_LOCATION=\<YOUR_AZURE_REGION>

AZ_KUBERNETES_CLUSTER_DNS_PREFIX=\<YOUR_UNIQUE_DNS_PREFIX_TO_ACCESS_YOUR_AKS_CLUSTER>

az login

az acr login -n \$AZ_CONTAINER_REGISTRY

**推送容器映像**

可以将新生成的容器映像推送到 Azure 容器注册表。
这样，你的容器映像将与你所有的 Azure 资源（例如 Azure Kubernetes
群集）密切相关。 最终要配置 AKS 以从 Azure 容器注册表拉取
flightbookingsystemsample 映像。

为了将容器映像推送到 Azure 容器注册表，请在 CLI 中运行以下三个命令：

登录 Azure 容器注册表：

Bash复制

az acr login

首先用 Azure 容器注册表标记前面生成的容器映像：

Bash复制

docker tag flightbookingsystemsample
\$AZ_CONTAINER_REGISTRY.azurecr.io/flightbookingsystemsample

接下来，将容器映像推送到 Azure 容器注册表：

Bash复制

docker push \$AZ_CONTAINER_REGISTRY.azurecr.io/flightbookingsystemsample

现在可以查看新推送的映像的 Azure 容器注册表映像元数据了。 在 CLI
中运行以下命令：

Bash复制

az acr repository show -n \$AZ_CONTAINER_REGISTRY \--image
flightbookingsystemsample:latest

你将看到如下内容：

JSON复制

{

\"changeableAttributes\": {

\"deleteEnabled\": true,

\"listEnabled\": true,

\"readEnabled\": true,

\"writeEnabled\": true

},

\"createdTime\": \"2021-10-08T00:51:43.5522013Z\",

\"digest\":
\"sha256:bc7613a5612c914d7a6bfc0f130d1f632a5bda362aa62bb3ac12304dc4ce94c1\",

\"lastUpdateTime\": \"2021-10-08T00:58:57.623821Z\",

\"name\": \"latest\",

\"signed\": false

}

容器映像现在驻留在 Azure 容器注册表中，并可供 Azure 服务（如 Azure
Kubernetes 服务）部署。

**单元6: 将容器映像部署到 Azure Kubernetes 服务**

-   9 分钟

在本单元中，你要将容器映像部署到 Azure Kubernetes 服务。

使用 Azure Kubernetes 服务，你将可以配置 Kubernetes
群集，使其在所需的状态下运行（通过部署），这是向 Pod 和 ReplicaSet
提供声明性更新的过程。 此状态声明在清单 (YAML) 文件中进行管理，而
Kubernetes 控制器会在收到指示时将当前状态更改为声明的状态。
你将在下面创建这个 deployment.yml 清单文件，并指示 Azure Kubernetes
服务在所需的状态下运行，并将 Pod 配置为拉取/运行驻留在 Azure
容器注册表中的 flightbookingsystemsample 容器映像（在上一单元中进行了推送）。
如果没有此部署，你将需要手动创建、更新和删除 Pod，而不是让 Kubernetes
来协调操作。

** 备注**

如果会话已经空闲，则在另一个时间点和/或从其他 CLI
执行此步骤时，可能需要重新初始化环境变量，并使用以下 CLI
命令重新进行身份验证。

AZ_RESOURCE_GROUP=javacontainerizationdemorg

AZ_CONTAINER_REGISTRY=\<YOUR_CONTAINER_REGISTRY>

AZ_KUBERNETES_CLUSTER=javacontainerizationdemoaks

AZ_LOCATION=\<YOUR_AZURE_REGION>

AZ_KUBERNETES_CLUSTER_DNS_PREFIX=\<YOUR_UNIQUE_DNS_PREFIX_TO_ACCESS_YOUR_AKS_CLUSTER>

az login

az acr login -n \$AZ_CONTAINER_REGISTRY

**部署容器映像**

我们将此 flightbookingsystemsample 容器映像部署到 Azure Kubernetes
群集。

在项目的根目录中
(Flight-Booking-System-JavaServlets_App/Project/Airlines)，创建一个名为
deployment.yml 的文件。 在 CLI 中运行以下命令：

Bash复制

vi deployment.yml

将以下内容添加到 deployment.yml 中，然后保存并退出：

** 备注**

你需要用先前设置的 AZ_CONTAINER_REGISTRY
环境变量值进行更新，例如：javacontainerizationdemoacr

yml复制

apiVersion: apps/v1

kind: Deployment

metadata:

name: flightbookingsystemsample

spec:

replicas: 1

selector:

matchLabels:

app: flightbookingsystemsample

template:

metadata:

labels:

app: flightbookingsystemsample

spec:

containers:

\- name: flightbookingsystemsample

image:
\<AZ_CONTAINER_REGISTRY>.azurecr.io/flightbookingsystemsample:latest

resources:

requests:

cpu: \"1\"

memory: \"1Gi\"

limits:

cpu: \"2\"

memory: \"2Gi\"

ports:

\- containerPort: 8080

\-\--

apiVersion: v1

kind: Service

metadata:

name: flightbookingsystemsample

spec:

type: LoadBalancer

ports:

\- port: 8080

targetPort: 8080

selector:

app: flightbookingsystemsample

** 备注**

或者，你的项目的根目录中的 deployment_solution.yml
包含所需的内容，你可能会发现，重命名/更新该文件的内容会更容易。

在上面的 deployment.yml 中，你会注意到此 deployment.yml 包含部署和服务。
部署用于管理一组 Pod，而服务用于允许网络访问这些 Pod。 你会注意到，Pod
已配置为从 Azure
容器注册表中拉取一个映像 \<AZ_CONTAINER_REGISTRY>.azurecr.io/flightbookingsystemsample:latest。
你还会注意到，该服务已配置为允许 HTTP Pod 流量传入端口
8080，这与使用 -p 端口参数在本地运行容器映像的方式类似。

现在，Azure Kubernetes 群集创建应已成功完成。

你需要将 Azure CLI 配置为通过 kubectl 命令访问 Azure Kubernetes 群集。
使用 az aks install-cli 命令在本地安装 kubectl。 在 CLI 中运行以下命令：

Bash复制

az aks install-cli

使用 az aks get-credentials 命令将 kubectl 配置为连接到 Kubernetes
群集。 在 CLI 中运行以下命令：

Bash复制

az aks get-credentials \--resource-group \$AZ_RESOURCE_GROUP \--name
\$AZ_KUBERNETES_CLUSTER

你将看到如下内容：

Bash复制

Merged AZ_KUBERNETES_CLUSTER as current context in \~/.kube/config

现在指示 Azure Kubernetes 服务将 deployment.yml 更改应用到群集。 在 CLI
中运行以下命令：

Bash复制

kubectl apply -f deployment.yml

你将看到如下内容：

Bash复制

deployment.apps/flightbookingsystemsample created

service/flightbookingsystemsample created

现在可以使用 kubectl 来监视部署状态。 在 CLI 中运行以下命令：

Bash复制

kubectl get all

你将看到如下内容：

Bash复制

NAME READY STATUS RESTARTS AGE

pod/flightbookingsystemsample-75647c4c98-v4v4r 1/1 Running 2 13d

NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE

service/kubernetes ClusterIP 10.0.0.1 \<none> 443/TCP 66d

service/flightbookingsystemsample LoadBalancer 10.0.34.128 20.81.13.151
8080:30265/TCP 66d

NAME READY UP-TO-DATE AVAILABLE AGE

deployment.apps/flightbookingsystemsample 1/1 1 1 66d

NAME DESIRED CURRENT READY AGE

replicaset.apps/flightbookingsystemsample-75647c4c98 1 1 1 66d

replicaset.apps/flightbookingsystemsample-7564c58f55 0 0 0 13d

如果 POD 状态为 Running，表示应用应可供访问。

此外，你还可以查看每个 Pod 中的应用日志。 在 CLI 中运行以下命令：

Bash复制

kubectl logs
pod/flightbookingsystemsample-\<POD_IDENTIFIER_FROM_YOUR_RUNNING_POD>

你将看到如下内容：

Bash复制

NOTE: Picked up JDK_JAVA_OPTIONS:
\--add-opens=java.base/java.lang=ALL-UNNAMED
\--add-opens=java.base/java.io=ALL-UNNAMED
\--add-opens=java.base/java.util=ALL-UNNAMED
\--add-opens=java.base/java.util.concurrent=ALL-UNNAMED
\--add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED

07-Oct-2021 18:31:14.073 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Server version
name: Apache Tomcat/8.5.71

07-Oct-2021 18:31:14.164 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Server built: Sep
9 2021 18:43:14 UTC

07-Oct-2021 18:31:14.164 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Server version
number: 8.5.71.0

07-Oct-2021 18:31:14.165 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log OS Name: Linux

07-Oct-2021 18:31:14.166 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log OS Version:
5.4.0-1051-azure

07-Oct-2021 18:31:14.166 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Architecture:
amd64

07-Oct-2021 18:31:14.166 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Java Home:
/usr/local/openjdk-11

07-Oct-2021 18:31:14.167 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log JVM Version:
11.0.12+7

07-Oct-2021 18:31:14.167 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log JVM Vendor: Oracle
Corporation

07-Oct-2021 18:31:14.167 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log CATALINA_BASE:
/usr/local/tomcat

07-Oct-2021 18:31:14.168 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log CATALINA_HOME:
/usr/local/tomcat

07-Oct-2021 18:31:14.261 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Command line
argument: \--add-opens=java.base/java.lang=ALL-UNNAMED

07-Oct-2021 18:31:14.261 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Command line
argument: \--add-opens=java.base/java.io=ALL-UNNAMED

07-Oct-2021 18:31:14.261 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Command line
argument: \--add-opens=java.base/java.util=ALL-UNNAMED

07-Oct-2021 18:31:14.262 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Command line
argument: \--add-opens=java.base/java.util.concurrent=ALL-UNNAMED

07-Oct-2021 18:31:14.262 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Command line
argument: \--add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED

07-Oct-2021 18:31:14.262 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Command line
argument:
-Djava.util.logging.config.file=/usr/local/tomcat/conf/logging.properties

07-Oct-2021 18:31:14.263 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Command line
argument:
-Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager

07-Oct-2021 18:31:14.263 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Command line
argument: -Djdk.tls.ephemeralDHKeySize=2048

07-Oct-2021 18:31:14.263 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Command line
argument: -Djava.protocol.handler.pkgs=org.apache.catalina.webresources

07-Oct-2021 18:31:14.263 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Command line
argument: -Dorg.apache.catalina.security.SecurityListener.UMASK=0027

07-Oct-2021 18:31:14.264 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Command line
argument: -Dignore.endorsed.dirs=

07-Oct-2021 18:31:14.264 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Command line
argument: -Dcatalina.base=/usr/local/tomcat

07-Oct-2021 18:31:14.264 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Command line
argument: -Dcatalina.home=/usr/local/tomcat

07-Oct-2021 18:31:14.265 INFO \[main\]
org.apache.catalina.startup.VersionLoggerListener.log Command line
argument: -Djava.io.tmpdir=/usr/local/tomcat/temp

07-Oct-2021 18:31:14.265 INFO \[main\]
org.apache.catalina.core.AprLifecycleListener.lifecycleEvent Loaded
Apache Tomcat Native library \[1.2.31\] using APR version \[1.7.0\].

07-Oct-2021 18:31:14.265 INFO \[main\]
org.apache.catalina.core.AprLifecycleListener.lifecycleEvent APR
capabilities: IPv6 \[true\], sendfile \[true\], accept filters
\[false\], random \[true\], UDS \[{4}\].

07-Oct-2021 18:31:14.266 INFO \[main\]
org.apache.catalina.core.AprLifecycleListener.lifecycleEvent APR/OpenSSL
configuration: useAprConnector \[false\], useOpenSSL \[true\]

07-Oct-2021 18:31:14.361 INFO \[main\]
org.apache.catalina.core.AprLifecycleListener.initializeSSL OpenSSL
successfully initialized \[OpenSSL 1.1.1k 25 Mar 2021\]

07-Oct-2021 18:31:14.763 INFO \[main\]
org.apache.coyote.AbstractProtocol.init Initializing ProtocolHandler
\[\"http-nio-8080\"\]

07-Oct-2021 18:31:14.962 INFO \[main\]
org.apache.tomcat.util.net.NioSelectorPool.getSharedSelector Using a
shared selector for servlet write/read

07-Oct-2021 18:31:15.071 INFO \[main\]
org.apache.catalina.startup.Catalina.load Initialization processed in
6497 ms

07-Oct-2021 18:31:15.771 INFO \[main\]
org.apache.catalina.core.StandardService.startInternal Starting service
\[Catalina\]

07-Oct-2021 18:31:15.772 INFO \[main\]
org.apache.catalina.core.StandardEngine.startInternal Starting Servlet
engine: \[Apache Tomcat/8.5.71\]

07-Oct-2021 18:31:16.261 INFO \[localhost-startStop-1\]
org.apache.catalina.startup.HostConfig.deployWAR Deploying web app
archive \[/usr/local/tomcat/webapps/FlightBookingSystemSample.war\]

07-Oct-2021 18:31:30.782 INFO \[localhost-startStop-1\]
org.apache.jasper.servlet.TldScanner.scanJars At least one JAR was
scanned for TLDs yet contained no TLDs. Enable debug logging for this
logger for a complete list of JARs that were scanned but no TLDs were
found in them. Skipping unneeded JARs during scanning can improve
startup time and JSP compilation time.

WARNING: An illegal reflective access operation has occurred

WARNING: Illegal reflective access by
com.sun.xml.ws.policy.privateutil.MethodUtil
(file:/usr/local/tomcat/webapps/FlightBookingSystemSample/WEB-INF/lib/webservices-rt-2.3.1.jar)
to method
sun.reflect.misc.MethodUtil.invoke(java.lang.reflect.Method,java.lang.Object,java.lang.Object\[\])

WARNING: Please consider reporting this to the maintainers of
com.sun.xml.ws.policy.privateutil.MethodUtil

WARNING: Use \--illegal-access=warn to enable warnings of further
illegal reflective access operations

WARNING: All illegal access operations will be denied in a future
release

07-Oct-2021 18:31:53.370 INFO \[localhost-startStop-1\]
com.sun.xml.ws.server.MonitorBase.createRoot Metro monitoring rootname
successfully set to:
com.sun.metro:pp=/,type=WSEndpoint,name=/FlightBookingSystemSample-PriceAndSeats-PriceAndSeatsPort

07-Oct-2021 18:31:54.864 INFO \[localhost-startStop-1\]
com.sun.xml.ws.transport.http.servlet.WSServletDelegate.\<init>
WSSERVLET14: JAX-WS servlet initializing

07-Oct-2021 18:32:02.869 INFO \[localhost-startStop-1\]
com.sun.xml.ws.transport.http.servlet.WSServletContextListener.contextInitialized
WSSERVLET12: JAX-WS context listener initializing

07-Oct-2021 18:32:02.870 INFO \[localhost-startStop-1\]
com.sun.xml.ws.transport.http.servlet.WSServletContextListener.contextInitialized
WSSERVLET12: JAX-WS context listener initializing

07-Oct-2021 18:32:03.069 INFO \[localhost-startStop-1\]
org.apache.catalina.startup.HostConfig.deployWAR Deployment of web app
archive \[/usr/local/tomcat/webapps/FlightBookingSystemSample.war\] has
finished in \[46,808\] ms

07-Oct-2021 18:32:03.165 INFO \[main\]
org.apache.coyote.AbstractProtocol.start Starting ProtocolHandler
\[\"http-nio-8080\"\]

07-Oct-2021 18:32:03.267 INFO \[main\]
org.apache.catalina.startup.Catalina.start Server startup in 48195 ms

现在可以使用 kubectl get services
flightbookingsystemsample 输出中的 EXTERNAL-IP 来访问 Azure Kubernetes
服务中正在运行的应用了。

** 备注**

需要将下面的 IP 地址 20.81.13.151 替换为先前执行的命令中的 EXTERNAL-IP。

打开浏览器并访问航班预订系统示例登陆页面，网址为 [[http://20.81.13.151:8080/FlightBookingSystemSample]{.underline}](http://20.81.13.151:8080/FlightBookingSystemSample)

你将看到如下内容：

![Screenshot showing the running
app](.//media/image5.png){width="5.768055555555556in"
height="2.884027777777778in"}

可以选择使用 tomcat-users.xml 中的任何用户登录，例如
someuser\@azure.com：密码

**单元7: 摘要**

已完成100 XP

-   2 分钟

恭喜！ 你已将 Java 应用容器化并部署到 Azure Kubernetes 服务。

你需要了解如何容器化 Java 应用并将其部署到 Azure。 你执行了以下步骤：

1.  容器化 Java 应用

2.  为 Java 应用生成容器映像

3.  在本地运行容器映像

4.  向 Azure 容器注册表推送容器映像

5.  将容器映像部署到 Azure Kubernetes 服务

你现在已经确信你可以容器化 Java 应用并将其部署到 Azure。

**启用诊断记录**

了解 Azure 如何提供[内置诊断以帮助进行调试]{.underline}。

**清理资源**

在本模块中，你在资源组创建了 Azure 资源。
如果以后不需要这些资源，请从门户中删除资源组。 或在 Azure Cloud Shell
中运行以下命令。

Bash复制

az group delete \--name \$AZ_RESOURCE_GROUP \--yes

运行此命令可能需要一分钟时间。

** 重要**

请取消预配本模块中所使用的 Azure 资源，以避免产生不必要的费用。

**其他资源**

详细了解 [[Docker]{.underline}](https://docs.docker.com/reference/)。

详细了解 [[Azure
容器注册表]{.underline}](https://azure.microsoft.com/services/container-registry/)。

了解有关 [[Azure Kubernetes
服务]{.underline}](https://azure.microsoft.com/services/kubernetes-service/)的详细信息。
