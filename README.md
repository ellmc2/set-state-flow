# useState flow

关于 `setState` 的流程

## 初次渲染

1. 自上而下执行组件（渲染阶段）
1. 若遇到 `useMemo` ，调用计算函数 `calculateValue`，缓存计算结果；
1. 遇到微任务（如：`queueMicrotask`、`Promise.resolve().then()`），将回调加入微任务队列；
1. 遇到宏任务（如：`setTimeout`、`setInterval` 等），将回调加入宏任务队列；

1. 提交渲染给浏览器（提交阶段）：
1. 执行 `useLayoutEffect` 的回调；
1. 执行 `useEffect` 的回调；
1. 查看微任务队列，若有任务，则执行微任务；
1. 看看宏任务队列，若有任务，则执行宏任务；
1. 若浏览器空闲，则执行 `requestIdleCallback`

## 第一次触发事件处理函数

1. 自上而下执行事件处理函数
   1. 遇到微任务（如：`queueMicrotask`、`Promise.resolve().then()`），将回调加入微任务队列；
   2. 遇到宏任务（如：`setTimeout`、`setInterval` 等），将回调加入宏任务队列；
   3. 遇到第一个 `setState` ，执行 `setState` 回调，并开启一个 `queueMircoTask`，将后续所有的 `setState` 回调装入一个队列；
   4. 主线程空出，查看微任务队列，若有任务，则执行一个微任务；
   5. 渲染阶段
      1. 执行 state 的更新队列的 callback；
   6. 提交阶段
      1. 依次执行 `useLayoutEffect` 的 `cleanup` 函数队列中的函数
      2. 执行 `useLayoutEffect` 的回调；
      3. 依次执行 `useEffect` 的 `cleanup` 函数队列中的函数
      4. 执行 `useEffect` 的回调；
      5. 查看微任务队列（事件处理函数的剩余微任务、渲染阶段新生成的微任务），若有任务，则执行微任务；
      6. 看看宏任务队列（事件处理函数的宏任务、渲染阶段新生成的宏任务），若有任务，则执行宏任务；
      7. 若浏览器空闲，则执行 `requestIdleCallback`

## 第二次触发事件处理函数

自上而下执行事件处理函数

1. 执行第一个 `setState` 前的微任务队列中的回调；
2. 渲染阶段
   1. 执行 state 的更新队列的 callback；
3. 提交阶段
   1. 依次执行 `useLayoutEffect` 的 `cleanup` 函数队列中的函数
   2. 执行 `useLayoutEffect` 的回调；
   3. 依次执行 `useEffect` 的 `cleanup` 函数队列中的函数
   4. 执行 `useEffect` 的回调；
   5. 查看微任务队列（事件处理函数的剩余微任务、渲染阶段新生成的微任务），若有任务，则执行微任务；
   6. 看看宏任务队列（事件处理函数的宏任务、渲染阶段新生成的宏任务），若有任务，则执行宏任务；
   7. 若浏览器空闲，则执行 `requestIdleCallback`
