.. _faq_disable_circuit_breaking:

有办法禁用熔断吗？
===========================================

对于每种类型的熔断，Envoy 都带有 :ref:`特定的默认值 <envoy_v3_api_msg_config.cluster.v3.CircuitBreakers.Thresholds>`。当前，还没有开关能完全关闭熔断机制； 然而， 你可以通过将这些阈值设置非常高来实现类似的效果，例如，将它们设为`std::numeric_limits<uint32_t>::max()`。

以下是一个示例配置，该配置尝试通过将阈值设置为 1000000000 来有效地禁用所有类型的熔断。

.. code-block:: yaml

  circuit_breakers:
    thresholds:
      - priority: DEFAULT
        max_connections: 1000000000
        max_pending_requests: 1000000000
        max_requests: 1000000000
        max_retries: 1000000000
      - priority: HIGH
        max_connections: 1000000000
        max_pending_requests: 1000000000
        max_requests: 1000000000
        max_retries: 1000000000

Envoy 在路由级别支持优先级路由。你可以相应地调整阈值。
