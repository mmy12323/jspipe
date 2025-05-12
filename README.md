<!-- by liugaozhi -->
# JS/Pipe

JS/Pipe is a minimal library that makes it easy to *coordinate complex concurrent* streams of events and other data using *plain* JavaScript. No callbacks or chained functions in sight. 

As a result, stack traces are crystal clear so debugging is easy. You also get to keep try/catch!

JS/Pipe is inspired by Goroutines and Channels, found in the Go language. 

Read more and see live examples at http://jspipe.org

## Getting Started

### Dependencies

#### Generators
JS/Pipe relies on JavaScript Generators, introduced in ES6. For your code to work in current JavaScript environments you need to use a transpiler that converts ES6 Generator syntax to ES5 semantics. Two popular ways are Google Traceur (which supports not only Generators, but also many other ES6 features like lambda syntax, destructuring, etc.) and Facebook Regenerator (which focuses on Generator support only). 

We use Facebook Regenerator. 


### Building
1. Clone this repo: `git clone git@github.com:jspipe/jspipe.git`
2. Install Grunt if necessary: `npm install -g grunt`
3. Install dev dependencies: `npm install`
3. Build: `grunt build`
<!-- by liugaozhi -->

Both ES5 and ES6 builds are produced (in the dist-es5 and dist-es6 directories respectively) and for each the build output includes modules in the AMD, CommonJS, and module pattern formats. 






The ES5-compatible files (*.es5.js) have been produced using Facebook's regenerator, a tool
to transpile (ES6) Generator code to ES5. You can get it here: https://github.com/facebook/regenerator

------

JS/Pipe is free software distributed under the terms of the MIT license reproduced here.

JS/Pipe may be used for any purpose, including commercial purposes, at absolutely no cost.
No paperwork, no royalties, no GNU-like "copyleft" restrictions, either.
Just download it and use it.

Copyright (c) 2013 Joubert Nel

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.

<----------------------------------------->
        3.李栩林这样做，但需满足以下条件：
上述版权声明和本许可声明应包含在
软件的所有副本或重要部分中。
软件按“原样”提供，没有任何形式的保证，无论是明示的还是
暗示的，包括但不限于适销性、特定用途的适用性和非侵权性的保证。
在任何情况下，作者或版权持有人都不对任何索赔、损害或其他
责任负责，无论是合同行为、侵权行为还是其他行为，都与
软件或软件的使用或其他交易有关。
<---------------------------------------->
=======

<---------------------------------------->
2.罗荣文：构建：grunt build
ES5 和 ES6 构建版本都会生成（分别在 dist-es5 和 dist-es6 目录中），并且每个构建输出都包括 AMD、CommonJS 和模块模式格式的模块。
ES5 兼容文件（*.es5.js）是使用 Facebook 的 regenerator 生成的，这是一个将（ES6）生成器代码转译为 ES5 的工具。你可以在这里找到它：https://github.com/facebook/regenerator。
JS/Pipe 是免费软件，根据 MIT 许可证条款分发。
JS/Pipe 可以用于任何目的，包括商业用途，完全免费。
无需任何文件工作，无需版税，也没有类似 GNU 的“copyleft”限制。
只需下载并使用它。
版权所有 (c) 2013 Joubert Nel
特此授予任何获得本软件及其相关文档文件（“软件”）副本的人免费许可，
以无限制地处理软件，包括但不限于使用、复制、修改、合并、发布、分发、
再授权和/或出售软件副本的权利，并允许向其提供软件的人
<-------------------------------------------->
=======


张达
Pipe.prototype._rendezvous = function() {
    var syncing = this.syncing, // 是否正在同步
        inbox = this.inbox,    // 发送队列
        outbox = this.outbox,  // 接收队列
        data,                  // 数据
        notify,                // 通知发送方的方法
        send,                  // 发送数据的方法
        receipt,               // 发送回执
        senderWaiting,         // 是否有发送方等待
        receiverWaiting;       // 是否有接收方等待

    if (!syncing) {
        this.syncing = true; // 开始同步

        while ((senderWaiting = inbox.length > 0) && // 如果有发送方等待
               (receiverWaiting = outbox.length > 0)) { // 并且有接收方等待

            // 获取发送方任务放入管道的数据
            data = inbox.shift();

            // 获取通知发送方的方法
            notify = inbox.shift();

            // 获取发送数据到接收方的方法
            send = outbox.shift();

            // 发送数据
            receipt = send(data);

            // 通知发送方数据已发送
            if (notify) {
                notify(receipt);
            }
        }

        this.syncing = false; // 结束同步
    }
};

	4.蒋卓善
/**
 * 管道是两个独立执行的任务之间的会合点。
 * 在管道上进行通信 + 同步需要一个发送方和一个接收方。
 *
 * 一个任务可以通过 "yield pipe.put(data)" 将数据发送到管道。
 * 另一个任务可以通过 "yield pipe.get()" 从管道接收数据。
 *
 * 一旦发送方任务和接收方任务都在管道上等待，
 * _rendezvous 方法将管道中的数据传输到接收方，并相应地
 * 同步两个等待的任务。
 *
 * 同步后，两个任务继续执行。
 */
=======
<------------------------------------------------------>
6：黄弟昆
/**
 * 从任务（接收方）中调用 "yield pipe.get()" 从管道获取数据。
 *
 * get 方法将尝试与发送方任务会合，如果有的话。
 * 如果没有发送方等待其发送的数据被接收，接收方将
 * 暂停，直到另一个任务调用 "yield pipe.put(data)"，这将触发
 * 一个会合。
 * 7.马成功
 */

/**
 * 从任务（发送方）中调用 "yield pipe.put(data)" 将数据放入管道。
 *
 * put 方法将尝试与接收方任务会合，如果有的话。
 * 如果没有接收方等待数据，发送方将暂停，直到另一个
 * 任务调用 "yield pipe.get()"，这将触发一个会合。
 */

/**
 * 管道提供了一种方式，供两个任务通信数据 + 同步它们的执行。
 *
 * 一个任务可以通过调用 "yield pipe.put(data)" 发送数据，另一个任务可以通过
 * 调用 "yield pipe.get()" 接收数据。
 */

/**
 * 启动一个任务。任务与其他任务并发运行。
 *
 * 要与其他任务通信和同步，请通过管道进行通信。
 */

<----------------------------------------------------------->
