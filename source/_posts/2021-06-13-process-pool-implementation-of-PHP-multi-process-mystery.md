---
cover: https://blog-1256184194.file.myqcloud.com/2021/06/13/052cb621cffde.jpg
copyright: true
title: PHP多进程奥秘之进程池实现
date: 2021-06-13 13:30
tags:
  - php
categories:
  - 分享
---

# PHP多进程奥秘之进程池实现

> 前几天公司分享会，大佬twosee讲了一节《PHP手写多进程服务》，自己也动手实现一下，哈哈哈

## 进程，啥是进程
简单点就是，正在进行的一个过程或者说一个任务。而负责执行该任务的是cpu。

![Snipaste_2021-06-13_11-08-52.png](https://blog-1256184194.file.myqcloud.com/2021/06/13/e77a60fd74ddc.png)

CPU同一时间只能干一件事，但是我们观察到的现象是，多个程序可以同时运行。因为我们的操作系统帮我们设计了一个牛逼的任务调度，采用时间片轮转的抢占式调度方式，正常来说，CPU一个内核一个时间只能干一件事，通过时间片的方式，无感切换执行。

![Snipaste_2021-06-13_11-13-26.png](https://blog-1256184194.file.myqcloud.com/2021/06/13/5144ce992b89a.png)

## 不同系统下的多进程编程
每个语言的多进程编程，底层其实都是调用操作系统提供的相关api

### Unix

![Snipaste_2021-06-13_11-17-06.png](https://blog-1256184194.file.myqcloud.com/2021/06/13/9e923f5b10d84.png)

### 跨平台

![Snipaste_2021-06-13_11-17-15.png](https://blog-1256184194.file.myqcloud.com/2021/06/13/5e4016fa80bbd.png)

## PHP实现
主要都是通过pcntl函数来进行多进程编程

### pcntl_fork

- 在当前进程当前位置产生分支（子进程）
- fork是创建了一个子进程，父进程和子进程都从fork的位置开始向下继续执行，不同的是父进程执行过程中，得到的fork返回值为子进程 号，而子进程得到的是0
- 子进程与父进程共享程序正文段
- 子进程拥有父进程的数据空间和堆、栈的副本，注意是副本，不是共享
- fork之后，是父进程先执行还是子进程先执行无法确认，取决于系统调度

> PS：这里说子进程拥有父进程数据空间以及堆、栈的副本，实际上，在大多数的实现中也并不是真正的完全副本。更多是采用了COW（Copy On Write）即写时复制的技术来节约存储空间。简单来说，如果父进程和子进程都不修改这些 数据、堆、栈 的话，那么父进程和子进程则是暂时共享同一份 数据、堆、栈。只有当父进程或者子进程试图对 数据、堆、栈 进行修改的时候，才会产生复制操作，这就叫做写时复制。

#### 程序从pcntl_fork()后父进程和子进程将各自继续往下执行代码
```php
<?php /** @noinspection ALL */

$parentPid = getmypid();
echo '目前父进程的pid为：' . $parentPid . PHP_EOL;

$pid = pcntl_fork();

// pcntl_fork()后父进程和子进程将各自继续往下执行代码
// 上面只fork了一次，但是下面的代码会执行两次，一次在父进程里，一次在子进程里
if ($pid > 0) {
    echo '我创建了一个子进程：' . $pid . PHP_EOL;
} else if (0 === $pid) {
    echo '我是新创建的子进程' . PHP_EOL;
} else {
    echo 'fork失败' . PHP_EOL;
}
```
执行结果可见，**执行了两次fork之后的代码，fork之前的只有一次**

![Snipaste_2021-06-13_11-25-08.png](https://blog-1256184194.file.myqcloud.com/2021/06/13/bc7b679248056.png)

#### 子进程拥有父进程的数据副本，而并不是共享
```
<?php /** @noinspection ALL */

$parentPid = getmypid();
echo '目前父进程的pid为：' . $parentPid . PHP_EOL;

// 初始化一个number变量 1
$number = 1;
$pid = pcntl_fork();

// pcntl_fork()后父进程和子进程将各自继续往下执行代码
// 上面只fork了一次，但是下面的代码会执行两次，一次在父进程里，一次在子进程里
if ($pid > 0) {
    $number += 1;
    echo '我创建了一个子进程：' . $pid . PHP_EOL;
    echo 'number+1 :' . $number . PHP_EOL;
} else if (0 === $pid) {
    $number += 2;
    echo '我是新创建的子进程' . PHP_EOL;
    echo 'number+2 :' . $number . PHP_EOL;
} else {
    echo 'fork失败' . PHP_EOL;
}
```

执行结果可见，**number在进程里执行时，数值的初始值都为1**

![Snipaste_2021-06-13_11-29-47.png](https://blog-1256184194.file.myqcloud.com/2021/06/13/1eb474da9f3fd.png)

#### 循环创建子进程会发生什么
```
<?php /** @noinspection ALL */

for ($i = 1; $i <= 3; $i++) {
    $parentPid = getmypid();
    echo $i.'目前父进程的pid为：' . $parentPid . PHP_EOL;
    $pid = pcntl_fork();
    if ($pid > 0) {
        echo $i.'我创建了一个子进程：' . $pid . PHP_EOL;
    } else if (0 === $pid) {
        echo $i.'我是新创建的子进程' . PHP_EOL;
    } else {
        echo $i.'fork失败' . PHP_EOL;
    }
}
```
执行结果

![Snipaste_2021-06-13_12-06-52.png](https://blog-1256184194.file.myqcloud.com/2021/06/13/d3d9c90a3b943.png)

这样不好看，我调整一下顺序
```
1目前父进程的pid为：2685
1我创建了一个子进程：2686
1我是新创建的子进程

2目前父进程的pid为：2685
2我创建了一个子进程：2687
2我是新创建的子进程

2目前父进程的pid为：2686
2我创建了一个子进程：2690
2我是新创建的子进程

3目前父进程的pid为：2685
3我创建了一个子进程：2688
3我是新创建的子进程

3目前父进程的pid为：2687
3我创建了一个子进程：2689
3我是新创建的子进程

3目前父进程的pid为：2686
3我创建了一个子进程：2691
3我是新创建的子进程

3目前父进程的pid为：2690
3我创建了一个子进程：2692
3我是新创建的子进程
```
![创建子进程流程图.png](https://blog-1256184194.file.myqcloud.com/2021/06/13/4020dda8e8cdd.png)

可以看到，创建了7个子进程，再次验证了pcntl_fork()后父进程和子进程将各自继续往下执行代码，也就是循环调用，子进程如果没有提前结束的话，会不断增加，形成僵尸进程

循环3次，创建了1+2+4个子进程，循环4次，创建了1+2+4+8个子进程，即1+2+4+···+2^(n-1)

## 孤儿进程和僵尸进程

### 孤儿进程

> 父进程在子进程结束之前提前退出，这些子进程将由init（进程ID为1）进程收养并完成对其各种数据状态的收集。因为子进程从此变得无依无靠、无家可归，变成了孤儿。

```php
<?php /** @noinspection ALL */

$pid = pcntl_fork();
if ($pid > 0) {
    // 显示父进程的进程ID，这个函数可以是getmypid()，也可以用posix_getpid()
    echo "Father PID:" . getmypid() . PHP_EOL;
    // 让父进程停止两秒钟，在这两秒内，子进程的父进程ID还是这个父进程
    sleep(2);
} else if (0 == $pid) {
    // 让子进程循环10次，每次睡眠1s，然后每秒钟获取一次子进程的父进程进程ID
    for ($i = 1; $i <= 10; $i++) {
        sleep(1);
        // posix_getppid()函数的作用就是获取当前进程的父进程进程ID
        echo posix_getppid() . PHP_EOL;
    }
} else {
    echo "fork error." . PHP_EOL;
}
```
这样的话，前2秒，父进程还在，后面父进程就结束了，那么子进程都会被变成孤儿进程然后被systemd收起

![Snipaste_2021-06-13_12-28-26.png](https://blog-1256184194.file.myqcloud.com/2021/06/13/ec97b8954df74.png)

可以看到后面的pid都变成1

### 僵尸进程

> 僵尸进程是指父进程在fork出子进程，而后子进程在结束后，父进程并没有调用wait或者waitpid等完成对其清理善后工作，导致改子进程进程ID、文件描述符等依然保留在系统中，极大浪费了系统资源。所以，僵尸进程是对系统有危害的，而孤儿进程则相对来说没那么严重。

刚刚我们前面演示的代码，就会造成僵尸进程，查一下看看

![Snipaste_2021-06-13_12-21-18.png](https://blog-1256184194.file.myqcloud.com/2021/06/13/4b0da5a258a7e.png)

可以看到有非常多状态为 z或者Z 的进程为僵尸进程

### pcntl_wait和pcntl_waitpid

#### pcntl_wait

> 这个函数的作用就是 “ 等待或者返回子进程的状态 ”，当父进程执行了该函数后，就会阻塞挂起等待子进程的状态一直等到子进程已经由于某种原因退出或者终止。换句话说就是如果子进程还没结束，那么父进程就会一直等等等，如果子进程已经结束，那么父进程就会立刻得到子进程状态。这个函数返回退出的子进程的进程ID或者失败返回-1。

```php
<?php /** @noinspection ALL */

$pid = pcntl_fork();
if ($pid > 0) {
    echo '父进程id：' . getmypid() . PHP_EOL;

    // 返回$wait_result，就是子进程的进程号，如果子进程已经是僵尸进程则为0
    // 子进程状态则保存在了$status参数中，可以通过pcntl_wexitstatus()等一系列函数来查看$status的状态信息是什么
    $wait_result = pcntl_wait($status);
    echo '回收子进程id：' . $wait_result . PHP_EOL;

    // 让主进程休息60秒钟
    sleep(60);
} else if (0 == $pid) {
    echo '子进程id：' . getmypid() . PHP_EOL;
    // 让子进程休息10秒钟，但是进程结束后，父进程不对子进程做任何处理工作，这样这个子进程就会变成僵尸进程
    sleep(10);
} else {
    exit('fork error.' . PHP_EOL);
}
```

![Snipaste_2021-06-13_12-42-41.png](https://blog-1256184194.file.myqcloud.com/2021/06/13/bd9d20ca1a467.png)

执行结果，可以看出，父进程会一直等待子进程结束（阻塞），结束后会进程回收，就避免了僵尸进程的产生

#### pcntl_waitpid

> pcntl_waitpid( $pid, &$status, $option = 0 )的第三个参数如果设置为WNOHANG，那么父进程不会阻塞一直等待到有子进程退出或终止，否则将会和pcntl_wait()的表现类似。

```php
<?php /** @noinspection ALL */

$pid = pcntl_fork();
if ($pid > 0) {
    echo '父进程id：' . getmypid() . PHP_EOL;

    // 返回值保存在$wait_result中
    // $pid参数表示 子进程的进程ID
    // 子进程状态则保存在了参数$status中
    // 将第三个option参数设置为常量WNOHANG，则可以避免主进程阻塞挂起，此处父进程将立即返回继续往下执行剩下的代码
    $wait_result = pcntl_waitpid($pid, $status, WNOHANG);
    echo '回收子进程id：' . $wait_result . PHP_EOL;

    echo "不阻塞，运行到这里".PHP_EOL;
    // 让主进程休息60秒钟
    sleep(60);
} else if (0 == $pid) {
    echo '子进程id：' . getmypid() . PHP_EOL;
    // 让子进程休息10秒钟，但是进程结束后，父进程不对子进程做任何处理工作，这样这个子进程就会变成僵尸进程
    sleep(10);
} else {
    exit('fork error.' . PHP_EOL);
}
```
执行结果，可以看出，父进程并没有等待子进程的状态，避免了非阻塞挂起

![Snipaste_2021-06-13_12-49-17.png](https://blog-1256184194.file.myqcloud.com/2021/06/13/8813a8ebcd819.png)

### 简单的进程池示例

```
<?php /** @noinspection ALL */

define('PROCESS_COUNT', 4);

$pidMap = [];

while (true) {
    $pid = pcntl_fork();
    if ($pid < 0) {
        echo "[Parent] Fork failed" . PHP_EOL;
        exit(1);
    } elseif ($pid > 0) {
        // 记录创建的子进程数
        $pidMap[$pid] = true;
        echo "[Parent] New Process {$pid}" . PHP_EOL;
        if (count($pidMap) === PROCESS_COUNT) {
            // 如果进程数达到最大值，进行子进程等等，并回收
            $pid = pcntl_wait($status);
            echo sprintf("[Parent] Process %s exit with status %d, signal=%d" . PHP_EOL, $pid, pcntl_wexitstatus($status), pcntl_wtermsig($status));
            unset($pidMap[$pid]);
        }
    } else {
        echo sprintf("[Child %d] running" . PHP_EOL, getmypid());
        sleep(mt_rand(1, 30));
        echo sprintf("[Child %d] exit(%d)" . PHP_EOL, getmygid(), $exStatus = mt_rand(0, 255));
        exit($exStatus);
    }
}
```

### 完整的进程池示例

```
<?php /** @noinspection ALL */

class Process
{
    protected $pid;

    public function __construct(callable $callable, string $name = null)
    {
        $pid = pcntl_fork();
        if ($pid < 0) {
            throw new Exception(pcntl_strerror(pcntl_get_last_error(), pcntl_get_last_error()));
        } else {
            if ($pid > 0) {
                echo "[Parent] New Process {$pid}" . PHP_EOL;
                $this->pid = $pid;
            } else {
                if ($name) {
                }
                $this->pid = getmypid();
                echo sprintf("[Child %d] running" . PHP_EOL, $this->pid);
                $exitStatus = $callable();
                echo sprintf("[Child %d] exit(%d)" . PHP_EOL, $this->pid, $exitStatus);
                exit($exitStatus);
            }
        }
    }

    public function getPid(): int
    {
        return $this->pid;
    }

    public function kill(int $signal = SIGTERM): void
    {
        if (!posix_kill($this->pid, $signal)) {
            throw new Exception(sprintf("Kill(%d, %d) fail, reason: %s"), $this->getPid(), $signal, posix_strerror(posix_get_last_error()));
        }
    }

    public static function setName(string $name)
    {
        if (function_exists('cli_set_process_title') && PHP_OS_FAMILY !== 'Darwin') {
            cli_set_process_title($name);
        }
    }
}

class ProcessPool
{
    protected $pool = [];
    protected $idMap = [];
    protected $callable;
    protected $count;

    public function __construct(callable $callable, int $count)
    {
        $this->callable = $callable;
        $this->count = $count;
        Process::setName('Parent');
    }

    public function run()
    {
        for ($id = 0; $id < $this->count; $id++) {
            $this->createProcess($id);
        }

        while (true) {
            $pid = pcntl_wait($status);
            echo sprintf("[Parent] Process %s exit with status %d, signal=%d" . PHP_EOL, $pid, pcntl_wexitstatus($status), pcntl_wtermsig($status));
            unset($this->pool[$pid]);
            $id = $this->idMap[$pid];
            unset($this->idMap[$pid]);
            $this->createProcess($id);
        }
    }

    protected function createProcess(int $id)
    {
        $process = new Process($this->callable, "Child-{$id}");
        $this->pool[$process->getPid()] = $process;
        $this->idMap[$process->getPid()] = $id;
    }
}

$processPool = new ProcessPool(function () {
    sleep(mt_rand(1, 30));
}, 4);
$processPool->run();
```

## 参考代码
https://github.com/lihq1403/gadget/tree/master/codeSnippet/pcntl

