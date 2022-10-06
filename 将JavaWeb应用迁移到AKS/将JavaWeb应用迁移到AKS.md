# 将Java Web应用迁移到AKS

• 单元1: 简介2 分钟

• 单元2: Azure Migrate 应用容器化概述4 分钟

• 单元3: 设置主机环境7 分钟

• 单元4: 发现 Java Web 应用5 分钟

• 单元5: 为 Java Web 应用构建容器映像5 分钟

• 单元6: 将应用容器部署到 AKS6 分钟

• 单元7: 总结2 分钟

**单元1:简介**

-   2 分钟

准备以数字方式转变其业务的组织必须确定如何对现有应用程序和基础结构进行现代化，以实现业务目标。
实现现代化的途径有很多 -
你可以从头开始重新构建应用程序，重构现有应用程序以使其更具有云原生性，或不改变应用程序结构的情况下重新平台化应用程序以实现云的某些优势。
每个应用程序现代化方法都有其自身的优势，对于你的组织和业务目标来说，决定项目组合中的哪些应用程序使用哪种方法是唯一的。

在此模块中，你将了解 Azure Migrate
应用容器化如何使用模板化的流程，帮助你对 Linux 计算机上运行的 Java Web
应用进行容器化并将其迁移到 Azure Kubernetes 服务
(AKS)，从而加快应用程序现代化的进程。

**学习目标**

学完本模块后，你将能够：

-   **使用 Azure Migrate 应用容器化发现 Linux 计算机上运行的 Java Web
    > 应用并对其进行容器化。**

-   **构建 Java Web 应用的容器映像。**

-   **使用 Azure Migrate 应用容器化将容器化应用程序部署到 AKS。**

**先决条件**

-   有效的 [[Azure
    > 订阅]{.underline}](https://azure.microsoft.com/free/services/kubernetes-service/?WT.mc_id=akspipeline_intro-learn-ludossan)

**单元2:Azure Migrate 应用容器化概述**

-   4 分钟

Azure Migrate 应用容器化是一个独立的实用工具，你可以下载并安装在所有
Windows 10 或 Windows Server
2016（或更高版本）计算机上，通过网络访问运行 Java Web
应用的应用程序服务器来进行容器化和迁移。
该工具的工作原理是使用服务器上应用程序的运行状态来确定应用程序组件，并帮助你将它们打包到容器映像中。
然后，可以将容器化应用程序部署到 Azure Kubernetes 服务 (AKS) 或 Azure
应用服务容器上。
容器化过程不需要访问代码库，并提供了一种简单的方法来容器化现有应用程序。

该工具当前支持对以下 Java Web 应用进行容器化和迁移 -

-   Tomcat 8 或更高版本上运行的应用程序。

-   Ubuntu Linux 16.04/18.04/20.04、Debian 7/8、CentOS 6/7 Red Hat
    > Enterprise Linux 5/6/7 上的应用程序服务器。

-   使用 Java 版本 7 或更高版本的应用程序。

Azure Migrate 应用容器化可帮助你实现以下目的：

-   发现应用程序：该工具远程连接到运行 Java Web
    > 应用程序的应用程序服务器（在 Apache Tomcat
    > 上运行），并发现应用程序组件。
    > 该工具将创建一个可用于为应用程序创建容器映像的 Dockerfile。

-   **生成容器映像**：可以根据应用程序要求检查并进一步自定义
    > Dockerfile，并使用它来生成应用程序容器映像。
    > 应用程序容器映像会被推送到指定的 Azure 容器注册表。

-   **部署到 Azure Kubernetes
    > 服务**：然后，该工具会生成将容器化应用程序部署到 Azure Kubernetes
    > 服务群集时所需的 Kubernetes 资源定义 YAML 文件。 可以自定义 YAML
    > 文件，并使用它们在 AKS 上部署应用程序。

![App Containerization process
overview](./media/media/image1.png){width="5.768055555555556in"
height="3.1770833333333335in"}

在此模块的后续部分中，你将设置一个示例双层 Java Web 应用，并使用 Azure
Migrate 应用容器化对应用程序进行容器化并将其迁移到 AKS。

**单元3: - 设置主机环境**

-   7 分钟

在此模块中，我们将尝试对 Airsonic 应用程序进行容器化和迁移。 Airsonic
是一个基于 Web 的免费音乐流式处理应用程序。 我们会在两层配置中部署
Airsonic，其中应用程序前端在 Linux 服务器的 Apache Tomcat
上运行，应用程序后端在 Azure Database for MySQL 上运行。

**准备 Azure 帐户**

如果没有 Azure
订阅，请在开始之前创建一个[[免费帐户]{.underline}](https://azure.microsoft.com/pricing/free-trial/)。

订阅设置完成后，你将需要一个拥有以下权限的 Azure 用户帐户：

-   Azure 订阅的所有者权限

-   注册 Azure Active Directory 应用的权限

如果你刚刚创建了免费的 Azure 帐户，那么你就是订阅的所有者。
如果你不是订阅所有者，请让所有者分配权限，如下所示：

1.  在 Azure 门户中，搜索"订阅"，然后在"服务"下选择"订阅" 。

> ![Search box to search for the Azure
> subscription.](./media/media/image2.png){width="5.768055555555556in"
> height="2.323611111111111in"}

2.  在"订阅"页上，选择要在其中创建 Azure Migrate 项目的订阅。

3.  在"订阅"中，选择"访问控制 (IAM)"\>"检查访问权限" 。

4.  在"检查访问权限"中，搜索相关的用户帐户。

5.  在"添加角色分配"中，单击"添加" 。

> ![Search for a user account to check access and assign a
> role.](./media/media/image3.png){width="5.768055555555556in"
> height="2.286111111111111in"}

6.  在"添加角色分配"中，选择"所有者"角色，然后选择帐户（本例中为
    > azmigrateuser）。 然后单击"保存" 。

> ![Opens the Add Role assignment page to assign a role to the
> account.](./media/media/image4.png){width="5.75in"
> height="6.527777777777778in"}

7.  Azure 帐户还需要注册 Azure Active Directory 应用的权限。

8.  在 Azure 门户中，导航到"Azure Active
    > Directory"\>"用户"\>"用户设置"。

9.  在"用户设置"中，验证 Azure AD
    > 用户是否可以注册应用程序（默认情况下设置为"是"） 。

> ![Verify in User Settings that users can register Active Directory
> apps.](./media/media/image5.png){width="5.768055555555556in"
> height="3.967361111111111in"}

10. 如果"应用注册"设置设置为"否"，请请求租户/全局管理员分配所需的权限。
    > 或者，租户/全局管理员可将"应用程序开发人员"角色分配给帐户，以允许注册
    > Azure Active Directory 应用。

**设置 Airsonic 应用程序**

1.  若要部署研讨会环境，请首先导航到 [[Azure
    > 门户]{.underline}](https://portal.azure.com/)。

2.  启动 Azure Cloud Shell 并运行以下命令。

> 复制
>
> git clone
> https://github.com/MicrosoftDocs/mslearn-azuremigrate-appcontainerization-javatomcat.git
>
> cd mslearn-azuremigrate-appcontainerization-javatomcat/Java\\
> Containerization/
>
> chmod +x scripts/deploy.sh
>
> ./scripts/deploy.sh \'westus2\' \'LearnAppContainerization\'

3.  部署完成后，资源组中将显示以下资源。

> ![Two-tier Java web application deployed for
> workshop](./media/media/image6.png){width="5.768055555555556in"
> height="2.404166666666667in"}

4.  要浏览应用程序，请选择"TomcatServer"虚拟机资源，复制服务器的公共
    > IP，并将其粘贴到浏览器窗口中。 在服务 IP 后面追加:8080/airsonic。

> ![Java web application to be containerized and
> migrated](./media/media/image7.png){width="5.768055555555556in"
> height="1.8777777777777778in"}

5.  你可以使用以下凭据登录到应用程序。

    -   用户名：admin

    -   密码：admin

**下载并安装 Azure Migrate: 应用容器化工具**

1.  在资源组中，选择"tomcatMigrate-toolclient"虚拟机资源并使用堡垒主机登录到该资源。
    > 使用"adminuser"作为用户名，"Password@123"作为密码。
    > 你将使用此计算机运行 Azure Migrate: 应用容器化工具来迁移
    > TomcatServer 上托管的 Java Web 应用。

2.  登录后，请[[下载]{.underline}](https://go.microsoft.com/fwlink/?linkid=2134571) Windows
    > 计算机上的 Azure Migrate: 应用容器化安装程序。

3.  在管理员模式下启动 PowerShell，并运行以下命令来安装该工具。

> PowerShell复制
>
> cd Downloads
>
> .\\AppContainerizationInstaller.ps1

4.  打开 Microsoft Edge
    > 浏览器并浏览到 **https://toolclient:44369** 以启动该工具。
    > 如果遇到警告，请单击"高级"和"继续 toolclient"。

# 单元4: - 发现 Java Web 应用

已完成100 XP

-   5 分钟

在此练习中，我们将使用 Azure
Migrate：应用容器化工具来发现需要进行容器化和迁移的 Java Web 应用。

## 完整工具必备组件

1.  选择"Java Web 应用"作为要进行容器化的应用程序类型。

2.  若要指定目标 Azure 服务，请选择"Azure Kubernetes 服务上的容器"。

> ![Default load-up for App Containerization
> tool.](./media/media/image8.png){width="5.768055555555556in"
> height="3.488888888888889in"}

3.  接受许可条款，并阅读第三方信息。

4.  此工具将自动检查是否有 Internet 连接，以及是否安装最新版本的 Azure
    > Migrate 应用容器化工具。

5.  该工具将通知你在应用程序服务器上启用 SSH，这是安装的一部分。
    > 单击"继续"。

## 登录 Azure

单击"登录"以登录 Azure 帐户。

1.  需要使用设备代码向 Azure 进行身份验证。
    > 单击"登录"将打开具有设备代码的模式。

2.  单击"复制代码并登录"以复制设备代码，并在新的浏览器标签页中打开 Azure
    > 登录提示。如果未显示，请确保在浏览器中禁用了弹出窗口阻止程序。

3.  在新选项卡中，粘贴设备代码，并使用 Azure 帐户凭证完成登录。
    > 登录完成后，可以关闭浏览器选项卡，然后返回到应用容器化工具的 Web
    > 界面。

4.  选择要使用的 Azure 租户。

5.  选择要使用的 Azure 订阅。

## 发现 Java Web 应用程序

应用容器化帮助程序工具使用提供的凭据远程连接到应用程序服务器，并尝试发现托管在应用程序服务器上的
Java Web 应用。

1.  若要发现 Parts Unlimited 应用程序，请使用以下值 -

    -   **服务器 IP/FQDN**：在 LearnAppContainerization 资源组中，导航到
        > TomcatServer 虚拟机，复制专用 IP
        > 地址，并在应用容器化工具中指定此值。

    -   凭据：将"adminUser"指定为用户名，将"Password@123"指定为密码。

```{=html}
<!-- -->
```
1.  单击"验证"，验证是否可以从运行该工具的计算机访问应用程序服务器，以及凭据是否有效。
    > 验证成功后，"状态"列会将状态显示为"已映射"。

> ![Screenshot for server IP and
> credentials.](./media/media/image9.png){width="5.768055555555556in"
> height="2.7055555555555557in"}

2.  单击"继续"，在选定的应用程序服务器上启动应用程序发现。

3.  成功完成应用程序发现后，可以选择要容器化的应用程序列表。

> ![Screenshot for discovered Java web
> application.](./media/media/image10.png){width="5.768055555555556in"
> height="2.8986111111111112in"}

4.  使用此复选框选择要进行容器化的 Airsonic 应用程序。

5.  **指定容器名称**：为每个选定的应用程序指定目标容器的名称。
    > 容器名称应指定为 \<name:tag\>，其中标记用于容器映像。
    > 例如，可以将目标容器名称指定为 airsonictest:v1。

### 参数化应用程序配置

参数化配置，使其可用作部署时间参数。
这使你可以在部署应用程序时配置此设置，而不是将其硬编码为容器映像中的特定值。
例如，此选项对数据库连接字符串等参数非常有用。

1.  单击"应用配置"以查看检测到的配置。

2.  选中所有复选框，对配置参数化（用户名、密码和 URL）。

3.  选择要参数化的配置后，单击"应用"。

> ![Screenshot for app configuration parameterization Java web
> application.](./media/media/image11.png){width="5.768055555555556in"
> height="2.7041666666666666in"}

### 外部化文件系统依赖项

可以添加应用程序使用的其他文件夹。
指定它们是否应为容器映像的一部分，或者是否要通过 Azure
文件共享上的永久性卷进行外部化。
对于将状态存储在容器外部或将其他静态内容存储在文件系统上的有状态应用程序，使用永久性卷非常有用。

1.  单击"应用文件夹"下的"编辑"以查看检测到的应用程序文件夹。
    > 检测到的应用程序文件夹已被识别为应用程序所需的必需项目，并将复制到容器映像中。

2.  单击"添加文件夹"，并指定要添加的文件夹路径。

3.  在文本框中添加"/var/airsonic"作为文件夹路径。

4.  选择"永久性卷"作为存储选项，以将容器外的文件夹存储到永久性卷上。

> ![Screenshot for externalizing app folders for Java web
> application.](./media/media/image12.png){width="5.768055555555556in"
> height="2.6118055555555557in"}

5.  单击"保存"。

**单元5: - 为 Java Web 应用构建容器映像**

-   5 分钟

在本练习中，我们将使用 Azure 容器注册表及其功能来生成和存储容器映像。

使用 Azure
容器注册表可以在专用注册表中为所有类型的容器部署生成、存储和管理容器映像与项目。
Azure Migrate 应用容器化使用 Azure 容器注册表任务在 Azure
中按需生成容器映像，然后存储这些映像。

**生成容器映像**

1.  **创建 Azure 容器注册表**：创建新的 Azure
    > 容器注册表以生成并存储应用的容器映像。
    > 选择"创建新注册表"选项以创建新的 ACR learnappcontainerizationacr。

> ![Screenshot for app ACR
> selection.](./media/media/image13.png){width="5.768055555555556in"
> height="3.436111111111111in"}

2.  **配置 Application Insights**：你可以在不检测代码的情况下为 Java
    > 应用启用监视功能。 该工具将安装 Java
    > 独立代理，作为容器映像的一部分。 在部署过程中进行配置后，Java
    > 代理会自动为应用程序收集可用于使用 Application Insights
    > 进行监视的各项请求、依赖项、日志和指标。 默认为所有 Java
    > 应用程序启用此选项，我们会保持原样。

3.  **查看 Dockerfile**：为每个所选应用程序生成容器映像所需的 Dockerfile
    > 是在生成步骤开始时生成的。 单击"查看"以查看 Dockerfile。
    > 还可以在查看步骤中将任何必要的自定义添加到
    > Dockerfile，并在开始生成过程之前保存更改。 在此练习中，我们不会对
    > Dockerfile 进行任何更改。

4.  **触发器生成过程**：选择要为其生成映像的应用程序，然后单击"生成"。
    > 单击"生成"将为每个应用程序启动容器映像生成。
    > 该工具会持续监视生成状态，并使你能够在成功完成生成后继续执行下一步。

5.  **跟踪生成状态** ：通过单击"状态"列下的"正在进行的生成"链接，还可以监视生成步骤的进度。
    > 触发生成过程后，该链接需要几分钟时间才能生效。

6.  完成生成后，单击"继续"以指定部署设置。

**单元6: - 将应用容器部署到 AKS**

-   6 分钟

生成容器映像后，下一步是将应用程序部署为 AKS 上的容器。

AKS 通过将操作开销分担到 Azure 来简化在 Azure 中部署托管 Kubernetes
群集的过程。 作为一个托管的 Kubernetes 服务，Azure
可以自动处理运行状况监视和维护等关键任务。 由于 Kubernetes 主节点由
Azure 管理，因此你只需要管理和维护代理节点。

**创建 Azure Kubernetes 服务群集**

该工具提供了选择现有 AKS 群集的选项，但在本练习中，我们将创建一个新的
AKS 群集。

1.  单击"新建 AKS 群集"。

2.  选择要使用的 Azure 订阅。

3.  选择"LearnAppContainerization"资源组。

4.  将 AKS 群集名称指定为"LearnAppContainerizationAKSCluster"。

5.  选择要创建的 AKS 群集的任何可用位置和 SKU。

单击"创建"后，该工具会触发 AKS 群集创建过程。 该工具将创建具有 Linux
节点池的 AKS 群集，并将该群集配置为有权从所选用于存储映像的 Azure
容器注册表中提取映像。

**指定密钥存储**

由于你选择了参数化应用程序配置（数据库连接字符串），则可选择 Azure Key
Vault 或 Kubernetes 机密来管理应用程序机密。

1.  选择"创建新的 Azure Key
    > Vault"选项，并将名称指定为"learnappcontainerizationkeyvault"。
    > 该工具会自动分配必要的权限，以便通过密钥保管库管理密钥。

2.  创建名为 learnappcontainerizationmanagedidentity 的新托管标识。 AKS
    > 将使用此托管标识访问 Key Vault 以装载应用程序的机密。

**创建 Application Insights 资源**

若要监视容器化的 Java Web 应用，我们需要创建一个新的 Application
Insights 资源。

1.  选择选项以创建新 Azure Application
    > Insights，并将名称指定为"learnappcontainerizationappinsight"。
    > 该工具将创建资源，且配置将在部署过程中自动执行。

**指定 Azure 文件共享**

如果你已添加了更多文件夹并选择了"永久性卷"选项，则在部署过程中，请指定
Azure Migrate 应用容器化工具应使用的 Azure 文件共享。 该工具将在此 Azure
文件共享中创建新目录，以复制为永久性卷存储配置的应用程序文件夹。
应用程序部署完成后，该工具会通过删除其创建的目录来清理 Azure 文件共享。

1.  新建选项以创建新的存储帐户和 Azure 文件共享。

2.  指定存储帐户的名称，并选择存储帐户的位置和 SKU。

3.  将 Azure 文件共享名称指定为"learnappcontainerizationfileshare"。

4.  单击"创建"。

**应用程序部署配置**

完成上述步骤后，需要指定应用程序的部署配置。
单击"配置"以自定义应用程序部署。 在配置步骤中，可以提供以下自定义项：

1.  **前缀字符串**：指定要在 AKS
    > 群集中为容器化应用程序创建的所有资源的名称中使用的前缀字符串。
    > 使用 airsonictest 作为本练习的前缀字符串。

2.  **副本数**：指定应在容器内运行的应用程序实例 (pod) 的数量。 使用值
    > 1。

3.  **负载均衡器类型**：选择"外部"，以便可以从公共网络访问容器化应用程序。

4.  **应用程序配置**：对于参数化的应用程序配置，将以下值用于当前部署。

    -   导航到 LearnAppContainerization 资源组 [[Azure
        > 门户]{.underline}](https://portal.azure.com/)。

    -   复制 MySQL 服务器的名称。 MySQL 服务器的名称将采用以下格式
        > -"airsonic-mysql-server-0000000000"。

    -   MySQL 服务器的名称将采用以下格式
        > -"airsonic-mysql-server-0000000000"。

        -   **用户名**：将用户名生成为"mysqladmin@\${MYSQL_SERVER_NAME}"。

        -   **密码**：将值指定为"SuperS3kretPasSw0rd"。

        -   **URL：**若要创建要指定的 URL，请替换以下连接字符串中的
            > MySQL 服务器名称，并将其粘贴到应用容器化工具中。\
            > **jdbc:mysql://\${MYSQL_SERVER_NAME}.mysql.database.azure.com:3306/airsonic?useSSL=false&sessionVariables=sql_mode=ANSI_QUOTES**

```{=html}
<!-- -->
```
1.  单击"应用"以保存部署配置。

2.  单击"继续"以部署应用程序。

> ![Screenshot for deployment app
> configuration.](./media/media/image14.png){width="5.768055555555556in"
> height="5.13125in"}

**部署应用**

保存应用程序的部署配置后，该工具将为应用程序生成 Kubernetes 部署 YAML
文件。 该文件将基于部署规范步骤中指定的输入创建。

1.  单击"编辑"，查看并自定义应用程序的 Kubernetes 部署 YAML。
    > 该工具提供了自定义 YAML
    > 文件的选项，但对于本练习，我们将使用该工具生成的默认文件。

2.  选择要部署的应用程序。

3.  单击"部署"启动所选应用程序的部署。

4.  部署应用程序后，可以单击"部署状态"列以跟踪为该应用程序部署的资源。

**浏览已部署的应用程序**

部署应用程序容器后，可以按如下所示浏览已迁移的应用程序：

1.  复制"部署状态"列中显示的 IP 地址。

2.  将 IP 地址粘贴到新的浏览器选项卡中。

3.  将 IP 地址追加到以下字符串 - :8080/airsonic

你应该能够访问应用程序。

**摘要**

-   2 分钟

干得漂亮! 在此模块中，你了解了使用 Azure Migrate 应用容器化对 Java Web
应用进行容器化并将其迁移到 AKS 来加快应用程序现代化进程。 你使用了 Azure
Migrate：应用容器化工具对 Linux 计算机上运行的 Java Web 应用和 Airsonic
进行容器化并将其迁移到 AKS。

应用容器化工具是一个独立的工具，可在所有 Windows 计算机上运行。
该工具可以远程连接到 Linux
服务器计算机，该计算机运行要进行容器化并迁移到 AKS 或 Azure 应用服务的
Java Web 应用。
该工具将标识需要一起打包到容器映像中的组件，并生成一个有助于生成该容器映像的
Dockerfile。 此外，还可使用基于 Azure
容器注册表生成的功能来生成该容器映像，并推送到容器注册表。
然后，该工具将生成在 AKS 上部署该应用程序所需的 Kubernetes 清单文件。
在此过程中，每个步骤中的自定义可帮助你参数化应用配置，使用 Azure Key
Vault 存储和管理应用程序机密，甚至有助于将应用内容移动到 AKS 永久性卷。
工具生成的项目（如 Dockerfile 和 Kubernetes
清单文件）可以下载、保存并进一步用于第二天操作。

**了解详细信息**

有关使用 Azure Migrate 对应用程序进行容器化并将其迁移到 AKS
的详细信息，请参阅以下文章：

-   [[对 ASP.NET 应用程序进行容器化并将其迁移到
    > AKS]{.underline}](https://go.microsoft.com/fwlink/?linkid=2173710)

-   [[对 Java Web 应用进行容器化并将其迁移到
    > AKS]{.underline}](https://go.microsoft.com/fwlink/?linkid=2173511)

-   [[对 ASP.NET 应用程序进行容器化并将其迁移到 Azure
    > 应用服务]{.underline}](https://go.microsoft.com/fwlink/?linkid=2173711)

-   [[对 Java Web 应用进行容器化并将其迁移到 Azure
    > 应用服务]{.underline}](https://go.microsoft.com/fwlink/?linkid=2173512)

**恭喜您！实验"将 Java Web 应用迁移到AKS"已完成！**
