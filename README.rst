============
vmod_cookie
============

----------------------
Varnish Cookie Module
----------------------

:Author: Lasse Karstensen
:Date: 2015-03-27
:Version: 1.02
:Manual section: 3

SYNOPSIS
========

import cookie;

DESCRIPTION
===========

Handle HTTP cookies easier in Varnish VCL. (without regex)

Parses a cookie header into an internal data store, where per-cookie
get/set/delete functions are available. A filter_except() method removes all
but a set comma-separated list of cookies. A filter() method removes a comma-
separated list of cookies.

A convenience function for formatting the Set-Cookie Expires date field
is also included. It might be needed to use libvmod-header if there might
be multiple Set-Cookie response headers.

Important: The state loaded with cookie.parse() can only be guaranteed
within a single VCL call. Do the cookie operations in vcl_recv, and store
any additional variables you need later on as fake request headers.

It is currently not safe/tested to call this VMOD in any fetch threads.
(vcl_backend_fetch and similar)


FUNCTIONS
=========

parse
-----

Prototype
        ::

                parse(STRING S)
Return value
	VOID
Description
	Parse the cookie string in string S. The parsed values are only guaranteed
	to exist within a single VCL function. Implicit clean() if run more than once.
Example
        ::

		sub vcl_recv {
			cookie.parse(req.http.Cookie);
		}


clean
-----

Prototype
        ::

                clean()
Return value
	VOID
Description
	Clean up all previously parse()-d cookies. Probably of limited
	use. It is not necessary to run clean() in normal operation.
Example
        ::

		sub vcl_recv {
			cookie.clean();
		}

get
-----

Prototype
        ::

                get(STRING cookiename)
Return value
	STRING
Description
	Get the value of a cookie, as stored in internal vmod storage. If the cookie name does not exists, an empty string is returned.

Example
        ::

		import std;
		sub vcl_recv {
			cookie.parse("cookie1: value1; cookie2: value2;");
			std.log("cookie1 value is: " + cookie.get("cookie1"));
		}

set
----

Prototype
        ::

                set(STRING cookiename, STRING cookievalue)
Return value
	VOID
Description
	Set the internal vmod storage value for a cookie to a value.

Example
        ::

		sub vcl_recv {
			cookie.set("cookie1", "value1");
			std.log("cookie1 value is: " + cookie.get("cookie1"));
		}

isset
-----

Prototype
        ::

                isset(STRING cookiename)
Return value
	BOOL
Description
	Check if a given cookie is set in the internal vmod storage.

Example
        ::

		import std;
		sub vcl_recv {
			cookie.parse("cookie1: value1; cookie2: value2;");
			if (cookie.isset("cookie2")) {
				std.log("cookie2 is set.");
			}
		}

delete
------

Prototype
        ::

                delete(STRING cookiename)
Return value
	VOID
Description
	Delete a cookie from internal vmod storage if it exists.

Example
        ::

		sub vcl_recv {
			cookie.parse("cookie1: value1; cookie2: value2;");
			cookie.delete("cookie2");
			// get_string() will now yield "cookie1: value1";
		}


filter_except
-------------

Prototype
        ::

                filter_except(STRING cookienames)
Return value
	VOID
Description
	Delete all cookies from internal vmod storage that is not in the
	comma-separated argument cookienames.

Example
        ::

		sub vcl_recv {
			cookie.parse("cookie1: value1; cookie2: value2; cookie3: value3");
			cookie.filter_except("cookie1,cookie2");
			// get_string() will now yield
			// "cookie1: value1; cookie2: value2;";
		}



filter
------

Prototype
        ::

                filter(STRING cookienames)
Return value
	VOID
Description
	Delete all cookies from internal vmod storage that is in the
	comma-separated argument cookienames.

Example
        ::

		sub vcl_recv {
			cookie.parse("cookie1: value1; cookie2: value2; cookie3: value3");
			cookie.filter("cookie1,cookie2");
			// get_string() will now yield
			// "cookie3: value3;";
		}



get_string
----------

Prototype
        ::

                get_string()
Return value
	STRING
Description
	Get a Cookie string value with all cookies in internal vmod storage.
Example
        ::

		sub vcl_recv {
			cookie.parse(req.http.cookie);
			cookie.filter_except("SESSIONID,PHPSESSID");
			set req.http.cookie = cookie.get_string();
		}

format_rfc1123
--------------

Prototype
        ::

                format_rfc1123(TIME, DURATION)
Return value
	STRING
Description
	Get a RFC1123 formatted date string suitable for inclusion in a
	Set-Cookie response header.

	Care should be taken if the response has multiple Set-Cookie headers.
	In that case the header vmod should be used.

Example
        ::

		sub vcl_deliver {
			# Set a userid cookie on the client that lives for 5 minutes.
			set resp.http.Set-Cookie = "userid=" + req.http.userid + "; Expires=" + cookie.format_rfc1123(now, 5m) + "; httpOnly";
		}


INSTALLATION
============

The source tree is based on autotools to configure the building, and
does also have the necessary bits in place to do functional unit tests
using the varnishtest tool.

Usage::

 ./configure --prefix=/usr

Make targets:

* make - builds the vmod
* make install - installs the vmod.
* make check - runs the unit tests in ``src/tests/*.vtc``

In your VCL you could then use this vmod along the following lines::

	import cookie;
	sub vcl_recv {
		cookie.parse(req.http.cookie);
		cookie.filter_except("SESSIONID,PHPSESSID");
		set req.http.cookie = cookie.get_string();
	}


COPYRIGHT
=========

This document is licensed under the same license as the
libvmod-example project. See LICENSE for details.

* Copyright (c) 2011-2013 Varnish Software
* Copyright (c) 2013-2015 Lasse Karstensen
