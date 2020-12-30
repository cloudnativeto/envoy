.. _config_http_conn_man_details:

响应代码详细信息
=====================

如果通过 :ref:`访问日志<config_access_log_format_response_code_details>`，或
:ref:`自定义头部<config_http_conn_man_headers_custom_request_headers>` 配置了 _%RESPONSE_CODE_DETAILS_，
Envoy 将传达一个给定流结束的详细原因。此页列出了 HttpConnectionManager、路由器过滤器和编解码器发送的详细信息。
但这是不全面的，因为其他任何过滤器都可能会发送带有自定义详细信息的本地答复。

下面列出了 HttpConnectionManager 或路由器过滤器可能会发送的响应或重置流的原因。

.. warning::
  以下列表不能保证是稳定的，因为细节可能会发生变化。

.. csv-table::
   :header: 名称, 描述
   :widths: 1, 2

   absolute_path_rejected, 由于在不支持请求的路由上使用绝对路径，请求被拒绝。
   admin_filter_response, 响应由管理过滤器生成。
   cluster_not_found, 由于找不到所选路由的集群，路由器过滤器拒绝了请求。
   downstream_local_disconnect, 由于未指定的原因，客户端连接在本地关闭。
   downstream_remote_disconnect, 客户端意外断开连接。
   duration_timeout, 已经超过最大连接持续时间。
   direct_response, 路由器过滤器生成直接响应。
   filter_chain_not_found, 由于没有匹配的过滤器链，请求被拒绝。
   internal_redirect, 原始流被内部重定向替换。
   low_version, 由于没有配置对 HTTP/1.0 的支持，HTTP/1.0 或 HTTP/0.9 的请求被拒绝。
   maintenance_mode, 由于集群于维护模式，请求被路由器过滤器拒绝。
   max_duration_timeout, 已超过每个流的最大持续超时时间。
   missing_host_header, 由于缺少 Host: 或 :authority 字段，请求被拒绝。
   missing_path_rejected, 由于缺少路径或 :path 头字段，请求被拒绝。
   no_healthy_upstream, 由于找不到健康的上游，路由器过滤器拒绝了请求。
   overload, 由于过载管理器达到配置的资源限制，请求被拒绝。
   path_normalization_failed, 由于配置了路径规范化，但失败了，请求被拒绝，可能是因为路径无效。
   request_headers_failed_strict_check, 由于 x-envoy-* 头部验证失败，请求被拒绝。
   request_overall_timeout, 已超过每个流的总请求超时时间。
   request_payload_exceeded_retry_buffer_limit, Envoy 正在执行流代理，但在等待重试时，到达的数据太多。
   request_payload_too_large, Envoy 正在执行非流代理，请求负载超出了配置的限制。
   response_payload_too_large, Envoy 正在执行非流代理，响应的负载超出了配置的限制。
   route_configuration_not_found, 由于找不到路由配置，请求被拒绝。
   route_not_found, 由于找不到路由，请求被拒绝。
   stream_idle_timeout, 已超出每个流的 keepalive 超时时间。
   upgrade_failed, 由于请求一个受支持的升级，请求被拒绝。
   upstream_max_stream_duration_reached, 由于请求超出了配置的最大流持续时间，请求被销毁。
   upstream_per_try_timeout, 最后一次上游尝试失败。
   upstream_reset_after_response_started{details}, 响应启动后，上游连接被重置。可能包括有关断开原因的更多详细信息。
   upstream_reset_before_response_started{details}, 响应启动前，上游连接被重置。可能包括有关断开原因的更多详细信息。
   upstream_response_timeout, 上游响应超时。
   via_upstream, 响应代码由上游服务器设置。


.. _config_http_conn_man_details_per_codec:

每个编解码器的详细信息
-----------------------

遇到错误时，每个编解码器都可能会发送编解码器特定的详细信息。

Http1 详细信息
~~~~~~~~~~~~~~~~~~~~

All http1 details are rooted at *http1.*

.. csv-table::
   :header: Name, Description
   :widths: 1, 2

   http1.body_disallowed, A body was sent on a request where bodies are not allowed.
   http1.codec_error, Some error was encountered in the http_parser internals.
   http1.connection_header_rejected, The Connection header was malformed or overly long.
   http1.content_length_and_chunked_not_allowed, A request was sent with both Transfer-Encoding: chunked and a Content-Length header when disallowed by configuration.
   http1.content_length_not_allowed, A content length was sent on a response it was disallowed on.
   http1.headers_too_large, The overall byte size of rquest headers was larger than the configured limits.
   http1.invalid_characters, The headers contained illegal characters.
   http1.invalid_transfer_encoding, The Transfer-Encoding header was not valid.
   http1.invalid_url, The request URL was not valid.
   http1.too_many_headers, Too many headers were sent with this request.
   http1.transfer_encoding_not_allowed, A transfer encoding was sent on a response it was disallowed on.
   http1.unexpected_underscore, An underscore was sent in a header key when disallowed by configuration.


Http2 details
~~~~~~~~~~~~~

All http2 details are rooted at *http2.*

.. csv-table::
   :header: Name, Description
   :widths: 1, 2

    http2.inbound_empty_frames_flood, Envoy detected an inbound HTTP/2 frame flood.
    http2.invalid.header.field, One of the HTTP/2 headers was invalid
    http2.outbound_frames_flood, Envoy detected an HTTP/2 frame flood from the server.
    http2.too_many_headers, The number of headers (or trailers) exceeded the configured limits
    http2.unexpected_underscore, Envoy was configured to drop requests with header keys beginning with underscores.
    http2.unknown.nghttp2.error, An unknown error was encountered by nghttp2
    http2.violation.of.messaging.rule, The stream was in violation of a HTTP/2 messaging rule.
