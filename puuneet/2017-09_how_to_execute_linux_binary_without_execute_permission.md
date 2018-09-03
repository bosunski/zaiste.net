---
created_at: 2017-09-18 
kind: article
publish: true
title: "How to execute Linux binary without execute permission"
tags:
- cli
- linux
- hack
---

You can execute a binary in Linux without the execute permission. Use `/lib/ld*.so`  or `/lib64/ld*.so` as an ELF interpreter. The actual name differs from architecture to architecture. `ld` is the program linker/loader. It finds and loads the shared libraries used by the program and executes it. `ld-linux` handles ELF binaries.

```
cp /bin/ls /tmp/ls
chmod a-x /tmp/ls
```

```
/lib/ld-linux.so.2 /tmp/ls
```

```
/lib64/ld-linux-x86-64.so.2 /tmp/ls
```

