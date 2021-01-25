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

断路器统计
-------------

断路器统计以 *cluster.<name>.circuit_breakers.<priority>.* 为根，包含如下信息：

.. csv-table::
  :header: 名称, 类型, 描述
  :widths: 1, 1, 2

  cx_open, Gauge, 连接断路器是关闭（0）还是打开（1）
  cx_pool_open, Gauge, 连接池断路器是关闭（0）还是打开（1）
  rq_pending_open, Gauge, 挂起的请求断路器是关闭（0）还是打开（1）
  rq_open, Gauge, 断路器是关闭（0）还是打开（1）
  rq_retry_open, Gauge, 重试断路器是关闭（0）还是打开（1）
  remaining_cx, Gauge, 断路器断开前的剩余连接数
  remaining_pending, Gauge, 断路器断开前的剩余未决请求数
  remaining_rq, Gauge, 断路器断开前的剩余请求数
  remaining_retries, Gauge, 断路器断开前的剩余重试次数

.. _config_cluster_manager_cluster_stats_timeout_budgets:

超时预算统计
-------------------------

如果打开了 :ref:`超时预算统计跟踪 <envoy_v3_api_field_config.cluster.v3.Cluster.track_timeout_budgets>`，
统计信息将添加到 *cluster.<name>* 并包含以下内容：

.. csv-table::
   :header: 名称, 类型, 描述
   :widths: 1, 1, 2

   upstream_rq_timeout_budget_percent_used, Histogram, 等待响应时使用的全局超时百分比是多少
   upstream_rq_timeout_budget_per_try_percent_used, Histogram, 每次尝试超时等待响应的百分比是多少

.. _config_cluster_manager_cluster_stats_dynamic_http:

动态 HTTP 统计
---------------

如果使用 HTTP，还可以使用动态 HTTP 响应代码统计信息。
这些信息由各种内部系统以及一些过滤器（如路由过滤器 :ref:`路由过滤器 <config_http_filters_router>`
和 :ref:`速率限制过滤器 <config_http_filters_rate_limit>`）发出。它们的根位于 *cluster.<name>* 并包含以下统计信息：

.. csv-table::
  :header: 名称, 类型, 描述
  :widths: 1, 1, 2

  upstream_rq_completed, Counter, "已完成的上游请求总数"
  upstream_rq_<\*xx>, Counter, "聚合HTTP响应代码（例如 2xx、3xx 等）"
  upstream_rq_<\*>, Counter, "特定的HTTP响应代码（例如 201、302 等）"
  upstream_rq_time, Histogram, 请求时间毫秒数
  canary.upstream_rq_completed, Counter, "已完成的上游金丝雀请求总数"
  canary.upstream_rq_<\*xx>, Counter, 上游金丝雀聚合HTTP响应代码
  canary.upstream_rq_<\*>, Counter, 上游特定于金丝雀的HTTP响应代码
  canary.upstream_rq_time, Histogram, 上游金丝雀请求时间毫秒
  internal.upstream_rq_completed, Counter, "已完成的内部来源请求总数"
  internal.upstream_rq_<\*xx>, Counter, 内部源聚合 HTTP 响应代码
  internal.upstream_rq_<\*>, Counter, 内部源指定 HTTP 响应代码
  internal.upstream_rq_time, Histogram, 内部原点请求时间（毫秒）
  external.upstream_rq_completed, Counter, "已完成的外部来源请求总数"
  external.upstream_rq_<\*xx>, Counter, 外部源聚合 HTTP 响应代码
  external.upstream_rq_<\*>, Counter, 外部源指定 HTTP 响应代码
  external.upstream_rq_time, Histogram, 外部原点请求时间（毫秒）

.. _config_cluster_manager_cluster_stats_alt_tree:

备用树动态 HTTP 统计信息
-----------------------

如果配置了备用树统计信息，它们将出现在 *cluster.<name>.<alt name>.* 命名空间中。
生成的统计信息与 :ref:`上面 <config_cluster_manager_cluster_stats_dynamic_http>` 的动态 HTTP 统计信息部分中记录的信息相同。

.. _config_cluster_manager_cluster_per_az_stats:

每个服务区域的动态 HTTP 统计信息
------------------------------

对于本地服务，如果服务区是可用的（通过 :option:`--service-zone`）
或者 :ref:`上游集群 <arch_overview_service_discovery_types_eds>`，
Envoy 将在 *cluster.<name>.zone.<from_zone>.<to_zone>.* 命名空间中跟踪以下统计信息：

.. csv-table::
  :header: 名称, 类型, 描述
  :widths: 1, 1, 2

  upstream_rq_<\*xx>, Counter, 聚合 HTTP 响应代码（例如 2xx、3xx 等）
  upstream_rq_<\*>, Counter, 特定 HTTP 响应代码（例如 201、302 等）
  upstream_rq_time, Histogram, 请求时间（毫秒）

负载均衡统计信息
-----------------

用于监视负载均衡器决策的统计信息。统计数据以 *cluster.<name>* 为根并包含以下信息：

.. csv-table::
  :header: 名称, 类型, 描述
  :widths: 1, 1, 2

  lb_recalculate_zone_structures, Counter, 用于快速决定上游位置选择，重新生成位置感知路由结构的次数
  lb_healthy_panic, Counter, 在紧急模式下与负载均衡器进行负载均衡的请求总数
  lb_zone_cluster_too_small, Counter, 由于上游集群规模较小，因此没有区域感知路由
  lb_zone_routing_all_directly, Counter, 将所有请求直接发送到同一区域
  lb_zone_routing_sampled, Counter, 向同一区域发送一些请求
  lb_zone_routing_cross_zone, Counter, 区域感知路由模式，但必须跨区域发送
  lb_local_cluster_not_ok, Counter, 未设置本地主机集或本地群集处于紧急模式
  lb_zone_number_differs, Counter, 本地和上游集群中的区域数不同
  lb_zone_no_capacity_left, Counter, 由于舍入误差以随机区域选择结束的总次数
  original_dst_host_invalid, Counter, 传递到原始目标负载均衡器的非法主机总数

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
