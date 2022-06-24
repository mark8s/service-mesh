# opentracing

## The OpenTracing Data Model
OpenTracing 中的跟踪由它们的 Span 隐式定义。特别是，可以将 Trace 视为 Spans 的有向无环图（DAG），其中 Spans 之间的边称为 References。

例如，以下是由 8 个 Span 组成的示例 Trace：

单个 Trace 中 Span 之间的因果关系
```shell
 [Span A]  ←←←(the root span)
            |
     +------+------+
     |             |
 [Span B]      [Span C] ←←←(Span C is a `ChildOf` Span A)
     |             |
 [Span D]      +---+-------+
               |           |
           [Span E]    [Span F] >>> [Span G] >>> [Span H]
                                       ↑
                                       ↑
                                       ↑
                         (Span G `FollowsFrom` Span F)
```

有时使用时间轴更容易可视化跟踪，如下图所示：

单个 Trace 中 Span 之间的时间关系

```shell
––|–––––––|–––––––|–––––––|–––––––|–––––––|–––––––|–––––––|–> time

 [Span A···················································]
   [Span B··············································]
      [Span D··········································]
    [Span C········································]
         [Span E·······]        [Span F··] [Span G··] [Span H··]
```

## Span
Span 是追踪链路中的基本组成元素，，使用 spanId 作为唯一标识.一个 Span 表示一个独立的工作单元，在链路追踪中可以表示一个接口的调用，一个数据库操作的调用等等。

一个 Span 中包含如下内容：
- 服务名称 (operation name，必选)
- 服务开始时间（必选）
- 服务的结束时间（必选）
- Tags：K/V 形式
- Logs：K/V 形式
- SpanContext
- Refrences：该 span 对一个或多个 span 的引用（通过引用 SpanContext）

### Tag & Log
Tag: 每个 Span 可以有多个键值 K/V 对形式的 Tags，Tags 是没有时间戳的，支持简单的对 Span 进行注解和补充。
Tags 是一个 K/V 类型的键值对，用户可以自定义该标签并保存。主要用于链路追踪结果对查询过滤。
如某 Span 是调用 Redis ，而可以设置 redis 的标签，这样通过搜索 redis 关键字，可以查询出所有相关的 Span 以及 trace；
又如 http.method="GET",http.status_code=200，其中 key 值必须为字符串，value 必须是字符串，布尔型或者数值型。
Span 中的 Tag 仅自己可见，不会随着 SpanContext 传递给后续 Span。

```shell
span.SetTag("http.method","GET")
span.SetTag("http.status_code",200)
```

Logs: Logs 也是一个 K/V 类型的键值对，与 Tags 不同的是，Logs 还会记录写入 Logs 的时间，因此 Logs 主要用于记录某些事件发生的时间。
```shell
span.LogFields(
	log.String("database","mysql"),
	log.Int("used_time":5),
	log.Int("start_ts":1596335100),
)
```

我们如何决定这些数据应该进入跨度Tags还是跨度Logs？ 
OpenTracing API 并没有规定我们如何做；一般原则是，适用于整个跨度的信息应记录为标签，而具有时间戳的事件应记录为日志。

### SpanContext（核心字段）
每个 Span 必须提供方法访问 SpanContext，SpanContext 代表跨越进程边界（在不同的 Span 中传递信息），传递到下级 Span 的状态。SpanContext 携带着一些用于跨服务通信的（跨进程）数据，主要包含：

- 该 Span 的唯一标识信息，如：span_id、trace_id、parent_id 或 sampled 等
- Baggage Items，为整条追踪连保存跨服务（跨进程）的 K/V 格式的用户自定义数据

### Trace
Trace 表示一次完整的追踪链路，trace 由一个或多个 Span 组成。它表示从头到尾的一个请求的调用链，它的标识符是 traceID。 下图示例表示了一个由 8 个 Span 组成的 trace:
```shell
[Span A]  ←←←(the root span)
            |
     +------+------+
     |             |
 [Span B]      [Span C] ←←←(Span C is a `ChildOf` Span A)
     |             |
 [Span D]      +---+-------+
               |           |
           [Span E]    [Span F] >>> [Span G] >>> [Span H]
                                       ↑
                                       ↑
                                       ↑
                         (Span G `FollowsFrom` Span F)
```

以时间轴的展现方式如下：
```shell
––|–––––––|–––––––|–––––––|–––––––|–––––––|–––––––|–––––––|–> time

 [Span A···················································]
   [Span B··············································]
      [Span D··········································]
    [Span C········································]
         [Span E·······]        [Span F··] [Span G··] [Span H··]
```

## OpenTracing API 的路径
在 OpenTracing API 中，有三个主要对象：

- Tracer
- Span
- SpanContext Tracer 可以创建 Spans 并了解如何跨流程边界对它们的元数据进行 Inject（序列化）和 Extract（反序列化），通常的有如下流程：
- 开始一个新的 Span
- Inject 一个 SpanContext 到一个载体
- 从载体 Extract 一个 SpanContext
- 由起点进程创建一个 Tracer，然后启动进程发起请求，每个动作产生一个 Span，如果有父子关系，Tracer 将它们关联
- 当请求 / Span 完成后，Tracer 将跟踪信息推送到 Collector

## Reference
[Take OpenTracing for a HotROD ride](https://medium.com/opentracing/take-opentracing-for-a-hotrod-ride-f6e3141f7941)

[在 gRPC 服务中应用 OpenTracing：使用 Zipkin 和 Jaeger 进行路追踪](https://pandaychen.github.io/2020/06/01/GOLANG-TRACING-WITH-ZIPKIN/)












