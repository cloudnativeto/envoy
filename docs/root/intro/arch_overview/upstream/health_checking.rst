.. _arch_overview_health_checking:

健康检查
===============

可以在每个上游集群的基础上 :ref:`配置 <config_cluster_manager_cluster_hc>` 主动健康检查。 如 :ref:`服务发现
<arch_overview_service_discovery>` 部分所述，主动健康检查和EDS服务发现类型是并行协作的。 但是，在其他情况下，即使使用其他服务发现类型，也需要进行主动健康检查。 Envoy支持三种不同类型的运行健康检查以及各种设置（检查时间间隔、主机不健康标记为故障、主机健康时标记为成功等）：

* **HTTP**: 在HTTP运行健康检查期间，Envoy将向上游主机发送HTTP请求。 默认情况下，如果主机运行状况良好，则期望200响应。 预期的响应代码是 :ref:`可配置 <envoy_v3_api_msg_config.core.v3.HealthCheck.HttpHealthCheck>`  的。 如果上游主机希望立即通知下游主机不再向其转发流量，则可以返回503。
* **L3/L4**: 在 L3/L4 健康检查期间，Envoy 会向上游主机发送一个可配置的字节缓冲区。如果主机被认为是健康的，字节缓冲区在响应中会被显示出来。Envoy 还支持仅连接 L3/L4 健康检查。
* **Redis**: Envoy 将发送 Redis PING 命令并期望 PONG 响应。如果上游 Redis 服务器使用 PONG 以外的任何其他响应命令，则会导致健康检查失败。或者，Envoy 可以在用户指定的键上执行 EXISTS。如果键不存在，则认为它是合格的健康检查。这允许用户通过将指定的键设置为任意值来标记 Redis 实例以进行维护直至流量耗尽。请参阅 
  :ref:`redis_key <envoy_v3_api_msg_config.health_checker.redis.v2.Redis>`.

运行健康检查通过为集群指定的传输套接字进行。这意味着，如果集群使用启用了TLS的传输套接字，则运行健康检查也将通过TLS进行。 可以指定用于运行健康检查连接的 :ref:`TLS options <envoy_v3_api_msg_config.core.v3.HealthCheck.TlsOptions>` ，，则此选项将很有用，如果相应的上游基于ALPN的 :ref:`FilterChainMatch <envoy_v3_api_msg_config.listener.v3.FilterChainMatch>` 使用带有不同协议的运行健康检查与数据连接协议。

.. _arch_overview_per_cluster_health_check_config:

每个集群成员运行健康检查配置
--------------------------------------

如果为上游集群配置了主动健康检查，则可以通过在 :ref:`ClusterLoadAssignment<envoy_v3_api_msg_config.endpoint.v3.ClusterLoadAssignment>` 中每个已定义的 :ref:`LocalityLbEndpoints<envoy_v3_api_msg_config.endpoint.v3.LocalityLbEndpoints>` 的 :ref:`LbEndpoint<envoy_v3_api_msg_config.endpoint.v3.LbEndpoint>` 的  :ref:`Endpoint<envoy_v3_api_msg_config.endpoint.v3.Endpoint>` 中设置 :ref:`HealthCheckConfig<envoy_v3_api_msg_config.endpoint.v3.Endpoint.HealthCheckConfig>` 来为每个注册成员指定特定的附加配置。


An example of setting up health check config to set a cluster member’s alternative health check port is:


以下示例为设置运行 :ref:`健康检查配置<envoy_v3_api_msg_config.endpoint.v3.Endpoint.HealthCheckConfig>`  以设置 
 :ref:`集群成员<envoy_v3_api_msg_config.endpoint.v3.Endpoint>`  可选运行健康检查的 :ref:`端口<envoy_v3_api_field_config.endpoint.v3.Endpoint.HealthCheckConfig.port_value>` 


.. code-block:: yaml

  load_assignment:
    endpoints:
    - lb_endpoints:
      - endpoint:
          health_check_config:
            port_value: 8080
          address:
            socket_address:
              address: localhost
              port_value: 80

.. _arch_overview_health_check_logging:

健康检查事件日志
-------------
Envoy 可以通过在 :ref:`HealthCheck 配置 <envoy_v3_api_field_config.core.v3.HealthCheck.event_log_path>` 中指定日志文件路径，选择性地生成包含弹出和添加事件的per-healthchecker日志。日志结构为 :ref:`HealthCheckEvent消息 <envoy_v3_api_msg_data.core.v3.HealthCheckEvent>` 的JSON dumps。

通过将 :ref:`always_log_health_check_failures
标志 <envoy_v3_api_field_config.core.v3.HealthCheck.always_log_health_check_failures>` 标志设置为true，特使可以配置为记录所有健康检查失败事件。

被动的健康检查
-----------------------
Envoy还支持通过 :ref:`异常值检测（outlier detection）
<arch_overview_outlier_detection>` 进行被动健康检查。


连接池的交互
----------------------------

请参阅 :ref:`此处 <arch_overview_conn_pool_health_checking>` 了解更多信息

.. _arch_overview_health_checking_filter:

HTTP健康检查过滤器
---------------------------

当部署 Envoy 网格并在集群之间进行主动健康检查时，会生成大量健康检查流量。Envoy 包含一个HTTP 健康检查过滤器，可以安装在配置的 HTTP 侦听器中。这个过滤器有几种不同的操作模式：

* **不穿过**: 在此模式下，运行健康检查请求永远不会传递给本地服务。Envoy 会根据服务器当前的耗尽状态以200或503响应。

* **从上游集群健康状况计算得出的不通过**:在此模式下，运行健康检查过滤器将返回200或503，具体取决于一个或多个上游集群中是否至少有 :ref:`指定百分比 <envoy_v3_api_field_extensions.filters.http.health_check.v3.HealthCheck.cluster_min_healthy_percentages>` 的服务器可用(运行状况+降级)。(但是，如果 Envoy 服务器处于耗尽状态，则无论上游集群运行状况如何，它都将使用503响应。)  

* **通过**：在此模式下，Envoy 会将每个健康检查请求传递给本地服务。根据该服务的健康状态返回200或503。

* **通过缓存传递**：在此模式下，Envoy 会将健康检查请求传递给本地服务，但会将结果缓存一段时间。如果在缓存有效期，在随后的健康检查请求会直接获取缓存的值。缓存到期后，下一个运行健康检查请求将传递给本地服务。操作大型网格时，这是推荐的操作模式。Envoy 会保存进行健康检查的连接，因此健康检查请求对 Envoy 本身的成本很低。因此，这种操作模式产生了每个上游主机的健康状态的最终一致的视图，而没有用大量的健康检查请求压倒本地服务。

进一步阅读：

* 健康检查过滤器 :ref:`配置 <config_http_filters_health_check>`.
* :ref:`/健康检查/失败 <operations_admin_interface_healthcheck_fail>` 管理端点.
* :ref:`/健康检查/ok <operations_admin_interface_healthcheck_ok>` 管理端点.

主动健康检查快速失败
-----------------------------------

在使用主动健康检查和被动运行健康检查(:ref:`异常值检测
<arch_overview_outlier_detection>`)时，通常使用较长的运行健康检查间隔来避免大量的主动健康检查流量。在这种情况下，当使用 :ref:`x-envoy-immediate-health-check-fail
<config_http_filters_router_x-envoy-immediate-health-check-fail>`  端点时，能够快速耗尽上游主机仍然很有用。为了支持这一点，:ref:`路由器过滤器 <config_http_filters_router>` 将响应 :ref:`x-envoy-immediate-health-check-fail <config_http_filters_router_x-envoy-immediate-health-check-fail>`。如果上游主机设置了此头，Envoy将立即将该主机标记为主动健康检查检查失败。注意，只有在主机集群 :ref:`配置
<config_cluster_manager_cluster_hc>` 了主动健康检查时才会发生这种情况。如果通过 :ref:`/健康检查/失败<operations_admin_interface_healthcheck_fail>`  管理端点将特使标记为失败，则运行 :ref:`健康检查筛选器<config_http_filters_health_check>` 将自动设置此头。


.. _arch_overview_health_checking_identity:

健康检查身份
---------------------

只验证上游主机是否响应特定的健康检查 URL 并不一定意味着上游主机有效。例如，当在自动缩放的云环境或容器环境中使用最终一致的服务发现时，被检查主机可能会消失，但是其他主机会以相同的 IP 地址返回。解决此问题的一个办法是针对每种服务类型都有不同的 HTTP 健康检查 URL。该方法的缺点是整体配置会变得更加复杂，因为每个健康检查 URL 都是完全自定义的。

Envoy HTTP 健康检查器支持 :ref:`service_name_matcher
<envoy_v3_api_field_config.core.v3.HealthCheck.HttpHealthCheck.service_name_matcher>` 选项。如果设置了此选项，健康检查程序还会使用 *x-envoy-upstream-healthchecked-cluster* 
响应标头的值与 *service_name_matcher* 进行比较。如果值不匹配，健康检查不通过。上游健康检查过滤器会将 *x-envoy-upstream-healthchecked-cluster* 附加到响应头。这个值由 :option:`--service-cluster` 命令行选项决定。


.. _arch_overview_health_checking_degraded:

健康状况下降
---------------
使用HTTP健康检查器时，上游主机可以返回 ``x-envoy-degraded`` 以通知健康检查器该主机已降级。 请参阅 :ref:`此处 <arch_overview_load_balancing_degraded>` 以了解这如何影响负载平衡。
