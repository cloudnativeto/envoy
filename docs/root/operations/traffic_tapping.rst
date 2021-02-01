.. _operations_traffic_tapping:

流量捕获
==========

Envoy目前提供了两个可以捕获流量的实验性扩展：

  * :ref:`HTTP 捕获过滤器 <config_http_filters_tap>`。更多信息参见过滤器文档。
  * :ref:`捕获传输套接字拓展 <envoy_v3_api_msg_config.core.v3.TransportSocket>` 可以拦截流量并写入 :ref:`protobuf 跟踪文件 <envoy_v3_api_msg_data.tap.v3.TraceWrapper>`。本文档的其余部分将介绍捕获传输套接字的配置。

捕获传输套接字传输配置 
----------------------------------

.. attention::

 捕获传输套接字是实验性的，正在积极地开发中。目前有一套非常有限的匹配条件、输出配置、输出汇等。随着时间的推移，能力将得到扩展，配置结构也可能发生变化。

捕获器可以在 :ref:`监听器
<envoy_v3_api_field_config.listener.v3.FilterChain.transport_socket>` 和 :ref:`集群
<envoy_v3_api_field_config.cluster.v3.Cluster.transport_socket>` 传输套接字上配置，提供分别在下行和上行L4连接上进行插接的能力。
要配置流量捕获，需要在监听器或集群中添加一个  `envoy.transport_sockets.tap`  传输套接字
:ref:`配置<envoy_v3_api_msg_extensions.filters.http.tap.v3.Tap>`。对于纯文本套接字来说，它可能是这样的：

.. code-block:: yaml

  transport_socket:
    name: envoy.transport_sockets.tap
    typed_config:
      "@type": type.googleapis.com/envoy.extensions.transport_sockets.tap.v3.Tap
      common_config:
        static_config:
          match_config:
            any_match: true
          output_config:
            sinks:
              - format: PROTO_BINARY
                file_per_tap:
                  path_prefix: /some/tap/path
      transport_socket:
        name: envoy.transport_sockets.raw_buffer

TLS 套接字，是这样的：

.. code-block:: yaml

  transport_socket:
    name: envoy.transport_sockets.tap
    typed_config:
      "@type": type.googleapis.com/envoy.extensions.transport_sockets.tap.v3.Tap
      common_config:
        static_config:
          match_config:
            any_match: true
          output_config:
            sinks:
              - format: PROTO_BINARY
                file_per_tap:
                  path_prefix: /some/tap/path
      transport_socket:
        name: envoy.transport_sockets.tls
        typed_config: <TLS context>

其中，TLS 上下文配置分别替换了监听器或集群上现有的 :ref:`下行<envoy_v3_api_msg_extensions.transport_sockets.tls.v3.DownstreamTlsContext>` 或 :ref:`上行 <envoy_v3_api_msg_extensions.transport_sockets.tls.v3.UpstreamTlsContext>`  TLS 配置。

每个独特的套接字实例都会生成一个以 path_prefix 为前缀的跟踪文件。例如：/some/tap/path_0.pb。

缓冲数据限制
--------------------

对于缓冲套接字的捕获，Envoy 会限制捕获的主体数据量，以避免出现 OOM 情况。收和传输数据的默认限制是1KiB，这可以通过  :ref:`max_buffered_rx_bytes
<envoy_v3_api_field_config.tap.v3.OutputConfig.max_buffered_rx_bytes>` 和
:ref:`max_buffered_tx_bytes
<envoy_v3_api_field_config.tap.v3.OutputConfig.max_buffered_tx_bytes>` 设置进行配置。 当缓冲套接字被截断时，跟踪将通过 :ref:`read_truncated
<envoy_v3_api_field_data.tap.v3.SocketBufferedTrace.read_truncated>` 和 :ref:`write_truncated
<envoy_v3_api_field_data.tap.v3.SocketBufferedTrace.write_truncated>` 字段以及 :ref:`truncated <envoy_v3_api_field_data.tap.v3.Body.truncated>` 字段显示截断情况。

流式传输
---------

捕获传输套接字支持缓冲和流式传输，由 :ref:`streaming
<envoy_v3_api_field_config.tap.v3.OutputConfig.streaming>` 设置控制。 缓冲时，会发出
:ref:`SocketBufferedTrace <envoy_v3_api_msg_data.tap.v3.SocketBufferedTrace>` 信息。当进行流式传输时，会发出一系列 :ref:`SocketStreamedTraceSegment
<envoy_v3_api_msg_data.tap.v3.SocketStreamedTraceSegment>` 消息。

更多信息参见 :ref:`HTTP tap filter streaming <config_http_filters_tap_streaming>` 文档，HTTP 过滤器和传输 套接字的大部分概念是重复的.

PCAP 传播
---------------

生成的跟踪文件可以转换为 `libpcap format
<https://wiki.wireshark.org/Development/LibpcapFileFormat>`_,格式， 可以使用如
 `Wireshark <https://www.wireshark.org/>`_ 和 tap2pcap 这样的工具进行分析, 例如:

.. code-block:: bash

  bazel run @envoy_api_canonical//tools:tap2pcap /some/tap/path_0.pb path_0.pcap
  tshark -r path_0.pcap -d "tcp.port==10000,http2" -P
    1   0.000000    127.0.0.1 → 127.0.0.1    HTTP2 157 Magic, SETTINGS, WINDOW_UPDATE, HEADERS
    2   0.013713    127.0.0.1 → 127.0.0.1    HTTP2 91 SETTINGS, SETTINGS, WINDOW_UPDATE
    3   0.013820    127.0.0.1 → 127.0.0.1    HTTP2 63 SETTINGS
    4   0.128649    127.0.0.1 → 127.0.0.1    HTTP2 5586 HEADERS
    5   0.130006    127.0.0.1 → 127.0.0.1    HTTP2 7573 DATA
    6   0.131044    127.0.0.1 → 127.0.0.1    HTTP2 3152 DATA, DATA
