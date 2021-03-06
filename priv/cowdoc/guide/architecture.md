Architecture
============

Cowboy is a lightweight HTTP server.

It is built on top of Ranch. Please see the Ranch guide for more 
information.

One process per connection
--------------------------

It uses only one process per connection. The process where your code 
runs is the process controlling the socket. Using one process instead 
of two allows for lower memory usage.

Because there can be more than one request per connection with the 
keepalive feature of HTTP/1.1, that means the same process will be used 
to handle many requests.

Because of this, you are expected to make sure your process cleans up 
before terminating the handling of the current request. This may 
include cleaning up the process dictionary, timers, monitoring and more.

Binaries
--------

It uses binaries. Binaries are more efficient than lists for 
representing strings because they take less memory space. Processing 
performance can vary depending on the operation. Binaries are known for 
generally getting a great boost if the code is compiled natively. 
Please see the HiPE documentation for more details.

Date header
-----------

Because querying for the current date and time can be expensive, Cowboy 
generates one `Date` header value every second, shares it to all other 
processes, which then simply copy it in the response. This allows 
compliance with HTTP/1.1 with no actual performance loss.

Max connections
---------------

By default the maximum number of active connections is set to a 
generally accepted big enough number. This is meant to prevent having 
too many processes performing potentially heavy work and slowing 
everything else down, or taking up all the memory.

Disabling this feature, by setting the `{max_connections, infinity}` 
protocol option, would give you greater performance when you are only 
processing short-lived requests.
