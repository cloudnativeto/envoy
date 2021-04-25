如何配置 Envoy 去使用 v3 API?
===========================================

默认情况下，Envoy 会试着将所有 YAML、JSON 或 文本格式的原型转换为 v2，如果转换失败，将考虑转换成 v3。
所以，如果你有一个简单的、基于文本引导程序的静态 Envoy，你只需要开始使用新的配置。二进制原型 bootstrap 配置，
请使用原型 :ref:`v3 Bootstrap <envoy_v3_api_msg_config.bootstrap.v3.Bootstrap>`。

动态配置，我们已经介绍过 :ref:`配置来源 <envoy_v3_api_msg_config.core.v3.ConfigSource>` 的两个字段，
传输 API 版本和资源 API 版本。区别如下：

* :ref:`传输 API 版本
  <envoy_v3_api_field_config.core.v3.ApiConfigSource.transport_api_version>` 表示 API 的端点和
  *DiscoveryRequest*/*DiscoveryResponse* 信息使用的版本。

* :ref:`资源 API 版本
  <envoy_v3_api_field_config.core.v3.ConfigSource.resource_api_version>` 表示 v2 还是 v3 版本的资源，
  例如：发送的是 v2 *RouteConfiguration* 或 v3 *RouteConfiguration*。

可以混合使用传输 API 和资源 API 版本，例如：基于 v2 ADS 传输，发送一个 v2 的
*监听器* 资源和 v3 的 *RouteConfiguration* 资源。 该特性用于 Envoy 的部署逐渐从 v2 向 v3 迁移。

通过适当地配置 :ref:`配置来源
<envoy_v3_api_msg_config.core.v3.ConfigSource>` 我们提供了在 vN 版本端点发送 vM 版本资源的操作优势和调用的灵活性。