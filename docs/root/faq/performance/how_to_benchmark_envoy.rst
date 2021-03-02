Envoy 基准测试的最佳实践
===============================================

没有 :ref:`单一的QPS，延迟或吞吐量的开销 <faq_how_fast_is_envoy>` 可以表征一个像 Envoy 这样的网络代理。取而代之的是，任何测量都需要考虑上下文，通过适当地配置和负载测试 Envoy 来确保与其他系统进行逐一比较。因此，我们无法提供通用的基准测试配置，而是提供以下指导：

* 应该使用发行版的 Envoy 二进制文件。如果要构建，请确保在 Bazel 命令行上使用 `-c opt`。使用 Envoy 发行版本时，请确保你使用的是最新发行版本。考虑到 Envoy 的开发速度，在声明 Envoy 的性能时选择较旧的版本是不合理的。同样，如果要处理主版本，请进行尽职调查，并确保没有任何回归或性能改进降到你的基准测试附近，并且你与 HEAD 接近。

* 这个 :option:`--concurrency` Envoy CLI 参数应该是未设置（你的机器上提供每个逻辑核心的一个工作线程）或设置为与在你的对照组中可用于其他网络代理的核心/线程数匹配。

* 禁用 :ref:`熔断器 <faq_disable_circuit_breaking>`。基准测试期间的一个常见问题是 Envoy 的默认断路器限制很低，从而导致连接和请求排队。

* 禁用 :ref:`generate_request_id
  <envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.generate_request_id>`。

* 禁用 :ref:`dynamic_stats <envoy_v3_api_field_extensions.filters.http.router.v3.Router.dynamic_stats>`。如果要测量开销与直接连接的开销，则可能需要考虑通过 :ref:`reject_all <envoy_v3_api_field_config.metrics.v3.StatsMatcher.reject_all>` 禁用所有统计信息。

* 确保网络和 HTTP 过滤器链反映了与 Envoy 进行比较的系统中的可比较功能。

* 确保 TLS 设置（如果有）是现实的，并且在任何比较中都使用一致的密码。会话重用可能会对结果产生重大影响，应通过 :ref:`监听器SSL统计信息 <config_listener_stats>` 进行跟踪。

* 确保 :ref:`HTTP/2 设置 <envoy_v3_api_msg_config.core.v3.Http2ProtocolOptions>`（尤其是那些影响流控制和流并发的设置）在任何对照组中都是一致的。在优化任何 HTTP/2 设置时，理想情况下应考虑 BDP 和网络链接延迟。

* 在监听器和群集统计信息中验证流，连接和错误的数量是否与任何给定实验中预期的数量匹配。

* 确保你了解由负载生成器创建的连接如何在 Envoy 工作线程之间分配。这对于使用较少连接数和完美 keep-alive 的基准测试尤其重要。你应该意识到， Envoy 会将给定连接的所有流分配给单个工作线程。这意味着，例如如果你有 72 个逻辑核心和工作线程，但是只有来自负载生成器的单个 HTTP/2 连接，则只有 1 个工作线程处于活动状态。

* 确保请求发布时间预期与预期目标保持一致。一些负载生成器会自然地产生抖动和/或分批的时序。在某些测试中，这可能最终成为意想不到的主导因素。

* 负载生成器如何重用连接的细节是一个重要因素（例如 MRU，随机，LRU 等），因为这会影响工作分配。

* 如果要测量较小的延迟（例如< 1 ms），请确保测量工具和环境具有所需的灵敏度，并且本底噪声足够低。

* 你要重视引导程序或 xDS 配置。理想情况下，每条线都有动机，并且对于所考虑的基准测试来说是必要的。

* 考虑将 `Nighthawk <https://github.com/envoyproxy/nighthawk>`_ 用作负载生成器和测量工具。我们致力于在此工具中建立基准测试和延迟测量最佳实践。

* 在基准测试程序运行时检查 Envoy 的 `perf` 文件，例如使用 `火焰图 <http://www.brendangregg.com/flamegraphs.html>`_ 。确认 Envoy 正在花费时间进行预期的测试中的重要工作，而不是一些无关或正交的工作。

* 熟悉 `延迟测量最佳实践 <https://www.youtube.com/watch?v=lJ8ydIuPFeU>`_。特别是，不要测量最大负载下的延迟时间，这通常没有意义，也不能反映真实的系统性能。旨在测量 QPS 延迟曲线的拐点以下。优先选择开环负载生成器与闭环负载生成器。

* 避免 `基准测试五宗罪 <https://www.cse.unsw.edu.au/~gernot/benchmarking-crimes.html>`_。
