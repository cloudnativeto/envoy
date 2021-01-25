.. _config_cluster_manager_cluster_stats:

统计
==========

.. contents::
  :local:

总则
-------

集群管理有一个根在 *cluster_manager.* 的统计树，有如下统计信息。在 stats 名称中的任何 ``:`` 字符都将被替换为 ``_``。
统计信息包含由集群管理器管理的所有集群，包括用于数据平面上游的集群和控制平面 xD 的集群。

.. csv-table::
  :header: 名称, 类型, 描述
  :widths: 1, 1, 2

  cluster_added, Counter, 添加的群集总数（通过静态配置或 CD）
  cluster_modified, Counter, 修改的群集总数（通过 CD）
  cluster_removed, Counter, 删除的群集总数（通过 CD）
  cluster_updated, Counter, 群集更新总数
  cluster_updated_via_merge, Counter, 作为合并更新应用的群集更新总数
  update_merge_cancelled, Counter, 已取消并提前交付的合并更新总数
  update_out_of_merge_window, Counter, 从合并窗口到达的总更新数
  active_clusters, Gauge, 当前活动（warmed）群集数
  warming_clusters, Gauge, 当前处在 warming（非活动）的群集数

每个集群有一个根在 *cluster.<name>.* 的统计树，有如下统计信息：

.. csv-table::
  :header: 名称, 类型, 描述
  :widths: 1, 1, 2

  upstream_cx_total, Counter, 连接总数
  upstream_cx_active, Gauge, 活动的连接总数
  upstream_cx_http1_total, Counter, HTTP/1.1 连接总数
  upstream_cx_http2_total, Counter, HTTP/2 连接总数
  upstream_cx_connect_fail, Counter, 连接失败总数
  upstream_cx_connect_timeout, Counter, 连接超时总数
  upstream_cx_idle_timeout, Counter, 连接 idle 超时总数
  upstream_cx_connect_attempts_exceeded, Counter, 超过配置连接尝试次数的连续连接失败总数
  upstream_cx_overflow, Counter, 群集连接断路器溢出的总次数
  upstream_cx_connect_ms, Histogram, 连接建立毫秒数
  upstream_cx_length_ms, Histogram, 连接长度毫秒数
  upstream_cx_destroy, Counter, 断开的连接总数
  upstream_cx_destroy_local, Counter, 本地断开的连接总数
  upstream_cx_destroy_remote, Counter, 远程断开的连接总数
  upstream_cx_destroy_with_active_rq, Counter, 1+ 个活动请求断开的连接总数
  upstream_cx_destroy_local_with_active_rq, Counter, 使用 1+ 个活动请求在本地断开的连接总数
  upstream_cx_destroy_remote_with_active_rq, Counter, 使用 1+ 个活动请求远程断开的连接总数
  upstream_cx_close_notify, Counter, 通过 HTTP/1.1 连接关闭头或 HTTP/2 GOAWAY 关闭的连接总数
  upstream_cx_rx_bytes_total, Counter, 接收的连接字节总数
  upstream_cx_rx_bytes_buffered, Gauge, 当前缓冲的接收连接字节数
  upstream_cx_tx_bytes_total, Counter, 发送的连接字节总数
  upstream_cx_tx_bytes_buffered, Gauge, 当前缓冲的发送连接字节数
  upstream_cx_pool_overflow, Counter, 群集的连接池断路器溢出的总次数
  upstream_cx_protocol_error, Counter, 连接协议错误总数
  upstream_cx_max_requests, Counter, 由于最大请求数而关闭的连接总数
  upstream_cx_none_healthy, Counter, 由于没有正常主机而未建立连接的总次数
  upstream_rq_total, Counter, 请求总数
  upstream_rq_active, Gauge, 活动的请求总数
  upstream_rq_pending_total, Counter, 挂起连接池连接的请求总数
  upstream_rq_pending_overflow, Counter, 溢出连接池或请求（主要针对 HTTP/2）断路并失败的请求总数
  upstream_rq_pending_failure_eject, Counter, 由于连接池连接失败或远程连接终止失败的请求总数
  upstream_rq_pending_active, Gauge, 挂起连接池连接的活动请求总数
  upstream_rq_cancelled, Counter, 在获取连接池连接之前取消的请求总数
  upstream_rq_maintenance_mode, Counter, 由于 :ref:`维护模式<config_http_filters_router_runtime_maintenance_mode>` 导致立即 503 的请求总数
  upstream_rq_timeout, Counter, 等待响应超时的请求总数
  upstream_rq_max_duration_reached, Counter, 由于达到最大持续时间而关闭的请求总数
  upstream_rq_per_try_timeout, Counter, 达到每次尝试超时的请求总数（启用请求对冲时除外）
  upstream_rq_rx_reset, Counter, 远程重置的请求总数
  upstream_rq_tx_reset, Counter, 本地重置的请求总数
  upstream_rq_retry, Counter, 请求重试总次数
  upstream_rq_retry_backoff_exponential, Counter, 使用指数退避策略的总重试次数
  upstream_rq_retry_backoff_ratelimited, Counter, 使用有限退避策略的总重试次数
  upstream_rq_retry_limit_exceeded, Counter, 由于超过 :ref:`配置的最大重试次数 <config_http_filters_router_x-envoy-max-retries>` 而未重试的请求总数
  upstream_rq_retry_success, Counter, 请求重试成功总数
  upstream_rq_retry_overflow, Counter, 由于线路中断或超出 :ref:`预置的重试次数 <envoy_v3_api_field_config.cluster.v3.CircuitBreakers.Thresholds.retry_budget>` 而未重试的请求总数
  upstream_flow_control_paused_reading_total, Counter, 流量控制从上游暂停读取的总次数
  upstream_flow_control_resumed_reading_total, Counter, 流量控制从上游恢复读取的总次数
  upstream_flow_control_backed_up_total, Counter, 上游连接备份并暂停从下游读取的总次数
  upstream_flow_control_drained_total, Counter, 上游连接从下游排出并恢复读取的总次数
  upstream_internal_redirect_failed_total, Counter, 失败的内部重定向导致向下游传递重定向的总次数。
  upstream_internal_redirect_succeed_total, Counter, 内部重定向导致第二个上行请求的总次数。
  membership_change, Counter, 集群成员更改总数
  membership_healthy, Gauge, 当前群集健康总数（包括健康检查和异常值检测）
  membership_degraded, Gauge, 当前集群降级总数
  membership_total, Gauge, 当前集群成员总数
  retry_or_shadow_abandoned, Counter, 由于缓冲区限制而取消映射或重试缓冲的总次数
  config_reload, Counter, 由于配置不同而导致配置重新加载的API获取总数
  update_attempt, Counter, 通过服务发现尝试的群集成员更新总数
  update_success, Counter, 通过服务发现列出的成功群集成员更新总数
  update_failure, Counter, 通过服务发现列出的失败群集成员更新总数
  update_empty, Counter, 集群成员更新以空集群负载分配结束并继续以前的配置总数
  update_no_rebuild, Counter, 未导致任何群集负载平衡结构重建的成功群集成员更新总数
  version, Gauge, 上次成功获取 API 的内容的哈希
  max_host_weight, Gauge, 群集中任意主机的最大权重
  bind_errors, Counter, 将套接字绑定到配置的源地址的错误总数
  assignment_timeout_received, Counter, 接收到的具有终结点租用信息的总分配数
  assignment_stale, Counter, 新分配到达之前接收的分配过期的次数

健康检查统计
------------

如果配置了健康检查，集群会有一个额外的根在 *cluster.<name>.health_check.* 的统计树，有如下统计信息：

.. csv-table::
  :header: 名称, 类型, 描述
  :widths: 1, 1, 2

  attempt, Counter, 健康检查次数
  success, Counter, 成功的健康检查次数
  failure, Counter, 立即失败的健康检查（例如 HTTP 503）以及网络故障数
  passive_failure, Counter, 由被动事件导致的健康检查失败次数（比如 x-envoy-immediate-health-check-fail）
  network_failure, Counter, 由于网络错误导致的健康检查失败数
  verify_cluster, Counter, 尝试群集名称验证的运行状况检查数
  healthy, Gauge, 健康成员数

.. _config_cluster_manager_cluster_stats_outlier_detection:

异常检测统计
----------------------------

如果在集群中配置了 :ref:`异常检测 <arch_overview_outlier_detection>`，
集群将有一个根在 *cluster.<name>.outlier_detection.* 的统计树，包含如下统计信息：

.. csv-table::
  :header: 名称, 类型, 描述
  :widths: 1, 1, 2

  ejections_enforced_total, Counter, 由于任何异常类型而强制弹出的次数
  ejections_active, Gauge, 当前弹出的主机数
  ejections_overflow, Counter, 由于最大弹出而中止的弹出数
  ejections_enforced_consecutive_5xx, Counter, 强制连续 5xx 弹出次数
  ejections_detected_consecutive_5xx, Counter, 检测到的连续的 5xx 弹出次数（即使未强制）
  ejections_enforced_success_rate, Counter, 强制成功率异常值弹出数。 此计数器的确切含义取决于 :ref:`outlier_detection.split_external_local_origin_errors<envoy_v3_api_field_config.cluster.v3.OutlierDetection.split_external_local_origin_errors>` 配置项。 详情参考 :ref:`异常检测文档<arch_overview_outlier_detection>`。
  ejections_detected_success_rate, Counter, 检测到的成功率异常值弹出数（即使未强制）。 此计数器的确切含义取决于 :ref:`outlier_detection.split_external_local_origin_errors<envoy_v3_api_field_config.cluster.v3.OutlierDetection.split_external_local_origin_errors>` 配置项。 详情参考 :ref:`异常过滤文档<arch_overview_outlier_detection>`。
  ejections_enforced_consecutive_gateway_failure, Counter, 强制的连续网关故障弹出次数
  ejections_detected_consecutive_gateway_failure, Counter, 检测到的连续网关故障弹出次数（即使未强制）
  ejections_enforced_consecutive_local_origin_failure, Counter, 强制的连续本地源故障弹出数
  ejections_detected_consecutive_local_origin_failure, Counter, 检测到的连续本地原点故障弹出次数（即使未强制）
  ejections_enforced_local_origin_success_rate, Counter, 本地发生的故障的强制成功率异常值弹出数
  ejections_detected_local_origin_success_rate, Counter, 检测到的本地故障的成功率异常值弹出数（即使未强制）
  ejections_enforced_failure_percentage, Counter, 强制失败百分比异常值弹出数。 此计数器的确切含义取决于 :ref:`outlier_detection.split_external_local_origin_errors<envoy_v3_api_field_config.cluster.v3.OutlierDetection.split_external_local_origin_errors>` 配置项。 详情参考 :ref:`异常过滤文档<arch_overview_outlier_detection>`。
  ejections_detected_failure_percentage, Counter, 检测到的故障百分比异常弹出数（即使未强制）。 此计数器的确切含义取决于 :ref:`outlier_detection.split_external_local_origin_errors<envoy_v3_api_field_config.cluster.v3.OutlierDetection.split_external_local_origin_errors>` 配置项。 详情参考 :ref:`异常过滤文档<arch_overview_outlier_detection>`。
  ejections_enforced_failure_percentage_local_origin, Counter, 本地发生的故障的强制故障百分比异常值弹出数
  ejections_detected_failure_percentage_local_origin, Counter, 本地故障的检测到的故障百分比异常值弹出数（即使未强制）
  ejections_total, Counter, 已弃用。 任何异常值类型导致的弹出次数（即使未强制）
  ejections_consecutive_5xx, Counter, 已弃用。 连续5xx弹出次数（即使未强制）

.. _config_cluster_manager_cluster_stats_circuit_breakers:

Circuit breakers statistics
---------------------------

Circuit breakers statistics will be rooted at *cluster.<name>.circuit_breakers.<priority>.* and contain the following:

.. csv-table::
  :header: Name, Type, Description
  :widths: 1, 1, 2

  cx_open, Gauge, Whether the connection circuit breaker is closed (0) or open (1)
  cx_pool_open, Gauge, Whether the connection pool circuit breaker is closed (0) or open (1)
  rq_pending_open, Gauge, Whether the pending requests circuit breaker is closed (0) or open (1)
  rq_open, Gauge, Whether the requests circuit breaker is closed (0) or open (1)
  rq_retry_open, Gauge, Whether the retry circuit breaker is closed (0) or open (1)
  remaining_cx, Gauge, Number of remaining connections until the circuit breaker opens
  remaining_pending, Gauge, Number of remaining pending requests until the circuit breaker opens
  remaining_rq, Gauge, Number of remaining requests until the circuit breaker opens
  remaining_retries, Gauge, Number of remaining retries until the circuit breaker opens

.. _config_cluster_manager_cluster_stats_timeout_budgets:

Timeout budget statistics
-------------------------

If :ref:`timeout budget statistic tracking <envoy_v3_api_field_config.cluster.v3.Cluster.track_timeout_budgets>` is
turned on, statistics will be added to *cluster.<name>* and contain the following:

.. csv-table::
   :header: Name, Type, Description
   :widths: 1, 1, 2

   upstream_rq_timeout_budget_percent_used, Histogram, What percentage of the global timeout was used waiting for a response
   upstream_rq_timeout_budget_per_try_percent_used, Histogram, What percentage of the per try timeout was used waiting for a response

.. _config_cluster_manager_cluster_stats_dynamic_http:

Dynamic HTTP statistics
-----------------------

If HTTP is used, dynamic HTTP response code statistics are also available. These are emitted by
various internal systems as well as some filters such as the :ref:`router filter
<config_http_filters_router>` and :ref:`rate limit filter <config_http_filters_rate_limit>`. They
are rooted at *cluster.<name>.* and contain the following statistics:

.. csv-table::
  :header: Name, Type, Description
  :widths: 1, 1, 2

  upstream_rq_completed, Counter, "Total upstream requests completed"
  upstream_rq_<\*xx>, Counter, "Aggregate HTTP response codes (e.g., 2xx, 3xx, etc.)"
  upstream_rq_<\*>, Counter, "Specific HTTP response codes (e.g., 201, 302, etc.)"
  upstream_rq_time, Histogram, Request time milliseconds
  canary.upstream_rq_completed, Counter, "Total upstream canary requests completed"
  canary.upstream_rq_<\*xx>, Counter, Upstream canary aggregate HTTP response codes
  canary.upstream_rq_<\*>, Counter, Upstream canary specific HTTP response codes
  canary.upstream_rq_time, Histogram, Upstream canary request time milliseconds
  internal.upstream_rq_completed, Counter, "Total internal origin requests completed"
  internal.upstream_rq_<\*xx>, Counter, Internal origin aggregate HTTP response codes
  internal.upstream_rq_<\*>, Counter, Internal origin specific HTTP response codes
  internal.upstream_rq_time, Histogram, Internal origin request time milliseconds
  external.upstream_rq_completed, Counter, "Total external origin requests completed"
  external.upstream_rq_<\*xx>, Counter, External origin aggregate HTTP response codes
  external.upstream_rq_<\*>, Counter, External origin specific HTTP response codes
  external.upstream_rq_time, Histogram, External origin request time milliseconds

.. _config_cluster_manager_cluster_stats_alt_tree:

Alternate tree dynamic HTTP statistics
--------------------------------------

If alternate tree statistics are configured, they will be present in the
*cluster.<name>.<alt name>.* namespace. The statistics produced are the same as documented in
the dynamic HTTP statistics section :ref:`above
<config_cluster_manager_cluster_stats_dynamic_http>`.

.. _config_cluster_manager_cluster_per_az_stats:

Per service zone dynamic HTTP statistics
----------------------------------------

If the service zone is available for the local service (via :option:`--service-zone`)
and the :ref:`upstream cluster <arch_overview_service_discovery_types_eds>`,
Envoy will track the following statistics in *cluster.<name>.zone.<from_zone>.<to_zone>.* namespace.

.. csv-table::
  :header: Name, Type, Description
  :widths: 1, 1, 2

  upstream_rq_<\*xx>, Counter, "Aggregate HTTP response codes (e.g., 2xx, 3xx, etc.)"
  upstream_rq_<\*>, Counter, "Specific HTTP response codes (e.g., 201, 302, etc.)"
  upstream_rq_time, Histogram, Request time milliseconds

Load balancer statistics
------------------------

Statistics for monitoring load balancer decisions. Stats are rooted at *cluster.<name>.* and contain
the following statistics:

.. csv-table::
  :header: Name, Type, Description
  :widths: 1, 1, 2

  lb_recalculate_zone_structures, Counter, The number of times locality aware routing structures are regenerated for fast decisions on upstream locality selection
  lb_healthy_panic, Counter, Total requests load balanced with the load balancer in panic mode
  lb_zone_cluster_too_small, Counter, No zone aware routing because of small upstream cluster size
  lb_zone_routing_all_directly, Counter, Sending all requests directly to the same zone
  lb_zone_routing_sampled, Counter, Sending some requests to the same zone
  lb_zone_routing_cross_zone, Counter, Zone aware routing mode but have to send cross zone
  lb_local_cluster_not_ok, Counter, Local host set is not set or it is panic mode for local cluster
  lb_zone_number_differs, Counter, Number of zones in local and upstream cluster different
  lb_zone_no_capacity_left, Counter, Total number of times ended with random zone selection due to rounding error
  original_dst_host_invalid, Counter, Total number of invalid hosts passed to original destination load balancer

.. _config_cluster_manager_cluster_stats_subset_lb:

Load balancer subset statistics
-------------------------------

Statistics for monitoring :ref:`load balancer subset <arch_overview_load_balancer_subsets>`
decisions. Stats are rooted at *cluster.<name>.* and contain the following statistics:

.. csv-table::
  :header: Name, Type, Description
  :widths: 1, 1, 2

  lb_subsets_active, Gauge, Number of currently available subsets
  lb_subsets_created, Counter, Number of subsets created
  lb_subsets_removed, Counter, Number of subsets removed due to no hosts
  lb_subsets_selected, Counter, Number of times any subset was selected for load balancing
  lb_subsets_fallback, Counter, Number of times the fallback policy was invoked
  lb_subsets_fallback_panic, Counter, Number of times the subset panic mode triggered
  lb_subsets_single_host_per_subset_duplicate, Gauge, Number of duplicate (unused) hosts when using :ref:`single_host_per_subset <envoy_v3_api_field_config.cluster.v3.Cluster.LbSubsetConfig.LbSubsetSelector.single_host_per_subset>`

.. _config_cluster_manager_cluster_stats_ring_hash_lb:

Ring hash load balancer statistics
----------------------------------

Statistics for monitoring the size and effective distribution of hashes when using the
:ref:`ring hash load balancer <arch_overview_load_balancing_types_ring_hash>`. Stats are rooted at
*cluster.<name>.ring_hash_lb.* and contain the following statistics:

.. csv-table::
  :header: Name, Type, Description
  :widths: 1, 1, 2

  size, Gauge, Total number of host hashes on the ring
  min_hashes_per_host, Gauge, Minimum number of hashes for a single host
  max_hashes_per_host, Gauge, Maximum number of hashes for a single host

.. _config_cluster_manager_cluster_stats_maglev_lb:

Maglev load balancer statistics
-------------------------------

Statistics for monitoring effective host weights when using the
:ref:`Maglev load balancer <arch_overview_load_balancing_types_maglev>`. Stats are rooted at
*cluster.<name>.maglev_lb.* and contain the following statistics:

.. csv-table::
  :header: Name, Type, Description
  :widths: 1, 1, 2

  min_entries_per_host, Gauge, Minimum number of entries for a single host
  max_entries_per_host, Gauge, Maximum number of entries for a single host

.. _config_cluster_manager_cluster_stats_request_response_sizes:

Request Response Size statistics
--------------------------------

If :ref:`request response size statistics <envoy_v3_api_field_config.cluster.v3.Cluster.track_cluster_stats>` are tracked,
statistics will be added to *cluster.<name>* and contain the following:

.. csv-table::
   :header: Name, Type, Description
   :widths: 1, 1, 2

   upstream_rq_headers_size, Histogram, Request headers size in bytes per upstream
   upstream_rq_body_size, Histogram, Request body size in bytes per upstream
   upstream_rs_headers_size, Histogram, Response headers size in bytes per upstream
   upstream_rs_body_size, Histogram, Response body size in bytes per upstream
