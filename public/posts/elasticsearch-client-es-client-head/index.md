# 简单好用的ElasticSearch可视化工具：es-Client和Head


&lt;!--more--&gt;

## 一、前言
- 使用 ElasticSearch(简称 es) 的过程中，经常有一些临时查询(如 排查问题、验证效果)，一个趁手的可视化工具 可以提高工作效率。
- 个人倾向于 免费(最好开源)、易于安装(如 浏览器插件)，`es-client` 就是 比较简单好用的一个，尤其是 查询。

## 二、[Kibana](https://www.elastic.co/cn/downloads/past-releases#kibana)
- es官方的可视化工具，天花板级别，当然也复杂一些，如要配置 Index Patterns 才能查询
- [7.11版本开始 需考虑许可证问题，也应该也是 阿里云es默认 7.10版本的原因吧](https://www.oschina.net/news/201376)，Kibana 提供给别人(如 公司的同事)使用收费
- [Kibana开源版](https://www.elastic.co/cn/downloads/past-releases#kibana-oss) 没有 性能分析工具 Search Profiler、Grok Debugger 等

## 三、Head 系列
1. [Head](https://github.com/mobz/elasticsearch-head)
- 多年前 刚接触es时，用的工具，浏览器插件 方式安装，简单方便，**只能保存 1个集群的连接信息**
- 集群、索引方面的功能可以，**数据浏览、基本查询 功能偏弱**
- 顶部 可以直观的看到 集群健康值，并以 颜色标识
- 主要分为：概览、索引、数据浏览、基本查询、符合查询，以及右上角的 信息

- 概览：页面是横向布局，可以直观的看到 集群节点列表，哪个是 主节点(最左侧 五角星标识)，索引的 分片、副本 分布在哪些节点
- 索引：列表，包含 名称、别名、创建时间、大小、文档数量、分片数、副本数。**以前就根据 大小 清理过数据，可惜不支持排序**
- 数据浏览：只能根据 索引、类型 筛选数据，**不支持自定义条件，且 不能翻页，最多显示 50条数据**
- **基本查询：还是不能翻页，可以选择显示 10、50、250、1000、5000、25000 条。索引、字段 下拉框 不支持 输入筛选，不太方便。查询条件不能 临时禁用，只能删除**
- 复合查询：竟然还要输入 集群地址

2. [Multi Elasticsearch Head](https://github.com/vorapoap/elasticsearch-head-chrome)
- 看名字就知道，是支持保存 多个集群连接信息的 Head 了
- 字体可能偏小，可以改插件的 css样式调整
- es有密码的情况下，每次重启浏览器以后，重新连接都需要输入 用户名、密码  
![Multi-Elasticsearch-Head-overview.png](https://img.890808.xyz/file/javalover123/2023/08/Multi-Elasticsearch-Head-overview.png)

## 四、[es-client](https://gitee.com/qiaoshengda/es-client)
- 数据浏览、基础查询 功能好用，开源免费，作者响应也比较及时
- 有 浏览器、utools、vscode、IDEA 插件版本，还有 windows安装包
- 支持保存 多个集群连接信息，重启浏览器重新连接 也不用输入 用户名、密码，更方便了
- 主要分为：概览、数据浏览、基础搜索、高级搜索、设置，以及右上角的 信息

- 概览  
支持 索引名称、状态 筛选，按 名称、大小、文档数量 正序、倒序 排列，**排查大索引 更方便了呀**

- 数据浏览  
**输入类似SQL的 查询条件、排序，有时候更高效。** 还用 `_id=null` 排查过数据同步的问题。  
![es-client-setting-browser.png](https://img.890808.xyz/file/javalover123/2023/08/es-client-setting-browser.png)

- 基础搜索  
**查询条件、排序 支持禁用，便于调整。**  
![es-client-setting-basic-search.png](https://img.890808.xyz/file/javalover123/2023/08/es-client-setting-basic-search.png)

- 高级搜索  
**注意：输入请求内容，才显示 执行 按钮**  
![es-client-setting-adv-search.png](https://img.890808.xyz/file/javalover123/2023/08/es-client-setting-adv-search.png)

- 设置  
支持 排除指定索引，显示指定索引，[本人贡献的PR](https://gitee.com/qiaoshengda/es-client/pulls/2)，索引比较多 而 关注的索引不多时，可以大幅降低 干扰  
![es-client-setting-basic.png](https://img.890808.xyz/file/javalover123/2023/08/es-client-setting-basic.png)

## 五、总结
- `es-client`、Head 更适合个人使用，其中 `es-client` 在 数据浏览、基础搜索、索引过滤 3方面明显更优，Head 在 集群健康度、索引分片副本分布 显示方面更好
- Kibana 更适合企业级使用，功能多，使用门槛高也一些。如配置好 时间字段，可以方便的 使用日期选择器筛选，还有 性能分析工具 Search Profiler、Grok Debugger 等，但是 7.11版本开始 需考虑许可证问题


![Gopher毛](https://blog.huchao.vip/picx-images-hosting/blog/%E5%85%AC%E4%BC%97%E5%8F%B7.58h8ppzfbd.webp)


---

> 作者: [胡超](https://github.com/mao888)  
> URL: http://localhost:1313/posts/elasticsearch-client-es-client-head/  
> 转载 URL: https://www.890808.xyz/elasticsearch-client-es-client-head/
