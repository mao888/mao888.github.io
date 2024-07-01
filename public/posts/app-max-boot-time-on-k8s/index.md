# Kubernetes(k8s)最大启动时长研究


&lt;!--more--&gt;

## 一、前言
应用部署在 Kubernetes(k8s)上，有些应用启动慢一些，没启动好 就又被 k8s 重启了

## 二、处理过程
### 1. 看日志
```
[2023-05-23 14:38:52.249]|-INFO |-[background-preinit]|-o.h.v.i.u.Version[0]|-[TID: N/A]|-HV000001: Hibernate Validator 6.1.7.Final
[2023-05-23 14:40:11.817]|-INFO |-...

2023-05-23 14:40:22 登录主机: aaaa失败!
原因:Failed to upgrade to websocket: Unexpected HTTP Response Status Code: 500 Internal Server Error
```

### 2. 看探针配置
```yaml
      livenessProbe:
        failureThreshold: 3
        initialDelaySeconds: 30
        periodSeconds: 10
        successThreshold: 1
        tcpSocket:
          port: 60001
        timeoutSeconds: 1
```

### 3. 分析
- 刚开始以为 80秒左右(14:38:52.249 到 14:40:11.817)，应用被重启了
- 发现和 探针配置的不一样，initialDelaySeconds &#43; periodSeconds * failureThreshold = 60秒
- 然后发现最终结束时间应该是 14:40:22 登录主机: aaaa失败，就是 90秒左右
- 最后发现还有个 宽限时长 terminationGracePeriodSeconds: 30，加上探针 60秒，刚好 90秒左右。至此终于 水落石出
- 建议运维把 initialDelaySeconds 改为 60 以后，成功启动

## 三、总结
- 最长重启时间：initialDelaySeconds &#43; (periodSeconds &#43; timeoutSeconds) * failureThreshold &#43; terminationGracePeriodSeconds(默认30秒)
- 建议 适当调大 initialDelaySeconds(如 60)、failureThreshold(如 6)、periodSeconds(如 20)，总之根据上面的公式计算的时长 要大于 实际启动时长(如 本地测试)
- [配置存活、就绪和启动探针 | Kubernetes](https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
- [kubernetes的三种探针startupprobe,ReadinessProbe,LivenessProbe记录 - 陈雷雷 - 博客园 (cnblogs.com)](https://www.cnblogs.com/superlinux/p/14933961.html)


![Gopher毛](https://blog.huchao.vip/picx-images-hosting/blog/%E5%85%AC%E4%BC%97%E5%8F%B7.58h8ppzfbd.webp)


---

> 作者: [胡超](https://github.com/mao888)  
> URL: http://localhost:1313/posts/app-max-boot-time-on-k8s/  
> 转载 URL: https://www.890808.xyz/app-max-boot-time-on-k8s/
