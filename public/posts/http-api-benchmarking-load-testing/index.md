# HTTP接口性能压力测试


&lt;!--more--&gt;

## 一、前言
- 开发接口以后，对性能有要求的 接口，需要做 性能压力测试
- 常见免费的如：经典的 ab，性能不太好的 jmeter、siege(有时候都怀疑程序性能不行了)，另介绍 hey、k6、vegeta、wrk

## 二、方案
### 1. [ab - Apache HTTP server benchmarking tool](https://httpd.apache.org/docs/2.4/programs/ab.html)
- C语言开发，适用于 Linux 平台
- 优劣：可能因为在 WSL里面运行，性能表现不太好，容易报错，apr_pollset_poll: The timeout specified has expired (70007)
- 参数：c 并发数、n 总请求数、k 表示 开启 Keep Alive 特性、r 表示 忽略 socket receive errors
- `ab -c 100 -k -r -n 10000 http://www.a.com/a`  
![http-bench-ab.png](https://img.890808.xyz/file/javalover123/2023/07/http-bench-ab.png)

### 2. [hey: HTTP load generator, ab replacement](https://github.com/rakyll/hey)
- GO语言开发，适用于 Linux、Mac、Windows 平台
- 性能高，跨平台，报表显示慢请求原因，最近发版是 2020年
- 参数：c 并发数、z 测试时长
- `hey -c 50 -z 5s http://www.a.com/a`  
![http-bench-hey.png](https://img.890808.xyz/file/javalover123/2023/07/http-bench-hey.png)

### 3. [jmeter](https://github.com/apache/jmeter)
- Java语言开发，适用于 多 平台
- 优劣：性能较差，跨平台  

### 4. [k6：load testing tool, using Go and JS](https://github.com/grafana/k6)
- GO语言开发，适用于 Linux、Mac、Windows 平台
- 优劣：性能较高，跨平台，支持 [请求](https://k6.io/docs/using-k6/checks/)、[统计结果](https://k6.io/docs/using-k6/thresholds/) 校验，非常适合开发人员(复制略作调整即可)做接口自动化测试
- 参数：u 并发数、d 测试时长
- 需用 JavaScript 脚本 定义测试内容(如保存为 k6.js)，`k6 run -u 200 -d 10s k6.js`  
```javascript
import http from &#34;k6/http&#34;;
import { check, sleep } from &#34;k6&#34;;
// Test configuration
export const options = {
};

export default function () {
  let res = http.get(&#34;url&#34;);
  // check(res, { &#34;status was 200&#34;: (r) =&gt; r.status == 200 });
}
```
  
![http-bench-k6.png](https://img.890808.xyz/file/javalover123/2023/07/http-bench-k6.png)

### 5. [siege](https://github.com/JoeDog/siege)
- C语言开发，适用于 Linux 平台
- 优劣：性能较低，不跨平台，报表没有显示慢请求原因
- 参数：c 并发数、t 测试时长(末尾单位必须大写)、b 表示 压测模式，请求不延迟(BENCHMARK: no delays between requests.)
- `siege -c 200 -t 10S -b  http://www.a.com/a`  
![http-bench-siege.png](https://img.890808.xyz/file/javalover123/2023/07/http-bench-siege.png)

### 6. [vegeta](https://github.com/tsenart/vegeta)
- GO语言开发，适用于 Linux、Mac、Windows 等5平台
- 优劣：性能高，跨平台，报表没有显示慢请求原因
- 参数：rate  指定并发，默认每秒 50个请求，0 表示不限制(用于测试接口极限性能，需和 max-workers 参数一起使用)
- 另 workers 参数 指定 初始 workers 数量，默认为 10，设置和 max-workers 相等可避免测试过程中创建连接耗时
- `echo &#34;GET http://www.a.com/a&#34; | vegeta attack -rate 0 -workers 200 -max-workers 200 -duration 10s | vegeta report`  
![http-bench-vegeta.png](https://img.890808.xyz/file/javalover123/2023/07/http-bench-vegeta.png)

### 7. [wrk](https://github.com/wg/wrk)
- C语言开发，适用于 Linux 平台
- 优劣：性能超高，不跨平台，最近发版是 2021年2月，另WSL里面运行卡住停不下来
- t 线程数(不宜过大，避免太多上下文切换，CPU核心数 1到3倍左右)，c 连接数，d 测试时长(末尾 s 表示秒)，latency 输出延迟统计
- `wrk -t12 -c100 -d10s --latency http://www.a.com/a`  

## 三、总结
[Open source load testing tool review 2020 (k6.io)](https://k6.io/blog/comparing-best-open-source-load-testing-tools/#end-summary)
| 工具                                        | RPS   | 开发语言 | 支持平台 | 备注                                                            |
|:------------------------------------------- |:----- | --- |:-------- |:--------------------------------------------------------------- |
| [ab](https://httpd.apache.org/docs/2.4/programs/ab.html) | 1929  |   C  | Linux    | 可能因为在 WSL里面运行，性能不太好，容易报错                    |
| [hey](https://github.com/rakyll/hey)        | 12000 |  GO   | 3平台    | 性能高，跨平台，报表显示慢请求原因，最近发版是2020年            |
| [jmeter](https://github.com/apache/jmeter)  | - |  Java  | 多平台    | 性能低，跨平台    |
| [k6](https://github.com/grafana/k6)         | 10000 |  GO  | 3平台    | 性能较高，跨平台，支持 请求、统计结果 校验，更适合自动化测试    |
| [siege](https://github.com/JoeDog/siege)    | 2253  |  C   | Linux    | 性能低，不跨平台，报表没有显示慢请求原因                        |
| [vegeta](https://github.com/tsenart/vegeta) | 10400 |  GO   | 5平台    | 性能高，跨平台，报表没有显示慢请求原因                          |
| [wrk](https://github.com/wg/wrk)            | -     |   C  | Linux    | 性能超高，不跨平台，最近发版是 2021年2月，另WSL里面运行没有效果 |

**本文遵守[【CC BY-NC】协议，转载请保留原文出处及本版权声明，否则将追究法律责任。](https://creativecommons.org/licenses/by-nc/4.0/)**   
***本文首先发布于 [https://www.890808.xyz/](https://www.890808.xyz/) ，其他平台需要审核更新慢一些。***   

![javalover123](https://img.890808.xyz/file/javalover123/2023/04/688b88cfd4ed9f6fcd56828b849ce47c.jpg)

---

> 作者: [胡超](https://github.com/mao888)  
> URL: http://localhost:1313/posts/http-api-benchmarking-load-testing/  
> 转载 URL: https://www.890808.xyz/http-api-benchmarking-load-testing/
