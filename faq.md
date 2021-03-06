Name
====

OpenResty Frequently Asked Questions

Table of Contents
=================

* [Name](#name)
* [Questions and Answers](#questions-and-answers)
    * [Does OpenResty follow NGINX's mainline releases?](#does-openresty-follow-nginxs-mainline-releases)
    * [How often does OpenResty make a new release?](#how-often-does-openresty-make-a-new-release)
    * [How should I report a problem?](#how-should-i-report-a-problem)
    * [Why can't I use Lua 5.2 or later?](#why-cant-i-use-lua-52-or-later)
    * [Can I use LuaJIT 2.0.x?](#can-i-use-luajit-20x)
    * [Why does OpenResty use LuaJIT 2.1 by default?](#why-does-openresty-use-luajit-21-by-default)
    * [Why can't I use duplicate configuration directives?](#why-cant-i-use-duplicate-configuration-directives)
* [Contributing to this FAQ](#contributing-to-this-faq)
* [Author](#author)
* [See Also](#see-also)

Questions and Answers
======================

Does OpenResty follow NGINX's mainline releases?
------------------------------------------------

Yes, sure. OpenResty is currently employing a "tick-tock" release model. Each "tock" release (usually) upgrades the bundled NGINX core to the latest
mainline release from the official NGINX team, followed by one ore more "tick" releases focusing on OpenResty's own features and enhancements without
upgrading the NGINX core.

The first part of the OpenResty release version numbers is the version of the bundled NGINX core. For instance, OpenResty 1.7.10.2 bundles
NGINX 1.7.10 while OpenResty 1.9.1.1 bundles NGINX 1.9.1.

How often does OpenResty make a new release?
--------------------------------------------

We are trying to make a new release of OpenResty every one or two months. It may sometimes take longer because this project is mainly based on volunteers.

[Back to TOC](#table-of-contents)

How should I report a problem?
------------------------------

Whenever you run into a problem, you are encouraged to report to the openresty-en (English) mailing list or the openresty (Chinese)
mailing list (depending on your language). See the [Community](http://openresty.org/#Community) page
for more details. But please don't cross-post.

You are highly recommended to provide as much details as possible while reporting a problem, for example,

* anything interesting in your Nginx's error log file (`logs/error.log`), if any,
* the exact versions of the related software you're using, including but not limited to the type/version of your
operating system, the version of your OpenResty (or the versions of your Nginx, ngx\_lua,
Lua/LuaJIT and other modules used if you are not using the OpenResty bundle),
* a minimal and standalone example that can reliably reproduce the issue on our side, and
* enable the Nginx/OpenResty's [debugging logs](http://nginx.org/en/docs/debugging_log.html) and
provide the *complete* logs for the guilty request you performed (the same `./configure --with-debug` command line also applies perfectly well to the OpenResty bundle).

The more details you provide, the more likely and faster we can help you out. Most of the time,
you will find your own mistakes while minimizing your problematic example or looking at your
Nginx error log files.

If you are absolutely certain that you have run into a real bug, then you are encouraged to
file a ticket in the corresponding project's GitHub Issues page. For example, the GitHub Issues page for
the ngx\_lua component is
https://github.com/openresty/lua-nginx-module/issues. The GitHub issues pages for OpenResty's components
are all considered English-only. Never use other languages like Chinese there. It is still fine, however, to just send a mail to one of the mailing lists mentioned above; but again, please do not cross-post.

[Back to TOC](#table-of-contents)

Why can't I use Lua 5.2 or later?
---------------------------------

Lua 5.2+ are incompatible with Lua 5.1 on both the C API land and
the Lua land (including various language semantics). If as you said there are quite
a few people already using ngx\_lua + Lua 5.1, then linking against Lua 5.2+ will
probably break these people's existing Lua code. Lua 5.2+ are essentially incompatible different languages.

Supporting Lua 5.2+ requires nontrivial architectural changes in ngx\_lua's basic infrastructure.
The most troublesome thing is the quite different "environments" model in Lua 5.2+.
At this point, we would hold back adding support for Lua 5.2+ to ngx\_lua. Also, we do not want to
create confusions and incompatibilities on the Lua land for applications running atop ngx\_lua, as well
as all the existing lua-resty-\* libraries written in the Lua 5.1 language.

We believe it is better to stick with one Lua language in ngx\_lua. Chasing the Lua language's version
number has not many practical technical merits (if there were some political ones).

[Back to TOC](#table-of-contents)

Can I use LuaJIT 2.0.x?
-----------------------

Yes sure. LuaJIT 2.0.x is always supported in OpenResty though LuaJIT 2.1+ is highly recommended.

[Back to TOC](#table-of-contents)

Why does OpenResty use LuaJIT 2.1 by default?
---------------------------------------------

As of this writing, LuaJIT 2.1 is still officially "beta" but OpenResty is using LuaJIT 2.1 by default
and encourages use of it in production because

1. We always run the latest LuaJIT v2.1 in our global network at CloudFlare.
2.  All the recent performance improvements we (CloudFlare) has
sponsored only land in v2.1.
3. For one of our typical Lua apps at the level of 10K+ LOC, LuaJIT
v2.1 gives over 100% over-all speedup as compared to LuaJIT 2.0.x
(when lua-resty-core is used).
4. Many important bug fixes in the v2.1 branch have been back ported
to 2.0.x series because these bugs were also in 2.0.x (just hidden for
long). The 2.0.3 release is such a proof.

We highly recommend LuaJIT 2.1 because we really need speed in OpenResty though you always have the freedom
to use LuaJIT 2.0.x or even the standard Lua 5.1 interpreter in OpenResty instead.

[Back to TOC](#table-of-contents)

Why can't I use duplicate configuration directives?
---------------------------------------------------

Most of the ngx\_lua module's configuration directives do not allow duplication in the same context.
For example, the following `nginx.conf` snippet

```nginx
    location / {
        content_by_lua_file conf/a.lua;
        content_by_lua_file conf/b.lua;
    }
```

will yield the following error while starting nginx:

```
nginx: [emerg] "content_by_lua_file" directive is duplicate in
```

People may want to use duplicate directives to split complicated Lua code base into multiple
separate `.lua` files. This is not allowed and it must be inefficient even if it were to be implemented in ngx\_lua.

The recommended way to organize your Lua code base is to use Lua's own module mechanism:

http://www.lua.org/manual/5.1/manual.html#5.3

You can put your unrelated Lua code into separate Lua module files, for example,

```lua
-- foo.lua
local _M = {}
function _M.go() ... end
return _M
```

```lua
-- bar.lua
local _M = {}
function _M.go() ... end
return _M
```

And finally, just `require` them in the entry point like this:

```lua
location / {
    content_by_lua '
        require("foo").go()
        require("bar").go()
    ';
}
```

You will need to add the path of your Lua modules to Lua's module search paths, for instance,

```nginx
lua_package_path "$prefix/conf/?.lua;;";
```

in the `http {}` block in your `nginx.conf`.

You can take a look at OpenResty's standard Lua libraries for real-world examples, like openresty/lua-resty-redis.

Use of Lua's native module mechanism is also very efficient. Thanks to the built-in caching mechanism
in Lua's built-in function `require()` (via the global `package.loaded` table anchored in the Lua registry, thus being shared
by the whole Lua VM).

[Back to TOC](#table-of-contents)

Contributing to this FAQ
=========================

This FAQ document is hosted on GitHub and periodically updated to the openresty.org site:

https://github.com/openresty/openresty.org

You can edit the `faq.md` file in the repository above and create pull requests so that I can incorporate your patches.

[Back to TOC](#table-of-contents)

Author
======

Yichun Zhang (agentzh)

[Back to TOC](#table-of-contents)

See Also
========

* [openresty.org](http://openresty.org)

[Back to TOC](#table-of-contents)

