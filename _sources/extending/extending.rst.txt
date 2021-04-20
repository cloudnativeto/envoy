.. _extending:

针对特定用例对 Envoy 进行扩展
====================================

Envoy 的架构使得它可以很容易地通过各种不同的扩展类型进行扩展，包括：

* :ref:`访问日志 <arch_overview_access_logs>`
* :ref:`访问日志过滤器 <arch_overview_access_log_filters>`
* :ref:`集群 <arch_overview_service_discovery>`
* :ref:`监听器过滤器 <arch_overview_listener_filters>`
* :ref:`网络过滤器 <arch_overview_network_filters>`
* :ref:`HTTP 过滤器 <arch_overview_http_filters>`
* :ref:`gRPC 凭据提供者 <arch_overview_grpc>`
* :ref:`健康检查器 <arch_overview_health_checking>`
* :ref:`资源监控 <arch_overview_overload_manager>`
* :ref:`重试实现 <arch_overview_http_routing_retry>`
* :ref:`统计 <arch_overview_statistics>`
* :ref:`追踪器 <arch_overview_tracing>`
* :ref:`请求 ID <arch_overview_tracing>`
* :ref:`传输 sockets <envoy_v3_api_field_extensions.transport_sockets.tls.v3.CommonTlsContext.CertificateProvider.typed_config>`
* BoringSSL 私有 key 方法
* :ref:`看门狗设置 <envoy_v3_api_msg_config.bootstrap.v3.Watchdog.WatchdogAction>`
* :ref:`内部重定向策略 <envoy_v3_api_field_config.route.v3.InternalRedirectPolicy.predicates>`
* :ref:`压缩库 <arch_overview_compression_libraries>`

在撰写本文时，还没有高级扩展开发人员文档。:repo:`已存扩展 <source/extensions>` 是一个很好的方式来了解什么是可能的。

有关如何添加网络筛选器、构造存储库和构建依赖项的示例，请参考 `envoy-filter-example <https://github.com/envoyproxy/envoy-filter-example>`_。
