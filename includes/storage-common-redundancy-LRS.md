本地冗余存储 (LRS) 在存储缩放单位中复制数据三次，该缩放单位托管在创建存储帐户的区域中的数据中心内。 仅在写入所有三个副本后，才成功返回写入请求。 这三个副本各驻留在同一存储缩放单位中的不同容错域和升级域中。

存储缩放单位是存储节点的机架的集合。 容错域 (FD) 是一组代表出错的物理单元的节点，可将其视为属于同一物理机架的节点。 升级域 (UD) 是一组在服务升级（推出）过程中一起升级的节点。 三个副本将分布在同一存储缩放单位中的 UD 和 FD 上，以确保即使在硬件故障影响单个机架时，或在推出期间升级节点时，数据也可用。

LRS 是成本最低的选项，与其他选项相比，提供最小的持久性。 如果发生数据中心级灾难（火灾、洪灾等），所有三个副本可能丢失或无法恢复。 为了减小此风险，建议大多数应用程序使用异地冗余存储 (GRS)。

在某些情况下，可能仍需要使用本地冗余存储：

* 提供 Azure 存储复制选项最高的最大带宽。
* 如果应用程序存储可轻松重构的数据，则可以选择 LRS。
* 由于数据治理需要，某些应用程序被限制为只能在一个国家/地区内复制数据。 配对区域可以在另一个国家/地区中。 有关区域对的详细信息，请参阅 [Azure 区域](https://azure.microsoft.com/regions/)。