Tests for WBE Chapter 1
=======================

Chapter 1 (Downloading Web Pages) covers parsing URLs, HTTP requests
and responses, and a very very simplistic print function that writes
to the screen. This file contains tests for those components.

Here's the testing boilerplate.

    >>> import test
    >>> _ = test.socket.patch().start()
    >>> _ = test.ssl.patch().start()
    >>> import web_browser
    
You can run this file with `doctest` to run the tests.

Testing `show`
--------------

The `show` function is supposed to print some HTML to the screen, but
skip the tags inside.

    >>> web_browser.show('<body>hello</body>')
    hello
    >>> web_browser.show('he<body>llo</body>')
    hello
    >>> web_browser.show('he<body>l</body>lo')
    hello
    >>> web_browser.show('he<body>l<div>l</div>o</body>')
    hello

Note that the tags do not have to match:

    >>> web_browser.show('he<body>l</div>lo')
    hello
    >>> web_browser.show('he<body>l<div>l</body>o</div>')
    hello

Testing `request`
-----------------

The `request` function makes HTTP requests.

To test it, we use the `test.socket` object, which mocks the HTTP server:

    >>> url = 'http://test.test/example1'
    >>> test.socket.respond(url, b"HTTP/1.0 200 OK\r\n" +
    ... b"Header1: Value1\r\n\r\n" +
    ... b"Body text")

Then we request the URL and test both request and response:

    >>> headers, body = web_browser.request(url)
    >>> test.socket.last_request(url)
    b'GET /example1 HTTP/1.0\r\nHost: test.test\r\n\r\n'
    >>> body
    'Body text'
    >>> headers
    {'header1': 'Value1'}

Testing SSL support
-------------------

Here we're making sure that SSL support is enabled.

    >>> url = 'https://test.test/example2'
    >>> test.socket.respond(url, b"HTTP/1.0 200 OK\r\n\r\n")
    >>> header, body = web_browser.request(url)
    >>> body
    ''

SSL support also means some support for ports:

    >>> url = 'https://test.test:400/example3'
    >>> test.socket.respond(url, b"HTTP/1.0 200 OK\r\n\r\nHi")
    >>> header, body = web_browser.request(url)
    >>> body
    'Hi'

Requesting the wrong port is an error:

    >>> test.errors(web_browser.request, "http://test.test:401/example3")
    True
