.. _faq_deprecation:

如何处理配置弃用？
===========================================

如 :repo:`CONTRIBUTING.md` 的“变更策略”中所述， 只要有可用的替换项，功能就可以在任何时候被标记为已弃用。每种弃用都在 API proto 文件中进行了注释，并且在 :ref:`Envoy 文档 <deprecated>` 进行了详细的说明。

在弃用后的前三个月中，使用弃用的字段将会在结果中打印出 warning 的日志，并且累加 :ref:`deprecated_feature_use <runtime_stats>` 的计数器。
此后，这个字段被默认注释为 fatal ，进一步该字段会被视为无效配置，除非 :ref:`runtime overrides <config_runtime_deprecation>` 被用于重启使用。
