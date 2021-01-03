
.. _config_http_filters_oauth:

OAuth2
======

* :ref:`v3 API 参考 <envoy_v3_api_msg_extensions.filters.http.oauth2.v3alpha.OAuth2>`
* 这个过滤器使用配置项 *envoy.filters.http.oauth2* 来配置。

.. attention::

  OAuth2 过滤器正在活跃开发中。

示例配置
---------------------

.. code-block::

   http_filters:
   - name: oauth2
     typed_config:
       "@type": type.googleapis.com/envoy.extensions.filters.http.oauth2.v3alpha.OAuth2
       token_endpoint:
         cluster: oauth
         uri: oauth.com/token
         timeout: 3s
       authorization_endpoint: https://oauth.com/oauth/authorize/
       redirect_uri: "%REQ(:x-forwarded-proto)%://%REQ(:authority)%/callback"
       redirect_path_matcher:
         path:
           exact: /callback
       signout_path:
         path:
           exact: /signout
      credentials:
        client_id: foo
        token_secret:
          name: token
        hmac_secret:
          name: hmac
      timeout: 3s
   - name: envoy.router

  clusters:
  - name: service
    ...
  - name: auth
    connect_timeout: 5s
    type: LOGICAL_DNS
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: auth
      endpoints:
      - lb_endpoints:
        - endpoint:
            address: { socket_address: { address: auth.example.com, port_value: 443 }}
    tls_context: { sni: auth.example.com }

注意
-----

这个模块目前还没有为重定向到 OAuth 服务器并且返回这一循环提供跨站点请求伪造（Cross-Site-Request-Forgery）保护。

服务必须基于 HTTPS，这样过滤器才能工作，同时 cookies 使用 `;secure`。

统计
----------

OAuth 过滤器在 *<stat_prefix>.* 命名空间下输出统计信息。

.. csv-table::
  :header: 名称, 类型, 描述
  :widths: 1, 1, 2

  oauth_failure, Counter, 总的拒绝请求数。
  oauth_success, Counter, 总的允许请求数。
  oauth_unauthorization_rq, Counter, 总的未授权请求数。
