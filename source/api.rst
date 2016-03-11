DiTrace gate API
================

Gate accept only the following http request.

.. http:post:: /spans?system=(string)

   System parameters is used if no system annotation in span json

   **Example request**:

   .. sourcecode:: http

      POST /spans?system=mysystem HTTP/1.1
      Content-Type: application/x-ldjson
      
      {
        "traceId":"c38efe4edb2d4a008af2805ee4e061c1",
        "spanId":"8256",
        "timeline":{
            "sr":"2015-04-24T09:53:49.5595869Z",
            "ss":"2015-04-24T09:53:50.5595869Z"
        },
        "annotations":{
            "url":"/url?arg1=arg1&arg2=arg2",
            "host":"hostname",
            "rqbl":"42",
            "rsbl":"4200",
            "targetId":"service-0"
        }
       }\r\n
       {
        "traceId":"c38efe4edb2d4a008af2805ee4e061c1",
        "parentSpanId":"8256",
        "spanId":"904a",
        "timeline":{
            "sr":"2015-04-24T09:53:49.5595869Z",
            "ss":"2015-04-24T09:53:50.5595869Z"
        },
        "annotations":{
            "url":"/url",
            "host":"hostname",
            "rqbl":"42",
            "rsbl":"4200",
            "targetId":"service-1"
        }
       }

   **Example response**:

   .. sourcecode:: http

      HTTP/1.1 200 OK

   :query system: spans source system name
   :statuscode 200: no error
   :statuscode 400: server can't parse request content or any of required field is missing (system, traceid, spanid)

.. important:: For better visibility example's jsons are formatted with additional new lines.
               Only the new lines that separate jsons should be in real requests.

Span format
^^^^^^^^^^^

Field, preferrable format, description

- (required) **TraceId**, UUID, unique identifier to group multiple spans into one trace
- (required) **SpanId**, part of UUID, unique identifier of span within one trace
- (required) **Annotations**, object of arbitrary annotations
- (required) **Timeline**, object of timestamps annotations
- (optional) **ParentSpanId**, identifier of parent span
- (optional) **ProfileId**, UUID, unique identifier to group multiple traces
- (optional) **System**, string, unique identifier of distributed system

Known annotations
^^^^^^^^^^^^^^^^^

UI counts on following annotations

- (required) **url** string (/path) 	 
- (optional) **url_method** string (POST, GET, DELETE, etc)
- (required) **host**	hostname string
- (required) **targetId**	string (microservice unique name)
- (optional) **rc**	int	(response code)
- (optional) **rqbl** long (request body length)
- (optional) **rsbl** long (response body length)
- (optional) **wrapper**	empty string (marks span as a wrapper of child spans to override targetid)
- (optional) **root**	empty string (marks span as a root span) 
- (optional) **revision**	int	(revision number to overwrite old annotations value)

Timeline annotations
^^^^^^^^^^^^^^^^^^^^

DiTrace gate and UI are using this timeline annotations to calculate trace and spans durations.
All timeline annotations should have RFC3389 datetime format.

- (optional) **cs** client has sent request
- (optional) **cr** client has recived response
- (optional) **sr** server has received request
- (optional) **ss** server has sent response