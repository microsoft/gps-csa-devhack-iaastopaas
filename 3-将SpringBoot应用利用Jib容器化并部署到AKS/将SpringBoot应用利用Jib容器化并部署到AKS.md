# **将 Spring Boot 应用程序部署到 Azure Kubernetes 服务**
 &nbsp;
 &nbsp;

本教程将指导用户将 Kubernetes 和 Docker 进行结合，从而将 Spring Boot
应用程序开发和部署到 Microsoft Azure。 具体而言，将使用 [Spring Boot](https://spring.io/projects/spring-boot/) 进行应用程序开发，使用 [Kubernetes](https://kubernetes.io/) 进行容器部署，使用 [Azure Kubernetes 服务(AKS)](https://azure.microsoft.com/services/kubernetes-service/) 来托管应用程序。

[Kubernetes](https://kubernetes.io/) 和 [Docker](https://www.docker.com/) 是开放源代码解决方案，可帮助开发人员自动部署、缩放和管理在容器中运行的应用程序。

**先决条件**

-   Azure 订阅；如果没有 Azure 订阅，可激活 [MSDN 订阅者权益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)或注册[免费的 Azure 帐户](https://azure.microsoft.com/pricing/free-trial/)。

-   [Azure 命令行接口(CLI)](https://learn.microsoft.com/zh-cn/cli/azure/overview)。

-   一个受支持的 Java 开发工具包 (JDK)。 有关在 Azure
    > 上进行开发时可供使用的 JDK 的详细信息，请参阅 [Azure 和 AzureStack 上的 Java 支持](https://learn.microsoft.com/zh-cn/azure/developer/java/fundamentals/java-support-on-azure)。

-   Apache 的 [Maven](http://maven.apache.org/) 生成工具（版本 3）。

-   [Git](https://github.com/) 客户端。

-   [Docker](https://www.docker.com/) 客户端。

-   [ACR Docker 凭据帮助器](https://github.com/Azure/acr-docker-credential-helper)。

**备注**

由于本教程中的虚拟化要求，无法在虚拟机上执行本文中的步骤；必须使用启用了虚拟化功能的物理计算机。

 &nbsp;
 &nbsp;
## **创建 Docker 上的 Spring Boot 入门 Web 应用**

以下步骤将指导用户构建 Spring Boot web 应用程序并在本地进行测试。

1.  打开命令提示符，创建本地目录以存放应用程序，并更改为以下目录；例如：

    > *Bash复制*
    >
   
    > mkdir C:\\SpringBoot
    >
    > cd C:\\SpringBoot
    >
    >  或 
    >
    > *Bash复制*
    >
    > mkdir /users/\$USER/SpringBoot
    >
    > cd /users/\$USER/SpringBoot
    
2.  将 [Docker 上的 Spring Boot 入门](https://github.com/spring-guides/gs-spring-boot-docker)示例项目克隆到目录。

    > *Bash复制*
    >
    > git clone https://github.com/spring-guides/gs-spring-boot-docker.git

3.  将目录更改为已完成项目。

    > *Bash复制*
    >
    > cd gs-spring-boot-docker
    >
    > cd complete

4.  使用 Maven 生成和运行示例应用。

    > *Bash复制*
    >
    > mvn package spring-boot:run

5.  通过浏览到 http://localhost:8080 或使用以下 curl 命令测试 Web 应用：

    > *Bash复制*
    >
    > curl http://localhost:8080

6.  应当会看到显示了以下消息：**Hello Docker World**

    > ![Browse Sample App
    > Locally](.//media/image1.png)

 &nbsp;
 &nbsp;

## **使用 Azure CLI 创建 Azure 容器注册表**

1.  打开命令提示符。

2.  登录到 Azure 帐户：

    > *Azure CLI复制*
    >
    > az login

3.  选择自己的 Azure 订阅：

    > *Azure CLI复制*
    >
    > az account set -s \<YourSubscriptionID>

4.  为本教程中使用的 Azure 资源创建资源组。

    > *Azure CLI复制*
    >
    > az group create \--name=wingtiptoys-kubernetes \--location=eastus

5.  在资源组中创建私有 Azure 容器注册表。
    > 本教程的后续步骤会将示例应用作为 Docker 映像推送到此注册表。
    > 用注册表的唯一名称替换 wingtiptoysregistry。

    > *Azure CLI复制*
    >
    > az acr create \--resource-group wingtiptoys-kubernetes \--location
    > eastus \\
    >
    > \--name wingtiptoysregistry \--sku Basic

 &nbsp;
 &nbsp;


## **通过 Jib 将应用推送到容器注册表**

1.  通过 Azure CLI 登录到 Azure 容器注册表。

    > *Azure CLI复制*
    >
    > \# set the default name for Azure Container Registry, otherwise you
    > will need to specify the name in \"az acr login\"
    >
    > az config set defaults.acr=wingtiptoysregistry
    >
    > az acr login

2.  使用文本编辑器（例如 [VS
    > Code](https://code.visualstudio.com/docs)）打开 pom.xml 文件，。

    > *Bash复制*
    >
    > code pom.xml

3.  将 *pom.xml* 文件中的 \<properties> 集合更新为你的 Azure
    > 容器注册表的注册表名称和 [jib-maven-plugin](https://github.com/GoogleContainerTools/jib/tree/master/jib-maven-plugin) 的最新版本。

    > *XML复制*
    >
    > \<properties>
    >
    > \<!\-- Note: If your ACR name contains upper case characters, be sure
    > to convert them to lower case characters. \--\>
    >
    > \<docker.image.prefix>wingtiptoysregistry.azurecr.io\</docker.image.prefix>
    >
    > \<jib-maven-plugin.version>2.5.2\</jib-maven-plugin.version>
    >
    > \<java.version>1.8\</java.version>
    >
    > \</properties>

4.  更新 pom.xml
    > 文件中的 \<plugins> 集合，使 \<plugin> 元素包含 jib-maven-plugin 的条目，如以下示例中所示。
    > 请注意，我们将使用 Microsoft 容器注册表 (MCR)
    > 中的基础映像：mcr.microsoft.com/java/jdk:8-zulu-alpine，其中包含
    > Azure 正式支持的 JDK。 有关包含正式支持的 JDK 的其他 MCR
    > 基础映像，请参阅 [Java SE
    > JDK](https://hub.docker.com/_/microsoft-java-jdk)、[Java SE
    > JRE](https://hub.docker.com/_/microsoft-java-jre)、[Java SE 无头
    > JRE](https://hub.docker.com/_/microsoft-java-jre-headless) 以及 [Java
    > SE JDK 和 Maven](https://hub.docker.com/_/microsoft-java-maven)。

    > *XML复制*
    >
    > \<plugin>
    >
    > \<artifactId>jib-maven-plugin\</artifactId>
    >
    > \<groupId>com.google.cloud.tools\</groupId>
    >
    > \<version>\${jib-maven-plugin.version}\</version>
    >
    > \<configuration>
    >
    > \<from>
    >
    > \<image>mcr.microsoft.com/java/jdk:8-zulu-alpine\</image>
    >
    > \</from>
    >
    > \<to>
    >
    > \<image>\${docker.image.prefix}/\${project.artifactId}\</image>
    >
    > \</to>
    >
    > \</configuration>
    >
    > \</plugin>

5.  导航到 Spring Boot
    > 应用程序的完成项目目录，然后运行以下命令以生成映像并将映像推送到注册表：

    > *Azure CLI复制*
    >
    > az acr login && mvn compile jib:build

**备注**

由于 Azure Cli 和 Azure 容器注册表的安全性问题，创建的az acr
login凭据有效期为 1 小时。 如果看到 *401
未经授权的* 错误，可以再次运行 az acr login -n \<your registry
name> 该命令以重新进行身份验证。
如果看到 *读取超时* 错误，可以尝试增加超时 mvn -Djib.httpTimeout=7200000
jib:dockerBuild，或者 -Djib.httpTimeout=0 无限超时。

 &nbsp;
 &nbsp;

## **使用 Azure CLI 在 AKS 上创建 Kubernetes 群集**

1.  在 Azure Kubernetes 服务中创建 Kubernetes 群集。 以下命令在
    > wingtiptoys-kubernetes 资源组中创建 kubernetes 群集（将
    > wingtiptoys-akscluster 作为群集名称，附加了 Azure
    > 容器注册表 wingtiptoysregistry，并将 wingtiptoys-kubernetes 作为
    > DNS 前缀 ）：

    > *Azure CLI复制*
    >
    > az aks create \--resource-group=wingtiptoys-kubernetes
    > \--name=wingtiptoys-akscluster \\
    >
    > \--attach-acr wingtiptoysregistry \\
    >
    > \--dns-name-prefix=wingtiptoys-kubernetes \--generate-ssh-keys
    >
    > 此命令可能需要一段时间才能完成。

2.  使用 Azure CLI 安装 kubectl。 Linux
    > 用户可能必须将 sudo 作为此命令的前缀，因为它将 Kubernetes CLI
    > 部署到 /usr/local/bin。

    > *Azure CLI复制*
    >
    > az aks install-cli

3.  下载群集配置信息，以便从 Kubernetes Web 界面和 kubectl 管理群集。

    > *Azure CLI复制*
    >
    > az aks get-credentials \--resource-group=wingtiptoys-kubernetes
    > \--name=wingtiptoys-akscluster

 &nbsp;
 &nbsp;

## **将映像部署到 Kubernetes 群集**

本教程使用 kubectl 部署应用，以便你可通过 Kubernetes Web 界面浏览部署。

## **使用 kubectl 进行部署**

1.  打开命令提示符。

2.  使用 kubectl run 命令在 Kubernetes 群集中运行容器。 指定 Kubernetes
    > 中应用的服务名称和完整映像名称。 例如：

    > *Bash复制*
    >
    > kubectl run gs-spring-boot-docker
    > \--image=wingtiptoysregistry.azurecr.io/gs-spring-boot-docker:latest
    >
    > 在此命令中：

-   容器名称 gs-spring-boot-docker 会立即在 run 命令后指定

-   \--image 参数将组合的登录服务器和映像名称指定为 wingtiptoysregistry.azurecr.io/gs-spring-boot-docker:latest

1.  使用 kubectl expose 命令外部公开你的 Kubernetes 群集。
    > 指定你的服务名称、用于访问应用的公开 TCP
    > 端口以及应用侦听的内部目标端口。 例如：

    > *Bash复制*
    >
    > kubectl expose pod gs-spring-boot-docker \--type=LoadBalancer
    > \--port=80 \--target-port=8080
>
> 在此命令中：

-   容器名称 gs-spring-boot-docker 紧跟在 expose pod 命令后指定。

-   \--type 参数指定此群集使用负载均衡器。

-   \--port 参数指定面向公众的 TCP 端口 80。 在此端口访问应用。

-   \--target-port 参数指定内部 TCP 端口 8080。
    > 负载均衡器在此端口将请求转发到你的应用。

2.  将应用部署到群集后，请查询外部 IP 地址并在 Web 浏览器中打开：

    > *Bash复制*
    >
    > kubectl get services
    > -o=jsonpath=\'{.items\[\*\].status.loadBalancer.ingress\[0\].ip}\'
    >
    > ![Browse Sample App on
    > Azure](.//media/image2.png)

 &nbsp;
 &nbsp;

## **使用 Kubernetes 资源视图进行部署**

1.  从任何资源视图（命名空间、工作负荷、服务和流入量、存储或配置）中选择"添加"。

    > ![Kubernetes resources
    > view.](.//media/image3.png)
2.  粘贴到以下 YAML 中：

    > *YAML复制*
    >
    > apiVersion: apps/v1
    >
    > kind: Deployment
    >
    > metadata:
    >
    > name: gs-spring-boot-docker
    >
    > spec:
    >
    > replicas: 1
    >
    > selector:
    >
    > matchLabels:
    >
    > app: gs-spring-boot-docker
    >
    > template:
    >
    > metadata:
    >
    > labels:
    >
    > app: gs-spring-boot-docker
    >
    > spec:
    >
    > containers:
    >
    > \- name: gs-spring-boot-docker
    >
    > image: wingtiptoysregistry.azurecr.io/gs-spring-boot-docker:latest

3.  选择 YAML 编辑器底部的"添加"，以部署应用程序。

    > ![Kubernetes resources view, add
    > resource.](.//media/image4.png)
    >
    > 部署 Deployment后，如上所示，选择 YAML
    > 编辑器底部的 **"添加** "，以使用以下 YAML 进行部署 Service ：
    >
    > *YAML复制*
    >
    > apiVersion: v1
    >
    > kind: Service
    >
    > metadata:
    >
    > name: gs-spring-boot-docker
    >
    > spec:
    >
    > type: LoadBalancer
    >
    > ports:
    >
    > \- port: 80
    >
    > targetPort: 8080
    >
    > selector:
    >
    > app: gs-spring-boot-docker

4.  添加 YAML 文件后，资源查看器会显示 Spring Boot 应用程序。
    > 外部服务包含链接的外部 IP
    > 地址，因此你可以轻松地在浏览器中查看应用程序。

    > ![Kubernetes resources view, services
    > list.](.//media/image5.png)
    >
    > ![Kubernetes resources view, services list, external endpoints
    > highlighted.](.//media/image6.png)

5.  选择"外部 IP"。 然后将看到 Spring Boot 应用程序在 Azure 上运行。

    > ![Browse Sample App on
    > Azure](.//media/image2.png)
