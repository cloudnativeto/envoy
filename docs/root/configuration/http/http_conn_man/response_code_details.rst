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
   cluster_not_found, 路由器过滤器拒绝了请求，因为找不到所选路由的集群。
   downstream_local_disconnect, 客户端连接由于未指定的原因在本地关闭。
   downstream_remote_disconnect, 客户端意外断开连接。
   duration_timeout, 已经超过最大连接持续时间。
   direct_response, 路由器过滤器生成直接响应。
   filter_chain_not_found, 由于没有匹配的过滤器链，请求被拒绝。
   internal_redirect, 原始流被内部重定向替换。
   low_version, 由于没有配置对 HTTP/1.0 的支持，HTTP/1.0 或 HTTP/0.9 的请求被拒绝。
   maintenance_mode, 由于集群于维护模式，请求被路由器过滤器拒绝。
   max_duration_timeout, The per-stream max duration timeout was exceeded. 已超过每个流的最大持续超时时间。
   missing_host_header, The request was rejected due to a missing Host: or :authority field.
   missing_path_rejected, The request was rejected due to a missing Path or :path header field.
   no_healthy_upstream, The request was rejected by the router filter because there was no healthy upstream found.
   overload, The request was rejected due to the Overload Manager reaching configured resource limits.
   path_normalization_failed, "The request was rejected because path normalization was configured on and failed, probably due to an invalid path."
   request_headers_failed_strict_check, The request was rejected due to x-envoy-* headers failing strict header validation.
   request_overall_timeout, The per-stream total request timeout was exceeded.
   request_payload_exceeded_retry_buffer_limit, Envoy is doing streaming proxying but too much data arrived while waiting to attempt a retry.
   request_payload_too_large, Envoy is doing non-streaming proxying and the request payload exceeded configured limits.
   response_payload_too_large, Envoy is doing non-streaming proxying and the response payload exceeded configured limits.
   response_payload_too_large, Envoy is doing non-streaming proxying and the response payload exceeded configured limits.
   route_configuration_not_found, The request was rejected because there was no route configuration found.
   route_not_found, The request was rejected because there was no route found.
   stream_idle_timeout, The per-stream keepalive timeout was exceeded.
   upgrade_failed, The request was rejected because it attempted an unsupported upgrade.
   upstream_max_stream_duration_reached, The request was destroyed because of it exceeded the configured max stream duration.
   upstream_per_try_timeout, The final upstream try timed out.
   upstream_reset_after_response_started{details}, The upstream connection was reset after a response was started. This may include further details about the cause of the disconnect.
   upstream_reset_before_response_started{details}, The upstream connection was reset before a response was started This may include further details about the cause of the disconnect.
   upstream_response_timeout, The upstream response timed out.
   via_upstream, The response code was set by the upstream.


.. _config_http_conn_man_details_per_codec:

Per codec details
-----------------

Each codec may send codec-specific details when encountering errors.

Http1 details
~~~~~~~~~~~~~

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
