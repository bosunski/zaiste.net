---
created_at: 2017-12-06 
kind: article
publish: true
title: "Encrypt external drive in MacOS High Sierra"
tags:
- security
- macos
---

Connect the drive via USB. List mounted devices:

```
diskutil list
```

Format a disk of choice as HSF+ (Journaled)

```
diskutil eraseDisk JHFS+ <your custom name> GPT <drive id>
```

Encrypt the partition 

```
diskutil cs convert <partition id> -passphrase
```

`<partition id>` looks like `disk2s2` for the `disk2` device. Choose it accordingly in your context.
