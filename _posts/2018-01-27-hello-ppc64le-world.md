---
title:	"Hello (ppc64le) World!"
date:	2018-01-27
---

# Hello (ppc64le) World

Our good old friend, Hello World:

```c
$ cat hello-world.c 
#include <stdio.h>

int main() {
	printf("Hello World!\n");
	return 0;
}
```

Build and run:

```
$ gcc -o hello-world hello-world.c

$ ./hello-world 
Hello World!
```

And welcome to `ppc64le`, a.k.a. _little-endian 64-bit PowerPC_:

```
$ file hello-world
hello-world: ELF 64-bit LSB executable, 64-bit PowerPC [...]

$ uname -sm
Linux ppc64le
```
