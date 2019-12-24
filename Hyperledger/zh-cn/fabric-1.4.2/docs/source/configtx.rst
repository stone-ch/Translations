通道配置（configtx）
================================

<<<<<<< HEAD
Hyperledger Fabric 区块链网络的共享配置存储在一个集合配置交易中，每个通道一个。
配置交易通常简称为 *configtx*。

通道配置有以下重要特性：

1. **版本化**：配置中的所有元素都有一个关联的、每次修改都提高的版本。此外，每个
   提交的配置都有一个序列号。 
2. **许可的**：配置中的所有元素都有一个关联的、控制对此元素的修改是否被允许的策略。
   任何拥有之前 configtx 副本（没有附加信息）的人均可以这些策略为基础来验证新配置
   的有效性。
3. **分层的**：根配置组包含子组，而且层次结构中的每个组都有关联的值和策略。这些
   策略可以利用层次结构从较低层级的策略派生出一个层级的策略。

配置解析
=======
Shared configuration for a Hyperledger Fabric blockchain network is
stored in a collection configuration transactions, one per channel. Each
configuration transaction is usually referred to by the shorter name
*configtx*.

超级账本 Fabric 区块链网络的共享配置存储在集合配置交易中，每个通道一个。每个配置交易通常用更短的名称 *configtx* 引用。

Channel configuration has the following important properties:

通道配置具有以下重要属性：

1. **Versioned**: All elements of the configuration have an associated
   version which is advanced with every modification. Further, every
   committed configuration receives a sequence number.
2. **Permissioned**: Each element of the configuration has an associated
   policy which governs whether or not modification to that element is
   permitted. Anyone with a copy of the previous configtx (and no
   additional info) may verify the validity of a new config based on
   these policies.
3. **Hierarchical**: A root configuration group contains sub-groups, and
   each group of the hierarchy has associated values and policies. These
   policies can take advantage of the hierarchy to derive policies at
   one level from policies of lower levels.

1. **版本化** ：配置的所有元素都有一个关联的版本，该版本在每次修改时都是增加的。此外，每个提交的配置都接收一个序列号。

2. **许可的** "
"：配置的每个元素都有相关的策略，该策略控制是否允许修改该元素。任何拥有先前版本的configtx副本（没有其他信息）的人，都可以基于这些策略验证新配置的有效性。

3. ** 层次结构 ** ：根配置组包含子组，层次结构的每个组都有关联的值和策略。这些策略可以利用层次结构从较低级别的策略派生本级别的策略。

Anatomy of a configuration
配置的解剖
>>>>>>> 3915c06381b1ecf6f1e7420afdf0ec132c8e954e
--------------------------

配置，作为一个类型为 ``HeaderType_CONFIG`` 的交易，被存储在一个没有其他交易的
区块中。这些区块被称为*配置区块*，其中的第一个就是*创世区块*。

<<<<<<< HEAD
配置的原型结构在 ``fabric/protos/common/configtx.proto`` 中。类型为 
``HeaderType_CONFIG`` 的信封将 ``ConfigEnvelope`` 消息编码为
``Payload`` ``data`` 字段。``ConfigEnvelope`` 的原型定义如下：
=======
配置作为类型为 ``HeaderType_CONFIG`` 的交易存储在一个没有其他交易的块中。这种块被称为 *配置区块* ，其中第一个被称为 "
"*创世区块* 。

The proto structures for configuration are stored in
``fabric/protos/common/configtx.proto``. The Envelope of type
``HeaderType_CONFIG`` encodes a ``ConfigEnvelope`` message as the
``Payload`` ``data`` field. The proto for ``ConfigEnvelope`` is defined
as follows:
>>>>>>> 3915c06381b1ecf6f1e7420afdf0ec132c8e954e

配置的原型结构存储在 ``fabric/protos/common/configtx.proto`` 中。类型 "
"``HeaderType_CONFIG`` 的信封将 ``ConfigEnvelope`` 消息编码为 ``Payload`` 、 ``data`` "
"字段。 ``ConfigEnvelope`` 的原型定义如下：

::

    message ConfigEnvelope {
        Config config = 1;
        Envelope last_update = 2;
    }

``last_update`` 字段在下面的 **配置更新** 一节定义，但只有当验证配置时才是必需的，
读取时则不是。当前提交的配置存储在 ``config`` 字段，包含 ``Config`` 消息。

``last_update`` 字段在下面的更新配置 **Updates to configuration** "
"小节中定义，但是只有在验证配置时才需要，而不是读取配置时。相反，当前提交的配置存储在 ``config`` 字段中，其中包含一条 ``Config`` "
"消息。

::

    message Config {
        uint64 sequence = 1;
        ConfigGroup channel_group = 2;
    }

``sequence`` 数字每次提交配置时增加1。``channel_group`` 字段是包含配置的根组。
``ConfigGroup`` 结构是递归定义的，并构建了一个组的树，每个组都包含值和策略。其
定义如下：

对于每个提交的配置， ``sequence`` 号将增加1。 ``channel_group`` 字段是包含整个配置的根组。 "
"``ConfigGroup`` 结构是递归定义的，并构建一个组树，每个组包含值和策略。定义如下：

::

    message ConfigGroup {
        uint64 version = 1;
        map<string,ConfigGroup> groups = 2;
        map<string,ConfigValue> values = 3;
        map<string,ConfigPolicy> policies = 4;
        string mod_policy = 5;
    }

因为 ``ConfigGroup`` 是递归结构，所以它有层次结构。为清楚起见，下面的例子使用 
golang 展现。

因为 ``ConfigGroup`` 是递归结构，所以它具有层次结构。为了清楚起见，下面的示例用 golang 表示法表示。

::

    // Assume the following groups are defined
    var root, child1, child2, grandChild1, grandChild2, grandChild3 *ConfigGroup

    // Set the following values
    root.Groups["child1"] = child1
    root.Groups["child2"] = child2
    child1.Groups["grandChild1"] = grandChild1
    child2.Groups["grandChild2"] = grandChild2
    child2.Groups["grandChild3"] = grandChild3

    // The resulting config structure of groups looks like:
    // root:
    //     child1:
    //         grandChild1
    //     child2:
    //         grandChild2
    //         grandChild3

每个组定义了配置层次中的一个级别，每个组都有关联的一组值（按字符串键索引）和策略
（也按字符串键索引）。

<<<<<<< HEAD
值定义：
=======
每个组在配置层次结构中定义一个级别，每个组都有一组关联的值（按字符串键 string key 索引）和策略（也按字符串键 string key 索引）。

Values are defined by:
>>>>>>> 3915c06381b1ecf6f1e7420afdf0ec132c8e954e

值这样定义：

::

    message ConfigValue {
        uint64 version = 1;
        bytes value = 2;
        string mod_policy = 3;
    }

策略定义：

"策略这样定义：

::

    message ConfigPolicy {
        uint64 version = 1;
        Policy policy = 2;
        string mod_policy = 3;
    }

<<<<<<< HEAD
注意，值、策略和组都有 ``version`` 和 ``mod_policy``。一个元素每次修改时 
``version`` 都会增长。 ``mod_policy`` 被用来控制修改元素时所需要的签名。
对于组，修改就是增加或删除值、策略或组映射中的元素（或改变 ``mod_policy``）。
对于值和策略，修改就是分别改变值和策略字段（或改变 ``mod_policy``）。每个元素
的 ``mod_policy`` 都在当前层级配置的上下文中被评估。下面是一个定义在 
``Channel.Groups["Application"]`` 中的修改策略示例（这里，我们使用 golang map 
语法，所以，``Channel.Groups["Application"].Policies["policy1"]`` 表示根组 
``Channel`` 的子组 ``Application`` 的 ``Policies`` 中的 ``policy1`` 所对应的
策略。）

* ``policy1`` 对应 ``Channel.Groups["Application"].Policies["policy1"]``
* ``Org1/policy2`` 对应 ``Channel.Groups["Application"].Groups["Org1"].Policies["policy2"]``
* ``/Channel/policy3`` 对应 ``Channel.Policies["policy3"]``

注意，如果一个 ``mod_policy`` 引用了一个不存在的策略，那么该元素不可修改。

=======
Note that Values, Policies, and Groups all have a ``version`` and a
``mod_policy``. The ``version`` of an element is incremented each time
that element is modified. The ``mod_policy`` is used to govern the
required signatures to modify that element. For Groups, modification is
adding or removing elements to the Values, Policies, or Groups maps (or
changing the ``mod_policy``). For Values and Policies, modification is
changing the Value and Policy fields respectively (or changing the
``mod_policy``). Each element's ``mod_policy`` is evaluated in the
context of the current level of the config. Consider the following
example mod policies defined at ``Channel.Groups["Application"]`` (Here,
we use the golang map reference syntax, so
``Channel.Groups["Application"].Policies["policy1"]`` refers to the base
``Channel`` group's ``Application`` group's ``Policies`` map's
``policy1`` policy.)

注意，值、策略和组都有一个 ``version`` 和一个 ``mod_policy`` 。元素的 ``version`` 在每次修改该元素时递增。 "
"``mod_policy`` 用于管理修改该元素所需的签名。对于组，修改是指向值、策略或组的map添加或删除元素（或更改 ``mod_policy`` "
"）。对于值和策略，修改是指分别更改值和策略字段（或更改 ``mod_policy`` ）。每个元素的 ``mod_policy`` "
"都是在当前配置级别的上下文中计算的。考虑下面在通道中定义的修改策略示例 ``Channel.Groups[\"Application\"]``  "
"（这里，我们使用golang map引用语法，因此 "
"``Channel.Groups[\"Application\"].Policies[\"policy1\"]`` 指基本 ``Channel`` 组的"
" ``Application`` 组的 ``Policies`` map的 ``policy1`` 政策）。

* ``policy1`` maps to ``Channel.Groups["Application"].Policies["policy1"]``
* ``Org1/policy2`` maps to
  ``Channel.Groups["Application"].Groups["Org1"].Policies["policy2"]``
* ``/Channel/policy3`` maps to ``Channel.Policies["policy3"]``

* ``policy1`` 映射到 ``Channel.Groups[\"Application\"].Policies[\"policy1\"]`` 
* ``Org1/policy2`` 映射到 ``Channel.Groups[\"Application\"].Groups[\"Org1\"].Policies[\"policy2\"]``
* ``/Channel/policy3`` 映射到 ``Channel.Policies[\"policy3\"]``

Note that if a ``mod_policy`` references a policy which does not exist,
the item cannot be modified.

注意，如果 ``mod_policy`` 引用不存在的策略，则无法修改该项。

Configuration updates
>>>>>>> 3915c06381b1ecf6f1e7420afdf0ec132c8e954e
配置更新
---------------------

配置更新被作为一个类型为 ``HeaderType_CONFIG_UPDATE`` 的 ``Envelope`` 消息提交。
交易中的 ``Payload`` ``data`` 是一个封送的 ``ConfigUpdateEnvelope``。 
``ConfigUpdateEnvelope`` 定义如下:

配置更新以 ``Envelope`` 形式提交，类型为 ``HeaderType_CONFIG_UPDATE`` 。交易的 ``Payload`` ``data`` 是封送的 ``ConfigUpdateEnvelope`` 。 ``ConfigUpdateEnvelope`` 的定义如下：

::

    message ConfigUpdateEnvelope {
        bytes config_update = 1;
        repeated ConfigSignature signatures = 2;
    }

``signatures`` 字段包含一组授权配置更新的签名。它的消息定义如下：

``signatures`` 字段包含一组授权配置更新的签名。其消息定义为：

::

    message ConfigSignature {
        bytes signature_header = 1;
        bytes signature = 2;
    }

``signature_header`` 是为标准交易定义的，而签名是通过 ``signature_header`` 字节
和 ``ConfigUpdateEnvelope`` 中的 ``config_update`` 字节串联而得。

<<<<<<< HEAD
``ConfigUpdateEnvelope`` ``config_update`` 字节是封送的 ``ConfigUpdate`` 
消息，定义如下：
=======
``signature_header`` 是为标准交易定义的，而签名是由 ``ConfigUpdateEnvelope`` 消息中的 ``signature_header`` 的字节和 ``config_update`` 的字节连接起来的。

The ``ConfigUpdateEnvelope`` ``config_update`` bytes are a marshaled
``ConfigUpdate`` message which is defined as follows:
>>>>>>> 3915c06381b1ecf6f1e7420afdf0ec132c8e954e

``ConfigUpdateEnvelope`` 的 ``config_update`` 字节是封装的 ``ConfigUpdate`` "
"消息，定义如下：

::

    message ConfigUpdate {
        string channel_id = 1;
        ConfigGroup read_set = 2;
        ConfigGroup write_set = 3;
    }

<<<<<<< HEAD
``channel_id`` 是更新所绑定的通道 ID，这对于确定支持此重配置的签名的作用域
是必需的。

``read_set`` 定义了现有配置的子集，属稀疏指定，其中只设置 ``version`` 字段，
其他字段不需要填充。尤其 ``ConfigValue`` ``value`` 或者 ``ConfigPolicy`` 
``policy`` 字段不应在 ``read_set`` 中设置。``ConfigGroup`` 可以有已填充
映射字段的子集，以便引用配置树中更深层次的元素。例如，要将 ``Application`` 组
包含在 ``read-set`` 中，其父组（``Channel`` 组）也必须包含在读集合中，但 
``Channel`` 组不需要填充所有键，例如 ``Orderer`` ``group`` 键，或任何 
``values`` 或 ``policies`` 键。

``write_set`` 指定了要修改的配置片段。由于配置的层次性，对层次结构中深层元素
的写入也必须在其 ``write_set`` 中包含更高级别的元素。但是，对于 ``read-set`` 
中也指定的 ``write-set`` 中的任何同一版本的元素，应该像在 ``read-set``中一样
稀疏地指定该元素。

例如，给定配置：
=======
The ``channel_id`` is the channel ID the update is bound for, this is
necessary to scope the signatures which support this reconfiguration.

``channel_id`` 是所要更新的通道的ID，这对于为支持本次重新配置的签名限定范围很有必要。

The ``read_set`` specifies a subset of the existing configuration,
specified sparsely where only the ``version`` field is set and no other
fields must be populated. The particular ``ConfigValue`` ``value`` or
``ConfigPolicy`` ``policy`` fields should never be set in the
``read_set``. The ``ConfigGroup`` may have a subset of its map fields
populated, so as to reference an element deeper in the config tree. For
instance, to include the ``Application`` group in the ``read_set``, its
parent (the ``Channel`` group) must also be included in the read set,
but, the ``Channel`` group does not need to populate all of the keys,
such as the ``Orderer`` ``group`` key, or any of the ``values`` or
``policies`` keys.

``read_set`` 指定现有配置的一个子集，在这里只设置 ``version`` 字段，而不需要填充其他字段。特定的 "
"``ConfigValue`` 的 ``value`` 或r ``ConfigPolicy`` 的 ``policy`` 字段不应该在 "
"``read_set`` 中设置。 ``ConfigGroup`` 可能填充了映射字段的子集，以便引用配置树中更深层的元素。例如，在 "
"``read_set`` 中包括 ``Application`` 组，其父级（ ``Channel`` 组）也必须包括在读集里面，但是， "
"``Channel`` 组不需要填充所有的键，如 ``Orderer`` ``group`` 键，或任何的 ``values`` 或 "
"``policies`` 键。

The ``write_set`` specifies the pieces of configuration which are
modified. Because of the hierarchical nature of the configuration, a
write to an element deep in the hierarchy must contain the higher level
elements in its ``write_set`` as well. However, for any element in the
``write_set`` which is also specified in the ``read_set`` at the same
version, the element should be specified sparsely, just as in the
``read_set``.

``write_set`` 指定修改的配置块。由于配置的层次性，对层次结构深处的元素的写操作也必须在其  ``write_set`` "
"中包含该元素上层各级别的元素。但是，对于同一版本的元素，同时在 ``read_set`` 和 ``write_set`` 中包含了，则在 "
"``write_set`` 中应该像在 ``read_set`` 中一样稀疏表示，即不需要填充无关字段。

For example, given the configuration:
>>>>>>> 3915c06381b1ecf6f1e7420afdf0ec132c8e954e

例如，给定的以下配置：

::

    Channel: (version 0)
        Orderer (version 0)
        Application (version 3)
           Org1 (version 2)

为了提交一个修改 ``Org1`` 的配置更新，``read_set`` 应如：

提交一个修改 ``Org1`` 的配置更新，其 ``read_set`` 应该是这样：

::

    Channel: (version 0)
        Application: (version 3)

``write_set`` 应如

其 ``write_set`` 应该是

::

    Channel: (version 0)
        Application: (version 3)
            Org1 (version 3)

<<<<<<< HEAD
收到 ``CONFIG_UPDATE`` 后，排序节点按以下步骤计算 ``CONFIG`` 结果。

1. 验证 ``channel_id`` 和 ``read_set``。``read_set`` 中的所有元素都必须
   以给定的版本存在。
2. 收集 ``write_set`` 中的所有与 ``read_set`` 版本不一致的元素以计算更新集。
3. 校验更新集合中版本号刚好增长了1的每个元素。
4. 校验附加到 ``ConfigUpdateEnvelope`` 的签名集是否满足更新集中每个元素的 
   ``mod_policy``。
5. 通过应用更新到到当前配置，计算出配置的新的完整版本。
6. 将配置写入 ``ConfigEnvelope``，包含作为 ``last_update`` 字段的
   ``CONFIG_UPDATE``，和编码为 ``config`` 字段的新配置, 以及递增的 
   ``sequence`` 值。
7. 将新 ``ConfigEnvelope`` 写入类型为 ``CONFIG`` 的 ``Envelope``，并最终将其
   作为唯一交易写入一个新的配置区块。

当节点（或其他任何 ``Deliver`` 的接收者）收到这个配置区块时，它应该，将 
``last_update`` 消息应用到当前配置并校验经过排序计算的 ``config`` 字段包含
当前的新配置，以此来校验这个配置是否得到了适当地验证。

组和值的许可更新
=======
When the ``CONFIG_UPDATE`` is received, the orderer computes the
resulting ``CONFIG`` by doing the following:

当接收到配置更新 ``CONFIG_UPDATE`` 之后，排序者会根据以下几步计算配置结果 ``CONFIG`` ：

1. Verifies the ``channel_id`` and ``read_set``. All elements in the
   ``read_set`` must exist at the given versions.
2. Computes the update set by collecting all elements in the
   ``write_set`` which do not appear at the same version in the
   ``read_set``.
3. Verifies that each element in the update set increments the version
   number of the element update by exactly 1.
4. Verifies that the signature set attached to the
   ``ConfigUpdateEnvelope`` satisfies the ``mod_policy`` for each
   element in the update set.
5. Computes a new complete version of the config by applying the update
   set to the current config.
6. Writes the new config into a ``ConfigEnvelope`` which includes the
   ``CONFIG_UPDATE`` as the ``last_update`` field and the new config
   encoded in the ``config`` field, along with the incremented
   ``sequence`` value.
7. Writes the new ``ConfigEnvelope`` into a ``Envelope`` of type
   ``CONFIG``, and ultimately writes this as the sole transaction in a
   new configuration block.

1. 验证通道ID ``channel_id`` 和读集 ``read_set`` 。所有 ``read_set`` 元素的给定版本必须存在。
2. 通过收集 ``write_set`` 中与 ``read_set`` 中的版本不同的所有元素来计算更新集。
3. 验证更新集中的每个元素，将元素更新的版本号精确地增加1。
4. 验证附加到 ``ConfigUpdateEnvelope`` 上的签名集是否满足更新集中每个元素的 ``mod_policy`` 。
5. 通过将更新集应用于当前配置，计算配置的新的完整版本。
6. 将新配置写入 ``ConfigEnvelope`` 中，其中包括值为 ``CONFIG_UPDATE`` 的 ``last_update`` "
"字段，以及编码进 ``config`` 字段的新配置，以及递增的 ``sequence`` 值。
7. 将新的 ``ConfigEnvelope`` 写入类型为 ``CONFIG`` 的 ``Envelope`` "
"中，并最终将其作为新配置块中的唯一交易写入。

When the peer (or any other receiver for ``Deliver``) receives this
configuration block, it should verify that the config was appropriately
validated by applying the ``last_update`` message to the current config
and verifying that the orderer-computed ``config`` field contains the
correct new configuration.

当Peer（或者任何其他的 ``Deliver`` 的接收方）接收到这个配置块时，它应该通过将 ``last_update`` "
"消息应用到当前配置并验证排序计算的 ``config`` 字段包含正确的新配置，来验证配置是否得到了适当的验证。

Permitted configuration groups and values
许可的配置组和值
>>>>>>> 3915c06381b1ecf6f1e7420afdf0ec132c8e954e
-----------------------------------------

任何有效配置都是以下配置的子集。在这里，我们用符号 ``peer.<MSG>`` 来定义一个 
``ConfigValue``，其 ``value`` 字段是一个封送的名为  ``<MSG>`` 的消息。它定义在 
``fabric/protos/peer/configuration.proto`` 中。符号 ``common.<MSG>``、
``msp.<MSG>`` 和 ``orderer.<MSG>`` 类似对应，它们的消息依次定义在
``fabric/protos/common/configuration.proto``、
``fabric/protos/msp/mspconfig.proto`` 和 
``fabric/protos/orderer/configuration.proto``中

<<<<<<< HEAD
注意，键 ``{{org_name}}`` 和 ``{{consortium_name}}`` 表示任意名称，指示一个
可以用不同名称重复的元素。
=======
任何有效的配置都是以下配置的子集。我们使用符号 ``peer.<MSG>`` 定义一个 ``ConfigValue`` ，它的 ``value`` "
"字段是在 ``fabric/protos/peer/configuration.proto`` 中定义的名为 ``<MSG>`` "
"的封装的原型消息。另外，符号 ``common.<MSG>`` 、 ``msp.<MSG>`` 和 ``orderer.<MSG>`` "
"都相似，不过他们的原型消息分别定义在 ``fabric/protos/common/configuration.proto`` 、 "
"``fabric/protos/msp/mspconfig.proto`` 和 "
"``fabric/protos/orderer/configuration.proto`` 。



Note, that the keys ``{{org_name}}`` and ``{{consortium_name}}``
represent arbitrary names, and indicate an element which may be repeated
with different names.
>>>>>>> 3915c06381b1ecf6f1e7420afdf0ec132c8e954e

注意，键 ``{{org_name}}`` 和 ``{{consortium_name}}`` 表示任意名称。

::

    &ConfigGroup{
        Groups: map<string, *ConfigGroup> {
            "Application":&ConfigGroup{
                Groups:map<String, *ConfigGroup> {
                    {{org_name}}:&ConfigGroup{
                        Values:map<string, *ConfigValue>{
                            "MSP":msp.MSPConfig,
                            "AnchorPeers":peer.AnchorPeers,
                        },
                    },
                },
            },
            "Orderer":&ConfigGroup{
                Groups:map<String, *ConfigGroup> {
                    {{org_name}}:&ConfigGroup{
                        Values:map<string, *ConfigValue>{
                            "MSP":msp.MSPConfig,
                        },
                    },
                },

                Values:map<string, *ConfigValue> {
                    "ConsensusType":orderer.ConsensusType,
                    "BatchSize":orderer.BatchSize,
                    "BatchTimeout":orderer.BatchTimeout,
                    "KafkaBrokers":orderer.KafkaBrokers,
                },
            },
            "Consortiums":&ConfigGroup{
                Groups:map<String, *ConfigGroup> {
                    {{consortium_name}}:&ConfigGroup{
                        Groups:map<string, *ConfigGroup> {
                            {{org_name}}:&ConfigGroup{
                                Values:map<string, *ConfigValue>{
                                    "MSP":msp.MSPConfig,
                                },
                            },
                        },
                        Values:map<string, *ConfigValue> {
                            "ChannelCreationPolicy":common.Policy,
                        }
                    },
                },
            },
        },

        Values: map<string, *ConfigValue> {
            "HashingAlgorithm":common.HashingAlgorithm,
            "BlockHashingDataStructure":common.BlockDataHashingStructure,
            "Consortium":common.Consortium,
            "OrdererAddresses":common.OrdererAddresses,
        },
    }

<<<<<<< HEAD
排序系统通道配置
=======
Orderer system channel configuration
排序者系统通道配置
>>>>>>> 3915c06381b1ecf6f1e7420afdf0ec132c8e954e
------------------------------------

排序系统通道需要定义一些排序参数，以及创建通道的联盟。一个排序服务有且只能有一个
排序系统通道，它是需要创建的第一个通道（或更准确地说是启动）。建议不要在排序系统
通道的创世配置中定义应用，但在测试时是可以的。注意，任何对排序系统通道具有读权限
的成员可能看到所有的通道创建，所以，这个通道的访问应用受到限制。

<<<<<<< HEAD
排序参数被定义在如下配置子集中：
=======
排序系统通道需要定义排序的参数，以及创建通道的联盟。排序服务必须只有一个排序系统通道，并且它是创建的第一个通道（或者更准确地说是引导启动的）。建议永远不要在排序系统通道初始配置中定义应用程序部分，但是可以进行测试。注意，任何对排序系统通道具有读权限的成员都可以看到所有通道的创建，因此应该限制该通道的读权限。

The ordering parameters are defined as the following subset of config:
>>>>>>> 3915c06381b1ecf6f1e7420afdf0ec132c8e954e

排序参数定义为配置的子集，如下：

::

    &ConfigGroup{
        Groups: map<string, *ConfigGroup> {
            "Orderer":&ConfigGroup{
                Groups:map<String, *ConfigGroup> {
                    {{org_name}}:&ConfigGroup{
                        Values:map<string, *ConfigValue>{
                            "MSP":msp.MSPConfig,
                        },
                    },
                },

                Values:map<string, *ConfigValue> {
                    "ConsensusType":orderer.ConsensusType,
                    "BatchSize":orderer.BatchSize,
                    "BatchTimeout":orderer.BatchTimeout,
                    "KafkaBrokers":orderer.KafkaBrokers,
                },
            },
        },

参与排序的每个组织在 ``Order`` 组下都有一个组元素。此组定义单个参数 ``MSP``，
其中包含该组织的加密身份信息。 ``Order`` 组的 ``Values`` 决定了排序节点的工作
方式。它们在每个通道中存在，因此，例如 ``orderer.BatchTimeout`` 可能在不同通
道上被不同地指定。

<<<<<<< HEAD
在启动时，排序节点将面临一个包含了很通道信息的文件系统。排序节点通过识别带有定义的
联盟组的通道来识别系统通道。联盟组的结构如下。
=======
每个参与排序的组织都有一个组元素，位于 ``Orderer`` 组下。这个组定义了一个参数 ``MSP`` ，其中包含该组织的密码标识信息。 "
"``Orderer`` 组的值 ``Values`` 决定排序节点如何工作。它们存在于每个通道中，所以，例如，在一个通道上指定的 "
"``orderer.BatchTimeout`` 可能与在另一个通道上指定的不同。

At startup, the orderer is faced with a filesystem which contains
information for many channels. The orderer identifies the system channel
by identifying the channel with the consortiums group defined. The
consortiums group has the following structure.
>>>>>>> 3915c06381b1ecf6f1e7420afdf0ec132c8e954e

在启动时，排序者面对的文件系统包含许多通道的信息。排序者通过使用定义的联盟组标识通道来标识系统通道。联盟组具有以下结构。

::

    &ConfigGroup{
        Groups: map<string, *ConfigGroup> {
            "Consortiums":&ConfigGroup{
                Groups:map<String, *ConfigGroup> {
                    {{consortium_name}}:&ConfigGroup{
                        Groups:map<string, *ConfigGroup> {
                            {{org_name}}:&ConfigGroup{
                                Values:map<string, *ConfigValue>{
                                    "MSP":msp.MSPConfig,
                                },
                            },
                        },
                        Values:map<string, *ConfigValue> {
                            "ChannelCreationPolicy":common.Policy,
                        }
                    },
                },
            },
        },
    },

注意，每个联盟定义一组成员，正如排序组织里的组织成员一样。每个联盟也定义一个 
``ChannelCreationPolicy``。这是一个应用于授权通道创建请求的策略。通常，该值
将被设置为一个 ``ImplicitMetaPolicy``，并要求通道的新成员签名以授权通道创建。
更多关于通道创建的细节，请参见下文。

<<<<<<< HEAD
应用通道配置
=======
"注意，每个联盟定义一组成员，就像排序组织集的组织成员一样。每个联盟还定义了一个通道创建策略 ``ChannelCreationPolicy`` "
"。这是一个用于授权通道创建请求的策略。通常，这个值将被设置为隐式元策略 ``ImplicitMetaPolicy`` "
"，要求通道的新成员签署授权通道创建。有关通道创建的更多细节将在本文档的后面介绍。"

Application channel configuration
应用程序通道配置
>>>>>>> 3915c06381b1ecf6f1e7420afdf0ec132c8e954e
---------------------------------

应用配置适用于为应用类型交易而设计的通道。它定义如下：

"应用程序配置用于通道，它为应用程序类型交易设计。定义如下："

::

    &ConfigGroup{
        Groups: map<string, *ConfigGroup> {
            "Application":&ConfigGroup{
                Groups:map<String, *ConfigGroup> {
                    {{org_name}}:&ConfigGroup{
                        Values:map<string, *ConfigValue>{
                            "MSP":msp.MSPConfig,
                            "AnchorPeers":peer.AnchorPeers,
                        },
                    },
                },
            },
        },
    }

正如 ``Orderer`` 部分，每个组织被编码为组。但是，并非仅仅编码 ``MSP`` 身份信息，
每个组织额外编码一个 ``AnchorPeers`` 列表。这个列表允许不同组织的节点互相联系以
建立对等 gossip 网络。

<<<<<<< HEAD
应用通道对排序组织副本和共识选项进行编码，以允许对这些参数进行确定的更新，因此排序
系统通道配置中相同的 ``Orderer`` 部分也被包括在内。但从应用程序角度来看，这在很大
程度上可能被忽略。

通道创建
----------------

当排序节点收到一个尚不存在的通道的 ``CONFIG_UPDATE`` 时，排序节点假定这是一个通道
创建请求并执行以下内容。

1. 排序节点识别将为之执行通道创建请求的联盟。它通过查看顶级组的 ``Consortium`` 值
   来完成这一操作。
2. 排序节点校验：包含在 ``Application`` 组中的组织是包含在对应联盟中的组织的一个子集，
   并且  ``ApplicationGroup`` ``version`` 被设为 ``1``。
3. 排序节点校验：如果联盟有成员，那么新通道也要有应用成员（创建没有成员的联盟和通道仅
   用于测试）。
4. 排序节点从排序系统通道中获取 ``Orderer`` 组，使用新指定的成员建立 ``Application`` 
   组，并按联盟配置中指定的 ``ChannelCreationPolicy`` 指定它的 ``mod_policy``，
   以此建立模板配置。注意，策略会在新配置的上下文被评估，所以，一个要求 ``ALL`` 成员
   的策略，会要求新通道所有成员的签名，而不是联盟所有成员的签名。
5. 然后排序节点将 ``CONFIG_UPDATE`` 作为一个更新应用到这个模板配置。 因为 
   ``CONFIG_UPDATE`` 将修改应用到 ``Application`` 组（它的 ``version`` 是 
   ``1``），配置代码按 ``ChannelCreationPolicy`` 验证这些更新。如果通道的创
   建包含任何其他修改，比如个别组织的锚节点，则这个元素相应的修改策略也会被调用。
6. 带有新通道配置的 ``CONFIG`` 交易被封装并发送给的排序系统通道以进行排序，排序后，
   通道被创建了。
=======
就像配置的 ``Orderer`` 部分一样，本部分的每个组织也都被编码为组。但是，不同的是，不只编码 ``MSP`` "
"身份信息，每个组织还额外编码了一个 ``AnchorPeers`` 列表。此列表允许不同组织的 Peers 相互联系，建立 gossip 通信网络。

The application channel encodes a copy of the orderer orgs and consensus
options to allow for deterministic updating of these parameters, so the
same ``Orderer`` section from the orderer system channel configuration
is included. However from an application perspective this may be largely
ignored.

应用程序通道编码了排序者组织和共识选项的副本，然后确定性地更新这些参数，所以配置中包含了与排序者系统通道配置相同的 ``Orderer`` "
"部分。然而，从应用程序的角度来看，很大程度上可能忽略这些。

Channel creation
通道的创建
----------------

When the orderer receives a ``CONFIG_UPDATE`` for a channel which does
not exist, the orderer assumes that this must be a channel creation
request and performs the following.
当排序者接收到一个不存在的通道的 ``CONFIG_UPDATE`` 交易时，排序者假定这肯定是一个通道创建请求，并执行以下操作。

1. The orderer identifies the consortium which the channel creation
   request is to be performed for. It does this by looking at the
   ``Consortium`` value of the top level group.
2. The orderer verifies that the organizations included in the
   ``Application`` group are a subset of the organizations included in
   the corresponding consortium and that the ``ApplicationGroup`` is set
   to ``version`` ``1``.
3. The orderer verifies that if the consortium has members, that the new
   channel also has application members (creation consortiums and
   channels with no members is useful for testing only).
4. The orderer creates a template configuration by taking the
   ``Orderer`` group from the ordering system channel, and creating an
   ``Application`` group with the newly specified members and specifying
   its ``mod_policy`` to be the ``ChannelCreationPolicy`` as specified
   in the consortium config. Note that the policy is evaluated in the
   context of the new configuration, so a policy requiring ``ALL``
   members, would require signatures from all the new channel members,
   not all the members of the consortium.
5. The orderer then applies the ``CONFIG_UPDATE`` as an update to this
   template configuration. Because the ``CONFIG_UPDATE`` applies
   modifications to the ``Application`` group (its ``version`` is
   ``1``), the config code validates these updates against the
   ``ChannelCreationPolicy``. If the channel creation contains any other
   modifications, such as to an individual org's anchor peers, the
   corresponding mod policy for the element will be invoked.
6. The new ``CONFIG`` transaction with the new channel config is wrapped
   and sent for ordering on the ordering system channel. After ordering,
   the channel is created.
>>>>>>> 3915c06381b1ecf6f1e7420afdf0ec132c8e954e

1. 排序者区分要为哪个联盟执行通道创建请求。它通过观察顶级配置组的 ``Consortium`` 的值，来做到这一点。
2. 排序者验证确认包含在 ``Application`` 组中的组织是相应联盟中的组织集合的子集，并且 ``ApplicationGroup`` 被设置为 "
"``version`` ``1`` 。
3. 排序者验证确认联盟是否有成员，那么新通道也能有应用程序成员（只能在测试时，创建没有成员的联盟和通道）。
4. 排序者通过从排序系统通道获取 ``Orderer`` 组来创建配置模板，并使用新指定的成员创建 ``Application`` 组，并将其 "
"``mod_policy`` 指定为联盟配置中指定的 ``ChannelCreationPolicy`` "
"。请注意，该策略是在新配置的上下文中评估的，因此要求 ``ALL`` 成员的策略是指要求所有新通道成员签名，而不是要求联盟的所有成员签名。
5. 然后排序者将本次配置更新 ``CONFIG_UPDATE`` 应用于此配置模板。因为配置更新 ``CONFIG_UPDATE`` 会将这些修改应用于 "
"``Application`` 组（它的 ``version`` is ``1`` ），所以配置代码会根据策略 "
"``ChannelCreationPolicy`` "
"来验证这些更新。如果通道创建包含任何其他修改，例如对单个组织的的锚节点的修改，则将调用元素的相应修改策略。
6. 带有新通道配置的新 ``CONFIG`` 交易被封装好，并发送到排序系统通道上进行排序。排序之后，将创建通道。

.. Licensed under Creative Commons Attribution 4.0 International License
   https://creativecommons.org/licenses/by/4.0/

