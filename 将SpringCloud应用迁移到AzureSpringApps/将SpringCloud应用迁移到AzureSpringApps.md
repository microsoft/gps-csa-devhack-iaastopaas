**将Spring Cloud应用迁移到Azure Spring Apps**

# 单元1 ：简介

-   3 分钟

在上一模块中，你建立了将 Spring Petclinic 应用程序迁移到 Azure
平台的计划。 现在可以执行第一个 Spring Petclinic 组件的实际迁移了。

Azure Spring Apps 服务为要使用的示例应用提供配置服务器。

在本模块中，首先创建 Azure Spring Apps 服务，然后创建 git 存储库。 创建
git 存储库后，将为 Azure Spring Apps 实例创建配置服务器以连接到 git
存储库。 然后，你将创建 MySQL 数据库服务。 在 MySQL
数据库服务到位后，可以将微服务部署到 Azure Spring Apps 服务。
最后，你将通过公开可用的终结点测试应用程序，以确保一切正常运行。

## 学习目标

完成此模块后，你将能够：

-   创建 Azure Spring Apps 服务。

-   配置配置服务器。

-   创建 Azure MySQL Database 服务。

-   将 Spring Petclinic 应用的第一个组件部署到 Azure Spring Apps 服务。

-   为 Spring Petclinic 应用程序提供公开可用的终结点。

-   通过公开可用的终结点测试应用程序。

## 先决条件

-   对以下工具的了解达到中等水平并在本地进行安装：Git、Java JDK 8
    > 或更高版本，以及 Java IDE 或文本编辑器。

-   基本 Git 命令，包括克隆、提交文件和将更改推送到 GitHub。

-   相对熟悉 Azure。

** 备注**

本练习中提供的说明假定你已成功完成前面的练习并使用相同的实验室环境，包括已设置相关环境变量的
Git Bash 会话。

若要下载此模块的说明，请参阅 GitHub 中的[在 Azure Spring
应用中部署和运行 Java
应用](https://github.com/MicrosoftLearning/Deploying-and-Running-Java-Applications-in-Azure-Spring-Apps/tree/master/Instructions/Labs)。

# 单元2: 创建 Azure Spring Apps 服务

-   10 分钟

Azure Spring Apps 是面向 Spring 开发人员的平台即服务 (PaaS)。
使用全面的监视和诊断、配置管理、服务发现、CI/CD 集成和蓝绿部署来管理
Spring Boot 应用程序的生命周期。

![关系图显示 Azure Spring Apps
服务如何交互。](.//media/image1.png){width="5.768055555555556in"
height="2.9881944444444444in"}

可使用 Azure 门户或 Azure CLI 部署 Azure Spring Apps 服务的实例。

工作站应安装以下组件：

-   [Visual Studio Code
    > 下载](https://code.visualstudio.com/download)中提供的 Visual
    > Studio Code

-   [Git 下载](https://git-scm.com/downloads)中提供的 Git

-   [Apache Maven Project
    > 下载](https://maven.apache.org/download.cgi)中提供的 Apache Maven
    > 3.8.5

-   [JDK 下载](https://download.oracle.com/java/18/latest/jdk-18_windows-x64_bin.msi)中提供的
    > Java 开发工具包 (JDK)。

-   若要下载此模块的说明，请参阅[在 Azure Spring Apps 中部署和运行 Java
    > 应用](https://github.com/MicrosoftLearning/Deploying-and-Running-Java-apps-in-Azure-Spring-Cloud)。

对于 Git 安装，请通过从 Git Bash shell 运行以下命令，设置全局配置变量
user.email 和 user.name：

Bash复制

git config \\

\--global user.email \"\<your-email-address>\"

git config \\

\--global user.email \"\<your-email-address>

若要安装 Apache Maven，请通过运行解压缩 apache-maven-3.8.5-bin.zip
提取.zip 文件的内容。 通过从 Git Bash shell
运行以下命令，将已提取内容的箱目录的路径添加到 PATH 环境变量：

Bash复制

export PATH=\~/apache-maven-3.8.5/bin:\$PATH

若要安装 JDK，请按照 [JDK
安装指南](https://docs.oracle.com/en/java/javase/18/install/installation-jdk-microsoft-windows-platforms.html)中提供的说明进行操作。
通过从 Git Bash shell 运行以下命令来设置 JAVA_HOME 环境变量：

Bash复制

export JAVA_HOME=\"/c/Program Files/Java/jdk-18.0.1.1\"

** 备注**

确保已启用适用于 AZ CLI 的 Azure Spring Apps 扩展。

## 创建 Azure Spring Apps 服务

以下过程使用 Azure CLI 扩展来部署 Azure Spring Apps 的实例。

1.  在 Git Bash 提示符下，运行以下命令登录到 Azure 帐户。

> Bash复制
>
> az login

2.  将扩展更新到最新版本。

> Azure CLI复制
>
> az extension update \\
>
> \--name spring-cloud

3.  确保已登录到连续命令的正确订阅。

> Azure CLI复制
>
> az account list -o table

4.  如果上述语句未显示将正确帐户指示为默认帐户，请使用以下命令将环境更改为正确的订阅，并替换 \<subscription-id>。

> Azure CLI复制
>
> az account set \\
>
> \--subscription \<subscription-id>

5.  运行以下命令以创建包含所有资源的资源组。
    > 将 \<azure-region> 占位符替换为可在其中创建 Azure Spring Apps
    > 服务的标准 SKU 实例和 Azure Database for MySQL
    > 单一服务器实例的任何 Azure 区域的名称。
    > 有关服务可用性，请参阅[可用产品(按区域)](https://azure.microsoft.com/global-infrastructure/services/?products=mysql%2Cspring-apps&regions=all)页。

> Azure CLI复制
>
> UNIQUEID=\$(openssl rand -hex 3)
>
> RESOURCE_GROUP=springappslab_rg\_\$UNIQUEID
>
> LOCATION=\<azure-region>
>
> az group create -g \$RESOURCE_GROUP -l \$LOCATION

6.  运行下面列出的命令，创建 Azure Spring Apps 服务的标准 SKU 的实例。
    > 服务的名称必须是全局唯一的，并且仅包含小写字母、数字和连字符。

> Azure CLI复制
>
> SPRING_APPS_SERVICE=springappssvc\$UNIQUEID
>
> \--name \$SPRING_APPS_SERVICE \\
>
> \--resource-group \$RESOURCE_GROUP \\
>
> \--location \$LOCATION \\
>
> \--sku Standard
>
> ** 备注**
>
> 如果需要，上述命令将自动注册 Spring 扩展。 使用 Y 确认扩展安装。
>
> ** 备注**
>
> 还可以使用 az 扩展 add \--name spring. 添加 spring 扩展
>
> ** 备注**
>
> 预配大约需要 5 分钟。

7.  可以设置默认的资源组名称和 Spring Apps 服务名称。
    > 通过设置这些默认值，无需为后续命令重复这些名称。

> Azure CLI复制
>
> az config set defaults.group=\$RESOURCE_GROUP
> defaults.spring-cloud=\$SPRING_APPS_SERVICE

8.  在新选项卡中，打开 [Azure 门户](https://portal.azure.com/)。

9.  使用"搜索资源、服务和文档"搜索使用 Azure Spring Apps 创建的资源组。

> 在资源组概述中，你将看到新创建的 Azure Spring Apps 实例。
>
> ![Azure Spring Apps
> 资源组的屏幕截图。](.//media/image2.png){width="5.768055555555556in"
> height="1.9583333333333333in"}
>
> ** 备注**
>
> 如果在资源组的概述列表中看不到 Azure Spring Apps
> 服务，可能需要刷新视图。

10. 选择 Azure Spring Apps 实例，然后选择"应用"。
    > 请注意，没有部署到实例的应用，但在即将到来的练习中将添加应用。

> ![所列出的 Azure Spring Apps
> 服务应用的屏幕截图。](.//media/image3.png){width="5.768055555555556in"
> height="2.9583333333333335in"}

# 单元3: 在 Azure Spring Apps 中配置托管的 Spring Cloud Config Server

-   12 分钟

Azure Spring Apps Config Server 是分布式系统的集中配置服务。
它使用当前支持本地存储、Git 和 Subversion 的可插入存储库层。
在此联系中，你将设置 Config Server 以从 Git
存储库获取数据。 ![练习源文件的屏幕截图。](.//media/image4.png){width="5.768055555555556in"
height="5.4743055555555555in"}在本模块中，将为 Spring Apps
应用设置配置服务器。 需要将配置服务器链接到 git 存储库。

Azure Spring Apps Config Server 是分布式系统的集中配置服务。
它使用当前支持本地存储、Git 和 Subversion 的可插入存储库层。 设置 Config
Server，将 Spring 应用部署到 Azure Spring Apps。

Spring 微服务使用的配置驻留在 [PetClinic GitHub
存储库](https://github.com/spring-petclinic/spring-petclinic-microservices-config)中。在本练习中，你将创建自己的专用
Git 存储库，然后更新配置设置。

使用 Web 浏览器，导航到 [GitHub](https://github.com/) 并登录自己的
GitHub 帐户。 如果没有 GitHub
帐户，请导航到["加入GitHub"页](https://github.com/join)，按照["注册新
GitHub
帐户"页](https://docs.github.com/en/get-started/signing-up-for-github/signing-up-for-a-new-github-account)上的说明创建一个帐户。

1.  在 GitHub 帐户中，导航到"存储库"页，并创建新的名为
    > spring-petclinic-microservices 的专用存储库。

> ** 备注**
>
> 请确保将 git 存储库配置为专用存储库。
>
> ** 备注**
>
> 记录新创建的 GitHub 存储库的 URL 的值。 本实验室后面将用到该 URL。
> 其值应为 [***https://github.com/your-github-username/spring-petclinic-microservices-private.git***](https://github.com/your-github-username/spring-petclinic-microservices-private.git)，其中
> your-github-username 占位符表示 GitHub 用户名。

2.  从 Git Bash 提示中，请确保不再位于 spring-petclinic-microservice
    > 文件夹中，并克隆 spring-petclinic-microservices-config 存储库。

> Bash复制
>
> cd \~/projects
>
> git clone
> https://github.com/\<your-github-username>/spring-petclinic-microservices-config.git

3.  从 Git Bash 提示中，移动到新创建的
    > spring-petclinic-microservices-config 文件夹，并运行以下命令。
    > 这些命令将所有配置服务器配置 yaml
    > 文件从 [spring-petclinic-microservices-config](https://github.com/spring-petclinic/spring-petclinic-microservices-config) 复制到实验室计算机上的本地文件夹。

> 命令应类似于：
>
> Bash复制
>
> cd spring-petclinic-microservices-config
>
> curl -o admin-server.yml
> https://raw.githubusercontent.com/spring-petclinic/spring-petclinic-microservices-config/main/admin-server.yml
>
> curl -o api-gateway.yml
> https://raw.githubusercontent.com/spring-petclinic/spring-petclinic-microservices-config/main/api-gateway.yml
>
> curl -o application.yml
> https://raw.githubusercontent.com/spring-petclinic/spring-petclinic-microservices-config/main/application.yml
>
> curl -o customer-service.yml
> https://raw.githubusercontent.com/spring-petclinic/spring-petclinic-microservices-config/main/customer-service.yml
>
> curl -o discovery-server.yml
> https://raw.githubusercontent.com/spring-petclinic/spring-petclinic-microservices-config/main/discovery-server.yml
>
> curl -o tracing-server.yml
> https://raw.githubusercontent.com/spring-petclinic/spring-petclinic-microservices-config/main/tracing-server.yml
>
> curl -o vets-service.yml
> https://raw.githubusercontent.com/spring-petclinic/spring-petclinic-microservices-config/main/vets-service.yml
>
> curl -o visit-service.yml
> https://raw.githubusercontent.com/spring-petclinic/spring-petclinic-microservices-config/main/visit-service.yml

4.  在 Git Bash 窗口中运行以下命令，将 [Spring
    > Petclinic](https://github.com/spring-petclinic/spring-petclinic-microservices) 应用程序克隆到工作站：

> Bash复制
>
> git add .
>
> git commit -m \'added base config\'
>
> git push

5.  在 Web 浏览器中，刷新新创建的 spring-petclinic-microservices-config
    > 存储库的页面，然后仔细检查这里是否包含所有配置文件。

完成托管服务器配置的 Git 存储库的初始更新后，需要为 Spring Apps
实例设置配置服务器。 在设置过程中，需要在 GitHub
存储库中创建个人访问令牌 (PAT)，并使其对配置服务器可用。

有关参考信息，请见以下信息：

-   [有关配置服务器设置的指南](https://github.com/MicrosoftDocs/azure-docs/blob/main/articles/spring-cloud/quickstart-setup-config-server.md)

-   [使用基本身份验证的专用存储库指南](https://github.com/MicrosoftDocs/azure-docs/blob/main/articles/spring-cloud/how-to-config-server.md)

若要创建个人访问令牌，请执行以下任务：

1.  若要创建个人访问令牌，请切换到显示专用 GitHub 存储库的 Web
    > 浏览器窗口，选择右上角的头像图标，然后选择"设置"。

2.  在垂直导航菜单底部，选择"开发人员设置"，选择"个人访问令牌"，然后选择"生成新令牌"。

3.  确认访问权限，输入 GitHub 帐户密码，然后选择"确认密码"。

4.  在"新建个人访问令牌"页上的"备注"文本框中，输入描述性名称，例如
    > spring-petclinic-config-server-token。

5.  确保"过期"下拉列表中的值设置为"30 天"。

6.  在"选择范围"部分中，选择"存储库"，然后选择"生成令牌"。

7.  记录生成的令牌。 你将在下一步中需要它。

8.  在 Git Bash 提示符下运行以下命令，设置托管 GitHub
    > 存储库的环境变量，并将配置服务器设置为指向 GitHub 存储库。

> ** 备注**
>
> 将 git_repository、git_username 和git_password 占位符替换为 GitHub
> 存储库的 URL、GitHub 用户帐户的名称和个人访问令牌值。
>
> Bash复制
>
> GIT_REPO=\[git repository\]
>
> GIT_USERNAME=\[git username\]
>
> GIT_PASSWORD=\[git password\]
>
> az spring-apps config-server git set \\
>
> \--name \$SPRING_APPS_SERVICE \\
>
> \--resource-group \$RESOURCE_GROUP \\
>
> \--label main \\
>
> \--password \$GIT_PASSWORD \\
>
> \--username \$GIT_USERNAME

# 单元4: 创建 Azure MySQL 数据库服务

-   18 分钟

现在，你拥有将托管应用程序的计算服务，以及迁移的应用程序将使用的配置服务器。
在开始将单个微服务部署为 Azure Spring Apps
应用程序之前，首先需要为其创建一个 Azure Database for MySQL
单一服务器托管的数据库。

![显示 MySQL 服务器与 Azure
门户和其他管理工具交互的关系图。](.//media/image5.png){width="5.768055555555556in"
height="3.261111111111111in"}

要详细了解如何使用 Azure CLI 创建 Azure Database for MySQL
服务器，请参阅本[快速入门](https://learn.microsoft.com/zh-cn/azure/mysql/quickstart-create-mysql-server-database-using-azure-cli)。

还需要更新应用程序的配置，以使用新预配的 MySQL 服务器来授权访问专用
GitHub 存储库。 你将使用 MySQL 服务器连接字符串中提供的值更新专用 git
配置存储库的 application.yml 配置文件。

1.  运行以下命令以创建 Azure Database for MySQL 单一服务器的实例。
    > 服务器的名称必须全局唯一，因此，如果已使用随机生成的名称，请相应地对其进行调整。
    > 请记住，该名称只能包含小写字母、数字和连字符。 此外，请将
    > myadmin_password 占位符替换为复杂密码并记录其值。

> Bash复制
>
> SQL_SERVER_NAME=springappsmysql\$RANDOM\$
>
> RANDOM SQL_ADMIN_PASSWORD=\<myadmin_password>
>
> DATABASE_NAME=petclinic
>
> az mysql server create \\
>
> \--admin-user myadmin \\
>
> \--admin-password \${SQL_ADMIN_PASSWORD} \\
>
> \--name \${SQL_SERVER_NAME} \\
>
> \--resource-group \${RESOURCE_GROUP} \\
>
> \--sku-name GP_Gen5_2 \\
>
> \--version 5.7 \\
>
> \--storage-size 5120
>
> ** 备注**
>
> 预配可能需要大约 3 分钟。

2.  创建 Azure Database for MySQL
    > 单一服务器实例后，它将输出有关其设置的详细信息。
    > 在输出中，你将找到服务器连接字符串。
    > 记录其值，因为稍后在本练习中需要它。

3.  运行以下命令可在 Azure Database for MySQL
    > 单一服务器实例中创建数据库。

> Bash复制
>
> az mysql db create \\
>
> \--server-name \$SQL_SERVER_NAME \\
>
> \--resource-group \$RESOURCE_GROUP \\
>
> \--name \$DATABASE_NAME

4.  需要允许从 Azure Spring Apps 连接到服务器。
    > 你将创建服务器防火墙规则，以允许来自所有 Azure 服务的入站流量。
    > 这样，在 Azure Spring Apps 中运行的应用将能够访问 MySQL
    > 数据库，从而为他们提供持久性存储。
    > 在即将进行的练习之一中，你将限制此连接，以将其设置为仅限 Azure
    > Spring Apps 实例托管的应用。

> Bash复制
>
> az mysql server firewall-rule create \\
>
> \--name allAzureIPs \\
>
> \--server \${SQL_SERVER_NAME} \\
>
> \--resource-group \${RESOURCE_GROUP} \\
>
> \--start-ip-address 0.0.0.0 \\
>
> \--end-ip-address 0.0.0.0

5.  从 Git Bash
    > 窗口，在本地克隆的配置存储库中，使用你喜欢的文本编辑器打开
    > application.yml 文件。 更改第 82 行、83 行和 84
    > 行中的条目，其中包含目标数据源终结点、相应的管理员用户帐户及其密码的值。
    > 使用此任务前面记录的 Azure Database for MySQL
    > 单一服务器连接字符串中的信息设置这些值。 你的配置应如下所示：

> ** 备注**
>
> application.yml 文件中这三行的原始内容具有以下格式：
>
> 复制
>
> url: jdbc:mysql://localhost:3306/db?useSSL=false
>
> username: root
>
> password: petclinic
>
> ** 备注**
>
> application.yml 文件中这三行的更新内容应具有以下格式（其中
> mysql-server-name 和 myadmin-password 占位符表示 Azure Database for
> MySQL 单一服务器实例的名称和在预配期间你分配给 myadmin 帐户的密码，
>
> 复制
>
> url: jdbc:mysql://localhost:3306/db?useSSL=false
>
> url:
> jdbc:mysql://\<mysql-server-name>.mysql.database.azure.com:3306/db?useSSL=true
>
> username: myadmin@\<mysql-server-name>
>
> password: \<myadmin-password>
>
> ** 备注**
>
> 确保将 useSSL 参数的值更改为 true，并在默认情况下由 Azure Database for
> MySQL 固定服务器强制实施。

6.  通过从 Git Bash 提示符运行以下命令，保存更改并将对 application.yml
    > 文件所做的更新推送到专用 GitHub 存储库：

> Bash复制
>
> git add .
>
> git commit -m \'azure mysql info\'
>
> git push
>
> ** 备注**
>
> 管理员帐户用户名和密码存储在 application.yml 配置文件中的明文中。
> 在即将进行的练习之一中，你将通过从配置中删除明文凭据来修正此潜在漏洞。

# 单元5:将应用程序部署到 Spring Cloud 服务

-   20 分钟

现在，你拥有可用于部署应用程序组件的计算和数据服务，包括：

-   spring-petclinic-admin-server

-   spring-petclinic-customers-service

-   spring-petclinic-vets-service

-   spring-petclinic-visits-service

-   spring-petclinic-api-gateway

在此任务中，你会将这些组件作为微服务部署到 Azure Spring Apps 服务中。
不会将 spring-petclinic-config-server 和
spring-petclinic-discovery-server 部署到平台将提供的 Azure Spring Apps
中。

** 备注**

spring-petclinic-api-gateway 和 spring-petclinic-admin-server
将分配有一个公共终结点。

** 备注**

部署 customers-service、vets-service 和 visits-service 时，应在激活
mysql 配置文件的情况下执行该部署。

1.  在主 pom.xml 文件中，将第 33 行的 spring-cloud.version 从版本
    > 2021.0.2 更改为 2021.0.0，并保存该文件。

> 复制
>
> \<spring-cloud.version>2021.0.0\</spring-cloud.version>

2.  首先生成 spring petclinic
    > 应用程序的所有微服务，方法是在应用程序的根目录中运行 mvn clean
    > package。

> 复制
>
> cd \~/spring-petclinic-microservices/
>
> mvn clean package -DskipTests

3.  通过查看 mvn clean package
    > -DskipTests 命令的输出来验证生成是否成功，该输出应采用以下格式：

> 复制
>
> \[INFO\] Reactor Summary for spring-petclinic-microservices 2.6.3:
>
> \[INFO\]
>
> \[INFO\] spring-petclinic-microservices \...\...\...\...\...\...\...
> SUCCESS \[ 0.224 s\]
>
> \[INFO\] spring-petclinic-admin-server \...\...\...\...\...\...\....
> SUCCESS \[ 5.665 s\]
>
> \[INFO\] spring-petclinic-customers-service \...\...\...\...\.....
> SUCCESS \[ 4.231 s\]
>
> \[INFO\] spring-petclinic-vets-service \...\...\...\...\...\...\....
> SUCCESS \[ 3.152 s\]
>
> \[INFO\] spring-petclinic-visits-service \...\...\...\...\...\.....
> SUCCESS \[ 2.902 s\]
>
> \[INFO\] spring-petclinic-config-server \...\...\...\...\...\...\...
> SUCCESS \[ 1.030 s\]
>
> \[INFO\] spring-petclinic-discovery-server \...\...\...\...\...\...
> SUCCESS \[ 1.429 s\]
>
> \[INFO\] spring-petclinic-api-gateway \...\...\...\...\...\...\.....
> SUCCESS \[ 8.277 s\]
>
> \[INFO\]
> \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--
>
> \[INFO\] BUILD SUCCESS
>
> \[INFO\]
> \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--
>
> \[INFO\] Total time: 27.310 s
>
> \[INFO\]
> \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

4.  对于每个应用程序，现在将在 Azure Spring Apps 服务中创建一个应用。
    > 将从 api-gateway 开始。 要部署它，请从 Git Bash
    > 提示符运行以下命令：

> Bash复制
>
> az spring-cloud app create \\
>
> \--service \$SPRING_APPS_SERVICE \\
>
> \--resource-group \$RESOURCE_GROUP \\
>
> \--name api-gateway \\
>
> \--assign-endpoint true
>
> ** 备注**
>
> 等待预配完成，大约 5 分钟。

5.  接下来，通过从 Git Bash 提示符运行以下命令，将 jar
    > 文件部署到此新创建的应用：

> Bash复制
>
> az spring-cloud app deploy \\
>
> \--service \$SPRING_APPS_SERVICE \\
>
> \--resource-group \$RESOURCE_GROUP \\
>
> \--name api-gateway \\
>
> \--no-wait \\
>
> \--artifact-path
> spring-petclinic-api-gateway/target/spring-petclinic-api-gateway-2.6.7.jar

6.  以同样的方法为 admin-server 微服务创建应用：

> Bash复制
>
> az spring-cloud app create \\
>
> \--service \$SPRING_APPS_SERVICE \\
>
> \--resource-group \$RESOURCE_GROUP \\
>
> \--name app-admin \\
>
> \--assign-endpoint true
>
> ** 备注**
>
> 等待操作完成，大约 5 分钟。

7.  接下来，将 jar 文件部署到此新创建的应用：

> Bash复制
>
> az spring-cloud app deploy \\
>
> \--service \$SPRING_APPS_SERVICE \\
>
> \--resource-group \$RESOURCE_GROUP \\
>
> \--name app-admin \\
>
> \--no-wait \\
>
> \--artifact-path
> spring-petclinic-admin-server/target/spring-petclinic-admin-server-2.6.7.jar

8.  接下来，将为 customers-service 微服务创建应用：

> Bash复制
>
> az spring-cloud app create \\
>
> \--service \$SPRING_APPS_SERVICE \\
>
> \--resource-group \$RESOURCE_GROUP \\
>
> \--name customers-service
>
> ** 备注**
>
> 等待操作完成，大约 5 分钟。

9.  你不会分配终结点，但会设置 mysql 配置文件（客户服务）：

> Bash复制
>
> az spring-cloud app deploy \\
>
> \--service \$SPRING_APPS_SERVICE \\
>
> \--resource-group \$RESOURCE_GROUP \\
>
> \--name customers-service \\
>
> \--no-wait \\
>
> \--artifact-path
> spring-petclinic-customers-service/target/spring-petclinic-customers-service-2.6.7.jar
> \\
>
> \--env SPRING_PROFILES_ACTIVE=mysql

10. 接下来，将为 visits-service 微服务创建应用：

> Bash复制
>
> az spring app create \\
>
> \--service \$SPRING_APPS_SERVICE \\
>
> \--resource-group \$RESOURCE_GROUP \\
>
> \--name visits-service
>
> ** 备注**
>
> 等待操作完成，大约 5 分钟。

11. 对于 visit-service，也将跳过终结点分配，但会设置 mysql 配置文件：

> Bash复制
>
> az spring app deploy \\
>
> \--service \$SPRING_APPS_SERVICE \\
>
> \--resource-group \$RESOURCE_GROUP \\
>
> \--name visits-service \\
>
> \--no-wait \\
>
> \--artifact-path
> spring-petclinic-visits-service/target/spring-petclinic-visits-service-2.6.7.jar
> \\
>
> \--env SPRING_PROFILES_ACTIVE=mysql

12. 最后，将为 vets-service 微服务创建应用：

> Bash复制
>
> az spring app create \\
>
> \--service \$SPRING_APPS_SERVICE \\
>
> \--resource-group \$RESOURCE_GROUP \\
>
> \--name vets-service
>
> ** 备注**
>
> 等待操作完成，大约 5 分钟。

13. 在这种情况下，你也将跳过终结点分配，但会设置 mysql 配置文件：

> Bash复制
>
> az spring app deploy \\
>
> \--service \$SPRING_APPS_SERVICE \\
>
> \--resource-group \$RESOURCE_GROUP \\
>
> \--name vets-service \\
>
> \--no-wait \\
>
> \--artifact-path
> spring-petclinic-vets-service/target/spring-petclinic-vets-service-2.6.7.jar
> \\
>
> \--env SPRING_PROFILES_ACTIVE=mysql

#  单元6: 使用公开可用的终结点测试应用程序

-   5 分钟

现在已部署所有微服务，请验证应用程序是否可使用 Web 浏览器访问。

1.  若要列出所有已部署的应用，请从 Git Bash shell 运行以下 CLI
    > 语句，该语句还将列出所有可公开访问的终结点：

> Bash复制
>
> az spring app list \\
>
> \--service \$SPRING_APPS_SERVICE \\
>
> \--resource-group \$RESOURCE_GROUP \\
>
> \--output table

2.  或者，可以切换到显示 Azure 门户界面的 Web 浏览器窗口，导航到 Spring
    > Apps 实例，然后从垂直导航菜单中选择"应用"。
    > 在应用列表中，选择"api-gateway \| 概述"页面上的"api-gateway"，记下
    > URL 属性的值。

3.  打开另一个 Web 浏览器选项卡，导航到 api-gateway 终结点的 URL
    > 以显示应用程序 Web 界面。

**单元7:总结**

-   3 分钟

通过学习本模块，你了解了如何：

-   创建 Azure Spring Apps 服务。

-   配置配置服务器。

-   创建 Azure MySQL Database 服务。

-   将 Spring Petclinic 应用的第一个组件部署到 Spring Cloud 服务。

-   为 Spring Petclinic 应用程序提供公开可用的终结点。

-   通过公开可用的终结点测试应用程序。
