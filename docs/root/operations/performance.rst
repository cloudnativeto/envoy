.. _operations_performance:

性能
===========

Envoy旨在通过在 :ref: 少量线程<arch_overview_threading> 上运行事件循环来优化可伸缩性和资源利用。
它的主线程负责控制平面的处理，每一个 worker 线程负责负责数据平面处理的一部分。
Envoy公开了两个统计指标，以监视所有这些线程上事件循环的性能。

* **循环持续时间:** 事件循环的每次迭代都会进行一定数量的处理。该数量会自然的随着负载的变化而变化。但是，如果一个或多个线程的循环持续时间异常长，则可能表示一个性能问题。例如，工作可能无法在工作线程之间平均分配，或者扩展程序中可能存在长时间的阻塞操作，从而阻碍了进度。

* **轮询延迟:** 在事件循环的每次迭代中，事件分派器都会轮询I/O事件，并在准备好要处理某些I/O事件或发生超时时（以先到者为准）“唤醒”。如果发生超时，我们可以测量预期唤醒时间与轮询后的实际唤醒时间之间的差，这种差异称为“轮询延迟”。正常情况下会看到一些小的轮询延迟，通常等于内核调度程序的“时间片”或“量子”，这取决于 Envoy 在其上运行的特定操作系统，但是，如果该数目大大高于正常情况下观察到的基线值，则可能表明内核调度程序延迟。

这些统计信息可以通过设置 :ref: enable_dispatcher_stats 
<envoy_v3_api_field_config.bootstrap.v3.Bootstrap.enable_dispatcher_stats> 为 true 来启用

.. 警告::
  请注意，启用调度程序统计信息会为每个线程上的事件循环的每次迭代记录一个值。
  通常这应该是最小的开销，但是当使用 :ref: statsd <envoy_v3_api_msg_config.metrics.v3.StatsdSink> 
  时,因为statsd协议无法表示直方图摘要，所以它将通过电线单独发送每个观测值。
  请注意，这可能是非常大量的数据。

事件循环统计
---------------------

主线程的事件分配器的统计树植根于服务器，分派器和每个工作线程的事件分派器都有一个统计树，该树根植于
*listener_manager.worker_ <id>.dispatcher.* ，每个都有以下统计信息：

.. csv-table::
  :header: 名称, 类型, 描述
  :widths: 1, 1, 2

  loop_duration_us, 直方图, 事件循环持续时间（以微秒为单位）
  poll_delay_us, 直方图, 轮询延迟（以微秒为单位）

请注意，此处不包括任何辅助线程。

.. _operations_performance_watchdog:

监视器
--------
除了事件循环统计信息外，Envoy还包括一个可配置的:ref: 监视器 
<envoy_v3_api_field_config.bootstrap.v3.Bootstrap.watchdogs> 系统，该系统可以在 Envoy 不响应时增加
统计信息并有选择地杀死服务器。此系统具有两个单独的看门狗配置，一个用于主线程，一个用于工作线程。
这是有用的，因为不同的线程有不同的工作负载。该系统还有一个扩展点，允许根据看门狗事件采取自定义操作。
这些统计信息对于从高层次了解 Envoy 的事件循环是否没有响应是有用的，这是因为它进行了过多的工作，正在
阻塞或未被操作系统调度。

监视器在 main_thread 和 worker 中都发出聚合的统计信息。另外，它在服务器下发出单独的统计信息
<thread_name> 树 <thread_name>等于 main_thread，worker_0，worker_1等。

.. csv-table::
  :header: 名称, 类型, 描述
  :widths: 1, 1, 2

  watchdog_miss, Counter, 标准未中次数
  watchdog_mega_miss, Counter, 大型未命中数
