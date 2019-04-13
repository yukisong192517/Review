# 常见的数据结构

## 时间复杂度

* 用来衡量算法的好坏
* 常见的时间复杂度：
    * O（1）操作与数据量没关系
    * O（aN + b）： n代表数据量，这种通常被记为O（n），数据量通常比较大，因此低阶
    项和常数项可以忽略不计，对于两个同时为O（n）复杂度的算法，就需要比较系数和常数了
    * O（nlogn）等

## 栈

* 特点：FILO,只能在某一端添加或者删除数据
* 实现
```
Class Stack{
    constructor(){
    this.stack = []
    }
    // 增
    push(item){
    this.stack.push(item);
    }
    // 删
    pop(){
    this.stack.pop();
    }
    // 返回栈顶元素却不删除
    peek(){
    return this.stack[this.getCount()-1]
    }
    // 长度
    getCount(){
    return this.stack.length
    }
    // 是否为空
    isEmpty(){
    return this.getCount() === 0
    }
}
```
* 应用：括号的匹配
[leetcode #20](https://leetcode.com/problems/valid-parentheses/submissions/)

```
/**
 * @param {string} s
 * @return {boolean}
 */
var isValid = function(s) {
    let map = {
        '(':-1,
        '[':-2,
        '{': -3,
        ')':1,
        ']':2,
        '}':3,
    }
    let stack = [];
    for(let i = 0; i< s.length;i++){
        if(map[s[i]]<0){
            stack.push(s[i])
        }
        else {
            let last = stack.pop();
            if(map[last]+map[s[i]]!==0) return false
        }
    }
    if(stack.length > 0 ){
        return false
    }
    return true
};
```

## 队列
* 特点： FIFO, 在一端添加数据，在另一侧删除数据
* 单链队列的实现：
```
class Queue{
    constructor(){
    this.queue = [];
    }
    enQueue(item){
    this.queue.push(item);
    }
    deQueue(){
    return this.queue.shift();
    }
    getHeader(){
    return this.queue[0]
    }
    getLength(){
    return this.queue.length
    }
    isEmpty(){
    return this.getLength === 0
    }
}
```
* 循环队列：
将向量空间想象为一个首尾相接的圆环，并称这种向量为循环向量。存储在其中的队列称为循环队列（Circular Queue）。循环队列中进行出队、入队操作时，头尾指针仍要加1，朝前移动。只不过当头尾指针指向向量上界（QueueSize-1）时，其加1操作的结果是指向向量的下界0。

## 链表

* 对于节点：值和指向下一个节点的指针。
* 优缺点：
    * 优点： 充分利用计算机存储空间，实现灵活的内存管理
    * 缺点： 无法随机读取，开销较大。
* 实现： singleLinkList.js

## 二叉树

* 二叉树有一个根节点，每个节点至多拥有两个子节点，左节点和右节点，树的最底部节点成为叶节点。当一棵树的叶数量数量为满，则称之为满二叉树。
* 二分搜索树：
每个节点的值比左子树的值要大，比右子树要小
