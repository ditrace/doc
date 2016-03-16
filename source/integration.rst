Integration
===========

.. _CSharp: https://github.com/ditrace/csharp

Distributed system's services should have an :doc:`/api/` client implementation.

The main requirement for clients is low load impact.

There are following techniques for achieve low impact:

- Sampling
- Ring buffering
- Async sending

Sampling
--------

There is no need to trace 100% of requests to get correct statistical results.

You can set sampling to 10% or even lower.

Ring buffering
--------------

Client should handle unavailability of "DiTrace" gate.
In the other hand, collecting of tracing data should not consume too much memory. 
Using ring buffer with certain limit is good practice to achieve that.

Async sending
-------------

Sending tracing data to the gate should be performed in async way, e.g. in separate thread.

C# client
---------

CSharp_ client utilize "Logical Call Context" to flow tracing data.

How to use
^^^^^^^^^^

1. Implement configuration provider

    .. code-block:: c#

       using Kontur.Tracing.Core.Config;

       public interface IConfigurationProvider
       {
           [NotNull]
           ITracingConfig GetConfig();
       }
    
2. Init tracing with your configuration provider

    .. code-block:: c#

       using Kontur.Tracing.Core.Config;

       Trace.Initialize(configProvider);

3. Create traces
 
    .. code-block:: c#
    
       using Kontur.Tracing.Core;
       

       using (var rootContext = Trace.CreateRootContext("Processing client request"))
       {
            rootContext.RecordTimepoint(Timepoint.Start);
            rootContext.RecordAnnotation(Annotation.RequestUrl, requestUrl);

            // ... somewhere deep in your code
            
            using (var childContext = Trace.CreateChildContext("Fetching data from database"))
            {
                childContext.RecordTimepoint(Timepoint.Start);
                data = db.Fetch();
                childContext.RecordTimepoint(Timepoint.Finish);
            }

            // ...
            rootContext.RecordTimepoint(Timepoint.Finish);
       }
       
4. Continue traces

    Assume that service A make a call to service B, so service B should continue tracing.

    .. code-block:: c#
    
       using Kontur.Tracing.Core;
       
       HttpListenerContext context;

       RequestExtensions.ExtractFromHttpHeaders(context.Request.Headers, out traceId, out contextId, out isActive);
       using (var serverContext = Trace.ContinueContext(traceId, contextId, isActive ?? false, isRoot: false))
       {
            serverContext.RecordTimepoint(Timepoint.ServerReceive);
            // ...
            // Handle service A request
            // ...
            serverContext.RecordTimepoint(Timepoint.ServerSend);
       }