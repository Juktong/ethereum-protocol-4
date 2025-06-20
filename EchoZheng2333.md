---
timezone: UTC+8
---

> 请在上边的 timezone 添加你的当地时区（UTC），这会有助于你的打卡状态的自动化更新，如果没有添加，默认为北京时间 UTC+8 时区


# Echo

1. 自我介绍
    菜鸟级 dapp 开发选手，希望通过这次 EPF 的学习能够加深对以太坊的了解
2. 你认为你会完成本次残酷学习吗？
    会
3. 你的联系方式（推荐 Telegram）
    @Echo_2333


<!-- Content_START -->

## 2025.06.16

### 一、前置信息介绍
1. 以太坊 The Merge（合并）是以太坊历史上最重要的升级之一，于 2022 年 9 月 15 日成功完成。它的核心内容是将以太坊主网（执行层，原本基于工作量证明 PoW）与信标链（共识层，基于权益证明 PoS）合并，正式将以太坊的共识机制从 PoW 切换为 PoS。
    - **简单流程：**
        - 信标链（Beacon Chain）于 2020 年 12 月上线，作为 PoS 共识层并行运行。
        - 2022 年 9 月 15 日，主网与信标链合并，PoW 被永久关闭，PoS 成为唯一共识机制。
2. Geth 属于执行层客户端，负责交易的执行、状态和数据的维护，使用 Go 语言开发，是公认的最稳定、久经考验的客户端

### 二、Geth 框架的整理

Geth 从逻辑上可以分为 6 个部分：

- EVM：负责执行交易，交易执行也是修改状态数的唯一方式
- 存储：负责 state 以及区块等数据的存储
- 交易池：用于用户提交的交易，暂时存储，并且会通过 p2p 网络在不同节点之间传播
- p2p 网络：用于发现节点、同步交易、下载区块等等功能
- RPC 服务：提供访问节点的能力，比如用户向节点发送交易，共识层和执行层之间的交互
- BlockChain：负责管理以太坊的区块链数据

[![2025-06-16-18-31-01.jpg](https://i.postimg.cc/hvWk5md7/2025-06-16-18-31-01.jpg)](https://postimg.cc/G8KgBHzc)

- 如果以一个抽象的维度来看以太坊的执行层，以太坊作为一台世界计算机，需要包括三个部分，网络、计算和存储，那么以太坊执行层中与这三个部分相对应的组件是：
    - 网络：devp2p
        - 以太坊本质还是一个分布式系统，每个节点通过 p2p 网络与其他节点相连。以太坊中的 p2p 网络协议的实现就是 devp2p。
        - devp2p 有两个核心功能，一个是节点发现，让节点在接入网络时能够与其他节点建立联系；另一个是数据传输服务，在与其他节点建立联系之后，就可以想换交换数据。
    - 计算：EVM
    - 存储：ethdb

#### Geth 完整模块：
1. **账户与加密相关**
    - **accounts**：管理以太坊账户，包括公私钥对的生成、签名验证、地址派生等
    - **crypto**：加密算法（椭圆曲线、哈希、签名验证）
    - **signer**：交易签名管理（含硬件钱包）

2. **共识与信标链**
    - **beacon**：处理与以太坊信标链（Beacon Chain）的交互逻辑，PoS共识相关
    - **consensus**：共识引擎（PoW、PoS、Clique 等）
    - **miner**：挖矿逻辑（PoW 场景）

3. **区块链核心与数据结构**
    - **core**：区块链核心逻辑，处理区块/交易生命周期、状态机、Gas 计算
    - **trie & triedb**：默克尔帕特里夏树，实现账户/合约状态存储
    - **ethdb**：数据库抽象层（LevelDB、内存等）
    - **rlp**：RLP 序列化协议实现

4. **网络与节点服务**
    - **eth**：以太坊协议实现，节点服务、区块同步、交易广播
    - **p2p**：点对点网络协议，节点发现、数据传输
    - **node**：节点服务管理，整合各模块
    - **ethstats**：节点状态上报与监控
    - **event**：事件订阅与发布机制

5. **接口与交互**
    - **cmd**：命令行工具入口
    - **console**：交互式 JavaScript 控制台
    - **rpc**：JSON-RPC 和 IPC 接口
    - **graphql**：GraphQL 接口
    - **ethclient**：Go 客户端库，封装 RPC 接口
    - **RPC服务**（见 rpc/ethclient/console 等）

6. **工具与通用组件**
    - **common**：通用工具类（字节处理、地址转换、数学函数等）
    - **params**：网络参数（主网、测试网、创世块等）
    - **log**：日志系统
    - **metrics**：性能指标收集（Prometheus 支持）
    - **internal**：内部工具/限制外部访问代码

7. **构建、文档与测试**
    - **build**：构建脚本、编译配置
    - **docs**：文档
    - **tests**：集成测试、状态测试

所以本次共学主要围绕 `core`、`eth`、`ethdb`、`node`、`p2p`、`rlp`、`trie` & `triedb` 部分进行源码学习


### 三、Geth 启动

Geth 教程：[https://geth.ethereum.org/docs/getting-started](https://geth.ethereum.org/docs/getting-started) （未维护？很多页面都无法访问）

- 在 Geth 中创建账户的方法有很多种。在本次学习中尝试使用了 Clef 来创建账户，它将用户的密钥管理与 Geth 解耦，使其更加模块化和灵活。
    ``` bash
    # 老版本：
    geth --datadir . account new

    # 新版本：
    clef newaccount --keystore geth-tutorial/keystore
    ```
- 在 geth 启动的过程中 Clef 终端应保持运行。
    ``` bash
    clef --keystore geth-tutorial/keystore --configdir geth-tutorial/clef --chainid 11155111

    # --keystore 私钥存储的位置： geth-tutorial/keystore 
    # --chainid 链 ID： 11155111 为 Sepolia 测试网的链 ID
    ```
- 在 geth 终端做账户交互时需要 Clef 终端进行操作。
    ``` bash
    # geth 终端
    > eth.accounts;

    # clef 终端
    -------- List Account request--------------
    A request has been made to list all accounts. 
    You can select which accounts the caller can see
        [x] 0x12B275ea188d8B2E368880c15A3d7d60563aF811
            URL: keystore:///Users/apple/Desktop/ethereum-protocol/geth/geth-tutorial/keystore/UTC--2025-06-16T07-42-55.120127000Z--12b275ea188d8b2e368880c15a3d7d60563af811
    -------------------------------------------
    Request context:
            NA -> ipc -> NA

    Additional HTTP header data, provided by the external caller:
            User-Agent: ""
            Origin: ""
    Approve? [y/N]:
    > y

    # geth 终端
    > ["0x12b275ea188d8b2e368880c15a3d7d60563af811"]
    ```
- 如果 Clef 审批时间过长，此请求也可能会超时

## 2025.06.17

### 以太坊 The Merge 升级笔记

#### 一、核心概念与背景

1. The Merge 的定义与意义
    - **定义**：The Merge 是以太坊从工作量证明（PoW，需要大量电力的「挖矿」系统）向权益证明（PoS，「存钱生利息」的系统）过渡的关键升级
    - **时间节点**：2022 年完成，至今已稳定运行近两年
    - **当前状态**：在稳定性、性能和避免中心化风险方面表现优异

2. 升级后的核心挑战
    - **技术特性改进**：稳定性、性能和小型验证者可访问性
    - **经济变革**：应对中心化风险的经济机制调整

#### 二、关键技术改进方向

1. **单时隙最终确定性（SSF）**

    **当前问题**
    - 需要 2-3 个 epoch （约 15 分钟） 完成区块最终确定（等太久）
    - 32 ETH 的质押门槛限制了验证者参与（太贵了）

    **解决方案对比**
    - 方案 A：超级计算机快速签名（暴力破解 SSF）
    - 方案 B：抽签选代表投票（Orbit SSF）
    - 方案 C：分 VIP 和普通用户（有双层质押机制的 SSF）

        | 方案 | 验证者数量 | 最终性时间 | 实现难度 | 优势 | 劣势 |
        |------|------------|------------|----------|------|------|
        | 现状 | ~30 万 | 15 分钟 | 低 | 稳定 | 效率低 |
        | 蛮力破解 | 100 万+ | 单时隙 | 极高 | 安全性最高 | 技术复杂 |
        | Orbit 委员会 | 8k-32k | 单时隙 | 中 | 平衡性好 | 安全性略降 |
        | 双层质押 | 分层 | 单时隙 | 中 | 门槛低 | 中心化风险 |

    **技术细节**
    - **Orbit 机制**：利用验证者存款规模异质性，通过中型随机委员会实现高效最终性
    - **签名聚合**：使用 ZK-SNARKs 等技术处理大规模验证者签名

2. **单一秘密领袖选举（SSLE）**

    **当前问题**
    - 现在大家都知道下一个出块的是谁 → 容易被黑客攻击

    **解决方案**
    - 像抽盲盒，只有中奖的人自己知道身份
        - **盲验证者 ID**：通过加密技术隐藏实际验证者身份
        - **混合网络机制**：对盲 ID 池进行改组和重新盲化


3. **更快的交易确认**

    **优化方向**
    1. **缩短时隙时间**：从 12 秒降至 4-8 秒
    2. **提议者预确认机制**：
        - 实时交易纳入与确认
        - 冲突处理方案：
            - 罚没提议者
            - 证明人投票选择

    **技术权衡**
    | 方案 | 平均确认时间 | 最坏情况 | 实现难度 |
    |------|--------------|----------|----------|
    | 缩短时隙 | 显著改善 | 仍受限于网络延迟 | 高 |
    | 预确认 | 0.5 秒（理想） | 12 秒（提议者离线） | 中 |

### 三、其他重要研究领域

1. **51% 攻击恢复机制**
    - **现状**：依赖社会共识（少数派软分叉）
    - **改进方向**：自动化恢复流程
        - 客户端自动检测长期被审查交易
        - 拒绝接受恶意链为有效链

2. **投票阈值提升**
    - **当前**：67% 质押者支持即最终确定
    - **提案**：提升至 80% 阈值
        - 减少「错误方获胜」情况
        - 增强独立质押者话语权

3. **抗量子攻击**
    - **威胁时间表**：预计 21 世纪 30 年代量子计算机可能破解现有密码学
    - **应对策略**：
        - 开发基于哈希的抗量子替代方案
        - 减少对 BLS 签名聚合的依赖

### 四、技术实现路径分析

1. **单时隙最终性实现路线**
    [![1.png](https://i.postimg.cc/Y96v2wzK/1.png)](https://postimg.cc/gX0z4Q7N)
2. **关键研究链接**
    1. [单时隙最终确定性路径](https://notes.ethereum.org/@vbuterin/single_slot_finality)
    2. [Orbit SSF 提案](https://ethresear.ch/t/orbit-ssf-solo-staking-friendly-validator-set-management-for-ssf/19928)
    3. [Whisk SSLE 协议](https://ethresear.ch/t/whisk-a-practical-shuffle-based-ssle-protocol-for-ethereum/11763)

### 五、为什么这些很重要？

| 好处 | 对你有什么影响 |
|------|----------------|
| 更快确认 | 转账不用等半天 |
| 降低门槛 | 存1个ETH也能当验证人 |
| 更安全 | 不怕大户操纵价格 |
| 更环保 | 减少 99% 能源消耗 🌱 |

### 六、举个生活例子 🌈

想象以太坊是个小区：
- **以前**：物业费交给挖矿最猛的人（电费超贵）
- **现在**：业主投票选物业（存ETH多的票多）
- **问题**：富人话语权太大，穷人参与难
- **改进**：
  - 让投票更快（不用开几天业主大会）
  - 隐藏投票顺序（防打击报复）
  - 降低参与门槛（小户型也能投票）

### 七、未来会怎样？ 🚀
- 2024 年：测试快速确认功能
- 2025 年：可能实现 1 ETH 就能质押
- 2030 年：防御量子计算机攻击


## 2025.06.18

### 以太坊协议 The Surge 学习笔记

### 一、基础概念理解

#### 1. 什么是以太坊的「The Surge」？
「The Surge」是以太坊 2023 年路线图中的重要部分，主要关注 **可扩展性（scalability）** 的提升。简单来说，就是让以太坊网络能够处理更多的交易，同时保持去中心化和安全性。



#### 2. 为什么需要扩展？
目前以太坊主网（L1）每秒只能处理约 15-30 笔交易，当网络拥堵时，交易费用（Gas费）会变得很高。The Surge 的目标是实现 **L1 + L2 上 100,000+ 的 TPS**（每秒交易数）。

### 二、 核心扩展策略

#### 1. 以 Rollup 为中心的路线图
以太坊采用「分层」架构：
- **L1（主网）**：保持去中心化和安全性
- **L2（二层网络）**：负责扩展，处理大量交易

这种分工类似于：
- 法院系统（L1）保护契约和产权
- 企业家（L2）在坚固基础上构建创新应用

#### 2. 可扩展性三难困境
这是一个重要概念，指区块链难以同时实现三个理想属性：
- **去中心化**：运行节点的成本低
- **可扩展性**：处理高交易量的能力
- **安全性**：攻击成本高

### 三、关键技术方案

#### 1. 数据可用性采样（DAS）
**解决的问题**：验证大量数据而不需要下载全部数据

**当前状态**：
- 每个时隙（12秒）有 3 个约 125kB 的「blob」
- 最大 TPS 约 173.6（仅 blob）或 607（加上 calldata）

**未来目标**：
- 通过 PeerDAS 将 blob 数量增加到 8-16 个
- 中期目标：每时隙 16 MB，约 58,000 TPS

#### 2. 数据压缩
**为什么需要**：减少每笔交易在链上占用的空间

**优化方法**：
1. 零字节压缩
2. 签名聚合（从 ECDSA 迁移到 BLS）
3. 地址指针替换（用 4 字节代替 20 字节地址）
4. 交易金额自定义序列化

#### 3. Plasma 架构
**与 Rollup 的区别**：
- Rollup：将完整区块数据上链
- Plasma：只在链上记录区块 Merkle 根

**优势**：
- 更高的扩展性
- 用户即使数据不可用也能提取资产

#### 4. L2 证明系统成熟化
**当前问题**：大多数 Rollup 尚未完全无信任

**发展阶段**：
- 阶段 0：中心化或基于信任的验证
- 阶段 1：无信任证明机制 + 安全委员会
- 阶段 2：完全无信任证明机制（目标）



### 四、跨 L2 互操作性改进

#### 1. 当前问题
不同 L2 间切换困难，用户体验差

#### 2. 解决方案方向
1. **链特定地址**：地址包含链标识
2. **链特定支付请求**：标准化跨链支付请求
3. **跨链兑换与 gas 支付**：标准化协议
4. **轻客户端**：直接验证交互的链
5. **密钥库钱包**：密钥集中管理

### 五、L1 上的扩展执行

#### 1。 为什么需要扩展 L1？
即使 L2 很成功，L1 仍需保持足够能力：
- 维护 ETH 资产价值
- 支持 L2 故障恢复
- 保持与 L2 生态的紧密联系

#### 2. 扩展策略
1. **直接增加 gas 上限**：需配合验证技术改进
2. **优化特定功能**：
   - EOF 新字节码格式
   - 多维 gas 定价
   - 降低特定操作码成本
3. **原生 Rollup**：协议内建并行 EVM

### 六、关键学习要点

1. 以太坊采用分层扩展策略，L1 保持安全，L2 负责扩展
2. 数据可用性采样是扩展的基础技术
3. Plasma 和 Rollup 是两种主要扩展方案，各有优劣
4. L2 需要逐步实现完全无信任的证明机制
5. 跨链互操作性是提升用户体验的关键
6. L1 自身也需要持续扩展

### 七、未来展望

The Surge 路线图的实现将使以太坊能够支持大规模应用，如：
- 消费支付
- 去中心化社交网络
- 高吞吐量 DApps

同时保持以太坊核心的去中心化和安全特性。

## 2025.6.19
### **学习总结：以太坊核心开发进展（Checkpoint #4: Berlinterop）**  

#### **1. 背景**  
以太坊核心开发者每周会议（All Core Developer calls）内容繁多，本系列“Checkpoint”旨在提供高层级更新。本次是特别版，聚焦柏林区块链周期间的**Berlinterop**（开发者协作周），重点包括：  
- **短期目标**：Fusaka 升级、Gas Limit 提升  
- **长期研究方向**：Slot 结构调整、历史数据清理（History Expiry）等  
- **L2 与 ZK 团队反馈**：优化 L1 与 L2 的协作  

---  

#### **2. 短期进展**  
**（1）Fusaka 升级**  
- 开发者启动了测试网：`fusaka-devnet-1` 和 `berlinterop-devnet-2`。  
- 发现需改进点，但需通过 ACD（All Core Developers）治理流程确认（本周四讨论）。  
- 乐观估计：夏季末推进至**Sepolia 测试网**，可能无需`devnet-3`。  

**（2）Gas Limit 提升**  
- 目标：安全提高网络吞吐量（如 4500 万 Gas/区块）。  
- 开发者通过压力测试挑战赛（Leaderboard）验证方案，Kamil 和 pk910 表现突出。  
- 共识达成后，优化方案将通过 **EthPandaOps 推特**和 **Eth R&D Discord** 发布。  

---  

#### **3. 长期研究方向**  
**（1）Slot 结构调整**  
- 讨论缩短 Slot 时间或调整子 Slot 时序，涉及提案：  
  - **ePBS**（提议者-构建者分离）  
  - **Delayed Execution**（延迟执行）  
  - **FOCIL**（快速确认）  
- **优势**：减少数据延迟、降低大区块影响、增强抗审查性。  
- 下一步：合并相关 PR，规范客户端验证逻辑。  

**（2）历史数据清理（History Expiry）**  
- 进展顺利！主网将默认清理合并前的历史数据（🎉）。  
- 已就 **Era 文件标准**达成一致，未来两个月将公布：  
  - 滚动清理机制  
  - 数据分发方案  
- 本周五召开社区会议讨论 **Portal 网络**的未来。  

**（3）共识层（CL）加固**  
- 针对 Holešky 分叉等问题，提出 26 项改进（如非最终状态同步、资源优化等）。  
- 目标：提升网络在非最终性期间的稳定性。  

---  

#### **4. L2 与 ZK 团队反馈**  
**（1）L2 日**  
- **需求**：更多 Blob 空间、更快最终性、EVM 变更提前沟通（如 calldata 定价）。  
- **协作建议**：L2 可分享高吞吐量网络经验，助力 L1 扩展设计。  

**（2）ZK 日**  
- **共识**：暂不固化特定 ISA（指令集架构），优先支持 RISC-V 目标。  
- **标准化**：统一系统调用和 Rust 库，300KB 证明大小合理。  
- **目标**：年底前实现 ZK 无状态客户端（基于 Reth）。  

---  

#### **5. 总结**  
- **Berlinterop 成果显著**：打破异步沟通壁垒，加速了长期停滞的项目。  
- **Fusaka 升级有望 2025 年落地**，Gas Limit 提升和 L2/ZK 协作是关键推动力。  
- **下一步**：关注 ACD 会议决议、Portal 网络讨论和 Ethereum/pm 仓库的详细笔记。  

---  
**延伸阅读**：  
- [以太坊基金会](https://ethereum.org) | [R&D Discord](https://discord.gg/ethereum)  
- 相关会议记录将发布于 [ethereum/pm GitHub](https://github.com/ethereum/pm)。
<!-- Content_END -->
