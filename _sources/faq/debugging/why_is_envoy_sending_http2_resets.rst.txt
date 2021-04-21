.. _why_is_envoy_sending_http2_resets:

为什么 Envoy 要发送 HTTP/2 重置？
===================================

HTTP/2 重置路径主要是由 Envoy 封装 HTTP/2、nghttp2 的编解码器来控制。
nghttp2 严格遵守 HTTP/2 规范，但是由于许多客户端不完全兼容，这种不完全匹配会导致意外重置。
不幸的是，与调试 :ref:`内部响应路径 <why_is_envoy_sending_internal_responses>` 不同，Envoy 在 nghttp2 中重置流的具体原因可见度有限。

如果你有一个可再现的失败案例，你可以用 "-l trace" 参数，在 debug 版本的 Envoy 中运行它，以便将获得详细的 nghttp2 错误日志，这些日志通常会指出哪些头信息未通过合规检查。
或者，如果你可以在一台遇到异常的机器上用 "-l trace" 参数运行，那么你可以从文件 "source/common/http/http2/codec_impl.cc" 中寻找格式为 `invalid http2: [nghttp2 error detail]` 的日志。
例如：`invalid http2: Invalid HTTP header field was received: frame type: 1, stream: 1, name: [content-length], value: [3]`

另外你也可以检查 :ref:`HTTP/2 统计数据`<config_http_conn_man_stats_per_codec>`: 在许多情况下，Envoy 会重置流。
例如，如果标头超过了配置所允许的范围，或者洪泛检测启动，则在重置流时， http2 计数器将增加。


