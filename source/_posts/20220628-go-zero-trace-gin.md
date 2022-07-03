---
title: go-zero gin jaeger trace
date: 2022-06-28 09:59:23
tags: [golang, jaeger, go-zero]
categories:
top_img:

---

## 0x00

最近玩玩链路追踪

<!--more-->

## 0x01 

- 什么是链路追踪？

> 链路追踪 Tracing Analysis 为分布式应用的开发者提供了完整的调用链路还原、调用请求量统计、链路拓扑、应用依赖分析等工具，可以帮助开发者快速分析和诊断分布式应用架构下的性能瓶颈，提高微服务时代下的开发诊断效率。

通俗一点讲就是多个服务之间调用可以串成一条链，例如 A 调用 B，B 调用 C

- 为什么需要链路追踪？

> 为了应对各种复杂的业务，开发工程师开始采用敏捷开发、持续集成等开发方式。系统架构也从单机大型软件演化成微服务架构。微服务构建在不同的软件集上，这些软件模块可能是由不同团队开发的，可能使用不同的编程语言来实现，还可能发布在多台服务器上。因此，如果一个服务出现问题，可能导致几十个应用都出现服务异常。

通俗一点讲还是多个服务之间调用，假设某个环节出现了问题，可能要查询哪个服务出现问题，一个一个排查成本过高也不科学，还有就是可以通过链路追踪查询出各个服务在一条链路中的请求响应耗时，从而进行一些针对性的优化等。

- 什么是调用链(Trace)？

> 在广义上，一个调用链代表一个事务或者流程在（分布式）系统中的执行过程。在 OpenTracing 标准中，调用链是多个 Span 组成的一个有向无环图（Directed Acyclic Graph，简称DAG），每一个 Span 代表调用链中被命名并计时的连续性执行片段。

可以大概理解为整条链路只有一个 traceId，但是可能会有多个 span，服务间的调用会产生对应的 span。

- OpenTelemetry

> OpenTelemetry, also known as OTel for short, is a vendor-neutral open-source [Observability](https://opentelemetry.io/docs/concepts/observability-primer/#what-is-observability) framework for instrumenting, generating, collecting, and exporting telemetry data such as [traces](https://opentelemetry.io/docs/concepts/observability-primer/#distributed-traces), [metrics](https://opentelemetry.io/docs/concepts/observability-primer/#reliability--metrics), [logs](https://opentelemetry.io/docs/concepts/observability-primer/#logs). As an industry-standard it is [natively supported by a number of vendors](https://opentelemetry.io/vendors).

otel 作为一个可观测性框架，主要可用于 traces(链路追踪)，metrics(指标)，logs(日志) 等数据的检测，生成，采集和导出。

概念性的东西可能比较多， 后面会贴上参考资料

[opentelemetry 可观测性框架](https://opentelemetry.io/)

[聊聊分布式链路追踪和可观察性](https://stanxing.hedwig.pub/i/liao-liao-fen-bu-shi-lian-lu-zhui-zong-he-ke-guan-cha-xing)

[aliyun tracing analysis](https://help.aliyun.com/document_detail/196637.html)

---

- 实战部分

1、[Jaeger](https://www.jaegertracing.io/): open source, end-to-end distributed tracing

为什么不使用 zipkin? 通常来讲 java 系 zipkin 用的多，go 项目一般用 jaeger

测试使用，直接 all-in-one

```go
$ docker run -d --name jaeger \
  -e COLLECTOR_ZIPKIN_HOST_PORT=:9411 \
  -e COLLECTOR_OTLP_ENABLED=true \
  -p 6831:6831/udp \
  -p 6832:6832/udp \
  -p 5778:5778 \
  -p 16686:16686 \
  -p 4317:4317 \
  -p 4318:4318 \
  -p 14250:14250 \
  -p 14268:14268 \
  -p 14269:14269 \
  -p 9411:9411 \
  jaegertracing/all-in-one:1.35
```

2、trace 接入

因为项目使用了 go-zero，go-zero 又直接集成了 opentelemetry，pr 见 [feat: opentelemetry integration, removed self designed tracing](https://github.com/zeromicro/go-zero/pull/1111)

所以理论上只需要进行相关配置即可使用链路追踪

不过凡事都有但是，目前只有 rpc 服务使用了 go-zero 的 zrpc，http web framework 部分使用了 gin，没有使用 go-zero 的 httpx

所以需要手动编写对应的 gin middleware 进行 trace ctx 的生成和 response header 的注入

```go
import (
	"github.com/gin-gonic/gin"
	"github.com/zeromicro/go-zero/core/trace"
	"go.opentelemetry.io/otel"
	"go.opentelemetry.io/otel/propagation"
	semconv "go.opentelemetry.io/otel/semconv/v1.4.0"
	oteltrace "go.opentelemetry.io/otel/trace"
)

// TracingHandler return a middleware that process the opentelemetry.
func TracingHandler(serviceName string) gin.HandlerFunc {
	return func(ginCtx *gin.Context) {
		propagator := otel.GetTextMapPropagator()
		tracer := otel.Tracer(trace.TraceName) // 影响的是 trace 中的 otel.library.name 字段

		ctx := propagator.Extract(ginCtx.Request.Context(), propagation.HeaderCarrier(ginCtx.Request.Header))
		spanName := ginCtx.Request.URL.Path
		//logx.Info("spanName is", spanName)
		spanCtx, span := tracer.Start(
			ctx,
			spanName,
			oteltrace.WithSpanKind(oteltrace.SpanKindServer),
			oteltrace.WithAttributes(semconv.HTTPServerAttributesFromHTTPRequest(
				serviceName, spanName, ginCtx.Request)...),
		)
		defer span.End()

		// convenient for tracking error messages 注入返回 response key: Traceparent
		propagator.Inject(spanCtx, propagation.HeaderCarrier(ginCtx.Writer.Header()))

		// 注入 context
		ginCtx.Request = ginCtx.Request.WithContext(spanCtx)

		ginCtx.Next()

		return
	}
}
```

这里其实有一个点，`ginCtx.Request = ginCtx.Request.WithContext(spanCtx)` 通过此方式才可以进行对应 context 的注入

gin.Context 是一个 struct 而不是 context.Context 所以导致有一些问题解决比较麻烦。

```yaml
// 服务端配置文件
Telemetry:
  Name: admin
  Endpoint: http://localhost:14268/api/traces
  Sampler: 1.0
  Batcher: jaeger
```

3、效果

![](https://blog-1253523830.cosgz.myqcloud.com/assets/img/202207031536610.png)

- 顺便讲讲 go-zero 升级

以上讲的示例是我个人的项目，工作上的项目因为之前 fork 并进行了修改，所以需要手动 merge 并重新打 tag 发布

还有一个比较麻烦的问题是 go module 名称的替换，原先 go-zero `github.com/tal-tech/go-zero` 后来变成了 `github.com/zeromicro/go-zero` ，官方 goctl 有提供相应 migrate 命令，不过我并没有进行使用，全局匹配 go 文件并进行相应替换。

还遇到一个有趣的问题，可见此 [issue](https://github.com/zeromicro/go-zero/issues/1344)

```sh
........\pkg\mod\k8s.io\klog\v2@v2.4.0\klog.go:702:10: invalid operation: logr != nil (mismatched types logr.Logger and nil)
........\pkg\mod\k8s.io\klog\v2@v2.4.0\klog.go:721:10: invalid operation: logr != nil (mismatched types logr.Logger and nil)
........\pkg\mod\k8s.io\klog\v2@v2.4.0\klog.go:739:10: invalid operation: logr != nil (mismatched types logr.Logger and nil)
........\pkg\mod\k8s.io\klog\v2@v2.4.0\klog.go:760:10: invalid operation: logr != nil (mismatched types logr.Logger and nil)
........\pkg\mod\k8s.io\klog\v2@v2.4.0\klog.go:779:11: invalid operation: loggr != nil (mismatched types logr.Logger and nil)
........\pkg\mod\k8s.io\klog\v2@v2.4.0\klog.go:791:11: invalid operation: loggr != nil (mismatched types logr.Logger and nil)
........\pkg\mod\k8s.io\klog\v2@v2.4.0\klog.go:915:9: invalid operation: log != nil (mismatched types logr.Logger and nil)
........\pkg\mod\k8s.io\klog\v2@v2.4.0\klog.go:1277:18: invalid operation: logging.logr == nil (mismatched types logr.Logger and nil)
........\pkg\mod\k8s.io\klog\v2@v2.4.0\klog.go:1278:21: cannot use nil as type logr.Logger in field value
```

解决方案也很简单，直接升级即可 `go get -v k8s.io/klog/v2`

接下来需要更新 goctl，不然生成的代码 import 的还是 `tal-tech` 的 module name

PS: 暂时没有将 demo 整理出来，后续考虑整理并开源一下，目前网上暂时没有看到 gin 结合 go-zero 上报 trace 相关。

## 0x02

半年就要过去了，那么加油。