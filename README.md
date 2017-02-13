Name
====
lua-resty-brotli - Lua bindings to Google
[Brotli](https://github.com/google/brotli) for LuaJIT using FFI.


Status
======
This library is still experimental and under early development.


Installation
============
To install `lua-resty-brotli` you need to install
[Brotli](https://github.com/google/brotli#build-instructions)
with shared libraries firtst.
Then you can install `lua-resty-brotli` by placing `lib/resty/brotli.lua` to
your lua library path.


Example
=======
```` lua
local brotli = require "resty.brotli"

local txt = string.rep("ABCD", 1000)
print("Uncompressed size:", #txt)
local c, err = brotli.compress(txt)
print("Compressed size:", #c)
local txt2, err = brotli.decompress(c)
assert(txt == txt2)
````

nginx.conf of Openresty
```` lua
location / {
    root   html;
    index  index.html index.htm;
    default_type  text/html;

    rewrite_by_lua_block {
        ngx.ctx.accept_br = false
        local header = ngx.var.http_accept_encoding
        if header then
           if string.find(header, "br") then
              ngx.ctx.accept_br = true
           end
        end
        ngx.req.set_uri(ngx.var.uri..".br")
    }
    
    header_filter_by_lua_block {
        ngx.header.Content_Encoding = "br"
        if not ngx.ctx.accept_br then
           ngx.header.Content_Length = nil
        end
    }
    
    body_filter_by_lua_block {
        local brotli = require "resty.brotli"
        if not ngx.ctx.accept_br then
           ngx.arg[1] = brotli.decompress(ngx.arg[1])
        end
    }
}
````

Methods
=======

compress
--------
`syntax: encoded_buffer, err = brotli.compress(input_buffer, options?)`

Compresses the data in input_buffer into encoded_buffer.

The `options` argument is a Lua table holding the following keys:

* `quality`

    Set Brotli quality (compression) level.
    Acceptable values are in the range from `0` to `11`.
    (Defaults to 11)

* `lgwin`

    Set Brotli window size. Window size is `(1 << lgwin) - 16`.

* `mode`

    The compression mode can be `BROTLI_MODE_GENERIC` (0, default),
   `BROTLI_MODE_TEXT` (1, for UTF-8 format text input) or
   `BROTLI_MODE_FONT` (2, for WOFF 2.0).

decompress
----------
`syntax: decoded_buffer, err = brotli.decompress(encoded_buffer)`

Decompresses the data in encoded_buffer into decoded_buffer.

