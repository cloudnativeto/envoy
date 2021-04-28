.. _config_http_filters_tap:

Tap
===

* :ref:`v3 API 参考 <envoy_v3_api_msg_extensions.filters.http.tap.v3.Tap>`
* 此过滤器的名称应该配置为 *envoy.filters.http.tap*。

.. attention::
  tap 过滤器是实验性的，目前正在积极开发中。当前可用的匹配条件、输出配置、输出接收器等还很有限。其功能会逐渐扩展，且配置结构可能会发生变化。

tap 过滤器用于插入和记录 HTTP 流量。 从宏观看，其配置由两项组成：

1. :ref:`匹配配置 <envoy_v3_api_msg_config.tap.v3.MatchPredicate>`：一个条件列表，若满足这些条件，过滤器会匹配一个 HTTP 请求并开启一个 tap 会话。
2. :ref:`输出配置 <envoy_v3_api_msg_config.tap.v3.OutputConfig>`：一个输出接收器列表，tap 过滤器会将匹配和过滤的数据写入这些接收器。

下一节将通过几个配置示例逐步介绍上述的各个概念。

配置示例
---------------------

过滤器配置示例

.. code-block:: yaml

  name: envoy.filters.http.tap
  typed_config:
    "@type": type.googleapis.com/envoy.extensions.filters.http.tap.v3.Tap
    common_config:
      admin_config:
        config_id: test_config_id

上面的代码段通过 :http:post:`/tap` admin handler 配置了过滤器以便管理。详情请参考下一节。

.. _config_http_filters_tap_admin_handler:

管理处理器（admin handler）
-------------

当 HTTP 过滤器指定了一个 :ref:`admin_config
<envoy_v3_api_msg_extensions.common.tap.v3.AdminConfig>`, 它将被配置用来管理员控制，且 :http:post:`/tap` 管理处理器将被安装。 管理处理器可以用来实时过滤和调试 HTTP 流量。工作流程如下：

1. POST 请求用于提供有效的 tap 配置。POST 请求体可以是 JSON 或 YAML 格式的 :ref:`TapConfig
   <envoy_v3_api_msg_config.tap.v3.TapConfig>` 消息。 
2. 如果该 POST 请求被接收，Envoy 会将 :ref:`HttpBufferedTrace
   <envoy_v3_api_msg_data.tap.v3.HttpBufferedTrace>` 信息转化成流（将其序列化为 JSON）直到该请求被终止。

一个 POST 请求体示例：

.. code-block:: yaml

  config_id: test_config_id
  tap_config:
    match_config:
      and_match:
        rules:
          - http_request_headers_match:
              headers:
                - name: foo
                  exact_match: bar
          - http_response_headers_match:
              headers:
                - name: bar
                  exact_match: baz
    output_config:
      sinks:
        - streaming_admin: {}

上述配置指示 tap 过滤器匹配所有同时包含请求头 ``foo: bar`` 和响应头 ``bar: baz`` 的 HTTP 请求。如果这两个条件都满足，该请求将被过滤，并流式传输到管理端点。

另一个 POST 请求体示例：

.. code-block:: yaml

  config_id: test_config_id
  tap_config:
    match_config:
      or_match:
        rules:
          - http_request_headers_match:
              headers:
                - name: foo
                  exact_match: bar
          - http_response_headers_match:
              headers:
                - name: bar
                  exact_match: baz
    output_config:
      sinks:
        - streaming_admin: {}

上述配置指示 tap 过滤器匹配所有包含请求头 ``foo: bar`` 或响应头 ``bar: baz`` 的 HTTP 请求。若任一个条件满足，该请求将被过滤，并流式传输到管理端点。

另一个 POST 请求体示例：

.. code-block:: yaml

  config_id: test_config_id
  tap_config:
    match_config:
      any_match: true
    output_config:
      sinks:
        - streaming_admin: {}

上述配置指示 tap 过滤器匹配所有 HTTP 请求，所有的请求都会被过滤并流式传输到管理端点。

另一个 POST 请求体示例：

.. code-block:: yaml

  config_id: test_config_id
  tap_config:
    match_config:
      and_match:
        rules:
          - http_request_headers_match:
              headers:
                - name: foo
                  exact_match: bar
          - http_request_generic_body_match:
              patterns:
                - string_match: test
                - binary_match: 3q2+7w==
              bytes_limit: 128
          - http_response_generic_body_match:
              patterns:
                - binary_match: vu8=
              bytes_limit: 64
    output_config:
      sinks:
        - streaming_admin: {}

上述代码指示 tap 过滤器匹配所有同时满足以下条件的 HTTP 请求：该请求包含请求头 ``foo: bar``；该请求的请求体包含字符串 ``test`` 且前 128 字节包含十六进制字节 ``deadbeef`` （base64 转换后为 ``3q2+7w==``）；响应体的前 64 字节包含十六进制字节 ``beef`` （base64 转换后为 ``vu8=``）。如果上述条件都满足，该请求将被过滤并流式传输到管理端点。

.. attention::

  在 HTTP 请求和响应体中使用正则表达式匹配可能会消耗大量 CPU 资源。因为对于每个指定的表达式，请求和响应体将被逐字节地扫描直到完成匹配。如果指定了多个表达式，对每个表达式都会执行该扫描过程。如果某表达式的位置是已知的，则应使用 ``bytes_limit`` 指定扫描位置。

输出格式
-------------

每个输出接收器都有其关联的 :ref:`格式
<envoy_v3_api_enum_config.tap.v3.OutputSink.Format>`。默认的格式是 :ref:`JSON_BODY_AS_BYTES
<envoy_v3_api_enum_value_config.tap.v3.OutputSink.Format.JSON_BODY_AS_BYTES>`。该模式易于读取 JSON，但缺点是该响应体是 base64 编码的。对于用户读取的数据，:ref:`JSON_BODY_AS_STRING
<envoy_v3_api_enum_value_config.tap.v3.OutputSink.Format.JSON_BODY_AS_STRING>` 格式可能对用户更友好，关于其他可用的格式，请查阅参考文档获取详细信息。

下面是一个使用 :ref:`JSON_BODY_AS_STRING
<envoy_v3_api_enum_value_config.tap.v3.OutputSink.Format.JSON_BODY_AS_STRING>` 格式配置 streaming admin tap 的示例：

.. code-block:: yaml

  config_id: test_config_id
  tap_config:
    match_config:
      any_match: true
    output_config:
      sinks:
        - format: JSON_BODY_AS_STRING
          streaming_admin: {}

缓存体限制
--------------------

对于应用缓存的 tap 过滤器，Envoy会限制过滤的请求和响应体的数据量以避免内存不足的情况。接收（请求）和发送（响应）数据的默认限制为 1KiB。这可以通过 :ref:`max_buffered_rx_bytes
<envoy_v3_api_field_config.tap.v3.OutputConfig.max_buffered_rx_bytes>` 和
:ref:`max_buffered_tx_bytes
<envoy_v3_api_field_config.tap.v3.OutputConfig.max_buffered_tx_bytes>` 设置。

.. _config_http_filters_tap_streaming:

流匹配
------------------

tap 过滤器支持“流匹配”，意思是该过滤器不会等待请求/响应序列的结束，而会随着请求的进行逐步匹配。即，首先匹配请求头，其次若有请求体则进行匹配，然后若有请求尾则进行匹配，再然后若有响应头则进行匹配，以此类推。

该过滤器还支持可选的流输出，由 :ref:`streaming
<envoy_v3_api_field_config.tap.v3.OutputConfig.streaming>` 设置管理。如果该项设置为 false（默认），Envoy 会发出 :ref:`fully buffered traces <envoy_v3_api_msg_data.tap.v3.HttpBufferedTrace>`。 在简单的情况下，用户可能会觉得这种格式更易于操作。

当全缓存跟踪不适用时（如，请求和返回体非常大，长连接的流 API，等等），可以将 streaming 设置为 true，且 Envoy会为每个 tap 过滤发出多个 :ref:`streamed trace segments <envoy_v3_api_msg_data.tap.v3.HttpStreamedTraceSegment>`。在这种情况下，要求执行后续处理，从而将所有跟踪段组合成一个可用的形式。另外请注意二进制的 protobuf 不是自定界的格式，如果需要二进制 protobuf 输出，则应使用 :ref:`PROTO_BINARY_LENGTH_DELIMITED
<envoy_v3_api_enum_value_config.tap.v3.OutputSink.Format.PROTO_BINARY_LENGTH_DELIMITED>` 格式输出。

一个启动流输出的静态过滤器配置如下所示：

.. code-block:: yaml

  name: envoy.filters.http.tap
  typed_config:
    "@type": type.googleapis.com/envoy.extensions.filters.http.tap.v3.Tap
    common_config:
      static_config:
        match_config:
          http_response_headers_match:
            headers:
              - name: bar
                exact_match: baz
        output_config:
          streaming: true
          sinks:
            - format: PROTO_BINARY_LENGTH_DELIMITED
              file_per_tap:
                path_prefix: /tmp/

上面的配置会匹配响应头，并缓存请求头、请求体和请求尾直到完成匹配（缓存数据的限制依然适用，如上节所述）。如果一个匹配完成，则缓存的数据将在单独的跟踪段中刷新，然后在数据到达时流式传输剩余的数据。输出的消息可能如下所示：

.. code-block:: yaml

  http_streamed_trace_segment:
    trace_id: 1
    request_headers:
      headers:
        - key: a
          value: b

.. code-block:: yaml

  http_streamed_trace_segment:
    trace_id: 1
    request_body_chunk:
      as_bytes: aGVsbG8=

等等。

统计数据
----------

tap 过滤器将统计信息输出在命名空间 *http.<stat_prefix>.tap.* 中。 :ref:`stat prefix
<envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.stat_prefix>` 来自其所属的 HTTP 连接管理器。

.. csv-table::
  :header: 名称, 类型, 描述
  :widths: 1, 1, 2

  rq_tapped, Counter, Total requests that matched and were tapped
