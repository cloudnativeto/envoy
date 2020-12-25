.. _config_http_conn_man_route_table_route_matching:

路由匹配
========= 

当 Envoy 匹配到一条路由时，它使用如下流程：

#. HTTP 请求的 *host* 和 *:authority* 头部会和一个 :ref:`虚拟主机 
   <envoy_v3_api_msg_config.route.v3.VirtualHost>` 相匹配。
#. 虚拟主机中的每一个 :ref:`路由入口 <envoy_v3_api_msg_config.route.v3.Route>` 都会 *按顺序地* 逐个被检查。
   如果匹配到了，则使用此路由且不再做其他路由检查。
#. 独立地，虚拟主机中的每一个 :ref:`虚拟集群 <envoy_v3_api_msg_config.route.v3.VirtualCluster>` 
   都会 *按顺序地* 被逐个检查。如果匹配到了，则使用此虚拟集群且不再做其他虚拟集群检查。
