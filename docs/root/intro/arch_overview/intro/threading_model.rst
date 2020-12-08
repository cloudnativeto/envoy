.. _arch_overview_threading:

线程模型
===============

Envoy 使用单进程-多线程架构。一个 *primary* 线程处理各种轻量协调任务，同时多个 *worker* 线程处理监听、过滤、转发。
当一个连接被监听接受，连接的剩余生命周期将绑定在当前 *worker* 线程。这使得 Envoy 大部分代码近似单线程运行（高度并行），
只有少量的复杂代码用于实现 *worker* 线程之间的协调。Envoy 基本实现了 100% 的非阻塞，对于大多数工作负载，
我们建议将 *worker* 线程数配置为物理机器的线程数。

监听器连接均衡
-----------------------------

默认情况下，*worker* 线程之间没有协调。这表示所有 *worker* 线程都独立的尝试从每一个监听器接受连接，
然后依赖内核在线程之间执行适当的均衡。对于大多数工作负载，内核都可以很好的均衡传入的连接。但是，
对于某些工作负载，特别是那些只有少量但非常长寿命的连接的工作负载（例如，服务网格 HTTP2/gRPC egress），
则可能希望让 Envoy 强制平衡工作线程之间的连接。为了支持这种行为，
Envoy 允许为每个 :ref:`监听器 <arch_overview_listeners>` 配置不同类型的 :ref:`连接均衡 <envoy_v3_api_field_config.listener.v3.Listener.connection_balance_config>`。