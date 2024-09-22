
- 分层队列
- 容量保证
- 安全
- **弹性**
- 多用户
- 可操作性
	- 运行时配置
	- 排空应用程序
- 基于资源的调度
- 基于默认或用户指定替换规则的队列映射接口
- 优先级调度
- 百分比资源配置
- 绝对资源配置
- 权重资源配置
- 通用容量向量资源配置
- 动态自动创建和管理叶队列


#### 资源配置
##### 01 队列模式

*`yarn.scheduler.capacity.legacy-queue-mode.enabled`*

指定是否启用 legacy-queue-mode (默认为`true`).

**legacy-queue-mode**

传统队列模式下, Capacity Scheduler 支持三种资源分配配置模式:

- 相对模式(relative mode).
- 权重模式(weight mode).
- 绝对模式(absolute mode).

**non-legacy-queue-mode**

非传统队列模式下, 支持使用通用容量向量(Universal Capacity Vector)格式指定队列容量, 不同资源的配置模式可以自由搭配.

例如`[memory=50%,vcores=2w,gpu=1]`表示:

- 分配 50% 内存, 2 倍权重虚拟内核, 1 枚 GPU.

集群资源在分配时,首先为队列分配绝对容量,其次分配相对容量,剩余容量用于按权重分配.

##### 02 队列资源分配

*`yarn.scheduler.capacity.<queue-path>.capacity`*

指定队列的最小容量, 及 **资源配置模式**.

当接受一个 **浮点数**, 将使用 **相对资源配置**, 需要确保每层所有队列容量之和等于 100;
当接受一个 **以`w`为后缀的浮点数**, 将使用 **权重资源配置**;
当接受一个 **绝对资源值**, 将使用 **绝对资源配置**, 需要确保每层所有队列容量之和小于其父队列.

相对资源配置和绝对资源配置可以混合使用, 需要确保子队列和父队列使用相同的资源配置模式.
当存在空闲资源时, 队列中的应用可以消耗超出队列最小容量的资源, 以提供**弹性**.
当禁用 legacy-queue-mode 时, 可以使用通用容量向量格式配置队列容量.

*`yarn.scheduler.capacity.<queue-path>.maximum-capacity`*

指定队列的最大容量, 用于限定队列中应用程序的弹性.

相对/权重资源配置下, 接受一个浮点数($[0, 100]$).
绝对资源配置下, 接受一个绝对资源值, 需要确保队列的绝对最大容量不小于绝对容量.

当值为`-1`时, 认为最大容量是 100%(即弹性最大化).
当禁用 legacy-queue-mode 时, 可以使用通用容量向量格式配置队列容量.

*`yarn.scheduler.capacity.minimum-user-limit-percent`*
*`yarn.scheduler.capacity.<queue-path>.minimum-user-limit-percent`*

接受一个整数(默认为`100`), 用于限制**同一队列**中**不同用户**所占用资源的最小占比.

例如, 当值为`25`, 每名用户的用户限制最小资源占比为 25%, 最大用户资源占比为 max(100/n, 25):

- 当两名用户向同一队列提交应用程序时, 每名用户的用户限制最大资源占比为 50%;
- 当三名用户向同一队列提交应用程序时, 每名用户的用户限制最大资源占比为 33%;
- 当四名及以上用户向同一队列提交应用程序时, 每名用户的用户限制最大资源占比为 25%.


*`yarn.scheduler.capacity.minimum-user-limit-factor`*
*`yarn.scheduler.capacity.<queue-path>.minimum-user-limit-factor`*

接受一个浮点数(默认为`1`), 用于限制**单个用户**所占用**集群**资源的最大值是队列容量的倍数.

当使用带权重的灵活自动队列创建(yarn.scheduler.capacity..auto-queue-creation-v2), 将自动把值设为`-1`.因为动态队列的权重被硬编码为`1`, 在集群空闲时本当使用更多资源.
当值为`-1`时, 禁用该功能(即不限制单个用户最大资源占用).

*`yarn.scheduler.capacity.<queue-path>.maximum-allocation-mb`*

为每个队列在 Resource Manager 申请的容器最大内存限制.

*`yarn.scheduler.capacity.<queue-path>.maximum-allocation-vcores`*

为每个队列在 Resource Manager 申请的容器最大虚拟内核限制.

*`yarn.scheduler.capacity.<queue-path>.user-settings.<user-name>.weight`*

接受一个浮点数(默认为`1`), 用于指定**同一队列**中**各个用户**所占用资源的权重.
