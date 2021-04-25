.. _faq_flow_control:

如何配置流量控制？
================================

当 Envoy 使用非流式传输的 L7 过滤器时，流量控制可能会导致许多问题，尤其是请求体或者响应体会超过 L7 缓存区限制。
对于请求体必须缓存并且超出配置限制的，Envoy 会给用户返回 413 并且增加 :ref:`downstream_rq_too_large <config_http_conn_man_stats>` 指标。
在响应路径中如果响应体必须缓存并超过限制，Envoy 会增加 :ref:`rs_too_large <config_http_conn_man_stats>` 指标并且断开中间响应（如果头已经发送到下游）然后发送 500 响应。

有三个控制点可以配置 Envoy 流量控制：
:ref:`监听器限制 <envoy_v3_api_field_config.listener.v3.Listener.per_connection_buffer_limit_bytes>` 、
:ref:`集群限制 <envoy_v3_api_field_config.cluster.v3.Cluster.per_connection_buffer_limit_bytes>` 和
:ref:`http2 流限制 <envoy_v3_api_field_config.core.v3.Http2ProtocolOptions.initial_connection_window_size>`

监听器限制适用于每个 read() 函数从下游读取多少原始数据，以及在 Envoy 和下游之间的用户空间缓存多少数据。

同样监听器限制也会传递到 HttpConnectionManager，在每个流的基础上应用于下面描述的 HTTP/1.1 L7 缓存区。
因此，是它们限制了可以缓冲的 HTTP/1 请求和响应体的大小。
对于 HTTP/2，由于许多流通过一个 TCP 连接进行多路复用，因此可以单独调节 L7 和 L4 的缓存区限制，并且可以配置 :ref:`http2 流限制 <envoy_v3_api_field_config.core.v3.Http2ProtocolOptions.initial_connection_window_size>` 选项应用于所有 L7 缓存区。
请注意，对于 HTTP/1 和 HTTP/2 请求，在所有 L7 过滤器都是流式传输的路由上，Envoy 可以并将代理任意大的实体，但许多过滤器（如转码器和缓冲过滤器）需要缓冲整个 HTTP 体，因此需要根据监听器的限制限制请求和响应的大小。

集群限制影响每个 read() 函数从上游读取多少原始数据，以及在 Envoy 和上游之间的用户空间缓存多少数据。

下面的代码库展示了如何调整如上所述的三个地方，通常情况下唯一需要修改的是 :ref:`per_connection_buffer_limit_bytes <envoy_v3_api_field_config.listener.v3.Listener.per_connection_buffer_limit_bytes>` 监听器。 

.. code-block:: yaml

  static_resources:
    listeners:
      name: http
      address:
        socket_address:
          address: '::1'
          portValue: 0
      filter_chains:
        filters:
          name: envoy.filters.network.http_connection_manager
          typed_config:
            "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
            http2_protocol_options:
              initial_stream_window_size: 65535
            route_config: {}
            codec_type: HTTP2
            http_filters: []
            stat_prefix: config_test
      per_connection_buffer_limit_bytes: 1024
    clusters:
      name: cluster_0
      connect_timeout: 5s
      per_connection_buffer_limit_bytes: 1024
      load_assignment:
        cluster_name: some_service
        endpoints:
          - lb_endpoints:
            - endpoint:
                address:
                  socket_address:
                    address: ::1
                    port_value: 46685
