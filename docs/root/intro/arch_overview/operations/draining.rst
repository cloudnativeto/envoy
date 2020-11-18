.. _arch_overview_draining:

排空
========

排空（Draining）是 Envoy 相应各种事件要求试图优雅地关闭连接的过程。例如，在服务器关闭期间，可以阻止现有请求，并将监听器设置为停止接受，以减少服务器关闭时打开的连接数。除个别监听器配置外，服务器选项还定义了排空行为。

排空发生在以下时机：

* 服务器 :ref:`热重启 <arch_overview_hot_restart>` 。
* 服务器通过 :ref:`drain_listeners?graceful <operations_admin_interface_drain>` 管理端点，开始优雅的排空序列。
* 服务器通过 :ref:`健康检查失败 <operations_admin_interface_healthcheck_fail>` 管理端点，手动设置健康检查失败。更多信息，请参阅 :ref:`健康检查过滤器 <arch_overview_health_checking_filter>` 架构概述。
* 通过 :ref:`LDS <arch_overview_dynamic_config_lds>` 修改或者移除单个监听器。

默认情况下，Envoy 服务器将在服务器关闭时立即关闭监听器。监听器将被直接停止，而没有任何优雅的排空行为，并立即停止接受新的连接。若要在服务器关闭之前的一段时间内排空监听器，可以在服务器关闭之前使用 :ref:`drain_listeners <operations_admin_interface_drain>` 。

若要在关闭监听器之前添加一个优雅的排空周期，请使用查询参数 :ref:`drain_listeners?graceful <operations_admin_interface_drain>` 。默认情况下，Envoy 会在一段时间（由 :option:`--drain-time-s` 确定）内阻止请求。请求阻止的行为由排空管理器确定。

请注意，尽管排空是每个监听器的概念，但前提必须是在网络过滤器级别上支持它。目前，支持优雅排空的过滤器只有 :ref:`Redis <config_network_filters_redis_proxy>` 、:ref:`Mongo <config_network_filters_mongo_proxy>` 和 :ref:`HTTP 连接管理器 <config_http_conn_man>` 。

默认情况下，:ref:`HTTP 连接管理器 <config_http_conn_man>` 过滤器会向 HTTP1 请求添加 "Connection: close" ，发送 HTTP2 GOAWAY，并在请求完成时（在延迟的关闭时间之后）终止连接。

每个 :ref:`已配置监听器 <arch_overview_listeners>` 都有一个 :ref:`drain_type <envoy_v3_api_enum_config.listener.v3.Listener.DrainType>` 设置，用来控制排空何时发生。目前支持的值有：

default
  Envoy 将在上述三种情况（健康检查失败、热重启和 LDS 更新/删除）发生时排空监听器。这是默认设置。
modify_only
  Envoy 只会在上述第二和第三种情况（热重启和 LDS 更新/删除）发生时排空监听器。如果 Envoy 同时托管了 ingress 和 egress 监听器，此设置会很有用。在 egress 监听器上设置 modify_only 是可取的，这样仅在 egress 监听器修改期间进行排空，而在尝试执行关机时，则依赖于 ingress 监听器上设置 modify_only 来执行全服务器的排空。
