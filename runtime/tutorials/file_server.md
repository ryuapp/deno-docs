---
title: "File Server"
oldUrl:
  - /runtime/manual/examples/file_server/
---

## Concepts

- Use [Deno.open](https://docs.deno.com/api/deno/~/Deno.open) to read a file's
  content in chunks.
- Transform a Deno file into a
  [ReadableStream](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream).
- Use Deno's integrated HTTP server to run your own file server.

## Overview

Sending files over the network is a common requirement. As seen in the
[Fetch Data example](./fetch_data.md), because files can be of any size, it is
important to use streams in order to prevent having to load entire files into
memory.

## Example

**Command:** `deno run --allow-read=. --allow-net file_server.ts`

```ts title="file_server.ts"
Deno.serve(
  { hostname: "localhost", port: 8080 },
  (request) => {
    // Use the request pathname as filepath
    const url = new URL(request.url);
    const filepath = decodeURIComponent(url.pathname);

    // Try opening the file
    try {
      const file = await Deno.open("." + filepath, { read: true });

      // Stream the file as it's read to the response. That way we don't
      // need to load the full file into memory.
      return new Response(file.readable);
    } catch {
      // If the file cannot be opened, return a "404 Not Found" response
      return new Response("404 Not Found", { status: 404 });
    }
  },
);
```

## Using the `std/http` file server

The Deno standard library provides you with a
[file server](https://jsr.io/@std/http/doc/file-server/~) so that you don't have
to write your own.

To use it, first install the remote script to your local file system. This will
install the script to the Deno installation root's bin directory, e.g.
`/home/alice/.deno/bin/file-server`.

```shell
# Deno 1.x
deno install --allow-net --allow-read jsr:@std/http@1/file-server
# Deno 2.x
deno install --global --allow-net --allow-read jsr:@std/http@1/file-server
```

You can now run the script with the simplified script name. Run it:

```shell
$ file-server .
Listening on:
- Local: http://0.0.0.0:8000
```

Now go to [http://0.0.0.0:8000/](http://0.0.0.0:8000/) in your web browser to
see your local directory contents.

The complete list of options are available via:

```shell
file-server --help
```

Example output:

```shell
Deno File Server 1.0.5
  Serves a local directory in HTTP.

INSTALL:
  deno install --allow-net --allow-read --allow-sys jsr:@std/http@1.0.5/file-server

USAGE:
  file_server [path] [options]

OPTIONS:
  -h, --help            Prints help information
  -p, --port <PORT>     Set port (default is 8000)
  --cors                Enable CORS via the "Access-Control-Allow-Origin" header
  --host     <HOST>     Hostname (default is 0.0.0.0)
  -c, --cert <FILE>     TLS certificate file (enables TLS)
  -k, --key  <FILE>     TLS key file (enables TLS)
  -H, --header <HEADER> Sets a header on every request.
                        (e.g. --header "Cache-Control: no-cache")
                        This option can be specified multiple times.
  --no-dir-listing      Disable directory listing
  --no-dotfiles         Do not show dotfiles
  --no-cors             Disable cross-origin resource sharing
  -v, --verbose         Print request level logs
  -V, --version         Print version information

  All TLS options are required when one is provided.
```
