.. _config_overload_manager:

负载管理器
================

:ref:`负载管理器 <arch_overview_overload_manager>` 被配置在引导中的
:ref:`overload_manager <envoy_v3_api_field_config.bootstrap.v3.Bootstrap.overload_manager>`
字段.

下面展现了一个负载管理器的配置样例。它展示了当堆内存使用达到 95% 时关闭 HTTP/1.x 的 keepalive 特性，并在堆内存使用达到 99% 时停止接收请求。

.. code-block:: yaml

   refresh_interval:
     seconds: 0
     nanos: 250000000
   resource_monitors:
     - name: "envoy.resource_monitors.fixed_heap"
       typed_config:
         "@type": type.googleapis.com/envoy.config.resource_monitor.fixed_heap.v2alpha.FixedHeapConfig
         max_heap_size_bytes: 2147483648
   actions:
     - name: "envoy.overload_actions.disable_http_keepalive"
       triggers:
         - name: "envoy.resource_monitors.fixed_heap"
           threshold:
             value: 0.95
     - name: "envoy.overload_actions.stop_accepting_requests"
       triggers:
         - name: "envoy.resource_monitors.fixed_heap"
           threshold:
             value: 0.99

资源监控器
------------

负载管理器使用 Envoy 的 :ref:`扩展 <extending>` 框架来定义资源监控器。Envoy 的内建资源监控器在 :ref:`这里 <config_resource_monitors>` 列出。 

触发器
-------------

触发器把资源监控器和动作联系在一起。 这里支持两类触发器:

.. list-table::
  :header-rows: 1
  :widths: 1, 2

  * - 类型
    - 描述
  * - :ref:`threshold <envoy_v3_api_msg_config.overload.v3.ThresholdTrigger>`
    - 当资源压力高于一个阈值设置动作的状态置为 1 (= *激活*), 否则置为 0.
  * - :ref:`scaled <envoy_v3_api_msg_config.overload.v3.ScaledTrigger>`
    - 当资源压力低于
      :ref:`scaling_threshold <envoy_v3_api_field_config.overload.v3.ScaledTrigger.scaling_threshold>` 设置动作状态为 0,
      当 `scaling_threshold < 压力 < saturation_threshold` 设置为 `(压力 - scaling_threshold)/(saturation_threshold - scaling_threshold)` , 当压力高于
      :ref:`saturation_threshold <envoy_v3_api_field_config.overload.v3.ScaledTrigger.saturation_threshold>` 设置为 1 (*饱和*)."

.. _config_overload_manager_overload_actions:

负载动作
----------------

支持下列负载动作:

.. csv-table::
  :header: 名字, 描述
  :widths: 1, 2

  envoy.overload_actions.stop_accepting_requests, Envoy 将会给新请求立即响应503响应码
  envoy.overload_actions.disable_http_keepalive, Envoy 将会停止接收入向 HTTP 连接流
  envoy.overload_actions.stop_accepting_connections, Envoy 将会在它配置的监听器上的停止接收新的网络连接
  envoy.overload_actions.shrink_heap, Envoy 将周期性的尝试缩小堆以释放内存给系统

限制活跃连接
-----------------

当前，只能通过给运行时关键字 ``overload.global_downstream_max_connections`` 制定一个整数值，这一种方式来限制跨越所有监听器的活跃连接总数。
考虑到上游连接，文件和其他文件描述符的使用，连接限制建议小于系统文件描述符限制的一半。
如果这个值未指定，活跃的下游连接数将没有全局的限制并且 Envoy 将在启动的时候发出一个警告提示。为了不设置活跃下游连接数限制并且禁用这个警告，
这个值可以被设置成一个非常大的限制（~2e9)。

如果希望只限制一个特定监听器的下游连接数，每监听器限制可以通过 :ref:`监听器配置 <config_listeners>` 设置。
可能同时指定每监听器和全局下游连接限制，这些约束会独立的执行。例如，如果已知一个特定的监听器应该比其他监听器有更少的打开连接数，
可能为此特定监听器指定一个更小连接限制同时允许全局限制所有的连接器的资源利用率。

在 :ref:`edge 最佳实践文档 <best_practices_edge>` 可以找到一个配置的样例.

统计
----------

每一个配置的资源监控器都有一个以 *overload.<name>.* 为根的统计树，包含下列统计值:

.. csv-table::
  :header: 名字, 类型, 描述
  :widths: 1, 1, 2

  pressure, 观测值, 资源压力百分比
  failed_updates, 计数器, 尝试更新资源压力失败的总数
  skipped_updates, 计数器, 由于挂起更新而尝试更新资源压力跳过的总数

每一个配置的负载动作都有一个以 *overload.<name>.* 为根的统计树，包含下列统计值:

.. csv-table::
  :header: 名字, 类型, 描述
  :widths: 1, 1, 2

  active, 观测值, "动作的活跃状态 (0=未激活, 1=激活)"
  scale_percent, 观测值, "动作的观测值百分比 (0-99=比例, 100=饱和)"
