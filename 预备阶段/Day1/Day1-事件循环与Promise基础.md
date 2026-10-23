# Day 1 · 事件循环 + Promise 基础

> 阶段：预备阶段 · JS/TS 内功
> 预计时长：2~3h
> 使用说明：题目看「题目区」，作答写在「答题区」。完成后把答题区内容发给 AI review。
> 铁律：代码自己敲，不许直接抄；每个知识点最后要能用自己的话讲一遍（费曼学习法）。

---

## 📌 知识点速览（原理，先建立心智模型）

1. **JS 单线程为何能并发**：靠事件循环（Event Loop）。调用栈跑同步代码，异步任务交给宿主（浏览器/Node），完成后把回调放进队列，栈空了再取出执行。
2. **宏任务 vs 微任务**：
   - 宏任务：`setTimeout`、`setInterval`、I/O、UI 渲染、整段 script。
   - 微任务：`Promise.then/catch/finally`、`queueMicrotask`、`MutationObserver`。
   - 规则：每执行完 **一个宏任务**，就 **清空整个微任务队列**，再进下一个宏任务。微任务会「插队」。
3. **Promise 三态**：`pending → fulfilled / rejected`，状态一旦改变 **不可逆**；`.then` 返回 **一个新 Promise**，这是链式调用的本质。

---

## 📝 题目区

### 任务 A：事件循环输出题

先在心里/纸上写出打印顺序，**再运行验证**：

```js
console.log('1');
setTimeout(() => console.log('2'), 0);
Promise.resolve().then(() => console.log('3'));
console.log('4');
```

**问题**：写出打印顺序，并解释为什么 `3` 在 `2` 前面。

### 任务 B：手写 `sleep` 函数

要求：`sleep(1000)` 返回一个 Promise，1 秒后 resolve。用它实现「等 1 秒 → 打印 A → 再等 1 秒 → 打印 B」。
（这是 Day 5 并发对比 demo 的地基，务必自己用 `new Promise` + `setTimeout` 写，不许抄库）

### 加餐自测

把任务 A 改成下面这样，预测新的打印顺序：

```js
console.log('1');
setTimeout(() => console.log('2'), 0);
Promise.resolve().then(() => console.log('3'));
Promise.resolve().then(() => console.log('5'));
console.log('4');
```

---

## ✍️ 答题区（在下面作答）
### 任务 A 作答

- 我的打印顺序：1432
- 我的解释（为什么 3 在 2 前）：
主线程执行完毕，会先执行微任务队列中的代码，后面才是宏任务
### 任务 B 作答（贴你写的 `sleep` 代码）

```js
// 在这里写你的代码
function sleep(ms) {
   return new Promise((resolve, reject) => {
      setTimeout(resolve, ms);
   })
}

await sleep(1000);
console.log('A');
await sleep(1000);
console.log('B')
```

### 加餐自测作答

- 新的打印顺序：14352
- 原因：主线程先跑，遇到宏任务和微任务都放入对应的队列，主线程执行完执行微任务队列，然后再执行宏任务

### 今日复盘

- 今天学到的点：
- 卡壳 / 不理解的地方：

### 验收自查

- [ ] 任务 A：能说出正确顺序并解释 3 在 2 前
- [ ] 任务 B：自己写出 `sleep` 并跑通 A、B 两次打印
- [ ] 加餐：能预测新顺序并说清原因
