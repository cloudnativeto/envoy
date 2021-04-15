API 版本控制如何与新扩展交互？
======================================================

对于扩展程序配置 API， 请按照样式指南中的 :repo:`新扩展程序配置步骤
<api/STYLE.md#adding-an-extension-configuration-to-the-api>` 进行操作。

扩展实现应在内部使用 v3 messages 进行操作，以实现其自身的配置和其他 Envoy 的配置 messages 。单元测试应针对 v3 配置进行编写。
