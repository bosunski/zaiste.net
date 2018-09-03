---
created_at: 2015-05-12
kind: article
publish: true
title: "How to join elements of an array in Bash"
tags:
- bash
- cli
---

Let's imagine we have an array defined in Bash

```
arr = ( 1 a 2 b c d e 3 )
```

```
echo ${arr[@]}
1 a 2 b c d e 3
```

We would like to join elements of that array using a custom 
separator e.g. `,` so it could be returned as a single string.

Here is the function that will do the job.

```
function join { local IFS="$1"; shift; echo "$*"; }
```

We can display it directly to `STDOUT`

```
join , ${arr[@]}
1,a,2,b,c,d,e,3
```

or, we can assign the return value to a variable

```
result=$(join , ${arr[@]})
echo $result
```