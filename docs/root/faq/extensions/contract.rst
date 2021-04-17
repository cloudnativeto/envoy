.. _faq_filter_contract:

我的 HTTP 过滤器必须遵守合约吗？
-----------------------------------

* 标头编码/解码

  * 在标头编码/解码期间，如果过滤器返回 ``FilterHeadersStatus::StopIteration``，则如果 ``encodeData()``/``decodeData()`` 返回 ``FilterDataStatus::Continue`` 或者通过显式调用 ``continueEncoding()``/``continueDecoding()`` 可以恢复处理。

  * 在标头编码/解码期间，如果过滤器返回 ``FilterHeadersStatus::StopAllIterationAndBuffer`` 或者 ``FilterHeadersStatus::StopAllIterationAndWatermark``，可以通过调用 ``continueEncoding()``/``continueDecoding()`` 可以恢复处理。

  * 在将 ``end_stream`` 设置为 *true* 的调用时，过滤器的 ``decodeHeaders()`` 实现不得返回 ``FilterHeadersStatus::ContinueAndEndStream``。在这种情况下应该返回 ``FilterHeadersStatus::Continue``。

  * 过滤器的 ``encode100ContinueHeaders()`` 必须返回 ``FilterHeadersStatus::Continue`` 或者 ``FilterHeadersStatus::StopIteration``。

* 数据编码/解码

  * 在数据编码/解码期间，如果过滤器返回 ``FilterDataStatus::StopIterationAndBuffer``、``FilterDataStatus::StopIterationAndWatermark``，或者``FilterDataStatus::StopIterationNoBuffer``，则如果 ``encodeData()``/``decodeData()`` 返回 ``FilterDataStatus::Continue`` 或者通过显式调用 ``continueEncoding()``/``continueDecoding()`` 可以恢复处理。

* Trailer 编码/解码

  * 在 Trailer 编码/解码期间，如果过滤器返回 ``FilterTrailersStatus::StopIteration``，通过显式调用 ``continueEncoding()``/``continueDecoding()`` 可以恢复处理。

Are there well-known headers that will appear in the given headers map of ``decodeHeaders()``?
是否存在将出现在 ``decodeHeaders()`` 给定标头映射中的知名标头？
---------------------------------------------------------------

The first filter of the decoding filter chain will have the following headers in the map:
解码过滤器链的第一个过滤器在映射中将具有以下标头：

* ``Host``
* ``Path`` （对于CONNECT请求可能会省略）。

尽管可以通过解码过滤器链上的任一过滤器省略这些标头，但应在触发终端过滤器之前重新插入它们。
