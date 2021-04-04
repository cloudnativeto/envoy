.. _common_configuration_zone_aware_routing:

如何配置区域感知路由？
======================================

在源服务（"cluster_a"）和目的服务（"cluster_b"）之间启用 :ref:`区域感知路由 <arch_overview_load_balancing_zone_aware_routing>` 需要经过几个必要的步骤。

在源服务端配置 Envoy
-----------------------------------------
这部分描述了与源服务并行运行的 Envoy 的特定配置。具有下述要求：

* Envoy 必须以 :option:`--service-zone` 选项启动，此选项定义了当前主机的区域。
* 源集群和目的集群的定义必须有 :ref:`EDS <envoy_v3_api_field_config.cluster.v3.Cluster.type>` 类型 。
* :ref:`local_cluster_name <envoy_v3_api_field_config.bootstrap.v3.ClusterManager.local_cluster_name>` 必须被设置为源集群相同。

  对于群集管理器，下面的配置中只列出了必要部分。

.. code-block:: yaml

  cluster_manager:
    local_cluster_name: cluster_a
  static_resources:
    clusters:
    - name: cluster_a
      type: EDS
      eds_cluster_config: ...
    - name: cluster_b
      type: EDS
      eds_cluster_config: ...

在目的服务端配置 Envoy
----------------------------------------------
没有必要与目的服务并行运行 Envoy，但是目标集群中的每个主机都要向 :ref:`源服务 Envoy 查询 <config_overview_management_server>` 的服务发现进行注册。:ref:` 区域 <envoy_v3_api_msg_config.endpoint.v3.LocalityLbEndpoints>` 信息必须作为响应的一部分能够被获取到。

下面的响应只列出了与区域相关的数据。

.. code-block:: yaml

  locality:
    zone: us-east-1d

基础设施设置
--------------------
上述的配置对于区域感知路由是非常必要的，但是在某些特定情况下，区域感知路由是 :ref:`不被执行的 <arch_overview_load_balancing_zone_aware_routing_preconditions>`。

验证步骤
------------------
* 使用 Envoy 的 :ref:`每一个区域 <config_cluster_manager_cluster_per_az_stats>` 统计来监控跨区域流量。
