
# remove和remove()
#bb1
```js
if (window.HappyFishing && window.HappyFishing.remove) {
	window.HappyFishing.remove(); 
}
```
#cgs1
```js
const removeGame = () => {
	console.log('HappyFishing 已关闭');
};
```
#cgs2
```js
window.HappyFishing = {
	spawn: (pos) => {
		localStorage.setItem(STORAGE_KEY.position, JSON.stringify(pos));
		initGame(pos); // 本页立即生效
	},
	remove: removeGame,
	reset: removeGame

};
```
**`remove` 是什么？**
它是对象的一个属性，但这个属性的值是一个函数。

**代码结构：**
1. **定义函数：** 你定义了一个箭头函数 `removeGame`。
    ```js
    const removeGame = () => { ... } // 这是一张“施工图纸”
    ```
2. **定义对象：** 你创建了一个对象 `window.HappyFishing`。
3. **赋值：** 你把 `removeGame` 这张“图纸”放进了对象的 `remove` 属性里。
    ```js
    window.HappyFishing = {
        // 属性名: 属性值 (是个函数)
        remove: removeGame
    };
    ```
在 JavaScript 中，当一个对象的**属性**是**函数**时，我们通常称之为**方法 (Method)**。所以，`remove` 既是属性，也是方法。

**为什么要加 `()`？**
这就像 **“菜谱”** 和 **“做菜”** 的区别。
- **`window.HappyFishing.remove` (不加括号)**
    - **含义：** 指的是**函数本身**（也就是 `removeGame` 的代码逻辑）。
    - **比喻：** 这是一个**菜谱**。你手里拿着菜谱，你可以把它复印给别人，也可以读它，但此时并没有饭菜被做出来。
    - **在代码中的效果：** 没有任何事情发生。游戏不会关闭，`console.log` 不会打印。浏览器只是确认了一下：“哦，这里有个函数。”
- **`window.HappyFishing.remove()` (加括号)**
	- **含义：** **执行**这个函数。
	- **比喻：** 这是按照菜谱**做菜**。只有当你开始做（加括号）时，切菜、开火（销毁游戏、清除缓存）这些动作才会真正发生。
	- **在代码中的效果：** 浏览器跳转到 `removeGame` 内部，逐行执行代码

**我怎么知道什么时候加，什么时候不加？**
这是一个非常实用的判断标准：

**情况 A**：你需要事情**立刻发生** -> **加 `()`**
如果你现在的目的是“赶紧把这个功能给我跑起来”，你就必须加括号。
- **场景：** 按钮点击后的逻辑、像你代码里那样在 `executeScript` 里直接执行命令。
- **代码：**
    ```js
    // 我现在就要关闭游戏！
    window.HappyFishing.remove();
    ```
    
**情况 B**：你只是想**传递/介绍**这个功能，稍后再用 -> **不加 `()`**
如果你是想把这个函数交给别人（比如浏览器、定时器、或者另一个函数），告诉他们：“嘿，以后有机会的时候，请帮我运行这个东西。”
- **场景 1：绑定点击事件**
    ```js
    // 错误：button.onclick = remove();
    // (这会立即执行删除，而不是等点击时才删除，且onclick绑定的是执行后的结果undefined)
    
    // 正确：
    button.onclick = window.HappyFishing.remove;
    // 意思是：当按钮被点击时，请去查阅 remove 这个菜谱并照做。
    ```
- **场景 2：定时器**
    ```js
    // 3秒后执行
    setTimeout(window.HappyFishing.remove, 3000);
    ```

**总结**
- **`remove`** = **名词**。它是指那个函数。
- **`remove()`** = **动词**。它是运行那个函数。

