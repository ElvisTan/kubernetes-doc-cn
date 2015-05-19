# 设计原则

	进行中

扩展Kubernetes时需要遵循的原则。

## API

参见[API约定](../api-conventions.md).

* 所有的API都应该是陈述式的（declarative）。
* API对象应该是可补充和可组合的，而不是不透明的封装。
* 控制平面（control plane）应该是透明的——没有隐藏的内部API。
* API操作的消耗和所操作的对象数量应该是成比例的。所以常见的过滤查询必须被索引。当心API会产生二次调用行为的模式。
* 对象的状态必须是100%可通过观察重新组织的。任何历史保存必须只是一个优化，而不要求是正确的操作（Any history kept must be just an optimization and not required for correct operation）。
* Cluster-wide invariants are difficult to enforce correctly. Try not to add them. If you must have them, don't enforce them atomically in master components, that is contention-prone and doesn't provide a recovery path in the case of a bug allowing the invariant to be violated. Instead, provide a series of checks to reduce the probability of a violation, and make every component involved able to recover from an invariant violation. 
* 底层的API应该被设计来被上层的系统控制。上层的API应该是面向意图的（想下SLO？），而不是面向实现的（想下控制旋钮）。

## 控制逻辑

* Functionality must be *level-based*, meaning the system must operate correctly given the desired state and the current/observed state, regardless of how many intermediate state updates may have been missed. Edge-triggered behavior must be just an optimization.
* Assume an open world: continually verify assumptions and gracefully adapt to external events and/or actors. Example: we allow users to kill pods under control of a replication controller; it just replaces them.
* Do not define comprehensive state machines for objects with behaviors associated with state transitions and/or "assumed" states that cannot be ascertained by observation. 
* Don't assume a component's decisions will not be overridden or rejected, nor for the component to always understand why. For example, etcd may reject writes. Kubelet may reject pods. The scheduler may not be able to schedule pods. Retry, but back off and/or make alternative decisions.
* Components should be self-healing. For example, if you must keep some state (e.g., cache) the content needs to be periodically refreshed, so that if an item does get erroneously stored or a deletion event is missed etc, it will be soon fixed, ideally on timescales that are shorter than what will attract attention from humans.
* Component behavior should degrade gracefully. Prioritize actions so that the most important activities can continue to function even when overloaded and/or in states of partial failure.

## 架构

* 只有apiserver应该和etcd/存储交互，而其他组件（scheduler、kubelet等）都不应该。
* 一个节点不应该连累到整个集群。
* 组件在收不到新指令的情况下（比如因为网络分区问题或者其他组件故障），应该继续做之前被要求做的事情。
* 所有的组件应该一直在内存中存放所有相关的状态。apiserver应该直接写etcd/存储，其他组件应该通过apiserver来写，并且它们应该监视其他客户端发起的更新操作。
* 优先使用监视的方法，而不是轮询。

## 可扩展性

TODO: 可插拔性（原文中的TODO）

## 自举（Bootstrapping）

* 目标是所有组件实现[自托管（Self-hosting）](https://github.com/GoogleCloudPlatform/kubernetes/issues/246)。
* 让依赖最小化，特别是那些需要稳态运行的组件。
* 通过有原则的分层方式对保留的依赖进行分层。
* Break any circular dependencies by converting hard dependencies to soft dependencies.
  * Also accept that data from other components from another source, such as local files, which can then be manually populated at bootstrap time and then continuously updated once those other components are available.
  * State should be rediscoverable and/or reconstructable.
  * Make it easy to run temporary, bootstrap instances of all components in order to create the runtime state needed to run the components in the steady state; use a lock (master election for distributed components, file lock for local components like Kubelet) to coordinate handoff. We call this technique "pivoting".
  * Have a solution to restart dead components. For distributed components, replication works well. For local components such as Kubelet, a process manager or even a simple shell loop works.

## 可用性

TODO（原文中的TODO）

## 一般原则

* [Eric Raymond's 17 UNIX rules](https://en.wikipedia.org/wiki/Unix_philosophy#Eric_Raymond.E2.80.99s_17_Unix_Rules)