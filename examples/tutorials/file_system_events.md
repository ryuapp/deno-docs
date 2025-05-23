---
title: "File system events"
description: "Tutorial on monitoring file system changes with Deno. Learn how to watch directories for file modifications, handle change events, and understand platform-specific behaviors across Linux, macOS, and Windows."
url: /examples/file_system_events_tutorial/
oldUrl:
  - /runtime/manual/examples/file_system_events/
  - /runtime/tutorials/file_system_events/
---

## Concepts

- Use [Deno.watchFs](https://docs.deno.com/api/deno/~/Deno.watchFs) to watch for
  file system events.
- Results may vary between operating systems.

## Example

To poll for file system events in the current directory:

```ts title="watcher.ts"
const watcher = Deno.watchFs(".");
for await (const event of watcher) {
  console.log(">>>> event", event);
  // Example event: { kind: "create", paths: [ "/home/alice/deno/foo.txt" ] }
}
```

Run with:

```shell
deno run --allow-read watcher.ts
```

Now try adding, removing and modifying files in the same directory as
`watcher.ts`.

Note that the exact ordering of the events can vary between operating systems.
This feature uses different syscalls depending on the platform:

- Linux: [inotify](https://man7.org/linux/man-pages/man7/inotify.7.html)
- macOS:
  [FSEvents](https://developer.apple.com/library/archive/documentation/Darwin/Conceptual/FSEvents_ProgGuide/Introduction/Introduction.html)
- Windows:
  [ReadDirectoryChangesW](https://docs.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-readdirectorychangesw)
