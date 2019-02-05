# 01
Try

```
make
```

No targets and no Makefile, so nothing to do.

Try

```
make five.c
```

There's nothing to be done. `five.c` already exists.

```
make five
```

Make shows you what command its running, and remarkably it knows to run a c
compiler on `five.c` to produce an executable called `five`. We can also ask
make to create an object file:

```
make five.o
objdump -d five.o
```

We can see all the rules that make looks for with `-d`:

```
make -d five
```

We can also ask make to explicitly disable builtin rules:

```
rm -rf five
make --no-builtin-rules five
```

https://www.gnu.org/software/make/manual/html_node/Implicit-Rules.html

# 02

Try:

```
make genesis
```

Delete `genesis` and build it again with no builtin rules and with debugging
output:

```
make --no-builtin-rules -d genesis
```

Delete `genesis` and build `death` which depends on `genesis`:

```
make --no-builtin-rules -d death
```

Now try `touch`ing `genesis` to update its filesystem timestamp, then re-make
`death`:

```
touch genesis
make --no-builtin-rules -d death
```

Try re-making `death` without changing anything:

```
make --no-builtin-rules -d death
```

# 03

The syntax `$(info ...)` prints the arguments to the shell after make variable
expansion.

The `=` syntax for variable assignment creates a *recursively expanded*
variable. Make variable expansion is conducted every time the variable is
referenced, and done so recursively.

# 04

The `:=` syntax for variable assignment creates a *simply expanded variable*,
this is expanded once at assignment time.

# 05

The `?=` syntax for variable assignment means set the variable if it is unset.

Try:

```
make
```

Make inherits variables from the environment and also accepts variable
assignments as arguments. Arguments from the user take precedence over
definitions given in the Makefile. The Makefile can override the user with
`override`. Environment variables *do not* take precedence over definitions
given in the Makefile. Consider the difference between these three commands

```
make
FOO=10 make
make FOO=10
```

Make also has append syntax `+=` which works as you expect for both flavors of
variables.

# 06

We can use `.PHONY` to indicate a target which:

 - does not correspond to a file on-disk

 - is always considered out-of-date (important: it always triggers rebuilds of
   its dependencies)


# 07

If a target depends on something that is not a file, like an environment
variable, then make cannot correctly determine when the target is out of date.

Try this:

```
make out
cat out
BAR=10 make out
cat out
rm out && BAR=10 make out
cat out
```

If we want to consider a target out of date when an environment variable
changes, we must make the environment variable into a file.

```
out2: env/BAR
	echo $(shell cat env/BAR) > $@
```

This prevents us from setting environment variables from the command line. We
must edit a file instead.

We can use conditional statements to check if an environment variable has
changed since the last invocation and force dependent targets to rebuild if it
has.

Try this:

```
make out2
cat out2
make out2
BAR=10 make out2
cat out2
```

# 08

Try this:

```
make child && ./child
```

Now try editing either `mother.h` or `father.h` and running:

```
make child && ./child
```

Note that the Makefile does not know that child depends on `mother.h` and
`father.h`. We could explicitly state these dependencies in the Makefile:

```
child: child.c mother.h father.h
	cc child.c -o child
```

But we are programmers. Let's write a rule to generate rules! Copy this into
`08/Makefile`:

```
include headers.mk

headers.mk: $(shell find . -name *.c)
	rm -f headers.mk
	$(foreach file,$^,grep $(file) -e '\#include' | sed -E 's/#include *"([^"]+)"/$(basename $(file)): \1/' >> $@)
```

`include headers.mk` suspends reading of the current Makefile and starts reading
`headers.mk`. When reading of `headers.mk` is complete, reading the current file
is resumed. After the expansion of all Makefiles is complete, make checks if any
of the Makefiles need to be rebuilt.

Since `headers.mk` depends on all the `.c` files, any time a `.c` file is
modified, it will be regenerated. For this reason this rule should finish
executing below the human-perception threshold (~100ms).

The rules above now causes Make to error on `make child`:

```
# make child
cc     child.c mother.h father.h   -o child
clang: error: cannot specify -o when generating multiple output files
make: *** [child] Error 1
```

The implicit rule for executables passes all the prerequisites to `cc`. We must
redefine the implicit rule to use only the first one:

```
%: %.c
	cc $< -o $@
```

Now try:

```
echo '#define MOTHER 10' > mother.h
make child && ./child
echo '#define MOTHER 30' > mother.h
make child && ./child
```
