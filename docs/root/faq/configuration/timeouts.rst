.. _faq_configuration_timeouts:

如何配置超时?
============================

Envoy 支持多种超时，用户可根据部署情况进行相应的配置。本页总结了在各种场景中使用的最重要的超时。

.. attention::

   本文并未列出所有 Envoy 支持的超时配置。根据部署情况，可能需要额外的配置。

HTTP/gRPC
---------

连接超时
^^^^^^^^^^^^^^^^^^^

连接超时适用于整个 HTTP 连接和连接携带的所有流。

* HTTP 协议中的 :ref:`idle timeout <envoy_v3_api_field_config.core.v3.HttpProtocolOptions.idle_timeout>` 是在 HTTP 连接管理器和上游集群 HTTP 连接使用的通用消息中定义的。空闲超时是在没有活动流的情况下，终止下游或上游连接的时间。如果没有指定，默认空闲超时时间为 1 小时。要修改下游连接的空闲超时时间，请使用 HTTP 连接管理器配置中的 :ref:`common_http_protocol_options
  <envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.common_http_protocol_options>` 字段。要修改上游连接的空闲超时时间，请使用集群配置中的 :ref:`common_http_protocol_options <envoy_v3_api_field_config.cluster.v3.Cluster.common_http_protocol_options>` 字段。

流超时
^^^^^^^^^^^^^^^

流超时适用于 HTTP 连接携带的单个流。请注意，流是 HTTP/2 和 HTTP/3 的概念，然而 Envoy 在内部将 HTTP/1 请求映射到流，因此在这种情况下，请求/流是可以互换的。

* HTTP 连接管理器中的 :ref:`请求超时 <envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.request_timeout>` 是连接管理器允许从客户端接收 *整个请求流* 的时间。

  .. attention::

    默认情况下，请求超时不是必须配置的，因为它与流请求（请求永不结束）不兼容。请查阅随后的流空闲超时。然而，如果使用 :ref:`缓冲区过滤器 <config_http_filters_buffer>`，建议配置请求超时。

* HTTP 连接管理器中的 :ref:`流空闲超时 <envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.stream_idle_timeout>` 是连接管理器允许流在没有下游或上游活动时，所能存在的时间。默认流空闲超时时间为 5 分钟。强烈建议对所有请求（不仅仅是流请求/响应）配置流空闲超时，因为它可以防止一旦将整个响应缓存发送给下游客户端，使用 HTTP/2 的客户端不会关闭流窗口。


* HTTP 协议中的 :ref:`最大流持续时间 <envoy_v3_api_field_config.core.v3.HttpProtocolOptions.max_stream_duration>` 是在 HTTP 连接管理器的通用消息中定义的。最大流持续时间是一个流的生命周期的最大时间。当需要定期重置 HTTP 请求/响应流时，可以使用此功能。在这种情况下，你不能使用 :ref:`请求超时<envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.request_timeout>` ，因为如果在请求/响应流中收到响应头，定时器将会被重置。该超时在上游和下游连接上都可用。

路由超时
^^^^^^^^^^^^^^

Envoy 另外还支持路由级别的流超时，可以覆盖上面已经介绍的一些流超时。

* 路由 :ref:`超时 <envoy_v3_api_field_config.route.v3.RouteAction.timeout>` 是 Envoy 等待上游完成响应的时间。*此超时在接收到整个下游请求流之前不会启动*。

 .. attention::

    该超时默认为 15 秒，然而它与流响应（响应永不结束）不兼容，需要禁用。流空闲超时应当在流 APIs 中使用，如本页在其他位置所描述的一样。

* 路由 :ref:`空闲超时 <envoy_v3_api_field_config.route.v3.RouteAction.idle_timeout>` 允许覆盖 HTTP 连接管理器中的 :ref:`流空闲超时 <envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.stream_idle_timeout>` ，并执行相同的动作。
* 在使用重试时，可以配置路由 :ref:`每次超时尝试 <envoy_v3_api_field_config.route.v3.RetryPolicy.per_try_timeout>` ，使得每次请求使用的超时时间比前面描述的总体请求超时时间更短。该超时仅在响应的任何部分发送到下游之前才适用，通常发生在上游发送响应头之后。如果上游在超时时间内无法响应，则可以使用该超时来重试流端点。
* 路由 :ref:`MaxStreamDuration proto <envoy_v3_api_msg_config.route.v3.RouteAction.MaxStreamDuration>` 可以用来覆盖 HTTP 连接管理器中单个路由的 :ref:`最大流持续时间 <envoy_v3_api_field_config.core.v3.HttpProtocolOptions.max_stream_duration>` ，也可以在 grpc-timeout 报头上设置限制和固定的时间偏移量。

TCP
---

* 集群 :ref:`连接超时 <envoy_v3_api_field_config.cluster.v3.Cluster.connect_timeout>` 指的是 Envoy 等待上游建立 TCP 连接的时间。该超时没有默认值，但是是必须配置的。

  .. attention::

     对于 TLS 连接，连接超时包括 TLS 握手。

* TCP 代理 :ref:`空闲超时 <envoy_v3_api_field_extensions.filters.network.tcp_proxy.v3.TcpProxy.idle_timeout>` 指的是 TCP 代理允许连接在没有下游或上游活动时，所能存在的时间。如果没有指定，默认的空闲超时时间是 1 小时。
