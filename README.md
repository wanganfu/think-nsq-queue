# think-nsq-queue for ThinkPHP6

## 安装

> composer require annon/think-nsq-queue

## 配置

> 配置文件位于 `config/nsq.php`


## 创建任务类
> 推荐使用 `app\job` 作为任务类的命名空间
> 也可以放在任意可以自动加载到的地方

任务类不需继承任何类，如果这个类只有一个任务，那么就只需要提供一个`fire`方法就可以了，如果有多个小任务，就写多个方法，下面发布任务的时候会有区别  
每个方法会传入两个参数 `annon\queue\job\NSQJob $job`（当前的任务对象） 和 `$data`（发布任务时自定义的数据）

还有个可选的任务失败执行的方法 `failed` 传入的参数为`$data`（发布任务时自定义的数据）

### 下面写个例子

```php
namespace app\job;

use annon\queue\job\NSQJob;

class Job1 {
    public function fire(Job $job, $data) {
        
        //....这里执行具体的任务 
        
        if ($job->attempts() > 3) {
             //通过这个方法可以检查这个任务已经重试了几次了
        }
        
        //如果任务执行成功后 记得删除任务，不然这个任务会重复执行，直到达到最大重试次数后失败后，执行failed方法
        $job->delete();
        
        // 也可以重新发布这个任务
        $job->release($delay); //$delay为延迟时间
    }
    
    public function failed($data) {
        // ...任务达到最大重试次数后，失败了
    }
}

```

```php

namespace app\job;

use annon\queue\job\NSQJob;

class Job2 {
    public function task1(Job $job, $data) {
          
    }
    
    public function task2(Job $job, $data) {
         
    }
    
    public function failed($data) {
        
    }
}

```


## 发布任务
> `annon\queue\Queue::push($job, $data = '', $queue = null)`，发布后立即执行

> `annon\queue\Queue::later($delay, $job, $data = '', $queue = null)`，在`$delay`毫秒后执行

`$job` 是任务名  
命名空间是`app\job`的，比如上面的例子一,写`Job1`类名即可  
其他的需要些完整的类名，比如上面的例子二，需要写完整的类名`app\lib\job\Job2`  
如果一个任务类里有多个小任务的话，如上面的例子二，需要用@+方法名`app\lib\job\Job2@task1`、`app\lib\job\Job2@task2`

`$data` 是你要传到任务里的参数

`$queue` 队列名，指定这个任务是在哪个队列上执行，同下面监控队列的时候指定的队列名,可不填

## 监听任务并执行

```bash
php think nsq:listen
```

两种，具体的可选参数可以输入命令加 `--help` 查看

> 可配合supervisor使用，保证进程常驻
