.. _config_http_filters_router:

路由器
======

路由过滤器实现了 HTTP 转发功能。几乎所有部署 Envoy 的 HTTP 代理场景都会使用到该功能。过滤器的主要任务是遵循 :ref:`路由表 <envoy_v3_api_msg_config.route.v3.RouteConfiguration>` 中所配置的指定说明进行操作。除转发和重定向外，过滤器还处理重试、统计信息等。

* :ref:`v3 API 参考 <envoy_v3_api_msg_extensions.filters.http.router.v3.Router>`
* 此过滤器应使用 *envoy.filters.http.router* 名称进行配置。

.. _config_http_filters_router_headers_consumed:

HTTP 头部 (从下游消费)
----------------------------------------

路由器在出口/请求路径和入口/响应路径上消费和设置各种 HTTP 头部。它们记录在本节中。

.. contents::
  :local:

.. _config_http_filters_router_x-envoy-max-retries:

x-envoy-max-retries
^^^^^^^^^^^^^^^^^^^
如果已经设置了 :ref:`路由重试策略 <envoy_v3_api_field_config.route.v3.RouteAction.retry_policy>` 或者 :ref:`虚拟主机重试策略 <envoy_v3_api_field_config.route.v3.VirtualHost.retry_policy>`，除非明确指定，Envoy 将会默认重试一次。重试的次数可以在虚拟主机重试配置、路由重试配置中明确设置，或者通过使用此头部。如果此头部已经使用，其值会优先于重试策略中设置的重试次数。如果没有配置重试策略并且 :ref:`config_http_filters_router_x-envoy-retry-on` 或者 :ref:`config_http_filters_router_x-envoy-retry-grpc-on` 头部也没有指定，Envoy 将不会重试失败的请求。

有关 Envoy 如何进行重试的一些说明：

* 路由超时（在路由配置中设置 :ref:`config_http_filters_router_x-envoy-upstream-rq-timeout-ms` 或 :ref:`timeout <envoy_v3_api_field_config.route.v3.RouteAction.timeout>`，或者通过在路由配置中指定 :ref:`max_grpc_timeout <envoy_v3_api_field_config.route.v3.RouteAction.timeout>` 以设置 `grpc 超时头部 <https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md>`_) **包括了** 所有重试。因此，如果请求超时设置为 3 秒，并且第一次请求尝试花费 2.7 秒，则重试（包括补偿）需要 0.3 秒才能完成。这样设计是为了避免指数级的超时/重试剧增。
* 默认情况下，Envoy 使用完整的抖动指数补偿算法在默认的 25 毫秒基本间隔时间中进行重试。给定一个基本间隔时间 B 和重试次数 N，则重试的补偿范围在 :math:`\big[0, (2^N-1)B\big)` 内。例如，给定默认间隔时间，第一次重试将随机延迟 0-24 毫秒，第二次为 0-74 毫秒，第三次为 0-174 毫秒，依此类推。间隔时间有个最大封顶，默认是基本间隔时间的 10 倍（250 毫秒）。默认的基本间隔时间（进而最大间隔时间）可以通过设置 upstream.base_retry_backoff_ms 运行时参数进行操控。补偿间隔时间可以通过配置重试策略的 :ref:`重试补偿 <envoy_v3_api_field_config.route.v3.RetryPolicy.retry_back_off>` 进行修改。
* Envoy 也可以配置为使用上游服务器的反馈来决定重试之间的间隔时间。响应头部如 ``Retry-After`` 或者 ``X-RateLimit-Reset`` 指示客户端重试之前要等待多长时间。重试策略的 :ref:`限流重试补偿 <envoy_v3_api_field_config.route.v3.RetryPolicy.rate_limited_retry_back_off>` 机制可以配置用于期望特定的头部，如果头部在响应中存在，Envoy 将会使用其值来决定补偿。如果头部不存在，或者如果无法成功解析，Envoy 将会使用默认指数补偿算法。

.. _config_http_filters_router_x-envoy-retry-on:

x-envoy-retry-on
^^^^^^^^^^^^^^^^

设置此头部将会导致 Envoy 尝试重试失败的请求（默认的重试次数为 1 并且可由 :ref:`x-envoy-max-retries <config_http_filters_router_x-envoy-max-retries>` 头部控制，或是 :ref:`路由配置重试策略 <envoy_v3_api_field_config.route.v3.RouteAction.retry_policy>`，或是 :ref:`虚拟主机重试策略 <envoy_v3_api_field_config.route.v3.VirtualHost.retry_policy>`）。
设置 x-envoy-retry-on 头部的值用以指定重试策略。可以使用 “，” 分隔符指定一项或多项策略。支持的策略包括：

5xx
  如果上游服务响应了任何 5xx 响应码或者完全不响应（断联/重置/读取超时）时 Envoy 将会尝试重试。（包括 *connect-failure* 和 *refused-stream*）

  * **注意：** 当请求超出了 :ref:`config_http_filters_router_x-envoy-upstream-rq-timeout-ms` （导致 504 错误码）时 Envoy 将不会重试。如果要在每次尝试花费太长时间时重试，请使用 :ref:`config_http_filters_router_x-envoy-upstream-rq-per-try-timeout-ms`。:ref:`config_http_filters_router_x-envoy-upstream-rq-timeout-ms` 是请求的外部时间限制，包含于任何发生的重试中。

gateway-error
  此策略类似于 *5xx* 策略，但只会对导致了 502、503 或 504 的请求进行重试。

reset
  当上游服务完全不响应（断联/重置/读取失败）时 Envoy 将尝试重试。

connect-failure
  如果请求因为连接上游服务失败（连接超时等）而导致失败时 Envoy 将尝试重试。（包含在 *5xx* 中）

  * **注意：** 连接失败/超时是指 TCP 级别，而不是指请求级别。这不包括通过 :ref:`config_http_filters_router_x-envoy-upstream-rq-timeout-ms` 或 :ref:`路由配置 <envoy_v3_api_field_config.route.v3.RouteAction.retry_policy>` 或 :ref:`虚拟主机重试策略 <envoy_v3_api_field_config.route.v3.VirtualHost.retry_policy>` 所指定的上游请求超时。

.. _config_http_filters_router_retry_policy-envoy-ratelimited:

envoy-ratelimited
  如果存在 :ref:`x-envoy-ratelimited<config_http_filters_router_x-envoy-ratelimited>` 头部，则 Envoy 将重试。

retriable-4xx
  如果上游服务响应了可重试的 4xx 响应码，则 Envoy 将尝试重试。当前，此分类中的唯一响应码是 409。

  * **注意：** 请小心启用该重试类型。在某些情况下，409 响应码表示调用方需要更新乐观锁的版本标识。因此，调用方不应该重试，而是需要先读取然后再尝试写入。如果在这种情况下发生了重试，他将始终导致另一方的 409 失败。

refused-stream
  如果上游服务使用 REFUSED_STREAM 错误码重置流，则 Envoy 将尝试重试。此重置类型表示请求是可以安全重试的。（包含在 *5xx* 中）

retriable-status-codes
  如果上游响应了任何与 :ref:`重试策略 <envoy_v3_api_field_config.route.v3.RetryPolicy.retriable_status_codes>` 或者 :ref:`config_http_filters_router_x-envoy-retriable-status-codes` 头部中相匹配的响应码，则 Envoy 将尝试重试。

retriable-headers
  如果上游响应了任何与 :ref:`重试策略 <envoy_v3_api_field_config.route.v3.RetryPolicy.retriable_headers>` 或者 :ref:`config_http_filters_router_x-envoy-retriable-header-names` 头部中相匹配的响应码，则 Envoy 将尝试重试。

重试次数可以通过 :ref:`config_http_filters_router_x-envoy-max-retries` 头部或 :ref:`路由配置 <envoy_v3_api_field_config.route.v3.RouteAction.retry_policy>` 或 :ref:`虚拟主机重试策略 <envoy_v3_api_field_config.route.v3.VirtualHost.retry_policy>` 控制。

注意重试策略也可以应用于 :ref:`路由级别 <envoy_v3_api_field_config.route.v3.RouteAction.retry_policy>` 或 :ref:`虚拟主机级别 <envoy_v3_api_field_config.route.v3.VirtualHost.retry_policy>`。

默认情况下，Envoy 将*不会*执行重试除非已按上述要求进行了配置。

.. _config_http_filters_router_x-envoy-retry-grpc-on:

x-envoy-retry-grpc-on
^^^^^^^^^^^^^^^^^^^^^
设置此头部将会导致 Envoy 尝试重试失败的请求（默认的重试次数为 1，且可通过 :ref:`x-envoy-max-retries <config_http_filters_router_x-envoy-max-retries>` 头部或 :ref:`路由配置重试策略 <envoy_v3_api_field_config.route.v3.RouteAction.retry_policy>` 或 :ref:`虚拟主机重试策略 <envoy_v3_api_field_config.route.v3.VirtualHost.retry_policy>` 控制）。
当前 gRPC 重试只支持了 gRPC 响应头部的状态码。trailers 中的 gRPC 状态码不会触发重试逻辑。可以使用 ‘，’ 分隔符指定一项或多项策略。支持的策略包括：

cancelled
  如果响应头部中的 gRPC 状态码为 “cancelled”（1） 则 Envoy 将尝试重试。

deadline-exceeded
  如果响应头部中的 gRPC 状态码为 “deadline-exceeded”（4） 则 Envoy 将尝试重试。

internal
  如果响应头部中的 gRPC 状态码为 “internal”（13） 则 Envoy 将尝试重试。

resource-exhausted
  如果响应头部中的 gRPC 状态码为 “resource-exhausted”（8） 则 Envoy 将尝试重试。

unavailable
  如果响应头部中的 gRPC 状态码为 “unavailable”（14） 则 Envoy 将尝试重试。

与 x-envoy-retry-grpc-on 头部相同，重试的次数可以通过 :ref:`config_http_filters_router_x-envoy-max-retries` 头部控制。

注意重试策略也可以应用于 :ref:`路由级别 <envoy_v3_api_field_config.route.v3.RouteAction.retry_policy>` 或 :ref:`虚拟主机级别 <envoy_v3_api_field_config.route.v3.VirtualHost.retry_policy>`。

默认情况下，Envoy 将*不会*执行重试除非您已按上述要求进行了配置。

.. _config_http_filters_router_x-envoy-retriable-header-names:

x-envoy-retriable-header-names
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
设置此头部会告知 Envoy 应将哪些响应头部视为可重试的依据。它与 :ref:`可重试头部 <config_http_filters_router_x-envoy-retry-on>` 重试策略一同使用。设置了相应的重试策略后，除了其他重试策略设置的响应头部外，此列表头部所提供的响应头部也将会被视为可重试的。

该列表是以逗号分隔的头部名称列表：如果启用了 “可重试头部” 重试策略，则 “X-Upstream-Retry,X-Try-Again” 将使包含指定头部之一的任何上游响应都可重试。头部名是区分大小写的。

通过请求头部只能指定可重试响应头部的名称。可以通过 :ref:`重试策略配置 <envoy_v3_api_field_config.route.v3.RetryPolicy.retriable_headers>` 重试策略使用任意头部匹配规则来指定基于响应头的更复杂的重试策略。

只有内部客户端发出的请求才会使用此头部。

.. _config_http_filters_router_x-envoy-retriable-status-codes:

x-envoy-retriable-status-codes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
当与 :ref:`retriable-status-code <config_http_filters_router_x-envoy-retry-on>` 重试策略结合使用时，设置此头部会告知 Envoy 应将哪些响应码视为可重试的依据。当相关的重试策略已设置，除了通过其他重试策略进行重试所启用的状态码外，可重试状态码列表也会做为可重试的依据。

该列表是以逗号分隔的整数列表：“409” 将会使 409 被认定为可重试，而 “504,409” 则会使 504 和 409 都被认定为可重试。

只有内部客户端发出的请求才会使用此头部。

.. _config_http_filters_router_x-envoy-upstream-alt-stat-name:

x-envoy-upstream-alt-stat-name
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

设置此头部会导致 Envoy 将上游响应码/时间统计信息发送到另外的统计树中。这对于 Envoy 以外的应用级分类统计很有帮助。输出树的文档记录在 :ref:`此处 <config_cluster_manager_cluster_stats_alt_tree>`。

请勿将此与 :ref:`alt_stat_name <envoy_v3_api_field_config.cluster.v3.Cluster.alt_stat_name>` 混淆，后者是在定义集群时指定的，而且是在为统计树根级别上的集群提供备用名称时。

.. _config_http_filters_router_x-envoy-upstream-rq-timeout-alt-response:

x-envoy-upstream-rq-timeout-alt-response
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

设置此头部将导致 Envoy 在请求超时的情况下设置 204 响应码（而不是 504）。实际的头部值会被忽略；仅判断其是否存在。另请参考 :ref:`config_http_filters_router_x-envoy-upstream-rq-timeout-ms`。

.. _config_http_filters_router_x-envoy-upstream-rq-timeout-ms:

x-envoy-upstream-rq-timeout-ms
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

设置此头部将导致 Envoy 覆盖 :ref:`路由超时配置 <envoy_v3_api_field_config.route.v3.RouteAction.timeout>` 或通过指定 `grpc 超时头部 <https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md>`_ 的 :ref:`max_grpc_timeout <envoy_v3_api_field_config.route.v3.RouteAction.timeout>` 设置的 gRPC 客户端超时。超时时间必须使用毫秒为单位指定。另请参考 :ref:`config_http_filters_router_x-envoy-upstream-rq-per-try-timeout-ms`。

.. _config_http_filters_router_x-envoy-upstream-rq-per-try-timeout-ms:

x-envoy-upstream-rq-per-try-timeout-ms
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

设置此头部将导致 Envoy 对路由的请求设置 *每次尝试* 的超时。如果配置了全局路由超时，此超时时间必须小于全局路由超时（查看 :ref:`config_http_filters_router_x-envoy-upstream-rq-timeout-ms`）否则会被忽略。这允许调用方设置严格的每次尝试超时时间用以进行尝试，同时又保持合理的总体超时时间。此超时仅在响应的任何部分发送到下游之前才适用，通常发生在上游发送响应头部之后。

x-envoy-hedge-on-per-try-timeout
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

设置此头部会导致 Envoy 在每次尝试发生超时的场景下使用请求规避策略。这将覆盖 :ref:`路由配置 <envoy_v3_api_field_config.route.v3.HedgePolicy.hedge_on_per_try_timeout>` 中所设置的值。这意味着将在不重置原始请求的情况下发出重试，从而使多个上游请求处于运行状态。

此头部值应当为 “true” 或 “false”，如果无效会被忽略。

.. _config_http_filters_router_x-envoy-decorator-operation:

x-envoy-decorator-operation
^^^^^^^^^^^^^^^^^^^^^^^^^^^

此头部值将覆盖由跟踪机制生成的服务器跨度上的任何本地定义的操作（跨度）名称。

从上游消费的 HTTP 响应头部
--------------------------------------------

x-envoy-decorator-operation
^^^^^^^^^^^^^^^^^^^^^^^^^^^

此头部值将覆盖由跟踪机制生成的服务器跨度上的任何本地定义的操作（跨度）名称。

x-envoy-upstream-canary
^^^^^^^^^^^^^^^^^^^^^^^

如果上游主机设置了此头部，则路由器将会用其生成金丝雀特定的统计信息。输出树的文档记录在 :ref:`此处 <config_cluster_manager_cluster_stats_dynamic_http>`。

.. _config_http_filters_router_x-envoy-immediate-health-check-fail:

x-envoy-immediate-health-check-fail
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

如果上游主机返回了此头部（设置为任何值），Envoy 将会立即假定上游主机的 :ref:`动态健康检查 <arch_overview_health_checking>` （如果集群已经 :ref:`配置 <config_cluster_manager_cluster_hc>` 了动态健康检查）已经失败。这可用于通过常规的数据平面快速处理使上游主机发生故障，而无需等待下一个健康检查间隔。主机可以通过常规的动态健康检查重新恢复正常。查看 :ref:`健康检查总览 <arch_overview_health_checking>` 获取更多信息。

.. _config_http_filters_router_x-envoy-ratelimited:

x-envoy-ratelimited
^^^^^^^^^^^^^^^^^^^

如果此头部在上游设置，Envoy 将不会重试除非重试策略 :ref:`envoy-ratelimited<config_http_filters_router_retry_policy-envoy-ratelimited>` 被启用。当前，头部的值不会被查看，只会判断其是否存在。当进行请求限流时，此头部通过 :ref:`限流过滤器 <config_http_filters_rate_limit>` 设置。

.. _config_http_filters_router_headers_set:

在上游调用中设置 HTTP 的请求头部
------------------------------------------

路由器在出口/请求路径以及入口/响应路径上都设置了各种 HTTP 头部。它们在本章节中记录。

.. contents::
  :local:

.. _config_http_filters_router_x-envoy-attempt-count:

x-envoy-attempt-count
^^^^^^^^^^^^^^^^^^^^^

发送到上游以指示当前请求是在一系列重试中的哪次尝试。初始请求的值会是 “1”，随着每次重试而递增。只有在 :ref:`include_request_attempt_count <envoy_v3_api_field_config.route.v3.VirtualHost.include_request_attempt_count>` 标记设置为 true 的情况下才做设置。

发送到下游以指示发生了多少上游请求。如果路由器没有发送任何上游请求则该头部将会消失。如果仅发送原始上游请求，其值将会是 “1”，且随着每次重试递增。只有在 :ref:`include_attempt_count_in_response <envoy_v3_api_field_config.route.v3.VirtualHost.include_attempt_count_in_response>` 标记设置为 true 的情况下才做设置。

.. _config_http_filters_router_x-envoy-expected-rq-timeout-ms:

x-envoy-expected-rq-timeout-ms
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

这是路由器期望请求完成的时间（以毫秒为单位）。Envoy 设置此头部以便于接受请求的上游主机可以基于请求超时做决定（例如提前退出）。这是在内部请求上设置的，可以按顺序从 :ref:`config_http_filters_router_x-envoy-upstream-rq-timeout-ms` 头部或 :ref:`路由超时 <envoy_v3_api_field_config.route.v3.RouteAction.timeout>` 中获取。

.. _config_http_filters_router_x-envoy-original-path:

x-envoy-original-path
^^^^^^^^^^^^^^^^^^^^^

如果路由使用 :ref:`prefix_rewrite <envoy_v3_api_field_config.route.v3.RouteAction.prefix_rewrite>` 或 :ref:`regex_rewrite <envoy_v3_api_field_config.route.v3.RouteAction.regex_rewrite>`，则 Envoy 会将原始路径头部放到此头部中。这对于日志记录和调试很有用。

在下游响应中设置 HTTP 响应头部
-------------------------------------------------

.. _config_http_filters_router_x-envoy-upstream-service-time:

x-envoy-upstream-service-time
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

包括上游主机处理请求所花费的时间（以毫秒为单位）以及 Envoy 和上游主机间的网络延迟。当客户端希望确定服务时间，而不是对比客户端和 Envoy 之间的网络延迟时会很有用。此头部是在响应上设置的。

.. _config_http_filters_router_x-envoy-overloaded_set:

x-envoy-overloaded
^^^^^^^^^^^^^^^^^^

如果请求由于 :ref:`维护模式 <config_http_filters_router_runtime_maintenance_mode>` 或上游 :ref:`断路 <arch_overview_circuit_break>` 而被丢弃，则 Envoy 将在下游响应上设置此头部。

.. _config_http_filters_router_stats:

统计
----------

路由器输出了许多统计信息到集群命名空间中（取决于所选路由指定的命名空间）。查看 :ref:`此处 <config_cluster_manager_cluster_stats>` 获取更多信息。

路由器过滤器输出统计信息到 *http.<stat_prefix>.* 命名空间中。:ref:`stat prefix <envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.stat_prefix>` 来自于 HTTP 连接管理器的的所有者。

.. csv-table::
  :header: 名称, 类型, 描述
  :widths: 1, 1, 2

  no_route, Counter, 找不到路由并导致 404 的请求总数
  no_cluster, Counter, 目标集群不存在并默认导致 503 的请求总数
  rq_redirect, Counter, 导致重定向响应的请求总数
  rq_direct_response, Counter, 导致直接响应的请求总数
  rq_total, Counter, 发生路由的总请求数
  rq_reset_after_downstream_response_started, Counter, 在下游响应开始后发生重置的请求总数

.. _config_http_filters_router_vcluster_stats:

虚拟集群
^^^^^^^^^^^^^^^^

虚拟集群统计信息会输出到 *vhost.<virtual host name>.vcluster.<virtual cluster name>.* 命名空间中并包含以下统计信息：

.. csv-table::
  :header: 名称, 类型, 描述
  :widths: 1, 1, 2

  upstream_rq_<\*xx>, Counter, "聚合 HTTP 响应码（例如：2xx、3xx 等）"
  upstream_rq_<\*>, Counter, "指定 HTTP 响应码（例如：201、302 等）"
  upstream_rq_retry, Counter, 请求重试总数
  upstream_rq_retry_limit_exceeded, Counter, 由于超出 :ref:`已配置的最大重试次数 <config_http_filters_router_x-envoy-max-retries>` 而没有重试的请求总数
  upstream_rq_retry_overflow, Counter, 由于断路或超出 :ref:`重试预算 <envoy_v3_api_field_config.cluster.v3.CircuitBreakers.Thresholds.retry_budget>` 而没有重试的请求总数
  upstream_rq_retry_success, Counter, 重试成功的请求总数
  upstream_rq_time, Histogram, 请求时间（以毫秒为单位）
  upstream_rq_timeout, Counter, 等待响应超时的请求总数
  upstream_rq_total, Counter, 路由器向上游发起的请求总数

运行时
-------

路由过滤器支持如下运行时设置：

upstream.base_retry_backoff_ms
  基本指数重试补偿时间。查看 :ref:`此处 <arch_overview_http_routing_retry>` 和 :ref:`config_http_filters_router_x-envoy-max-retries` 获取更多信息。默认为 25 毫秒。
  默认的最大重试补偿时间为该值的 10 倍。

.. _config_http_filters_router_runtime_maintenance_mode:

upstream.maintenance_mode.<cluster name>
  将会立即导致 503 响应的请求百分比。这将覆盖 <cluster name> 中所定义的请求的任何路由行为。可用于释放负载、故障注入等。默认禁用。

upstream.use_retry
  有重试资格的请求百分比。此配置在其他重试配置之前检查，并且可以按需在所有 Envoy 节点完全禁用重试。
