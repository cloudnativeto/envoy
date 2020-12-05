.. _install_sandboxes_jaeger_native_tracing:

Jaeger 原生跟踪
================

Jaeger 跟踪沙盒使用 `Jaeger <https://jaegertracing.io/>`_ 作为跟踪提供程序，并使用 Jaeger 的原生
`C++ 客户端 <https://github.com/jaegertracing/jaeger-client-cpp>`_ 
作为插件，演示了 Envoy 的  :ref:`请求跟踪 <arch_overview_tracing>`  功能。 使用 Jaeger 及其原生客户端而不是Envoy内置的Zipkin客户端具有以下优点：

- 跟踪传播将与 Jaeger 的其他服务一起使用，而无需进行配置 `更改 <https://github.com/jaegertracing/jaeger-client-go#zipkin-http-b3-compatible-header-propagation>`_。
- 可以使用各种不同的 `采样策略 <https://www.jaegertracing.io/docs/sampling/#client-sampling-configuration>`_
  ，包括概率采样或远程采样，可以从 Jaeger 的后端集中控制采样。
- 跨度以更有效的二进制编码发送到收集器。


该沙盒与前面描述的 front proxy 体系结构非常相似，但有一个区别： service1 在返回响应之前对 service2 进行 API 调用。 这三个容器将部署在称为 ``envoymesh`` 的虚拟网络中。

.. 注意::

   Jaeger 原生跟踪只能工作在 x86-64 架构.

所有的传入请求都通过前端 Envoy 路由，该前端 Envoy 是作为位于 ``envoymesh`` 网络边缘的反向代理。``8000`` 端口由 docker compose 公开（请参阅 :repo:`/examples/jaeger-native-tracing/docker-compose.yaml`)。请注意，所有 Envoy 都配置为收集请求的跟踪（例如， :repo:`/examples/jaeger-native-tracing/front-envoy-jaeger.yaml` 中的 http_connection_manager/config/tracing 设置) 并进行设置以传播 Jaeger 所生成的跨度(spans)，以及跟踪到 Jaeger 集群（在 :repo:`/examples/jaeger-native-tracing/front-envoy-jaeger.yaml`  中跟踪驱动程序设置）。

在将请求路由到适当的服务 Envoy 或应用程序之前，Envoy 将负责生成用于跟踪的适当跨度（父/子上下文跨度）。在较高层级上，每个跨度都记录上游 API 调用的延迟以及将跨度与其他相跨度相关联的所需信息（例如，跟踪 ID）。

从 Envoy 进行跟踪的最重要好处之一是，它将负责将跟踪信息传播到 Jaeger 服务集群。但是，为了充分利用跟踪的优势，应用程序必须传播 Envoy 生成的跟踪头部信息，同时调用其他服务。在我们提供的沙盒当中，有个简单 flask 应用程序（请参阅 :repo:`/examples/front-proxy/service.py` 中的跟踪功能 ）充当 service1 的传播跟踪标头，同时对 service2 进行出站调用。


运行沙盒
~~~~~~~~~

.. include:: _include/docker-env-setup.rst

步骤 3: 构建沙盒
***************

要构建这个沙盒示例，并启动示例应用程序，请运行以下命令:


.. code-block:: console

    $ pwd
    envoy/examples/jaeger-native-tracing
    $ docker-compose pull
    $ docker-compose up --build -d
    $ docker-compose ps

                Name                              Command                State                                                      Ports
    -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    jaeger-native-tracing_front-envoy_1   /start-front.sh                Up      10000/tcp, 0.0.0.0:8000->8000/tcp, 0.0.0.0:8001->8001/tcp
    jaeger-native-tracing_jaeger_1        /go/bin/all-in-one-linux - ... Up      14250/tcp, 14268/tcp, 0.0.0.0:16686->16686/tcp, 5775/udp, 5778/tcp, 6831/udp, 6832/udp, 0.0.0.0:9411->9411/tcp
    jaeger-native-tracing_service1_1      /start-service.sh              Up      10000/tcp, 8000/tcp
    jaeger-native-tracing_service2_1      /start-service.sh              Up      10000/tcp, 8000/tcp

步骤 4: 生成一切负载
*******************

你现在可以通过 front-envoy 向 service1 发送以下请求:

.. code-block:: console

    $ curl -v localhost:8000/trace/1
    *   Trying 192.168.99.100...
    * Connected to 192.168.99.100 (192.168.99.100) port 8000 (#0)
    > GET /trace/1 HTTP/1.1
    > Host: 192.168.99.100:8000
    > User-Agent: curl/7.54.0
    > Accept: */*
    >
    < HTTP/1.1 200 OK
    < content-type: text/html; charset=utf-8
    < content-length: 89
    < x-envoy-upstream-service-time: 9
    < server: envoy
    < date: Fri, 26 Aug 2018 19:39:19 GMT
    <
    Hello from behind Envoy (service 1)! hostname: f26027f1ce28 resolvedhostname: 172.19.0.6
    * Connection #0 to host 192.168.99.100 left intact

步骤 5: 在 Jaeger 图形界面中查看跟踪
************************************

用你的浏览器打开 http://localhost:16686 。 你应该可以看到 Jaeger 的仪表盘。
设置服务为 "front-proxy" 然后点击 'Find Traces'. 你应该看到 front-proxy 的跟踪信息。
单击跟踪，以查看从 front-proxy 到 service1 到 service2 的请求所采取的路径，以及每次跳转时引起的延迟。
