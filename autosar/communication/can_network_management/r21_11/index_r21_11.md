<section id="title">AUTOSAR CAN NM</section>

# 1. 简介和功能概述

本文档描述了AUTOSAR CAN网络管理 (**CanNm**) 的概念、核心功能、可配置特性、接口和配置问题。

AUTOSAR CAN网络管理是一个独立于硬件的协议，只能在**CAN**上使用（有关限制，请参阅[章节4.1](#41-限制)）。其主要目的是协调网络正常运行和总线睡眠模式之间的转换。

除了核心功能外，还提供了可配置的功能，例如：实现一个服务来检测所有当前节点或检测所有其他节点是否准备好休眠。

CAN网络管理 (**CanNm**) 功能提供网络管理接口 (**NmIf**) 和 CAN接口 (**CanIf**) 模块之间的适配。有关AUTOSAR网络管理功能的一般理解，请参阅[7]。

# 2. 缩略语

## 2.1. 首字母缩写

下面的词汇表包括与**CanNm**模块相关的首字母缩略词和缩写词，它们未包含在**AUTOSAR**词汇表中。

**CanIf**
> CAN接口（**CAN Interface**）的缩写

**CanNm**
> CAN网络管理（**CAN Network Management**）的缩写

**CBV**
> 控制位向量（Control Bit Vector）

**CWU**
> 整车唤醒（**Car Wakeup**）

**ERA**
> 外部请求数组（**External Request Array**）

**EIRA**
> 外部和内部请求数组（**External and Internal Request Array**）

**NM**
> 网络管理（**Network Management**）

**PNC**
> 部分网络集群（**Partial Network Cluster**）

**PNI**
> 部分网络信息（**Partial Network Information**）

**PNL**
> 部分网络学习（**Partial Network Learning**）

**SNI**
> 源节点标识符（**Source Node Identifier**）


## 2.2. 术语 

**NM-Timeout Timer**
> NM超时计时器

**PDU传输能力被禁用**（**PDU transmission ability is disabled**）
> 这意味着网络管理**PDU**传输已被服务**CanNm_DisableCommunication**禁用。

**重复报文请求位指示**（**Repeat Message Request Bit Indication**）
> **CanNm_RxIndication**在收到的网络管理**PDU**的控制位向量中找到**RptMsgRequest**集。

**PN过滤遮罩**（**PN filter mask**）
> 由配置容器定义的过滤器掩码字节向量**CanNmPnFilterMaskByte**。

**顶级PNC协调员**（**Top-level PNC coordinator**）
> 顶级PNC协调器是一个**ECU**，它充当网络中的PNC网关，并在所有分配的通道上主动协调处理至少一个PNC。如果启用了同步PNC关闭，如果网络中没有其他ECU请求它们，顶级PNC协调器将为这些PNC触发关闭。

**中间PNC协调器**（**Intermediate PNC coordinator**）
> 中间PNC协调器是一个**ECU**，它充当网络中的PNC网关，并在至少一个分配的通道上将至少一个PNC处理为被动协调。如果启用了同步PNC关闭，它会将收到的这些PNC的关闭请求转发到相应的主动协调通道，并相应地启动它们的关闭。

**PNC叶节点**（**PNC leaf node**）
> PNC叶节点是一个**ECU**，在网络中根本不充当PNC协调器。它像通常的NM消息一样处理PN关闭消息。

**PN关闭消息**（**PN shutdown message**）
> 顶级PNC协调器传输PN关闭消息以指示跨PN拓扑的同步PNC关闭。PN关闭消息是作为**NM**消息，它在控制位向量中具有**PNSR**位，并且所有被指示用于同步关闭的PNC都设置为**1**

# 3. 相关文档

## 3.1. 输入文件

[1] General Requirements on Basic Software Modules
> AUTOSAR_SRS_BSWGeneral.pdf

[2] Specification of the AUTOSAR Network Management Protocol
> AUTOSAR_PRS_NetworkManagementProtocol.pdf

[3] Requirements on Network Management
> AUTOSAR_RS_NetworkManagement.pdf

[4] Specification of CAN Interface
> AUTOSAR_SWS_CANInterface.pdf

[5] Specification of Communication Stack Types
> AUTOSAR_SWS_CommunicationStackTypes.pdf

[6] Specification of ECU Configuration
> AUTOSAR_TPS_ECUConfiguration.pdf

[7] Specification of Generic Network Management Interface
> AUTOSAR_SWS_NetworkManagementInterface.pdf

[8] Specification of Communication Manager
> AUTOSAR_SWS_ComManager.pdf

[9] Specification of Standard Types
> AUTOSAR_SWS_StandardTypes.pdf

[10] General Specification of Basic Software Modules 
> AUTOSAR_SWS_BSWGeneral.pdf

[11] Specification of SystemTemplate 
> AUTOSAR_TPS_SystemTemplate

## 3.2. 相关规范

AUTOSAR 提供了基础软件模块的通用规范 [10] (**General Specification of Basic Software Modules**)，它也适用于**CAN**网络管理。

因此，规范基础软件模块的通用规范应被视为**CAN**网络管理的附加和必需规范。

# 4. 约束和假设

## 4.1. 限制

1. **CanNm**的一个通道只关联一个网络中的一个网络管理集群（**network management cluster**）。
2. 一个网络管理集群的一个节点只能有一个**CanNm**通道。
3. **CanNm**的一个通道只关联同一个ECU内的一个网络。
4. **CanNm**仅适用于**CAN**系统。

图4-1展示了一个示例**ECU**中的**AUTOSAR**网络管理堆栈，该示例**ECU**至少包含一个**CanNm**集群。

![Figure4_1.png](Figure4_1.png)

## 4.2. 适用于汽车领域

**CanNm**模块可以在上述限制下应用于任何汽车领域。

# 5. 对其他模块的依赖

CAN网络管理（**CanNm**）主要使用CAN接口（**CanIf**[4]）的服务，并为通用网络管理接口（**NmIf**[7]）提供服务。

# 6. 功能规格

## 6.1. 协调算法

**AUTOSAR CanNm**基于分散的直接网络管理策略，这意味着每个网络节点仅根据在通信系统内接收或传输的网络管理**PDU**执行自给自足的活动。

### 6.1.1. 概念

**AUTOSAR CanNm**算法基于周期性的网络管理**PDU**，集群中的所有节点通过广播传输接收这些**PDU**。网络管理**PDU**的接收表明传输节点希望保持网络管理集群处于唤醒状态。如果任何节点准备好进入总线睡眠（**Bus-Sleep**）模式，它就会停止传输网络管理**PDU**，但只要收到来自其他节点的网络管理**PDU**，节点就需推迟转换到总线睡眠模式。最后，如果由于不再接收到网络管理**PDU**而导致专用计时器超时，则每个节点都会启动到总线睡眠模式的转换。

如果网络管理集群中的任何节点需要总线通信，它可以通过传输网管**PDU**将网络管理集群从总线睡眠模式唤醒。有关唤醒过程本身的更多详细信息，请参阅通讯管理器规格[8]。

**AUTOSAR CanNm**算法的主要概念可以通过以下两个关键要求来定义：**CanNm**集群中的每个网络节点只要需要总线通信，就应该定期传输网络管理**PDU**；否则它将无需传输网络管理**PDU**。

如果**CanNmStayInPbsEnabled**被禁用，并且**CanNm**集群中的总线通信被释放，并且在可配置的（CanNmTimeoutTime + CanNmWaitBusSleepTime）时间里，总线上没有任何网络管理**PDU**，则需执行转换到总线睡眠模式。

### 6.1.2. 状态机

**AUTOSAR CanNm**算法的整体状态机可以定义如下：

**AUTOSAR CanNm**状态机应包含从网络管理集群中单个节点的角度来看的**AUTOSAR CanNm**算法所需的状态、转换和触发器。

**注意：**

状态转换必须在下一个主函数中最晚执行。

## 6.2. 操作模式

以下章节将详细介绍**AUTOSAR CanNm**算法的操作模式。

**AUTOSAR CanNm**应包含在模块接口上可见的三种操作模式：

* 网络模式（**Network Mode**）
* 准备总线休眠模式（**Prepare Bus-Sleep Mode**）
* 总线睡眠模式（**Bus-Sleep Mode**）

**AUTOSAR CanNm**操作模式的变化需通过回调函数通知上层。当**CanNm_GetState**被调用时，**CanNm**将返回当前的**NM**状态和模式。

### 6.2.1. 网络模式

网络模式应包括三个内部状态：

* 重复消息状态（**Repeat Message State**）
* 正常操作状态（**Normal Operation State**）
* 就绪睡眠状态（**Ready Sleep State**)

当从总线睡眠模式（**Bus-Sleep Mode**）进入网络模式（**Network Mode**）时，**CanNm**模块需默认进入重复消息状态（**Repeat Message State**）。

当从准备总线休眠模式（**Prepare Bus-Sleep Mode**）进入网络模式（**Network Mode**）时，**CanNm**模块需默认进入重复消息状态（**Repeat Message State**）。

当进入网络模式时，**CanNm**模块需启动NM超时计时器（**NM-Timeout Timer**），同时通过调用回调函数**Nm_NetworkMode**通知上层新的当前操作模式。

在网络模式下成功接收网络管理PDU（调用**CanNm_RxIndication**）后，如果启用了**PDU**传输能力，**CanNm**模块需重新启动NM超时计时器。

在网络模式下成功传输网络管理PDU（使用**E_OK**调用**CanNm_TxConfirmation**）时，**CanNm**模块需重新启动NM超时计时器。

**注意：**

如果启用**CanNmImmediateTxConfEnabled**，则假定每个网络管理**PDU**传输请求都会导致网络管理**PDU**传输成功。

**CanNm**模块需在每次启动或重启时，重置NM超时计时器。

如果在网络模式下调用**CanNm_PassiveStartUp**，**CanNm**模块将不执行该服务并返回**E_NOT_OK**。

如果在**CanNmDynamicPncToChannelMappingEnabled**设置为**TRUE**，且**CanNm**处于网络模式的通道上调用函数**CanNm_PnLearningRequest**，则 **CanNm**模块需在该通道上将**CBV**中的重复消息位和部分网络学习（**Partial Network Learning**）位设置为**1**，并更改为或重新启动重复消息状态（**Repeat Message State**）。

如果在**CanNmDynamicPncToChannelMappingEnabled**设置为**TRUE**，且**CanNm**处于网络模式的通道上接收到的部分网络学习（**Partial Network Learning**）位和 **Repeat Message Request**位的值为**1**，则**CanNm**应在该通道上将**CBV**中的**Partial Network Learning Bit**设置为**1**，并更改或重新启动重复消息状态。

#### 6.2.1.1. 重复消息状态（Repeat Message State）

对于不处于被动模式的节点（请参阅[第6.9.3章节](#693-被动模式)），重复消息状态确保从总线睡眠或准备总线睡眠到网络模式的任何转换，对网络上的其他节点都是可见的。 此外它还确保任何节点在最短的时间内保持活动状态。它可用于存在节点的检测。

当进入重复消息状态时，**CanNm**模块需（重新）开始传输网络管理PDU，除非被动模式被启用或者通信被禁止。

当NM超时计时器在重复消息状态中超时到期时，**CanNm**模块需（重新）启动NM超时计时器，同时需向**DET**报告**CANNM_E_NETWORK_TIMEOUT**。

网络管理状态机应在由配置参数**CanNmRepeatMessageTime**确定的可配置时间量内保持在重复消息状态；在此时间后，**CanNm**模块需离开重复消息状态。

当离开重复消息状态时，如果网络已被请求，**CanNm**模块需进入正常操作状态（**Normal Operation State**）；如果网络已经被释放，则**CanNm**模块需进入就绪睡眠状态（**Ready Sleep State**）。

如果**CanNmNodeDetectionEnabled**设置为**TRUE**，**CanNm**将在离开重复消息状态时，清除重复消息位。

如果在重复消息状态（**Repeat Message State**）、总线睡眠模式（**Prepare Bus-Sleep Mode**）或总线睡眠模式（**Bus-Sleep Mode**）中调用服务**CanNm_RepeatMessageRequest**，CanNm模块应不执行该服务并返回**E_NOT_OK**。

如果**CanNmDynamicPncToChannelMappingEnabled**设置为**TRUE**，**CanNm**将在离开重复消息状态时清除部分网络学习位。

#### 6.2.1.2. 正常操作状态

正常操作状态确保只要网络被请求，任何节点都可以保持网络管理集群处于唤醒状态。

当从就绪睡眠状态（**Ready Sleep State**)进入正常操作状态（**Normal Operation State**）时，**CanNm**模块将开始传输网络管理**PDU**。

**注意：**

如果被动模式被启用或者网络管理PDU传输被禁用，则**NM PDU**不会被传输，所以无任何操作需被执行。

当NM超时计时器在正常操作状态下超时到期，**CanNm**模块需（重新）启动 NM超时计时器，同时需向**DET**报告**CANNM_E_NETWORK_TIMEOUT**。

当前状态为正常操作状态（**Normal Operation State**），如果网络被释放，则**CanNm**模块需进入就绪睡眠状态（**Ready Sleep State**)。

如果**CanNmNodeDetectionEnabled**设置为**TRUE**，并且在正常操作状态（**Normal Operation State**）下接收到重复报文请求位，则**CanNm**模块需进入重复消息状态（**Repeat Message State**）。

如果**CanNmNodeDetectionEnabled**设置为**TRUE**，并且在正常操作状态（**Normal Operation State**）下调用函数**CanNm_RepeatMessageRequest**，则**CanNm**模块也需进入重复消息状态（**Repeat Message State**），并且CanNm模块需设置重复消息位（**Repeat Message Bit**）。

#### 6.2.1.3. 就绪睡眠状态

就绪睡眠状态（**Ready Sleep State**）确保只要有其他节点保持网络管理集群处于唤醒状态，网络管理集群中的任何节点就会等待进入准备总线休眠模式。

当从重复消息状态（**Repeat Message State**）或正常操作状态（**Normal Operation State**）进入就绪睡眠状态（**Ready Sleep State**）时，**CanNm**模块将停止网络管理**PDU**的传输。

**注意：**

如果被动模式被启用，则**NM PDU**不会被传输，所以无任何操作需被执行。

在某些情况下，如果被动模式被禁用，则需要在就绪睡眠状态（**Ready Sleep State**）下传输**NM PDU**，以保证网络同步关闭。例如：PN关闭消息（PN shutdown messages）的重新传输。

当NM超时计时器在就绪睡眠状态（**Ready Sleep State**）下超时到期，**CanNm**模块将进入准备总线休眠模式（**Prepare Bus-Sleep Mode**）。

当网络请求时，当前状态为就绪睡眠状态（**Ready Sleep State**）时，**CanNm**模块需进入正常操作状态（**Normal Operation State**）。

如果**CanNmNodeDetectionEnabled**设置为**TRUE**，并且在就绪睡眠状态（**Ready Sleep State**）下收到重复报文请求位，则**CanNm**模块进入重复消息状态（**Repeat Message State**）。

如果**CanNmNodeDetectionEnabled**设置为**TRUE**，并且在就绪睡眠状态（**Ready Sleep State**）下调用**CanNm_RepeatMessageRequest**函数，则**CanNm**模块需进入重复消息状态（**Repeat Message State**），并且CanNm模块需设置重复消息位（**Repeat Message Bit**）。

### 6.2.2. 准备总线休眠模式

准备总线休眠模式的目的是确保所有节点在进入总线休眠模式之前有时间停止它们的网络活动。在准备总线休眠模式下，总线活动平静下来(即，为了使所有的tx -buffer为空，传输排队的消息)，最后，在准备总线休眠模式下，总线上没有活动。

当进入准备总线休眠模式（**Prepare Bus-Sleep Mode**）时，**CanNm**模块将通过调用**Nm_PrepareBusSleepMode**通知上层。

如果**CanNmStayInPbsEnabled**被禁用，**CanNm**应在由配置参数**CanNmWaitBusSleepTime**确定的可配置时间内保持在准备总线休眠模式（**Prepare Bus-Sleep Mode**）。当时间超时后，需离开准备总线休眠模式（**Prepare Bus-Sleep Mode**）并进入总线睡眠模式（**Bus-Sleep Mode**）。

**注意：**

此需求隐含地包含，如果启用**CanNmStayInPbsEnabled**，**CanNm**将永远不会因为超时而离开，即**CanNm**将一直保持在准备总线休眠模式（**Prepare Bus-Sleep Mode**），直到**ECU**进入电源关闭或重启。

在准备总线休眠模式（**Prepare Bus-Sleep Mode**）下成功接收到网络管理**PDU**后，**CanNm**模块需进入网络模式（**Network Mode**）。在默认情况下，**CanNm**模块需进入重复消息状态（**Repeat Message State**）。

在准备总线休眠模式（**Prepare Bus-Sleep Mode**）下，当网络被请求后，**CanNm**模块需进入网络模式。并且默认情况下，**CanNm**模块需进入重复消息状态（**Repeat Message State**）。

在准备总线休眠模式（**Prepare Bus-Sleep Mode**）下，当网络被请求后，**CanNm**模块已进入网络模式，如果配置参数**CanNmImmediateRestartEnabled**设置为**TRUE**，则**CanNm**模块需传输网络管理**PDU**。

**理由：**

集群中的其他节点仍处于准备总线休眠模式（**Prepare Bus-Sleep Mode**）。在上述异常情况下，应避免过渡到总线睡眠模式（**Bus-Sleep Mode**），并且需尽快恢复总线通信。

由于**CanNm**中网络管理**PDU**的传输偏移，第一个处于重复消息状态的网络管理**PDU**的传输可能会显着延迟。为了避免网络延迟重新启动，可以立即请求网络管理**PDU**的传输。

**注意：**

如果**CanNmImmediateRestartEnabled**设置为**TRUE**，并且唤醒硬性（**wake-up line**）被使用，如果所有网络节点在准备总线休眠模式（**Prepare Bus-Sleep Mode**）下收到网络请求，则会发生网络管理**PDU**突发。

### 6.2.3. 总线睡眠模式

总线睡眠模式（**Bus-Sleep Mode**）的目的是在当没有消息交换时降低节点的功耗。通信控制器（**communication controller**）被切换到睡眠模式（**sleep mode**），相应的唤醒机制被激活，最终功耗降低到总线睡眠模式中的适当水平。

如果**CanNmStayInPbsEnabled**被禁用，可通过配置参数**CanNmTimeoutTime + CanNmWaitBusSleepTime**来确定的进入总线睡眠模式的时间。当整个网络管理集群中的所有节点都配置了相同的时间，则网络管理集群中使用**AUTOSAR**网络管理算法协调的所有节点，几乎会在非常接近的一个时间点执行到总线睡眠模式（**Bus-Sleep Mode**）的转换。

**注意：**

对于在这个网络管理集群的所有网络节点，参数**CanNmTimeoutTime**和**CanNmWaitBusSleepTime**需要具有相同的配置值。但是根据具体实现，总线睡眠模式（**Bus-Sleep Mode**）的转换会发生可能会在完全相同或者大致相同的一个时间点。

此转换的时间上的误差主要取决于以下因素：

* 内部时钟精度（振荡器漂移）。
* 网络任务的周期时间。（如：任务并未进行全局的时间同步）。
* Tx队列中的网络管理**PDU**等待时间。（如果在传输请求后立即进行传输确认）。

其实在最佳情况下，一般只需考虑振荡器的漂移即可，该时间可通过值配置参数**CanNmTimeoutTime + CanNmWaitBusSleepTime**来确定。

除了初始化时默认进入总线睡眠模式（**Bus-Sleep Mode**）的情况，其他进入总线睡眠模式（**Bus-Sleep Mode**）时，**CanNm**模块需通过调用回调函数**Nm_BusSleepMode**通知上层。

当**CanNm**模块在总线睡眠模式（**Bus-Sleep Mode**）下成功接收到网络管理**PDU**（**CanNm_RxIndication**调用）时，**CanNm**模块将通过调用回调函数**Nm_NetworkStartIndication**通知上层，同时应向**DET**报告错误**CANNM_E_NET_START_IND**。

**理由：**

为了避免网络和模式管理之间的竞争条件和状态不一致，**CanNm**不会自动执行从总线睡眠模式（**Bus-Sleep Mode**）到网络模式（**Network Mode**）的转换。**CanNm**只会通知必须做出唤醒决策的上层。总线睡眠模式下（**Bus-Sleep Mode**）的网络管理**PDU**接收必须根据**ECU**关闭/启动过程的当前状态进行处理。

如果在总线睡眠模式（**Bus-Sleep Mode**）或者准备总线休眠模式（**Prepare Bus-Sleep Mode**）下，调用**CanNm_PassiveStartUp**，**CanNm**模块应进入网络模式（**Network Mode**）。同时在默认情况下，**CanNm**模块需进入重复消息状态（**Repeat Message State**）。

**注意：**

在准备总线休眠模式（**Prepare Bus-Sleep Mode**）和总线睡眠模式（**Bus-Sleep Mode**）中，除非有明确的总线通信请求，一般都假定网络已经被释放。

当在总线睡眠模式（**Bus-Sleep Mode**）模式下，网络被再次请求，**CanNm**模块应进入网络模式（**Network Mode**）。同时在默认情况下，**CanNm**模块需进入重复消息状态（**Repeat Message State**）。

## 6.3. 网络状态

网络状态包括两类：被请求（**requested**）和被释放（**released**）。它是**AUTOSAR CanNm**状态机的两个附加的状态，并与状态机同时存在。

网络状态表示软件组件是否需要在总线上通信。当软件组件需要在总线上通信时，网络状态转换为被请求。当软件组件无需在总线上通信时，网络状态转换为被释放；但需注意的是即使网络被释放了，一个**ECU**仍然可以通信，因为其他一些ECU仍然在请求网络。

通过调用函数**CanNm_NetworkRequest**，可以用来请求网络。**CanNm**模块需将当前的网络状态更改为被请求。

通过调用函数**CanNm_NetworkRelease**，可以用来释放网络。**CanNm**模块需将当前的网络状态更改为被释放。

## 6.4. 初始化

如果**CanNm**模块初始化成功，即：调用函数**CanNm_Init**成功，则**CanNm**模块应将网络管理状态设置为总线睡眠模式（**Bus-Sleep Mode**）。

**注意：**

**CanNm**模块应该在**CanIf**初始化之后，并且在任何其他网络管理服务被调用之前，进行初始化。

初始化后，默认情况下，**CanNm**模块需将网络状态设置为已释放（**released**），**CanNm**模块需进入到总线睡眠模式（**Bus-Sleep Mode**）。

**CanNm_Init**函数应通过传递的配置指针参数选择活动配置集。

如果**CanNmGlobalPnSupport**设置为**TRUE**，并且**CanNm**已被初始化（调用**CanNm_Init**），则**CanNm**需停止NM消息传输超时计时器（**NM Message Tx Timeout Timer**）。

在初始化期间，**CanNm**模块应停用总线，减少总线负载。

初始化后，**CanNm**模块应通过停止消息周期定时器（**Message Cycle Timer**）来停止网络管理**PDU**的传输。

**注意：**

如果**CanNmPassiveModeEnabled**设置为**TRUE**，因为此类节点不传输网络管理 PDU，所有不需要**CanNm**消息周期（**Message Cycle**）。

在初始化期间，**CanNm**模块应将用户数据的每个字节设置为**0xFF**，并且控制位向量设置为**0x00**

在初始化期间，如果**CanNmPnEnabled**为**TRUE**，**CanNm**模块应将**PNC**位向量的每个字节设置为**0x00**。

## 6.5. 执行

### 6.5.1. 处理器架构

**AUTOSAR CanNm**算法需独立于处理器。这也就意味着，它不应依赖于任何处理器特定的硬件支持，所以它需要支持在**AUTOSAR**范围内的任何处理器架构上的实现。

### 6.5.2. 时间参数

配置参数**CanNmTimeoutTime**定义了**AUTOSAR CanNm**的NM超时的时间（**NM-Timeout Time**）。

配置参数**CanNmRepeatMessageTime**定义了**AUTOSAR CanNm**的重复消息发送的时间（**Repeat Message Time**）。

配置参数**CanNmWaitBusSleepTime**定义了**AUTOSAR CanNm** 的等待总线睡眠时间（**Wait Bus-Sleep Time**）。

配置参数**CanNmRemoteSleepIndTime**定义了**AUTOSAR CanNm**远程睡眠指示时间（**Remote Sleep Indication Time**）。

## 6.6. 网络管理PDU结构

下图显示了网络管理**PDU**的格式，以**8**字节为例，其中：

1. 源节点标识符 (**SNI**) 位于第一个字节。
2. 控制位向量 (**CBV**) 位于第二个字节。
3. 同时用户数据（**user data**）和部分网络（**partial network**）已被启用。同时用户数据的范围位于：系统字节（**System Bytes**）和**PNC**位向量之间。

![Figure7_1.png](Figure7_1.png)

源节点标识符（**Source Node Identifier**）的位置应可通过**CanNmPduNidPosition**配置为字节**0**，字节**1**或者关闭（**off**）。

**注意：**

将**CanNmPduNidPosition**设置为**off**，意味着在**NM PDU**中没有空间被源节点标识符占用，所以会多一个字节可用于用户数据或**PNC**位向量。

控制位向量（**Control Bit Vector**）的位置应可通过**CanNmPduCbvPosition**配置为字节**0**，字节**1**或者关闭（**off**）。

**注意：**

* 将**CanNmPduCbvPosition**设置为**Off**，意味着在**NM PDU**中没有空间被控制位向量（**CBV**）占用。所有会多一个字节可用于用户数据。
* **PNC**位向量的位置可通过相应**NM**通道的**NmPncBitVectorOffset**和**NmPncBitVectorLength**进行配置。**PNC**位向量的位置位于系统字节（**CBV**和**SNI**）之后，并且在**NM-PDU**的**PduLength**内。

未分配给**Nm**系统字节或**PNC**位向量的剩余字节，可以用于用户数据（**User Data**）。

**注意：**

用户数据的使用和位置是可配置的。如果使用了用户数据，则用户数据被放置在**NM-PDU**的**PduLength**内，并且不与系统字节或**PNC**位向量的范围重叠。

如果启用了部分网络功能（即：**CanNmPnEnabled**设置为**TRUE**），并且使用了用户数据，则用户数据范围位于系统字节和**PNC**位向量之间，或者**PNC**位向量和**NM-PDU**的末尾之间。

用户数据范围的长度可以根据以下限制计算：
* 如果用户数据范围位于系统字节和**PNC**位向量之间，则用户数据范围的长度等于**PNC**位向量偏移量和系统字节长度的差值。
* 如果用户数据范围位于**PNC**位向量和**NM-PDU**的末尾之间，则用户数据范围的长度等于**NM-PDU**长度与最后一个PNC位向量的字节的位置的差值。
  
如果禁用了部分网络功能（即：**CanNmPnEnabled**设置为**FALSE**，并且使用了用户数据，则用户数据范围等于**NM-PDU**长度和系统字节长度的差值。

网络管理**PDU**的长度由全局**ECUC**模块中的**PduLength**参数定义。

下图描述了控制位向量的格式：

![Figure7_2.png](Figure7_2.png)

注意：在**AUTOSAR**的**R3.2**版本中，位**1**和位**2**用作NM协调器ID（低位）。

控制位向量应包括：

1. **位 0**：重复消息请求位
    * 0：未请求重复消息状态
    * 1：请求重复消息状态
2. **位 1**：PN关闭请求位 (PNSR)
    * 0：NM 消息不包含同步的部分网络关闭请求
    * 1：NM 消息确实包含至少一个 PNC 的同步部分网络关闭请求
3. **位 3**：NM协调器休眠位
    * 0：主协调器未请求启动同步关机
    * 1：主协调器请求启动同步关机
4. **位 4**: 主动唤醒位
    * 0：节点没有唤醒网络（被动唤醒）
    * 1：节点已唤醒网络（主动唤醒）
5. **位 5**: 部分网络学习位 (PNL)
    * 0：不请求 PNC 学习
    * 1：请求 PNC 学习
6. **位 6**：部分网络信息位 (PNI)
    * 0：NM PDU 不包含部分网络请求信息
    * 1：NM PDU 包含部分网络请求信息
7. **位 2**和**位 7**：保留用于将来的扩展
    * 0：禁用/保留以供将来使用

**注意：**

**CBV**在初始化时用**0x00**进行初始化。

**CanNm**模块需使用配置参数**CanNmNodeId**设置源节点标识符，除非**CanNmPduNidPosition**设置为关闭。

如果**CanNm**由于调用**CanNm_NetworkRequest**（即：主动唤醒）而执行从总线睡眠模式（**Bus Sleep Mode**）或者准备总线休眠模式（**Prepare Bus Sleep Mode**）到网络模式（**Network Mode**）的状态更改，并且**CanNmActiveWakeupBitEnabled**为**TRUE**，则**CanNm**需在**CBV**中设置**ActiveWakeupBit**。

如果**CanNm**模块离开网络模式并且**CanNmActiveWakeupBitEnabled**为**TRUE**，则**CanNm**模块需清除**CBV**中的**ActiveWakeupBit**。

## 6.7. 通信调度

### 6.7.1. 传输（Transmission）

本节中所描述的传输机制仅适用在网络管理**PDU**传输能力被启用的情况。网络管理**PDU**传输能力可通过**CanNmPassiveModeEnabled**进行配置。

**注意：**

本节中描述的传输机制仅在**CanNmPassiveModeEnabled**为**FALSE**时被开启。

**CanNm**模块需提供周期性传输模式。在这种传输模式下，**CanNm**模块将定期发送网络管理**PDU**。

**CanNm**模块需可选地提供具有降低总线负载的周期性传输模式。在这种传输模式下，**CanNm**模块将根据特定算法传输网络管理**PDU**。

周期性传输模式可用于重复消息状态（**Repeat Message State**）和正常操作状态（**Normal Operation State**）。具有降低总线负载的周期性传输模式仅在正常操作状态（**Normal Operation State**）下可用。

**注意：**

如果禁用了降低总线负载机制，则在重复消息状态（**Repeat Message State**）和正常操作状态（**Normal Operation State**）中可使用周期性传输模式。

如果启用了降低总线负载机制，则仅在正常操作状态（**Normal Operation State**）中使用具有降低总线负载的周期性传输模式。

即时传输确认机制（**Immediate transmission confirmation**）应可通过**CanNmImmediateTxConfEnabled**进行配置。

**注意：**

即时传输确认机制用于不想使用**CanIf**的真实确认的系统。

**理由：**

如果总线访问完全通过离线系统设计工具进行调节，则通知网络管理传输成功的实际传输确认可被视为是多余的。由于最大仲裁时间是已知的，所以在传输请求时间立即提出确认是可以接受的。

此外，在这样的系统中仅针对一条**NM**消息，执行多余的实际传输确认，将意味着与整个Can接口（**CanIf**）和Can驱动程序层的执行时间有关的重大性能损失，从而使计算的时间的效率低下。

如果不是通过调用**CanNm_NetworkRequest**，进入重复消息状态（**Repeat Message State**），或者**CanNmImmediateNmTransmissions**为零，则在进入重复消息状态（**Repeat Message State**）后，**NM PDU**的传输应延迟**CanNmMsgCycleOffset**。

如果从就绪睡眠状态（**Ready Sleep State**）进入正常操作状态（**Normal Operation State**），则需立即开始**NM PDU**的传输。

如果**CanNmPnHandleMultipleNetworkRequests**设置为**TRUE**，**CanNm_NetworkRequest**将触发从网络模式（**Network Mode**）到重复消息状态（**Repeat Message State**）的状态转换。如果启用了**PDU**传输能力，则需使用**CanNmImmediateNmCycleTime**作为周期时间来传输**NM PDU**。第一帧**NM PDU**的传输需尽快地被触发。 在**PDU**被传输后，消息周期计时器需用**CanNmImmediateNmCycleTime**进行重新加载 。在这种情况下不应使用**CanNmMsgCycleOffset**。

**注意：**

在这种情况下，由于**ECUC_CanNm_00056**，所以**CanNmImmediateNmTransmissions**必须大于零。

如果**NM PDU**需使用**CanNmImmediateNmCycleTime**来进行传输，**CanNm**需确保具有此时间的**CanNmImmediateNmTransmission**（包括第一次立即传输）被成功请求。 如果对CanIf的传输请求失败（即：返回**E_NOT_OK**），**CanNm**将在下一个主函数中重试传输请求。

接着**CanNm**需使用**CanNmMsgCycleTime**来继续传输**NM PDU**。

**注意：**

当使用**CanNmImmediateNmCycleTime**传输**NM PDU**时，不得传输其他**Nm PDU**（即：**CanNmMsgCycleTime**传输周期已经被停止）。

如果**NM PDU**的传输已经开始，在下列的两种场景中，**CanNm**模块需通过调用**CanIf_Transmit**，发送**NM PDU**。

1. **CanNmSynchronizedPncShutdownEnabled**设置为**FALSE**，并且**CanNm**消息周期计时器（**Message Cycle Timer**）超时。
2. **CanNmSynchronizedPncShutdownEnabled**设置为**TRUE**，并且没有同步**PNC**关闭的请求在等待处理（**pending**），

**注意：**

如果**CanIf_Transmit**的函数调用失败，需按[章节6.12中所描述的传输错误处理](#612-传输错误处理)，通知**CanNm**模块。

如果**NM PDU**的传输已经开始，**CanNm**消息周期计时器到期，并且**CanNmSynchronizedPncShutdownEnabled**设置为**TRUE**，同步PNC关闭的请求正等待处理（**pending**），**NM PDU**的传输应推迟到下一个**CanNm_Main**函数调用。

**注意：**

* 同步的**PNC**关闭必须立即发送，所以使用**CanNmMsgCycleTime**传输的周期的**NM**消息的处理必须被延迟。在极少数情况下，这可能会导致一个以上主函数周期时间的延迟。
* 在网络管理时序上，因为必须考虑使用**CanNmMsgCycleTime**传输的**NM**消息可能会延迟一个以上的主函数周期时间，所以必须满足以下条件，以容忍这些**NM**消息的多次延迟：(CanNmPnResetTime – CanNmMsgCycleTime) > n * CanNmMainFunctionPeriod，其中**n**表示如果没有收到**NM**消息，则在**PnResetTime**超时到期之前允许的延迟数。

如果**CanNm**消息周期计时器到期，则**CanNm**模块必须使用**CanNmMsgCycleTime**来重新启动。

如果**NM PDU**的传输已停止，**CanNm**模块需取消消息周期计时器。

### 6.7.2. 接收（Reception）

如果成功接收到**NM PDU**，**CanIf**模块将调用回调函数**CanNm_RxIndication**。

在调用回调函数**CanNm_RxIndication**时，**CanNm**模块应将函数参数中引用的**NM PDU**的数据复制到内部缓冲区。

## 6.8. 降低总线负载机制

**NM PDU**的传输周期通常由时间参数**CanNmMsgCycleTime**决定。对于属于网络管理集群的所有**NM**节点，该参数必须相等。如果不采取任何行动，这将导致一个总线负载（**bus load**）的问题。同时问题是依据网络管理集群成员数量所决定的。即使可以通过节点特定时间参数**CanNmMsgCycleOffset**来防止突发，但也需要一种独立于网络管理集群的大小的机制来减少总线负载。

为了实现这哥1机制，以下两个方面必须予以考虑：

1. 当接收到**NM PDU**，**CanNm**消息周期计时器需使用此节点特定的时间参数**CanNmMsgReducedTime**进行重新加载。则此节点特定时间**CanNmMsgReducedTime**应大于 **½ CanNmMsgCycleTime**且小于**CanNmMsgCycleTime**。
2. 当**NM PDU**被传输后，**CanNm**消息周期定时器需使用此网络管理集群特定的时间参数**CanNmMsgCycleTime**进行重新加载。

这会导致以下行为：

只有两个具有最小**CanNmMsgReducedTime**时间的节点，在网络上交替传输**NM PDU**。如果其中一个节点停止传输，则下一个最小**CanNmMsgReducedTime**时间的节点将开始传输**NM PDU**。如果网络上只有一个节点需要总线通信，则每个**CanNmMsgCycleTime**传输一个**NM PDU**。

该算法确保总线负载限制为每个**CanNmMsgCycleTime**最多只有两个**NM PDU**。

降低总线负载机制需通过参数**CanNmBusLoadReductionEnabled**进行静态配置。

当从总线睡眠模式（**Bus-Sleep Mode**）、准备总线休眠模式（**Prepare Bus-Sleep Mode**）、正常操作状态（**Normal Operation State**）或就绪睡眠状态（**Ready Sleep State**）进入重复消息状态（**Repeat Message State**）时，**CanNm**模块应停用降低总线负载机制。

当从重复消息状态（**Repeat Message State**）或就绪睡眠状态（**Ready Sleep State**）进入正常操作状态（**Normal Operation State**）并且 **CanNmBusLoadReductionEnabled**为**TRUE**，**CanNm**模块需激活降低总线负载机制。

如果降低总线负载机制全局启用，即**CanNmBusLoadReductionEnabled**为**TRUE**。对于激活的特定网络，**PDU**传输能力被启用，并为此网络调用了函数**CanNm_RxIndication**，**CanNm**模块需使用节点特定的时间参数**CanNmMsgReducedTime**，重新启动**CanNm**消息周期计时器。

## 6.9. 附加的功能

### 6.9.1. 远程睡眠指示检测

远程睡眠指示（**Remote Sleep Indication**）表示为以下的一种特殊的场景。

1. 网络管理集群中某一个节点处于正常操作状态（**Normal Operation State**）。
2. 网络管理集群中的所有其他节点都准备好进入睡眠状态，即：都处于就绪睡眠状态（**Ready Sleep State**）。 
3. 处于正常操作状态（**Normal Operation State**）的节点仍将继续保持总线唤醒状态。

远程睡眠指示的检测应使用配置参数**CanNmRemoteSleepIndEnabled**进行静态配置。

如果**CanNm**模块在由配置参数**CanNmRemoteSleepIndTime**决定的可配置时间量内，没有接收到处于正常操作状态的**NM PDU**，则**CanNm**模块需调用回调函数 **Nm_RemoteSleepIndication**。

通过调用**Nm_RemoteSleepIndication**，**CanNm**通知模块**Nm**模块，集群中的所有节点都准备好​​进入睡眠状态，也就是所谓的远程睡眠指示（**Remote Sleep Indication**）。

先前已检测到远程睡眠指示，并且在正常操作状态（**Normal Operation State**）或就绪睡眠状态（**Ready Sleep State**）中接收到**NM PDU**，则模块**CanNm**需调用回调函数**Nm_RemoteSleepCancellation**。

先前已检测到远程睡眠指示，并且从正常操作状态（**Normal Operation State**）或就绪睡眠状态（**Ready Sleep State**）进入重复消息状态（**Repeat Message State**），则**CanNm**模块需调用回调函数**Nm_RemoteSleepCancellation**。

通过调用**Nm_RemoteSleepCancellation**，**CanNm**通知模块**Nm**模块，集群中的某些节点不再准备好进入睡眠状态，也就是所谓的远程睡眠取消（**Remote Sleep Cancellation**）。

当服务**CanNm_CheckRemoteSleepIndication**被调用，并且当前状态为总线睡眠模式（**Bus-Sleep Mode**）、准备总线休眠模式（**Prepare Bus-Sleep Mode**）或者重复消息状态（**Repeat Message State**）时，**CanNm**模块不应执行该服务并应返回**E_NOT_OK**。

### 6.9.2. 用户数据

为了支持网络管理用户数据，可以使用配置参数**CanNmUserDataEnabled**开关进行静态配置。

当**CanNm_SetUserData**被调用时，**CanNm**模块需为接下来在总线上传输的**NM PDU**设置网络管理用户数据。

当**CanNm_GetUserData**被调用时，**CanNm**模块需将返回最近收到的**NM PDU**的网络管理用户数据。

**注意：**

如果配置了用户数据，它肯定会在重复消息状态（**Repeat Message State**）下发送。在正常操作状态（**Normal Operation State**）下，是否发送用户数据取决于降低总线负载机制的配置。处于就绪睡眠状态（**Ready Sleep State**）的用户数据将不会被发送。

#### 6.9.2.1. COM 用户数据 

除了使用**CanNm API**来设置和获取用户数据之外，**CanNm**还可以使用**COM**来检索其用户数据。

如果**CanNmComUserDataSupport**被启用，则**API** **CanNm_SetUserData**将不再可用。

如果**CanNmComUserDataSupport**被启用，并且**NM-PDU** 未配置为**CanIf**中的触发传输，即：**CanIfTxPduTriggerTransmit**设置为**FALSE**，则**CanNm**需通过调用**PduR_CanNmTriggerTransmit**，从被引用的**NM I-PDU**收集网络管理用户数据。同时在它每次请求传输相应的**NM PDU**之前，将用户数据与**NM**字节组进行合并。

**注意：**

在触发式传输的情况下，传输请求不需要数据，只需要长度。数据将在**CanNm_TriggerTransmit**中收集。

如果**CanNmComUserDataSupport**被启用，并且**PduR_CanNmTriggerTransmit** 返回**E_NOT_OK**，则**NM**将使用最后传输的**NmUserData**值。

**注意：**

避免过时的**NM**数据的传输，可以通过不停止**COM**中用于网络管理用户数据传输的**I-PDU**。

如果**CanNmComUserDataSupport**被启用，并调用**CanNm_TxConfirmation**，**CanNm**将通过调用**PduR_CanNmTxConfirmation**将传输确认结果转发给**PduR**。

如果**CanNmComUserDataSupport**被启用，并且可用的用户数据字节数与引用的**I-PDU**的长度不匹配，则应在生成配置时报告错误。

### 6.9.3. 被动模式

在被动模式下，节点仅接收**NM PDU**，但不发送任何**NM PDU**。

被动模式需使用配置参数**CanNmPassiveModeEnabled** 开关进行静态配置。

**注意：**

必须为一个**ECU**中的所有**NM**网络启用或禁用被动模式。

### 6.9.4. 网络管理PDU的Rx指示

在调用回调函数**CanNm_RxIndication**时，当且仅当配置参数**CanNmPduRxIndicationEnabled**设置为**TRUE**时，**CanNm**模块需调用**Nm**的回调函数**Nm_PduRxIndication**。

### 6.9.5. 状态变化通知

如果回调**Nm_StateChangeNotification**被启用，即：配置参数**CanNmStateChangeIndEnabled**设置为**TRUE**，**AUTOSAR CanNm**状态的所有变化都应通过调用**Nm_StateChangeNotification**通知上层。

### 6.9.6. 通讯控制

通信控制（**Communication Control**）需通过配置参数**CanNmComControlEnabled**进行静态配置。

如果服务**CanNm_DisableCommunication**被调用，则**CanNm**模块需禁用**NM PDU**的传输能力。

**注意：**

此行为也同样适用于重复消息状态（**Repeat Message State**）。通信控制功能不影响重复消息状态（**Repeat Message State**）的持续时间。

当**NM PDU**传输能力被禁用时，需执行以下操作：

1. **CanNm**模块需停止**CanNm**消息周期定时器（**CanNm Message Cycle Timer**），以停止网络管理**PDU**的传输。
2. **CanNm**模块需停止NM超时计时器（**NM-Timeout Timer**）。
3. **CanNm**模块需停止远程睡眠指示检测（**Remote Sleep Indication Detection**）。

当**NM PDU**传输能力被启用时，需执行以下操作：

1. **NM PDU**的传输应最晚在下一个**NM**主函数中重新开始。
2. **CanNm**模块将重新启动NM超时计时器（**NM-Timeout Timer**）。
3. 如果**CanNmRemoteSleepIndEnabled**设置为**TRUE**，则**CanNm**模块应重新启动远程睡眠指示检测。

如果**NM PDU**传输能力被禁用，服务**CanNm_RequestBusSynchronization**需返回**E_NOT_OK**。

### 6.9.7. 协调器同步支持

当有多个协调器连接到同一总线时，**CBV**中有一个特殊位**NmCoordinatorSleepReady**，用于指示主协调器请求启动关闭序列（**shutdown sequence**）。该算法的主要功能已在**Nm**模块中进行了描述。

如果**CanNmCoordinatorSyncSupport**设置为**TRUE**，**CanNm**已进入网络模式（**Network Mode**）或已调用**Nm_CoordReadyToSleepCancellation**，则它应在第一次接收到**NmCoordinatorSleepReady**位设置为**1**的**NM PDU**时，通过调用**Nm_CoordReadyToSleepIndication**来通知**Nm**模块。

如果**CanNmCoordinatorSyncSupport**设置为**TRUE**，**CanNm**调用**Nm_CoordReadyToSleepIndication**并且仍处于网络模式（**Network Mode**），则它应在第一次接收到**NmCoordinatorSleepReady**位设置为**0**的**NM PDU**时，通过调用**Nm_CoordReadyToSleepCancellation**来通知**Nm**模块。

如果**CanNmCoordinatorSyncSupport**设置为**TRUE**，并且**CanNm_SetSleepReadyBit**函数被调用，则**CanNm**应将NM协调器睡眠就绪位（**NM Coordinator Sleep ready Bit**）设置为传递的值，并触发单个**NM PDU**。

## 6.10. 整车唤醒

整车唤醒位（**Car Wakeup bit**）在**NM-PDU**中的位置，由配置参数**CanNmCarWakeUpBytePosition**和**CanNmCarWakeUpBitPosition**定义。

### 6.10.1. 接收路径（Rx Path）

如果接收到的**NM PDU**中的整车唤醒位为**1**，并且**CanNmCarWakeUpRxEnabled**为**TRUE**，**CanNmCarWakeUpFilterEnabled**为**FALSE**, 则**CanNm**需调用**Nm_CarWakeUpIndication**，并执行标准**Rx**指示处理。

如果在**Nm_CarWakeUpIndication**上下文中，**CanNm_GetPduData**被调用，并且**CanNmNodeDetectionEnabled**或者**CanNmUserDataEnabled**或者**CanNmNodeIdEnabled**设置为**TRUE**，**CanNm**需返回导致**Nm_CarWakeUpIndication**被调用的**PDU**中所包含的**PDU**数据。

**注意：**

这是使**ECU**能够识别有关整车唤醒请求发送者的详细信息所必需的。

如果**CanNmCarWakeUpFilterEnabled**为**TRUE**，接收到的**NM PDU**中的整车唤醒位为**1**，同时**CanNmCarWakeUpRxEnabled**为**TRUE**，并且接收到的**NM PDU**中的节点**ID**等于**CanNmCarWakeUpFilterNodeId**，**CanNm**模块应调用**Nm_CarWakeUpIndication**，并执行标准的**Rx**指示处理。

**注意：**

整车唤醒过滤器对于只考虑实现中央网关（**Central Gateway**）的整车唤醒的子网关是必需的，可以避免错误唤醒。

### 6.10.2. 传输路径（Tx Path）

整车唤醒位的传输需由应用程序来处理，可以通过使用**CanNm**模块提供的网络管理用户数据（**NM user data**）机制实现。

## 6.11. 部分联网

### 6.11.1. NM PDU的Rx处理

如果**CanNmPnEnabled**为**FALSE**，**CanNm**不应从进一步的**Rx**指示处理中丢弃**NM PDU**，并且部分网络（**partial networking**）扩展需被禁用。

如果**CanNmPnEnabled**为**TRUE**，接收到的**NM-PDU**中的**PNI位**为**0**，并且**CanNmAllNmMessagesKeepAwake**为**TRUE**，则**CanNm**模块不应从进一步的 **Rx**指示处理中丢弃**NM PDU**，并需省略部分网络的扩展。

**注意：**

这是使网关在任何类型的**NM-PDU**上都能保持唤醒所必需的。

如果**CanNmPnEnabled**为**TRUE**，接收到的**NM-PDU**中的**PNI位**为**0**，并且**CanNmAllNmMessagesKeepAwake**为**FALSE**，则**CanNm**模块需忽略所接收到的**NM-PDU**。

#### 6.11.1.1. 基于NM-PDU的过滤算法（已过时）

如果**CanNmPnEnabled**为**TRUE**，接收到的**NM-PDU**中的**PNI位**为**1**，则**CanNm**模块将按照[第6.11.4章节NM PDU 过滤算法中的描述](#6114-网络管理pdu过滤算法已过时)处理NM-PDU的部分网络信息。

#### 6.11.1.2. 基于PNC位向量

如果**CanNmPnEnabled**为**TRUE**，接收到的**NM-PDU**中的**PNI位**为**1**，**PNSR位**为**0**，则**CanNm**模块将根据部分网络配置从接收到的**NM-PDU**中提取 PNC位向量（**NmPncBitVectorOffset** 和**NmPncBitVectorLength**对应的**NM**通道），并通过调用**Nm_PncBitVectorRxIndication**转发**PNC**位向量。

如果**CanNmPnEnabled**为**TRUE**, 并且**Nm_PncBitVectorRxIndication**已经被调用，则接收到的**NM PDU**仅应在以下某一个条件满足的情况下，考虑进行进一步处理：

* **CanNmAllNmMessagesKeepAwake**设置为**TRUE**
* **CanNm_ConfirmPnAvailability**尚未被调用。
* **RelevantPncRequestDetectedPtr**的输出值被设置为**TRUE**

**注意：** 

* 为了让网关在任何类型的**NM-PDU**上保持唤醒，需要设置参数**CanNmAllNmMessagesKeepAwake**。
* 如果**PN**可用性未被**CanSM**确认，则所有**PNC**请求都被认为是相关的，所以**Nm**在接收到**NM-PDU**时重新启动NM超时计时器。这是允许故障的部分网络依赖硬件（例如：支持**PN**的**CAN**收发器）与其余网络关闭同步所必需的。
* 如果不是所有消息都应使**ECU**保持唤醒，并且**PN**可用性已经被确认但未检测到相关**PN**信息，则不考虑对**NM PDU**进行进一步处理。

#### 6.11.1.3. 举例

- CanNmPduCbvPosition = 0
- CanNmPduNidPosition = 1
- NmPncBitVectorOffset = 4
- NmPncBitVectorLength = 4
- Calculated length of user data range = 2

**NM PDU**的字节**2**和字节**3**包含用户数据，**NM PDU**的字节**4**到字节**7**包含**PNC**位向量：

![Figure7-3](Figure7-3.png)

对于此示例，定义了四个**NmPnFilterMaskBytes**。**PN**滤波器掩码的值根据部分网络设计使用。

例如：

- NmPnFilterMaskByteIndex = 0，NmPnFilterMaskByteValue = 0x01
- NmPnFilterMaskByteIndex = 1，NmPnFilterMaskByteValue = 0x97
- NmPnFilterMaskByteIndex = 2，NmPnFilterMaskByteValue = 0x00
- NmPnFilterMaskByteIndex = 3，NmPnFilterMaskByteValue = 0x00

**注意：**

**PNC**位向量的偏移量来自**Nm**模块（**NmPncBitVectorOffset**）。 **PNC**位向量长度则是来自**Nm**模块的每个**NM**通道（**NmPncBitVectorLength**）。PN过滤器掩码（**NmPnFilterMaskByteIndex**和**NmPnFilterMaskByteValue**）位于**Nm**模块中并被使用。

#### 6.11.1.4. 同步PNC关闭

如果**CanNmSynchronizedPncShutdownEnabled**设置为**TRUE**，当接收到**PNI位**和**PNSR位**为**1**的**NM PDU**时，同时由**CanNmComMNetworkHandleRef**配置的**ComMChannel**被主动协调，即**ComMPncGatewayType**设置为**COMM_GATEWAY_TYPE_ACTIVE**，**CanNm**模块需忽略接收到的**NM-PDU**。

此外，**CanNm**模块还需：
* 向**DET**模块报告运行时错误**CANNM_E_INVALID_PN_SYNC_SHUTDOWN_REQUEST**。
* 如果**CanNmPnSyncShutdownErrorReactionEnabled**设置为**TRUE**，则最晚在下一个主函数（**Main Function**）调用中请求发送具有当前**PN**信息**NM PDU**。

如果**CanNmSynchronizedPncShutdownEnabled**设置为**TRUE**，则接收到的**NM-PDU**中的**PNI位**需设置为**1**，**PNSR位**也需设置为**1**，**CanNm**模块应根据部分网络配置（即：对应的NM通道的**NmPncBitVectorOffset**和**NmPncBitVectorLength**），从收到的**NM-PDU**中提取**PNC**位向量，并通过调用**Nm_ForwardSynchronizedPncShutdown**转发**PNC**位向量。

**注意：**

仅当请求同步**PNC**关闭时，**PNSR**位才可能设置为**1**。同步的**PNC**关闭需在**PN**拓扑中处理。所以假设所有协调器都启用了同步**PNC**关闭，或者所有协调器都禁用了同步**PNC**关闭。两者的混合会导致非同步的**PNC**关闭。这种情况是需要避免的。

### 6.11.2. NM PDU的Tx处理

如果**CanNmPnEnabled**为**TRUE**，**CanNm**模块应将发送的**PNI位**的值设置为**1**。

如果**CanNmPnEnabled**为**FALSE**，**CanNm**模块应将发送的**PNI位**的值始终设置为**0**。

**注意：**

如果使用部分网络，则必须使用**CBV**。

#### 6.11.2.1. 同步PNC关闭

如果**CanNmPnEnabled**为**TRUE**，则**NM-PDU**未配置为**CanIf**中的触发传输，即：**CanIfTxPduTriggerTransmit**设置为**FALSE**，没有同步**PNC**关闭的请求正待处理，并且NM-PDU必须被传输，**CanNm**模块需按照给定的顺序执行以下操作：

* 调用**Nm_PncBitVectorTxIndication**(\<NM-channel\>, \<buffer to store the unfiltered PNC bit vector of aggregated internal PNC requests\>) 来指示传输请求和取得内部**PNC**请求。
* 通过相应**NM**通道的**NmPncBitVectorOffset**和**NmPncBitVectorLength**，将接收到的用于内部**PNC**请求的**PNC**位向量复制到**NM-PDU**。
* 如果启用了用户数据，则获取可用数据（启用了**CanNmComUserDataSupport**，则从**Com**获取。否则从内部存储中获取）并复制**NM-PDU**的用户数据范围中的数据。
* 通过调用**CanIf_Transmit**触发**NM-PDU**的传输。

如果**CanNmSynchronizedPncShutdownEnabled**为**TRUE**，并且服务**CanNm_RequestSynchronizedPncShutdown**被调用，**CanNm**模块应将每个给定**NM**通道 (**nmChannelHandle**) 的给定**PNC**（**pncId**）存储为一个待处理的同步PNC关闭的请求。

**注意：**
所有同步**PNC**关闭请求的**PNC**的聚集，作为**PN**关闭消息进行传输（将**CBV**中的**PNSR**位设置为**1**），并在**CanNm_MainFunction**的上下文中异步完成。

### 6.11.3. 内部请求的部分网络集群的处理

所有内部**PNC**请求均由**ComM**维护。**ComM**将每个通道的聚集内部**PNC**请求作为**PNC**位向量转发到**NmIf**。这个**PNC**位向量携带了所谓的内部请求数组IRA（**Internal Request Array**）。每次发送**NM PDU**时，**CanNm**都必须从**NmIf**获取最新的**IRA**。**NmIf**将**IRA**信息提供给**CanNm**，并更新**PNC**重置计时器。即每次发送相关**PNC**时，重新启动**PNC**重置计时器。

**注意：**

对于**CanNmPnEnabled**为**TRUE**的所有已配置**NM**通道，**CanNm**将调用**Nm_PncBitVectorTxIndication**(\<NM-channel\>, \< buffer to store the unfiltered PNC bit vector of the aggregated internal PNC requests\>)，来指示传输并获取当前内部**PNC**请求作为相应配置**NmPncBitVectorLength**的**PNC** 位向量。**CanNm**将接收到的内部**PNC**请求复制到**NM-PDU**的**PNC**位向量字节。

### 6.11.4. 网络管理PDU过滤算法（已过时）

### 6.11.5. 内部和外部请求的部分网络的聚合（已过时）

### 6.11.6. 外部请求的部分网络的聚合（已过时）

### 6.11.7. 通过CanNm_NetworkRequest自发传输NM PDU

## 6.12. 传输错误处理

## 6.13. CanNm API 的功能要求

## 6.14. 主函数调度







