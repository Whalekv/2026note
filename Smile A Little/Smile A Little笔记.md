## 函数引用 vs 函数调用
#p1
```js
btn.addEventListener('click', showRandomJoke);
```
**这段代码中 showRandomJoke为什么不写成 showRandomJoke()?**

|写法|什么时候执行|实际效果（在这个场景下）|是否正确|
|---|---|---|---|
|`showRandomJoke`|点击按钮时才执行|浏览器等到用户真的点按钮，才去调用函数|**正确**（你要的效果）|
|`showRandomJoke()`|代码运行到这一行就立刻执行|页面刚打开、按钮还没被点，笑话就已经换了一次|**错误**（大多数情况下不是想要的）|

这个点（“为什么事件监听器里传函数名不加 ()”）属于 **JavaScript 中“函数作为一等公民** + 回调 / 延迟执行” 这块知识的典型表现。

**函数引用 vs 函数调用（Function Reference vs Function Invocation）**
- 这是最核心、最经常让初学者卡住的地方
- showRandomJoke → 函数本身（一个值，可以被传递、赋值、存起来）
- showRandomJoke() → 立刻执行函数，并得到它的返回值
- 很多人学到函数的时候，主要练习的是“调用它得到结果”，很少刻意练习“把函数当值传给别人”这种用法。 → 所以一看到函数名，脑子第一反应就是“要加 () 才执行”，而忽略了“它也可以不执行、只是被传递”这个视角。

---

## 安全地获取一个数组
#p2

```js
async function markAsSeen (item) {

    if (!item || !item.id) return;

    const key = item.id.startsWith("j_") ? "seenJokes" : "seenMedia";
    const data = await chrome.storage.local.get(key);
    const list = data[key] || []; //p2


    if(!list.includes(item.id)) {
        list.push(item.id);
        await chrome.storage.local.set({ [key]: list});

    }

}
```

在这段代码中，
```js
const data = await chrome.storage.local.get(key);
```
这一步结束后，data 可能的几种样子：
- 情况1：存储里本来就有这个 key，且值是数组 ` data = { seenJokes: ["j_001", "j_002"] }`
- 情况2：这个 key 从来没存过，或者被清空了 ` data = {} `← 空对象
- 情况3：理论上可能存了非数组（比如以前代码写错了） `data = { seenJokes: null } `或` { seenJokes: "字符串" }` 或 `{ seenJokes: 123 }`

然后执行
```js
const list = data[key] || [];
```
这行代码的语义是：
- 先看 `data[key]` 是什么
- 如果`data[key]` 是**真值**（数组、非空字符串、非零数字、对象等），就用它
- 如果 `data[key]`是**假值**（undefined、null、false、0、""、NaN），就用` []`

**为什么需要这一行**？
如果直接写：
```JavaScript
const list = data[key];   // ← 危险写法
```

那么当存储里没有这个 key 时：
```JavaScript
list → undefined
```

后面再执行：
```JavaScript
list.includes(item.id)   // → TypeError: Cannot read properties of undefined (reading 'includes')
list.push(item.id)       // → TypeError: list.push is not a function
```

程序就会直接崩溃。

而加了 ||` []` 之后，无论存储里有没有数据，list 永远是一个数组，后续的 .includes()、.push() 都能安全执行。

---
## 计算属性名
```js
async function markAsSeen (item) {
    if (!item || !item.id) return;

    const key = item.id.startsWith("j_") ? "seenJokes" : "seenMedia";
    const data = await chrome.storage.local.get(key);
    const list = data[key] || []; 
  
    if(!list.includes(item.id)) {
        list.push(item.id);
        await chrome.storage.local.set({ [key]: list});// +++p3
    }
}
```
```js
await chrome.storage.local.set({ [key]: list});
``` 
这里的 `[key] `是 JavaScript 里计算属性名（computed property name） 的写法。
简单说： 当对象的属性名不是写死（literal）的，而是来自一个变量时，就必须用` [ ] `来包住这个变量。

```js
let key = "seenJokes";

// 错误写法（大多数人第一次会这样写）
chrome.storage.local.set({ key: list })  
// 结果存成了： { "key": [...] }   ← 属性名真的是字符串 "key"，而不是 "seenJokes"

// 正确写法
chrome.storage.local.set({ [key]: list })  
// 结果存成了： { "seenJokes": [...] }   ← 属性名才是我们想要的 "seenJokes"
```


---
## 访问对象属性的常见方式

**1. 点表示法 (Dot Notation): `obj.key`**
当你使用 `obj.key` 时，JavaScript 会直接去对象中寻找名字**刚好叫做 "key"** 的那个属性。

- **特点：** 它后面的词必须是一个有效的 JavaScript 标识符（不能以数字开头，不能包含空格，不能是连字符 `-` 等特殊字符）。
    
- **不支持变量：** 它**不会**把 `key` 当作一个变量去计算它的值。当你使用 `obj.key` 时，JavaScript 会直接去对象中寻找名字**刚好叫做 "key"** 的那个属性。

示例：
```js
const obj = {
  name: "张三",
  key: "这是一个叫key的属性"
};

let myVariable = "name";

console.log(obj.key);        // 输出: "这是一个叫key的属性" (直接找字面名字)
console.log(obj.myVariable); // 输出: undefined (对象中没有叫 "myVariable" 的属性)
```

**2. 括号表示法 (Bracket Notation): `obj[key]`**
当你使用 `obj[key]` 时，JavaScript 会**先计算括号里面表达式的值**，然后把这个值转换成字符串，再去对象里找对应的属性。

- **特点：** 括号里可以放任何表达式，最常见的就是放一个**变量**。
    
- **支持特殊字符：** 如果你的属性名包含空格、特殊符号，或者以数字开头，你只能通过字符串加括号的方式来访问。

示例：
```js
const obj = {
  name: "张三",
  "home address": "北京", // 属性名有空格
  123: "数字属性"
};

let key = "name";

// 1. 使用变量
console.log(obj[key]);    // 输出: "张三" (因为 key 的值是 "name"，相当于 obj["name"])

// 2. 访问包含特殊字符的属性
// console.log(obj.home address); // 报错！语法错误
console.log(obj["home address"]); // 输出: "北京"

// 3. 访问数字开头的属性
// console.log(obj.123); // 报错！
console.log(obj[123]);   // 输出: "数字属性" (123 会被自动转为字符串 "123")
```
**总结**
如果属性名是写死的，用 `.`；如果属性名存在变量里或者需要动态计算，用 `[]`。

---

## new Set()
#p4
```js
const data = await chrome.storage.local.get(["seenJokes", "seenMedia"]);
const seenJokes = new Set(data.seenJokes || []);
const seenMedia = new Set(data.seenMedia || []); // +++p4
```
**new Set() 是什么？**
new Set() 是 JavaScript 内置的一个**集合（Set）** 类型。

它和数组（Array）很像，但有几个非常关键的区别，正是因为这些区别，在“记录已经看过的 id”这种场景下，**Set 比数组更合适、更高效**。

**Set 和 Array 的核心区别**

|特性|Array（数组）|Set（集合）|对“已看 id 列表”这个场景的影响|
|---|---|---|---|
|是否允许重复元素|允许重复|**不允许重复**（自动去重）|Set 天然防止重复添加同一个 id|
|检查某个元素是否存在|includes() → O(n) 线性查找|has() → O(1) 接近常数时间|Set 查询速度快很多|
|添加元素|push()|add()|—|
|删除元素|splice() 或 filter()|delete()|Set 删除更简单|
|典型内存占用|一般|通常略高一些（但差别不大）|几千条 id 完全无压力|
|是否有索引/顺序保证|有序、可通过下标访问|**插入顺序**（现代浏览器保证）但无索引|你不需要按顺序遍历时无所谓|

---

## 代码解析
#p5

```js
async function getRandomFavorite() {
    const data = await chrome.storage.local.get(["favoriteJokes", "favoriteMedia"]);
    const favJokes = data.favoriteJokes || [];
    const favMedia = data.favoriteMedia || [];

    const allFavorites = [
        ...favJokes.map(id => jokes.find(j => j.id === id)).filter(Boolean), //+++p5
        ...favMedia.map(id => mediaItems.find(m => m.id === id)).filter(Boolean)
    ];

    if (allFavorites.length === 0) {
        return null; // 还没有收藏
    }

    return randomFrom(allFavorites);

}
```

```js
export const jokes = [
    { id: "j_60311_001", text: "程序员最害怕的两个字是——“重构”" },
    { id: "j_60311_002", text: "为什么程序员喜欢黑暗模式？因为灯亮了工资就没了" },
    { id: "j_60311_003", text: "我老婆让我别买游戏机，我说这是投资——投资我开心" },
    { id: "j_60311_004", text: "前端和后端分手了，因为后端总说：你样式我不管" },
    { id: "j_60311_005", text: "程序员谈恋爱就像debug：到处都是bug，还不让说话" },
    { id: "j_60311_006", text: "代码写得再好，也不如领导一句话：这个需求改一下" },
];
```
```js
favJokes.map(id => jokes.find(j => j.id === id)).filter(Boolean)
```
**`.map()`**
用一个函数，把数组 A 映射成数组 B。
```js
const arr = [1, 2, 3];
const result = arr.map(x => x * 2);
console.log(result); //[2, 4, 6]
```
所以，`favJokes.map(id => jokes.find(j => j.id === id))`是对数组favJokes中的每一个元素（这里就是id）进行`jokes.find(j => j.id === id)`操作。

**`.find()`**
在数组中找到第一个满足条件的元素
```js
const arr = [5, 12, 8, 130, 44];  
const result = arr.find(x => x > 10);  
console.log(result); //12
```

所以`jokes.find(j => j.id === id)`就是在jokes数组中找到一个元素j，j满足j的id等于前面的id。返回的是一个类似`{ id: "j_60311_001", text: "程序员最害怕的两个字是——“重构”" },`的对象。也可能返回`id = "j_999" → jokes.find(...) → undefined ← 因为数据里没有这条`

**`.filter(Boolean)`**
它会把数组中所有 falsy 值移除，只保留 truthy 值。

在 JavaScript 中，以下值在布尔上下文中会被视为“假”（falsy）：
- false
- 0
- -0
- 0n (BigInt 零)
- "" (空字符串)
- null
- undefined
- NaN

```js
const arr = [0, 1, "", "hello", null, undefined, " ", false, NaN, {name:"tom"}];
const result = arr.filter(Boolean);
// 结果：
//[1, "hello", " ", {name:"tom"}]
```
本质是
```js
.filter(item => Boolean(item))
```
的简写

等价于
```js
.filter(item => item !== undefined && item !== null && item !== 0 && item !== "" && item !== false && !Number.isNaN(item) && ... )
```

所以`.filter(Boolean)`会把`jokes.find(j => j.id === id)`返回的undefined移除，只保留有效数据

---

## 可选链运算符
#p6
```js
if(!item?.id) return;
```
```js
item?.id
```
等价于
```js
item == null ? undefined : item.id
```
也就是说：

- 如果 `item` 是 `null` 或 `undefined` → 返回 `undefined`
- 如果 `item` 有值 → 正常访问 `item.id`


---
## 变量重新赋值
#p7

```js
const list = data[key] || [];
list.push(item.id);
```
这个不是重新赋值，只是修改了list的内部内容
```js
let list = data[key] || []; 
list = list.filter(id => id !== item.id);
```
是重新赋值，因为后面是`list = ....`