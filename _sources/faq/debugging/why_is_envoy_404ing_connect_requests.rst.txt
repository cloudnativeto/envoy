.. _faq_why_is_envoy_404ing_connect_requests:

Envoy 为什么会发送 404 给连接请求？
==============================================

Envoy 的默认匹配器是基于主机和路径进行匹配。
由于 CONNECT 请求（通常）不会携带路径，所以大多数匹配器去匹配 CONNECT 请求都会失败，而 Envoy 响应 404 是由于未找到对应的路由。
HTTP/1.1 CONNECT 请求的解决办法是使用 :ref:`connect_matcher <envoy_v3_api_msg_config.route.v3.RouteMatch.ConnectMatcher>` ，如 :ref:`升级文档<arch_overview_upgrades>` 的 CONNECT 部分中所述。
