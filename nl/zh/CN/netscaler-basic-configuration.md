---
copyright:
  years: 1994, 2017
lastupdated: "2017-12-06"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}

# 基本负载均衡配置
假设有一家公司具有基本社交社区 Web 站点，最终用户可以在该 Web 站点中注册不需要敏感信息的帐户，注册后用户可以登录并发布宠物的照片。有三个 Web/应用程序服务器和一个用于对其备份的数据库服务器。域和 DNS 通过 {{site.data.keyword.BluSoftlayer_notm}} 托管，因为它们拥有的环境不大，所以 NetScaler 和 Web/应用程序服务器全都位于同一 VLAN 中。这简化了操作，因为无需对 NetScaler 做进一步配置，就可以设置基本负载均衡策略。以下过程是此实例中流量流的极简说明：

1. 用户在其浏览器中输入 URL。
2. URL 的 DNS 记录指向 NetScaler 上的某个公共 VIP。
3. NetScaler 接收该 VIP 上的流量，并记下正在使用的流量协议（HTTP 端口 80 流量）。
4. 接着，NetScaler 根据定义的均衡方法（循环法、持久性 IP 等）将该流量传递到服务器池中的其中一个服务器。
5. 然后，该服务器接受流量，用户连接并登录。

为了完成此任务，需要配置 NetScaler 来处理此流量。由于已配置 VIP、DNS 服务器的 IP 以及 SNIP，因此简化了配置操作。 

在 NetScaler GUI 中的“配置”屏幕上，展开左侧的**流量管理**。展开标题为**负载均衡**的子区域。然后，通过执行以下过程，指示 NetScaler 哪些目标服务器将包含在负载均衡策略中：

1. 在“负载均衡”下，单击**服务器**。
2. 单击**添加**。
3. 输入服务器的服务器名称（例如，Web1）。
4. 输入服务器的 IP 地址。
5. 使**流量域**字段保留为空，因为在此场景中，您只关注使用缺省流量域。
6. 输入关于此服务器的任何所需注释。
7. 单击**创建**。

对池中的所有服务器重复此过程。  

**提示：**为了始终能轻松识别服务器，请对相同池中的服务器使用类似的命名约定（例如，Web1、Web2、Web3 等）。

接下来，创建服务。您将为刚才输入的每个服务器创建一个服务。该服务用于配置 NetScaler 与池中服务器之间的连接。每个服务都具有一个名称，并指定一个 IP 地址、一个端口以及所服务的数据类型。

1. 单击**流量管理 > 负载均衡 > 服务**。
2. 单击**添加**。
3. 使用相同的信息为您先前创建的每个服务器创建一个服务。

接下来，创建虚拟服务器。虚拟服务器是用于负载均衡的服务器的 VIP 与先前所创建服务之间的一种虚拟连接。

1. 单击**流量管理 > 负载均衡 > 虚拟服务器**。
2. 单击**添加**。
3. 对虚拟服务器命名。
4. 指定将进行均衡的协议 (HTTP)。
5. 使“IP 地址类型”保留为缺省值（IP 地址）。“IP 地址”字段用于输入作为所有用户入口点的 VIP。
6. 指定端口。缺省值为端口 80。
7. 单击**确定**。

现在，将创建的服务绑定到虚拟服务器。

1. 在“虚拟服务器”屏幕上，单击**无负载均衡虚拟服务器服务绑定**链接。
2. 将先前创建的每个服务绑定到虚拟服务器。
3. 单击**完成**。
4. 单击**刷新**按钮。“状态”和“有效状态”将显示为绿色。

您已为 Web 站点创建负载均衡池和策略。

**注：**要了解有关 Citrix NetScaler VPX 设备配置的更多信息，请访问 [Citrix 文档页面 ![外部链接图标](../../icons/launch-glyph.svg "外部链接图标")](https://docs.citrix.com/en-us/netscaler.html)。要获取进一步的帮助，请联系 {{site.data.keyword.BluSoftlayer_notm}} 支持和销售。
