# Processing oversized requests

Each API provider has its own limits on the size of the request. These limits vary a lot across different providers, but most of the cases refer to one of the following:

- **max-item-length** is a maximum size of text segment in a request.
- **max-array-length** is a maximum number of segments in array in a request.
- **max-total-length** is a maximum total length of all segments in a request.
- **max-request-length** is the maximum total size of a request in bytes.

To process data which hits any of those limits, you may either split it into a number of requests by yourself or enable the [Intento async mode](https://github.com/intento/intento-api#async-mode), sit back and relax.

In the async mode, if the request fits in all the limits of the selected API provider, it is sent to the provider in a single request, much like it happens in the synchronous mode.

If the request contains an array which violates any of these three limits: max-array-length, max-request-length or max-total-length, we split it into multiple small sub-arrays. Those sub-arrays are sent to the provider concurrently and merged when the async job is completed.

Another situation is when the original request exceeds max-item-length. It may be a single large text segment (e.g. html or xml) that breaks limits or any number of such large segments in the array. We use a specific text segmentation tool to split those large segments into smaller ones, concurrently send them for the processing and then compile results in the original order.

Please note that formatted text is correctly processed only by providers that support formatted files and the file format should be set in the request as [described in the Intento API docs](https://github.com/intento/intento-api/blob/master/ai.text.translate.md#supported-formats).

One of the subrequests may fail due to sporadic or persistent errors on the provider API. In such case, we make three retries for each such subrequest. If it still failed, the whole large request is returned with the “failed” status.  


