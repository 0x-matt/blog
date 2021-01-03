---
layout: post
title: 声明式和命令式编程
date: 2019-12-28
tags: ["paradigm","日志","programming","函数式","前端知识","命令式","声明式","客户端开发知识","编程范式","计算机基础知识"]
categories:
- 计算机
---

作者：xiaobo

* * *

![paradigm](paradigm.png "paradigm")

最近，我才知道原来计算机基础科学中，比较强调一个词，叫编程范式 (programming paradigm)。并且，到目前为止，我们常用的编程范式有四种：**声明式、命令式、函数式、和面向对象；其中，函数式被认为是声明式的子集**。本节，记录下我对声明式和命令式的理解。其实这两种方式，我们开发中都用到过。

### 声明式 declarative

> 通过描述事情的状态和结果，来表示计算的逻辑，不关心具体的逻辑细节和流程控制。
> 
>   关键词是**做什么(what do)**，而不关心**怎样做(how do)**

### 命令式 imperative

> 通过使用更改程序状态的语句，一步步得到结果的编程方式。
> 
>   关键词是怎样做 **(how do)**，强调实现的过程。

## 语法层面的区别

* * *

大部分情况下 HTML、CSS，都是**声明式**的写法，更典型的**声明式**编程是 SQL 语句：

      SELECT score FROM games WHERE id < 100;

上述 SQL 语句的特点是，只描述清楚想要的结果，并不操纵过程。

这种声明式的编程方式，也正在被越来越多的现代编程语言借鉴，比如 swift ：

    var list = [1, 3, 10, 4, 5];
    // 找到数组中的第一个偶数
    var result = list.first(where: {(num) -> Bool in num % 2 == 0})

语句中的 `first` 和 `where` 关键字，就是声明式编程的典型特征。

上边的实现也可以用**命令式**来写:

    var result = 0;
    for num in list {
        if num % 2 == 0 {
            result = num;
            break;
        }
    }

`if` `for` 都是典型的更改程序执行状态的语言，所以可以简单的认为 `if` `for` `while` ... 就是命令式编程的标志。

## 框架层面的区别

* * *

除了语法层面的区别，很多框架都逐渐采用了声明式的编程方式，典型的如 React Flutter，下面就是一段声明式的 React 代码：

    class Button extends React.Component{
      this.state = { color: 'red' }
      handleChange = () => {
        const color = this.state.color === 'red' ? 'blue' : 'red';
        this.setState({ color });
      }
      render() {
        return (<div>
          <button 
             className=`btn ${this.state.color}`
             onClick={this.handleChange}>
          </button>
        </div>);
      }
    }

这段代码中，通过点击按钮，来改变自身的颜色。同样的逻辑，我们可以用命令式来实现：

    const container = document.getElementById('container');
    const btn = document.createElement('button');
    btn.className = 'btn red';
    btn.onclick = function(event) {
     if (this.classList.contains('red')) {
       this.classList.remove('red');
       this.classList.add('blue');
     } else {
       this.classList.remove('blue');
       this.classList.add('red');
     }
    };
    container.appendChild(btn);

这两段，代码最大的区别是，声明式的写法，不会直接操纵 DOM，而是声明了一个状态；然后我们应该只关心组件的状态，而不是思考完整的实现过程，通过不同的状态值，来得到不同的结果。

flutter 中也使用了类似的声明式设计，所以，我可能考虑用不同的思考方式去学习 flutter 了。

## 参考链接

* * *

1.  https://stackoverflow.com/questions/1784664/what-is-the-difference-between-declarative-and-imperative-programming
2.  https://codeburst.io/declarative-vs-imperative-programming-a8a7c93d9ad2

(全文完)