Getting started
===============

Erlang is more than a language, it is also an operating system for your 
applications. Erlang developers rarely write standalone modules, they 
write libraries or applications, and then bundle those into what is 
called a release. A release contains the Erlang VM plus all 
applications required to run the node, so it can be pushed to 
production directly.

This chapter walks you through all the steps of setting up Cowboy, 
writing your first application and generating your first release. At 
the end of this chapter you should know everything you need to push 
your first Cowboy application to production.

Bootstrap
---------

We are going to use the 
[erlang.mk](https://github.com/ninenines/erlang.mk) build system. It 
also offers bootstrap features allowing us to quickly get started 
without having to deal with minute details.

First, let's create the directory for our application.

``` bash
$ mkdir hello_erlang
$ cd hello_erlang
```

Then we need to download `erlang.mk`. Either use the following command 
or download it manually.

``` bash
$ wget https://raw.githubusercontent.com/ninenines/erlang.mk/master/erlang.mk
```

We can now bootstrap our application. Since we are going to generate a 
release, we will also bootstrap it at the same time.

``` bash
$ make -f erlang.mk bootstrap bootstrap-rel
```

This creates a Makefile, a base application, and the release files 
necessary for creating the release. We can already build and start this 
release.

``` bash
$ make
...
$ ./_rel/hello_erlang_release/bin/hello_erlang_release console
...
(hello_erlang@127.0.0.1)1>
```

Entering the command `i().` will show the running processes, including 
one called `hello_erlang_sup`. This is the supervisor for our 
application.

The release currently does nothing. In the rest of this chapter we will 
add Cowboy as a dependency and write a simple "Hello world!" handler.

Cowboy setup
------------

To add Cowboy as a dependency to your application, you need to modify 
two files: the Makefile and the application resource file.

Modifying the Makefile allows the build system to know it needs to 
fetch and compile Cowboy. To do that we simply need to add one line to 
our Makefile to make it look like this:

``` Makefile
PROJECT = hello_erlang
DEPS = cowboy
include erlang.mk
```

Modifying the application resource file, `src/hello_erlang.app.src`, 
allows the build system to know it needs to include Cowboy in the 
release and start it automatically. This is a different step because 
some dependencies are only needed during development.

We are simply going to add `cowboy` to the list of `applications`, 
right after `stdlib`. Don't forget the comma separator.

``` erlang
{application, hello_erlang, [
	{description, "Hello Erlang!"},
	{vsn, "0.1.0"},
	{modules, []},
	{registered, []},
	{applications, [
		kernel,
		stdlib,
		cowboy
	]},
	{mod, {hello_erlang_app, []}},
	{env, []}
]}.
```

You may want to set a description for the application while you are 
editing the file.

If you run `make` now and start the release, Cowboy will be included 
and started automatically. This is not enough however, as Cowboy 
doesn't do anything by default. We still need to tell Cowboy to listen 
for connections.

Listening for connections
-------------------------

We will do this when our application starts. It's a two step process. 
First we need to define and compile the dispatch list, a list of routes 
that Cowboy will use to map requests to handler modules. Then we tell 
Cowboy to listen for connections.

Open the `src/hello_erlang_app.erl` file and add the necessary code to 
the `start/2` function to make it look like this:

``` erlang
start(_Type, _Args) ->
	Dispatch = cowboy_router:compile([
		{'_', [{"/", hello_handler, []}]}
	]),
	cowboy:start_http(my_http_listener, 100, [{port, 8080}],
		[{env, [{dispatch, Dispatch}]}]
	),
	hello_erlang_sup:start_link().
```

The dispatch list is explained in great details in the 
[Routing](routing) chapter. For this tutorial we map the path `/` to 
the handler module `hello_handler`. This module doesn't exist yet, we 
still have to write it.

If you build the release, start it and open 
[http://localhost:8080](http://localhost:8080) now, you will get an 
error because the module is missing. Any other URL, like 
[http://localhost:8080/test](http://localhost:8080/test), will result 
in a 404 error.

Handling requests
-----------------

Cowboy features different kinds of handlers, including REST and 
Websocket handlers. For this tutorial we will use a plain HTTP handler.

First, let's generate a handler from a template.

``` bash
$ make new t=cowboy_http n=hello_handler
```

You can then open the `src/hello_handler.erl` file and modify the 
`handle/2` function like this to send a reply.

``` erlang
handle(Req, State=#state{}) ->
	{ok, Req2} = cowboy_req:reply(200,
		[{<<"content-type">>, <<"text/plain">>}],
		<<"Hello Erlang!">>,
		Req),
	{ok, Req2, State}.
```

What the above code does is send a `200 OK` reply, with the 
`content-type` header set to `text/plain` and the response body set to 
`Hello Erlang!`.

If you build the release, start it and open 
[http://localhost:8080](http://localhost:8080) in your browser, you 
should get a nice `Hello Erlang!` displayed!
