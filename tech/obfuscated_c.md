# C Obfuscation Tricks

_A primer on some C obfuscation tricks_ was originally [authored](https://github.com/ColinIanKing/christmas-obfuscated-C/blob/master/tricks/obfuscation-tricks.txt) by [Colin Ian King](https://github.com/ColinIanKing)

## Abusing `typedef`

1. Normal expected form:
```cpp
typedef signed int my_int;
```

2. Abuse it by:
```cpp
signed int typedef my_int;
int typedef signed my_int;
```
3. or since int is default type, one can do:
```cpp
typedef my_int;
```

## Abusing Arrays

```cpp
char x[];
int index;

x[index] is *(x+index)
index[x] is legal C and equivalent too
```

### Abusing String Literals

Literal string abuse, like above in principle

```cpp
int index = 0;

printf("%c\n", index["MyString"]);
```

## Abuse unexpected results

```cpp
#include <stdio.h>

#define X (-2147483648)

int foo(void) { return -X; }
```

`-2147483648` is positive.  This is because `2147483648` cannot fit in the type `int`, so (following the ISO C rules) its data type is `unsigned long int`. Negating this value yields `2147483648` again."


## Surprising Math

```cpp
int x = 0xfffe+0x0001;
```
looks like 2 hex constants, but in fact it is not.

## Abusing pre-processor

```cpp
#define  A case _COUNTER_: n = (__COUNTER__ - 1) & 0xfffff;break;
#define B A A A A A A A A A A A A A A A A
#define C B B B B B B B B B B B B B B B B
#define D C C C C C C C C C C C C C C C C
#define E D D D D D D D D D D D D D D D D
#define F E E E E E E E E E E E E E E E E
main(){ int n = 0; for (;;) switch(n) { F } }
```

## Indirection on function pointers with comments

```cpp
(**/**/**/**/**/**/**write)(1, "Hello World\n", 12);
```

### Do confusing things

```cpp
unsigned char count = 1;

printf("%d %d\n", 0 == sizeof(count = 2, count++), count);
```

## Make it hard to be traced

```cpp
#include <unistd.h>
#include <sys/ptrace.h>
#include <signal.h>
#include <setjmp.h>

sigjmp_buf e;

void h(int sig) { siglongjmp(e, 1); }

int main(void)
{
        signal(SIGTRAP, h);
        if (sigsetjmp(e, 0) != 1)
                while (1)__asm__("int3");
        if (!ptrace(PTRACE_TRACEME, 0, 1, 0))
                write(1, "gdb or strace me\n", 17);
        return 0;
}
```

## Insert URLs for fun

```cpp
int main(int argc, char *argv[])
{
        http://smackerelofopinion.blogspot.co.uk/search/label/C

        return 0;
}
```

## URL comments ;-)

Useful background reading:
http://www.se.rit.edu/~tabeec/RIT_441/Resources_files/How%20To%20Write%20Unmaintainable%20Code.pdf

## Fun with predefined values (yep, this compiles)

```cpp
int main(){ return linux > unix; }
```

## Break code over lines. Makes it hard to grep.

```cpp
ret\
urn 0;
```

## Make macros with ( { ; imbalancing to make it more confusing

```cpp
#define P	printf(
#define X	);}

main() {
	P("Hello Word" X
```

## Use gcc alternate digraphs for fun

```cpp
Digraph:        <%  %>  <:  :>  %:  %:%:
Punctuator:     {   }   [   ]   #    ##
```

## Use macro definitions of numbers in roman numerals ...

... and make them wrong to confuse reader

```cpp
#define XII	(10+3)
```

## use offputting variable names, eg;

	float Not, And, Or;
	double Int8;
	int _, __, o, O; Oo, Oo, OO, oo;

so you end up with code like:

	while (!Not & And != (Or | 2))...

## Shove all variables into one array

	don't have lots of ints; just have one array of ints and
	reference these using:

	x[0], 1[x], *(x+4), *(8+x).. etc

## Use cpp pasting ## and stringification for fun and laughs


## Abuse `switch` and `while`

```cpp
void sw(int s)
{
  switch (s) while (0) {
  case 0:
          printf("zero\n");
          continue;
  case 1:
          printf("one\n");
          continue;
  case 2:
          printf("two\n");
          continue;
  default:
          printf("something else\n");
          continue;
  }
}
```

## Use confusing coding idioms

Replace:

```cpp
if (c)
    x = v;
else
    y = v;
```

With:

```cpp
*(c ? &x : &y) = v;
```

## don't use loops, abuse goto, setjmp, longjmp, recursion

	make it confusing, don't make it obvious

## use a smart algorithms

	make it so smart that it is hard to figure out
	what the code is really doing

## Abuse `sizeof`

int foo(const int i
{
        char x[i];

        return sizeof x;
}

int
main(void)
{
        printf("%d\n", foo(-1));
}

##

A void function that returns void(!)

void a (void) { }
void b (void) { return a(); };


## Make it look different, format code into pictures

```cpp
#include <stdio.h>
#include <stdlib.h>
#define K continue
#define t /*|+$-*/9
#define _l /*+$*/25
#define s/*&|+*/0xD
#define _/*&|+*/0xC
#define _o/*|+$-*/2
#define _1/*|+$-*/3
#define _0/*|+$*/16
#define J/*&|*/case
char typedef signed
B;typedef H;H main(
){B I['F'],V=0,E[]=
{s,0,s,31,t,1,s,111
,_,t,-3,_,s,50,_l,-
1,t,1,s,0x48>>2,_l,
-2,_,_1,5,_o,s,0,_1
 ,-8,s,0,s,-65,t,75,
 s,100,_,t,8,_,_1,-5
  ,s,82,t,32,s,111,s,
  20,_l,-2,t,7,_,_1,5
   ,_o,s,0,_1,-8,s,0,\
  _0,};B*P=E;while(P)
  {B L=*P,l=*(P+1),U=
 I[V-1],A=(L>>2)&1,C
 =(V-(1-A)),i;switch
(L)while(0){J _l:i=
l>0?U>>l:U<<-l;K;J\
t:i=U+l;K;J _:i=U;
K;J s:i=l;K;J _o:
putchar(U);K;J _1:
P+=U?0:l;K;J _0:e\
xit(0);}C[I]=(L&8)?
i:I[C];P+=(L&1)+1;V
+=A-((L&2)>>1);}re\
turn/*c.i.king*/0;}
```

## Doing nothing

        ({_:&&_;});

        ({});

        ({;});

	switch (0);

## abusing __VA_ARGS__

One can pass an entire function body into a macro using __VA_ARGS__

#define F(f, ...) f __VA_ARGS__

F(void foo(void) { printf("foo\n"); })

## Turn array to a pointer using `+`

	foo(&array[0]);

	instead use:

	foo(+array);

## ???

	int x = 1;

	foo(&x);

	use:

	foo( (int[]){1} );

## Zero'ing

	a ^= a;
	a = '-'-'-';

## Variable names

```cpp
char broiled;
double burger;
short cake;
float icecream;
long story;
signed sincerely;
static electricity;
auto mechanic;
volatile substance;
register electorat;
unsigned autobiography;
void *gazing_back;
char witness;
const*able;
union dues {};
```

## Precendence rules abuse:

```cpp
#include <stdio.h>
int main(void) { int array[] = {1, 2, 3}; int *p = &array[1]; printf("%d\n", -1[p]); }
```

`-1[p]` is `-(*(1 + p))` and not `*(-1 + p)`

## Make simple expressions more complex

	u8 val;

	tricky				simple
	if (val && ~val)		if (val)
		break;				break;

## Fun ways to increment:

	i-=-1;

## Yes/No?

	return (char *[]){"No", "Yes"}[!!x];

## Use local typedefs to confuse

```cpp
int x(int y)
{
        return y;
}

int main(void)
{
        printf("%d\n", (x)(257));
        typedef char x;
        printf("%d\n", (x)(257));
}
```
