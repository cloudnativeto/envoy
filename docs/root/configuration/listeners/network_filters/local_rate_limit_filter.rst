.. _config_network_filters_local_rate_limit:

本地限流
================

* 本地限流 :ref:`architecture overview <arch_overview_local_rate_limit>`
* :ref:`v3 API 参考 <envoy_v3_api_msg_extensions.filters.network.local_ratelimit.v3.LocalRateLimit>`
* 过滤器应该以 *envoy.filters.network.local_ratelimit* 的名称进行配置。

.. note::
  全局限流是通过 :ref:`全局限流过滤器 <config_network_filters_rate_limit>` 进行支持的。

总述
------

本地限流过滤器将 :ref:`token 桶 <envoy_v3_api_field_extensions.filters.network.local_ratelimit.v3.LocalRateLimit.token_bucket>` 速率限制应用于由过滤器的过滤器链处理的传入连接。
过滤器处理的每个连接都会使用一个 token，如果连接中没有可用的 token，那么连接将会被立刻关闭，而不需要进一步的迭代过滤器。

.. note::
  在当前实现的每一个过滤器链中都有一个独立的限流过滤器。

.. _config_network_filters_local_rate_limit_stats:

统计
------

每个本地限流过滤器的配置都有一个基于 *local_ratelimit.<stat_prefix>.* 如下所示的统计信息：

.. csv-table::
  :header: 名称, 类型, 描述
  :widths: 1, 1, 2

  rate_limited, Counter, 因超出速率限制而被关闭的连接总数

运行时
-------

本地限流过滤器可以通过 :ref:`启用 <envoy_v3_api_field_extensions.filters.network.local_ratelimit.v3.LocalRateLimit.runtime_enabled>` 的配置字段标记运行时功能。
