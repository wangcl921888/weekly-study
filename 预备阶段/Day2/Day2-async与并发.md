# Day 2 · async/await + 并发 + 错误处理 + AbortController

> 阶段：预备阶段 · JS/TS 内功
> 预计时长：2~3h
> 使用说明：题目看「题目区」，作答写在「答题区」。先预测/写代码，再运行验证。完成后发给 AI review。
> 铁律：代码自己敲；每个知识点要能用自己的话讲一遍。
> ⚠️ 环境提醒：本文件里出现顶层 `await` 时，请在 **ESM**（`.mjs` 或 `"type":"module"`）里跑，或自己包一层 `(async () => { ... })()`。

---

## 📌 知识点速览（原理）

1. **async/await 本质**：`async` 函数**返回一个 Promise**；`await` 会「暂停」当前 async 函数、把后面的代码登记为微任务，然后**让出主线程**去跑别的。底层相当于 Generator + 自动执行器。
2. **串行 vs 并发**：
   - 多个 `await` 一行行写 = **串行**，总耗时 ≈ 各自耗时**相加**。
   - 彼此**无依赖**的任务应该用 `Promise.all` **并发**，总耗时 ≈ 最慢的那个。
   - 有依赖（后一个要用前一个的结果）才必须串行。
3. **四个组合方法的语义**：
   - `Promise.all`：全成功才成功；**一个失败立即失败**（fail-fast）。
   - `Promise.allSettled`：等**全部结束**，返回每项 `{status:'fulfilled'|'rejected', value|reason}`，**不 fail-fast**。
   - `Promise.race`：**第一个 settled**（无论成功/失败）的结果决定最终结果。
   - `Promise.any`：第一个**成功**的决定结果；全失败才失败。
4. **错误处理**：async 里用 `try/catch`，或对 Promise 链用 `.catch()`；未处理的 rejection 会触发 `unhandledrejection`。
5. **AbortController**：`new AbortController()` → 把 `controller.signal` 传给可中断操作 → 需要时调 `controller.abort()`，signal 触发 `abort` 事件，让操作 reject（通常是 `AbortError`）。

---

## 📝 题目区

### 任务 A：串行 vs 并发耗时预测

```js
const sleep = (ms, v) => new Promise((r) => setTimeout(() => r(v), ms));

async function serial() {
  const t0 = Date.now();
  const a = await sleep(1000, 'A');
  const b = await sleep(2000, 'B');
  console.log('serial', Date.now() - t0, [a, b]);
}

async function concurrent() {
  const t0 = Date.now();
  const [a, b] = await Promise.all([sleep(1000, 'A'), sleep(2000, 'B')]);
  console.log('concurrent', Date.now() - t0, [a, b]);
}
```

**问题**：`serial` 和 `concurrent` 各约耗时多少 ms？为什么差这么多？什么情况下**必须**用串行？

### 任务 B：all / allSettled / race 语义预测

```js
const p1 = sleep(1000, 'ok1');
const p2 = Promise.reject('fail');
const p3 = sleep(500, 'ok3');
```

先**预测**再运行验证，写出下面每个的结果（成功值 / 抛出的错误 / 数组内容）：

1. `Promise.all([p1, p2, p3])`
2. `Promise.allSettled([p1, p2, p3])`
3. `Promise.race([p1, p2, p3])`

### 任务 C：错误处理（两种方式）

写一段代码：定义一个必定 reject 的 async 操作，分别用 **① `try/catch`** 和 **② `.catch()`** 两种方式捕获它，并各打印出错误信息。

### 任务 D：实现可被中断的 sleep（AbortController）

实现 `sleep(ms, { signal })`：
- 正常情况 `ms` 后 resolve；
- 若传入的 `signal` 被 `abort()`，则**立即 reject**（错误名 `AbortError`）；
- 记得在结束/中断时 `clearTimeout`，避免定时器泄漏。

然后用它演示：启动一个 `2000ms` 的 sleep，**1 秒后中断它**，并捕获打印中断原因。

---

## ✍️ 答题区（在下面作答）

### 任务 A 作答

- serial 约耗时：3000____ms
- concurrent 约耗时：2000____ms
- 为什么差这么多：serial是串行，concurrent是并行。串行要把任务一个个走完，并行取最长的那个任务时间就行
- 什么时候必须串行：根据需求，如果后一个需要依赖前一个任务的结果，就必须串行

### 任务 B 作答

1. `Promise.all` 结果：报错
2. `Promise.allSettled` 结果：返回一个数组[成功Primose, 失败Promise, 成功Promise]
3. `Promise.race` 结果：返回一个失败的Promise

### 任务 C 作答（贴代码）

```js
// 在这里写你的代码
Promise.reject('err').then(res => {}).catch(err => console.log(err))

async function testCatch() {
  try {
    const result = await Promise.reject("err");
  } catch (err) {
    console.log(err);
  }
}

testCatch();
```

### 任务 D 作答（贴代码）

```js
// 在这里写你的 sleep(ms, { signal }) 和中断演示
function sleep(ms, {signal}) {
            return new Promise((resolve, reject) => {
                const timer = setTimeout(resolve, ms);

                signal?.addEventListener("abort", () => {
                  clearTimeout(timer);
                  reject(signal.reason)
                });
            });
        }


        const controller = new AbortController();
        const timer = setTimeout(() => controller.abort(), 1000);
        sleep(2000, {signal: controller.signal}).catch((err) => {
            clearTimeout(timer);
            console.log(err);
        });
```

### 今日复盘

- 今天学到的点：Promise,async,AbortController概念
- 卡壳 / 不理解的地方：AbortController以前没有用过，初步了解并使用

### 验收自查

- [ ] 任务 A：能说出两者耗时并解释串行/并发的取舍
- [ ] 任务 B：三个结果预测正确（尤其 allSettled 的数组结构）
- [ ] 任务 C：两种捕获方式都能跑通
- [ ] 任务 D：sleep 能被 abort 提前中断并正确 reject
