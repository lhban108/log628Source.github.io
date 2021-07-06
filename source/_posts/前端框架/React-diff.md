---
layout: React
title: React Diff
date: 2018-05-29 22:48:29
tags: [React,框架]
---

# diff

### 初始化 fiber tree 流程

### 根节点 fiberRootNode 创建

### 组件 fiberNode 创建

### reconciler 阶段

### renderer 阶段

### commit 阶段

### beginWork 流程

### completeWork 流程

### commitWork 流程

### alternate fiber tree 流程 （diff 过程）


## workLoop 环节


### performConcurrentWorkOnRoot 流程

并发模式 concurrent 级别的调度任务


### performSyncWorkOnRoot 流程

> syncLane 级别的调度任务

常量集合

**react执行上下文状态**

```js
export const NoContext = /*             */ 0b0000000;
const BatchedContext = /*               */ 0b0000001;
const EventContext = /*                 */ 0b0000010;
const DiscreteEventContext = /*         */ 0b0000100;
const LegacyUnbatchedContext = /*       */ 0b0001000;
const RenderContext = /*                */ 0b0010000;
const CommitContext = /*                */ 0b0100000;
export const RetryAfterError = /*       */ 0b1000000;
```

**Root 节点状态**

```js
type RootExitStatus = 0 | 1 | 2 | 3 | 4 | 5;
const RootIncomplete = 0;
const RootFatalErrored = 1;
const RootErrored = 2;
const RootSuspended = 3;
const RootSuspendedWithDelay = 4;
const RootCompleted = 5;
```

**需要注意的很多标记属性**

```js
// Describes where we are in the React execution stack
let executionContext: ExecutionContext = NoContext;
// The root we're working on
let workInProgressRoot: FiberRoot | null = null;
// The fiber we're working on
let workInProgress: Fiber | null = null;
// The lanes we're rendering
let workInProgressRootRenderLanes: Lanes = NoLanes;

export let subtreeRenderLanes: Lanes = NoLanes;
```

#### flushPassiveEffects

遍历执行 useEffect 的回调

#### renderRootSync

这里应该就是 reconciler 流程的入口了

####