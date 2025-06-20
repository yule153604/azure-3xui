# **指南：使用 Azure学生订阅和 3X-UI 搭建个人代理服务器**

本指南将引导您完成申请 Azure学生订阅、创建 Linux 虚拟机 (VM) 以及安装 3X-UI 面板来管理您的代理服务的全过程。

## **第 1 步：申请 Azure 学生订阅**

1. **申请订阅**：  
   * 您可以参考此处的详细步骤进行申请： https://zhuanlan.zhihu.com/p/629311513  
   * **重要提示**：填写个人资料时，建议不要选择“中国大陆”。  
2. **检查额度**：  
   * 成功申请后，您应该能在您的 Azure 账户中看到 100 美元的额度。  
   * (图片示例：Azure 学生订阅额度，类似于 \[来源文档图片 1\])
![](https://github.com/yule153604/azure-3xui/blob/main/Image/image001.png)
## **第 2 步：创建 Linux 虚拟机 (VM)**

1. **登录 Azure 门户**：  
   * 使用您的学生账户访问 Azure 门户。  
   * 在主页上，点击“**创建资源**”。  
   * (图片示例：Azure 创建资源按钮，类似于 \[来源文档图片 2\])
         ![](https://github.com/yule153604/azure-3xui/blob/main/Image/image003.png)
2. **查找并创建虚拟机**：  
   * 在“创建资源”页面，搜索“虚拟机”，然后点击“**创建**”。  
3. **配置 VM 基本信息**：  
   * **订阅**：选择“**Azure for Students**”。  
   * **资源组**：新建一个（例如，“yule-rg”）。  
   * **虚拟机名称**：为您的 VM 指定一个名称（例如，“my-proxy-vm”）。  
   * **区域**：选择您的服务器所在的数据中心位置。此选项会显著影响连接延迟。通常，选择地理位置较近的区域会获得更快的速度。  
   * **可用性选项**：无需基础结构冗余（默认）。  
   * **安全类型**：标准。  
   * **映像**：选择一个 Ubuntu 服务器映像（例如，Ubuntu Server 22.04 LTS \- Gen2 或 Ubuntu Server 20.04 LTS \- Gen2）。  
   * **VM 体系结构**：x64。  
   * **大小**：选择合适的 VM 大小。“B2ats” (Standard\_B2ats) 通常是学生额度下性价比较高的入门选择。  
   * (图片示例：Azure VM 创建 \- 实例详细信息，类似于 \[来源文档图片 3\])
     ![](https://github.com/yule153604/azure-3xui/blob/main/Image/image004.png)
   * **身份验证类型**：选择“**SSH 公钥**”。  
   * **用户名**：选择一个用户名（例如，“azureuser”）。  
   * **SSH 公钥源**：选择“**生成新密钥对**”。  
   * **密钥对名称**：为您的密钥对指定一个名称（例如，“yule1”）。  
     * **关键步骤**：继续操作时，Azure 会提示您下载私钥（.pem 文件）。**请务必下载并将其保存在安全的位置。** 您将需要此文件来连接到您的 VM。  
   * **公共入站端口**：选择“**允许所选端口**”。  
   * **选择入站端口**：确保已选择 **SSH (22)**。我们稍后会配置其他端口。  
4. **配置磁盘**：  
   * **OS 磁盘类型**：选择“**高级 SSD (本地冗余存储)**”以获得更好的性能。文档中提到选择“p6”，这通常对应于高级 SSD 的特定大小/性能层 (例如 64 GiB P6)。  
   * (图片示例：Azure VM 创建 \- OS磁盘，类似于 \[来源文档图片 4\])
     ![](https://github.com/yule153604/azure-3xui/blob/main/Image/image006.png)
   * 除非有特定需求，否则将其他磁盘设置保留为默认值。  
   * 点击 **下一步：网络 \>**。  
5. **配置网络**：  
   * **虚拟网络、子网、公共 IP**：Azure 默认会创建新的。这通常没有问题。  
     * 关于公共 IP，文档中提到静态 IP 可能产生费用。如果选择新建公共 IP，可以先使用默认的动态分配。  
     * (图片示例：创建公共 IP 地址，如果选择新建静态IP，类似于 \[来源文档图片 5\])
       ![](https://github.com/yule153604/azure-3xui/blob/main/Image/image007.png)
   * **NIC 网络安全组**：选择“**高级**”。  
   * **配置网络安全组**：点击“**新建**”。  
     * 如果“基本信息”选项卡中尚未设置，请添加入站规则以允许 SSH（端口 22）。我们稍后将为 3X-UI 面板和代理服务添加更多规则。  
   * 最初，您可以使用动态 IP。文档指出，在某些情况下，微软已停止提供免费的动态 IP，静态 IP 可能会产生费用并从您的额度中扣除。在本指南中，我们将使用 VM 创建期间分配的 IP，如果需要，稍后将讨论通过 CLI 创建动态 IP。  
6. **配置管理和高级选项**：  
   * **监视**：在“监视”选项卡下（或某些布局中的“管理”然后“监视”），您可以通过选择“**禁用**”来禁用**启动诊断**，以节省少量成本/资源。  
   * (图片示例：禁用启动诊断，类似于 \[来源文档图片 10\])
     ![](https://github.com/yule153604/azure-3xui/blob/main/Image/image018.png)
   * 将其余选项保留为默认值。  
7. **查看并创建**：  
   * 检查所有设置。  
   * (图片示例：创建虚拟机 \- 摘要/验证通过，类似于 \[来源文档图片 11\])
     ![](https://github.com/yule153604/azure-3xui/blob/main/Image/image020.png)
   * 点击“**查看 \+ 创建**”，然后点击“**创建**”以部署您的虚拟机。等待部署完成。

## **第 2.1 步 (可选)：通过 Azure CLI 创建动态公共 IP**

如果您需要单独创建一个*新的*动态公共 IP 地址（例如，如果在 VM 创建期间没有获取到，或者需要额外的 IP），您可以使用 Azure Cloud Shell。微软可能会对静态 IP 地址收费，这可能会从您的额度中扣除。以下命令用于创建动态 IP。

1. **打开 Azure Cloud Shell**：  
   * 在 Azure 门户中，点击顶部工具栏中的 Cloud Shell 图标 (类似 \>\_)。  
   * (图片示例：指向 Cloud Shell 图标，类似于 \[来源文档图片 6\])
     ![](https://github.com/yule153604/azure-3xui/blob/main/Image/image011.png)
2. **列出位置 (可选)**：  
   * 要查找您的资源组对应的正确 location 名称，请执行：  
     az account list-locations \--output table

   * 查找与您为 VM 选择的区域对应的“Name”（例如，japaneast、southcentralus）。  
   * (图片示例：Cloud Shell az account list-locations 输出，类似于 \[来源文档图片 7\])
     ![](https://github.com/yule153604/azure-3xui/blob/main/Image/image012.png) 
3. **创建动态公共 IP**：  
   * 执行以下命令，并将占位符替换为您的实际值：  
     az network public-ip create \--resource-group 您的资源组名称 \--name 您期望的IP名称 \--sku Basic \--allocation-method Dynamic \--version IPv4 \--location 您的VM区域名称

     * \--resource-group：您的资源组名称（例如，“yule”）。  
     * \--name：此公共 IP 资源的名称（例如，“JP-IP”）。  
     * \--location：上一步中找到的区域名称（例如，“japaneast”）。  
   * (图片示例：Cloud Shell az network public-ip create 输出，类似于 \[来源文档图片 8\])
     ![](https://github.com/yule153604/azure-3xui/blob/main/Image/image014.png)
   * 这个新创建的动态 IP 将在您的网络资源中可用。然后，您需要将此 IP 与您的 VM 的网络接口控制器 (NIC) 相关联。  
   * (图片示例：网络接口中选择新创建的动态IP，类似于 \[来源文档图片 9\])
     ![](https://github.com/yule153604/azure-3xui/blob/main/Image/image016.png)
   * （对于本指南，通常使用随 VM 自动创建的公共 IP 就足够了。）

## **第 3 步：通过 SSH 连接到您的 VM**

1. **获取您的 VM 的公共 IP 地址**：  
   * 在 Azure 门户中导航到您新创建的 VM。  
   * 在“概述”页面上，您会找到“公共 IP 地址”。  
2. **打开终端**：  
   * **Windows**：使用 PowerShell、命令提示符或专用的 SSH 客户端（如 PuTTY）。  
   * **macOS/Linux**：使用内置的终端。  
3. **SSH 命令**：  
   * 使用以下命令，将 您的私钥路径 替换为您下载的 .pem 文件的实际路径，并将 您的VM公共IP 替换为您的 VM 的公共 IP 地址：  
     ssh \-i /path/to/您的私钥文件名.pem azureuser@您的VM公共IP

     例如：  
     ssh \-i C:\\Users\\yyul\\.ssh\\yule1.pem azureuser@52.185.129.180

4. **连接**：  
   * 首次连接时，系统可能会要求您确认主机的真实性。输入“**yes**”并按 Enter。  
   * (图片示例：SSH 成功连接后的终端界面，类似于 \[来源文档图片 10之后的SSH连接图\])
        ![](https://github.com/yule153604/azure-3xui/blob/main/Image/image022.png)
   * 连接成功后，切换到 root 用户以获取管理员权限：  
     sudo \-i

## **第 4 步：安装 3X-UI 面板**

1. **运行安装脚本**：  
   * 在您的 VM 终端中执行以下命令，以下载并运行 3X-UI 安装脚本：  
     bash \<(curl \-Ls https://raw.githubusercontent.com/xeefei/3x-ui/master/install.sh)

   * 按照脚本的屏幕提示进行操作。它将安装 3X-UI 及相关组件（如 Xray-core）。  
   * (图片示例：3X-UI 安装脚本运行过程中的终端输出，类似于 \[来源文档图片 11之后的安装图1\])
         ![](https://github.com/yule153604/azure-3xui/blob/main/Image/image024.png)
2. **访问 3X-UI 菜单和面板信息**：  
   * 安装完成后，脚本通常会提供一个管理 3X-UI 的命令（通常就是 x-ui）。运行此命令将显示一个菜单。  
   * (图片示例：3X-UI 管理脚本菜单，类似于 \[来源文档图片 11之后的安装图2\])
         ![](https://github.com/yule153604/azure-3xui/blob/main/Image/image026.png)
   * 输入选项 10（或“查看面板信息”的相关选项）以显示您的面板访问详细信息，包括用户名、密码、端口和 webBasePath。  
   * (图片示例：查看 3X-UI 面板信息，类似于 \[来源文档图片 11之后的安装图3\])
         ![](https://github.com/yule153604/azure-3xui/blob/main/Image/image028.png)
   * 记下 username、password、port 和 webBasePath。面板访问 URL 通常是 http://您的VM公共IP:面板端口/您的webBasePath/。

## **第 5 步：配置防火墙并访问面板**

您需要在两个地方开放端口：

* **Azure 网络安全组 (NSG)**：这是 Azure 为您的 VM 提供的防火墙。  
* **VM 的内部防火墙** (例如 ufw)：3X-UI 脚本提供了管理此防火墙的选项。  
1. **在 Azure NSG 中打开 3X-UI 面板端口**：  
   * 转到 Azure 门户中的您的 VM。  
   * 导航到 **设置 \> 网络**。  
   * 点击“**添加入站端口规则**”。  
   * (图片示例：“添加入站端口规则”按钮，类似于 \[来源文档中防火墙部分图片1\])
         ![](https://github.com/yule153604/azure-3xui/blob/main/Image/image030.png)
   * 进行如下设置：  
     * **源**：任何  
     * **源端口范围**：\*  
     * **目标**：任何 (或选择您的公共 IP)  
     * **服务**：自定义  
     * **目标端口范围**：输入您的 3X-UI 面板的 端口 (例如，从面板信息中获取的 10000 或源文档中的 19000，请根据实际情况填写)。  
     * (图片示例：为面板设置目标端口范围，类似于 \[来源文档中防火墙部分图片3\])
           ![](https://github.com/yule153604/azure-3xui/blob/main/Image/image032.png)
     * **协议**：TCP  
     * **操作**：允许  
     * **优先级**：分配一个唯一的优先级编号（例如，310）。  
     * **名称**：为其指定一个描述性名称（例如，“Allow-3XUI-Panel”）。  
   * 点击“**添加**”。  
2. **在 VM 的防火墙中打开 3X-UI 面板端口 (使用 3X-UI 脚本)**：  
   * 在您的 SSH 会话中，运行 3x-ui 命令调出菜单。  
   * 选择选项 21（或类似选项，用于“防火墙管理”/“开放端口”）。  
   * 出现提示时，输入相同的面板端口（例如，10000 或实际面板端口）。  
3. **访问 3X-UI 面板**：  
   * 打开 Web 浏览器并转到面板 URL：http://您的VM公共IP:面板端口/您的webBasePath/。  
     * 例如：http://52.185.129.180:10000/3x-ui/ (请根据您的实际端口和路径修改)  
   * 使用您之前记下的 username 和 password 登录。  
   * (图片示例：3X-UI 面板登录页面，类似于 \[来源文档中防火墙部分图片4\])
    ![](https://github.com/yule153604/azure-3xui/blob/main/Image/image034.png)
## **第 6 步：在 3X-UI 中配置入站代理服务**

1. **添加入站连接**：  
   * 在 3X-UI 面板中，导航到“Inbounds”或“入站列表”。  
   * 点击“+ Add Inbound”或“添加入站”。  
   * 配置您想要的代理协议（例如，VLESS、VMess、Trojan）。  
     * **备注**：为其指定一个名称。  
     * **端口**：为此代理服务选择一个端口（例如，示例图片中显示的 29989）。**此端口也必须在 Azure NSG 和 VM 的防火墙中打开。**  
     * **协议**：选择您想要的协议。  
     * **传输设置**：根据需要进行配置（例如，TCP、WebSocket、gRPC）。示例图片显示的是 TCP。  
     * **安全**：根据协议配置安全设置，如 reality 或 tls。  
     * 如果使用 TLS，您可能需要配置域名和证书（图片显示了一个使用 tesla.com 作为 SNI 和 Get New Cert 选项的示例）。  
   * (图片示例：3X-UI 添加入站代理服务配置，类似于 \[来源文档中入站规则图片1\])
         ![](https://github.com/yule153604/azure-3xui/blob/main/Image/image036.png)
   * 保存入站配置。  
2. **在 Azure NSG 中打开代理服务端口**：  
   * 返回 Azure 门户 \> 您的 VM \> 网络 \> 添加入站端口规则。  
   * 为您刚配置的代理服务端口添加一条新规则（例如，29989）。  
     * **服务**：自定义  
     * **目标端口范围**：您选择的代理端口（例如，29989）  
     * **协议**：TCP (如果您的代理服务使用 UDP，则选择 UDP)  
     * **名称**：例如，“Allow-VLESS-TCP”  
   * 点击“**添加**”。  
3. **在 VM 的防火墙中打开代理服务端口 (使用 3X-UI 脚本)**：  
   * 在您的 SSH 会话中，运行 3x-ui 调出菜单。  
   * 再次使用选项 21 打开新的代理服务端口（例如，29989）。

## **第 7 步：将配置导出到客户端 (例如 V2RayN)**

1. **从 3X-UI 面板导出**：  
   * 在 3X-UI 面板中，找到您新创建的入站服务。  
   * 应该有一个“导出链接”、“复制二维码”或“复制配置”的选项。这将为您提供客户端应用程序可以导入的配置字符串或二维码。  
   * (图片示例：“导出链接”选项，类似于 \[来源文档中导出链接图片1\])
         ![](https://github.com/yule153604/azure-3xui/blob/main/Image/image038.png)
2. **导入到 V2RayN (或其他客户端)**：  
   * 打开您的 V2RayN 客户端软件。  
   * (图片示例：V2RayN 导入配置选项，类似于 \[来源文档中导出链接图片2\])
         ![](https://github.com/yule153604/azure-3xui/blob/main/Image/image040.png)
   * 导入配置（例如，通过粘贴链接、扫描二维码或从文件导入）。  
3. **配置 V2RayN**：  
   * **系统代理**：设置为“**自动配置系统代理**”。  
   * **路由模式**：根据您的偏好选择“**全局**”(所有流量都通过代理) 或“**绕过大陆**”(绕过中国大陆地址，仅代理海外流量)。  
   * (图片示例：V2RayN 系统代理和路由设置，类似于 \[来源文档中导出链接图片3\])
         ![](https://github.com/yule153604/azure-3xui/blob/main/Image/image042.png)
4. **测试连接**：  
   * 在 V2RayN 中选择导入的服务器并连接。  
   * 通过在浏览器中检查您的 IP 地址或尝试访问有区域限制的内容来测试您的互联网流量现在是否通过您的 Azure VM 路由。  
   * (图片示例：Google 搜索页面，暗示连接成功，类似于 \[来源文档中测试图片1\])
         ![](https://github.com/yule153604/azure-3xui/blob/main/Image/image044.png)
   * (图片示例：Speedtest 测速结果，类似于 \[来源文档中测试图片2\])
    ![](https://github.com/yule153604/azure-3xui/blob/main/Image/image046.png)
您现在应该在您的 Azure VM 上拥有一个由 3X-UI 面板管理的工作正常的代理服务器了。请记住监控您的 Azure 额度，以确保不会意外用尽。
