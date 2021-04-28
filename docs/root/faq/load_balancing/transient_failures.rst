.. _common_configuration_transient_failures:

如何处理瞬时故障？
===================================

在服务网格中使用 Envoy 的一个最大优势就是它让服务从实现复杂的弹性功能解脱出来，如熔断、异常检测以及重试，这些让服务应对实际情况时更具弹性，常见的诸如滚动升级、动态基础设施变更以及网络故障。Envoy 中实现的上述功能不仅仅提高了服务可用性和韧性，而且在行为和观测性方面带来了一致性。

这部分将从一个较高层次来解释 Envoy 支持的配置以及如何一起使用这些功能来处理这些场景。

熔断
----------------

:ref:`熔断 <arch_overview_circuit_break>` 是分布式系统非常重要的一环。熔断可以让应用程序配置故障阈值，以确保最大限度的安全，允许组件快速失败然后尽可能快的恢复正常。使用正确的熔断阈值有助于节省那些浪费在请求等待（超时）或者非必要的请求重试中的资源。对于在 Envoy 中实现的熔断来讲 ，一个主要的优势是熔断限制可以被应用在网络层。
.. _common_configuration_transient_failures_retries:

重试
-------
自动的 :ref:`请求重试 <config_http_filters_router>` 是另外一种保证服务弹性的方法。请求重试通常应用于防范瞬时故障。Envoy 支持非常丰富的可配置参数来指定请求重试的类型、请求重试的次数以及请求重试的超时时间等。

gRPC 服务中的重试
------------------------
对于 gRPC 服务，Envoy 会在响应中查看 gRPC 状态，然后基于在 :ref:`x-envoy-retry-grpc-on <config_http_filters_router_x-envoy-retry-grpc-on>` 中配置的状态来尝试重试。

对于自动重试来讲，gRPC 中的下述应用程序状态码被认为是安全的。

* *CANCELLED* - 在服务中如果有一个可重试的错误，则返回此码。
* *RESOURCE_EXHAUSTED* - 如果服务所依赖的某些资源在该实例中耗尽，则重试至另外一个实例来获取帮助，在这种情况下返回此码。请注意对于共享资源的耗尽，返回此码将不会有任何帮助。应使用 :ref:`速率限制 <arch_overview_global_rate_limit>` 来处理这种情况。


HTTP 状态码 *502 (Bad Gateway)* 、*503 (Service Unavailable)* 及 *504 (Gateway Timeout)* 都和 gRPC 状态码 *UNAVAILABLE* 做了映射。针对自动重试来讲，这可被视为是安全的。


在配置重试时，请求的幂等性需要被重点考虑。

Envoy 同样支持对重启策略的扩展。:ref:`重试插件 <arch_overview_http_retry_plugins>` 允许为应用程序来定制 Envoy 重试实现。

异常检测
-----------------

:ref:`异常检测 <arch_overview_outlier_detection>` 是一种在上游集群中动态检测行为异常的主机的一种方法。通过探测这些主机并将它们从健康的负载均衡集中临时驱逐出去，Envoy 能够提高集群的成功率。Envoy 支持基于连续的 *5xx*、连续的网关失败和成功率来配置异常检测。

Envoy 也允许你配置驱逐周期。

**配置**

下述配置有助于优化：

* 常见场景下的最大请求成功率（比如，滚动升级）
* 速度
* 避免级联故障


*熔断*

.. code-block:: json

  {
     "thresholds": [
       {
         "max_retries": 10,
       }
    ]
  }


在这个特定用例中，需要配置上游集群的重试预算来开启并控制并发重试次数。如果配置的值过低，部分请求将不会被重试，可以通过 :ref:`upstream_rq_retry_overflow <config_cluster_manager_cluster_stats>` 来进行一个衡量。如果值配置的过高，服务可能会被重试请求淹没。

*异常检测*

.. code-block:: json

  {
     "consecutive_5xx": 5,
     "base_ejection_time": "30s",
     "max_ejection_percent": 50,
     "consecutive_gateway_failure": 5,
  }


如果有 5 个连续的 *5xx* 或者 *网关失败*，上述设置就会开启异常检测，而且会将驱逐的主机数量限制为上游集群大小的 50%。此配置对删除的主机数设置了安全限制。请注意，一旦一个主机被驱逐，它将会在一个驱逐周期（通常等于 *base_ejection_time* 与主机被驱逐次数的乘积）过后重新回到主机池。

*请求重试*

.. code-block:: json

  {
     "retry_on": "cancelled,connect-failure,gateway-error,refused-stream,reset,resource-exhausted,unavailable",
     "num_retries": 1,
     "retry_host_predicate": [
     {
        "name": "envoy.retry_host_predicates.previous_hosts"
     }
    ],
    "host_selection_retry_max_attempts": "5"
  }

请求将会基于在 *retry_on* 中配置的内容进行重试。此设置还将对 Envoy 进行配置，以使用 :ref:`前一个主机重试预测 <arch_overview_http_retry_plugins>` 来允许其选择与前一个失败请求不同的主机，因为同一台主机上的故障通常会持续一段时间，立即重试成功的概率较低。
