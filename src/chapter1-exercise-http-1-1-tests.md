Tests for WBE Chapter 1 Exercise `HTTP/1.1`
===========================================

Testing boilerplate:

    >>> import time
    >>> import test
    >>> _ = test.socket.patch().start()
    >>> _ = test.ssl.patch().start()
    >>> import web_browser

Mock the HTTP server, as before:

    >>> url = 'http://test.test/example1'
    >>> test.socket.respond(url=url,
    ...   response=("HTTP/1.0 200 OK\r\n" +
    ...             "Header1: Value1\r\n" + 
    ...             "\r\n" +
    ...             "Body text").encode(),
    ...   method="GET")

This request/response pair was tested in the base tests, but now we are 
  checking that the 'connection' header is present and contains 'close', that a
  'user-agent' header is present, and that the request is HTTP 1.1:

    >>> response_headers, response_body = web_browser.request(url)
    >>> command, path, version, headers = test.socket.parse_last_request(url)
    >>> command
    'GET'
    >>> path
    '/example1'
    >>> headers["connection"]
    'close'
    >>> "user-agent" in headers
    True
    >>> version
    'HTTP/1.1'

In addition to these changes, the signature of your `request` function should 
  now include an optional argument, `headers`, which is a dictionary mapping 
  strings to strings representing header names mapping to their values.
Optional arguments in python may not behave as you expect.
We recommend reading 
[this description](https://docs.python-guide.org/writing/gotchas/#default-args)
    
    >>> extra_client_headers = {"ClientHeader" : "42"}
    >>> headers, body = web_browser.request(url, headers=extra_client_headers)
    >>> command, path, version, headers = test.socket.parse_last_request(url)
    >>> headers["clientheader"]
    '42'

If the headers given are already present in the query they should overwrite the
  value for that request.
In other words, the request generated should only contain one occurrence of each
  header.
  
    >>> extra_client_headers = {"User-Agent" : "different/1.0"}
    >>> headers, body = web_browser.request(url, headers=extra_client_headers)
    >>> raw_request = test.socket.last_request(url)
    >>> raw_request.count("User-Agent".encode())
    1
    >>> command, path, version, headers = test.socket.parse_last_request(url)
    >>> headers["user-agent"]
    'different/1.0'