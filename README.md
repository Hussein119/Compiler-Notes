# Compiler Nots

_This compilation highlights key points from each chapter that I deem essential for memorization. Following each chapter, you'll find solutions to the challenges presented, providing a comprehensive understanding of the material._

[![Static Badge](https://img.shields.io/badge/Processing-8A2BE2%2CProcessing?label=Status)](https://github.com/Hussein119/Compiler-Nots)

## Table of Contents

- [Chapter 1: Introduction](#chapter-1-introduction)
- [Chapter 2: A Map of the Territory](#chapter-2-a-map-of-the-territory)
- [Chapter 3: The Lox Language](#chapter-3-the-lox-language)
- [Chapter 4: Scanning](#chapter-4-scanning)

## Chapter 1 : Introduction

### NOTES

1. little languages = domain-specific languages : These are pidgins
   tailor-built to a specific task.

![domain-specific languages](ch1-1.png)

2. A compiler reads files in one language. translates them, and outputs files in another
   language. You can implement a compiler in any language, including the same language it
   compiles, a process called _self-hosting_.
3. You can’t compile your compiler using itself yet, but if you have another compiler for your
   language written in some other language, you use that one to compile your compiler once.
   Now you can use the compiled version of your own compiler to compile future
   versions of itself and you can discard the original one compiled from the other
   compiler. This is called _bootstrapping_ from the image of pulling yourself up by your own
   bootstraps.
4. Bytecode is an intermediate representation (IR) of a program's source code that is generated by
   a compiler or an interpreter before it is executed. It is a low-level, platform-independent
   set of instructions that can be executed by a virtual machine (VM) or interpreter. Bytecode is often used to bridge the gap between high-level programming languages and machine code.

### CHALLENGES

1. There are at least six domain-specific languages used in the little system I cobbled
   together to write and publish this book. What are they?

   > HTML , CSS , SQL , JSON , XML , XAML , Bash

2. Get a “Hello, world!” program written and running in Java. Set up whatever
   Makefiles or IDE projects you need to get it working. If you have a debugger, get
   comfortable with it and step through your program as it runs.

   > Chapter1.java

   ```java
   public class Chapter1 {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
   }
   ```

   > Compile the program with the following command:

   ```bash
   javac Chapter1.java
   ```

   > Run the compiled program with:

   ```bash
    java Chapter1
   ```

3. Do the same thing for C. To get some practice with pointers, define a doubly-linked
   list of heap-allocated strings. Write functions to insert, find, and delete items from
   it. Test them.

   > doubly-linked-list.c

   ```c
   #include <stdio.h>
   #include <stdlib.h>
   #include <string.h>

   typedef struct Node {
       char* data;
       struct Node* prev;
       struct Node* next;
   } Node;

   typedef struct {
       Node* head;
       Node* tail;
       int size;
   } LinkedList;

   void initList (LinkedList *list){
       list->head = NULL;
       list->tail = NULL;
       list->size = 0;
   }

   void insert(LinkedList *list , const char* data) {
       Node *newNode = (Node*) malloc(sizeof(Node));
       newNode->data = strdup(data);
       newNode->prev = list->tail; // list->tail to add the new node at end
       newNode->next = NULL; // now the new node at the end of the list
       if(list->tail != NULL) {
           list->tail->next = newNode;
       }
       list->tail = newNode;
       if(list->head == NULL) {
           list->head = newNode;
       }
       list->size++;
   }

   Node *find (LinkedList *list , const char* data) {
       Node *cuur = list->head;
       while (cuur != NULL) {
           if (strcmp(cuur->data, data) == 0) {
               return cuur;
           }
           cuur = cuur->next;
       }
       return NULL;
   }

   void deleteNode(LinkedList *list, Node *node) {
       if (node == NULL) {
           return;
       }
       if(node->prev != NULL) {
           node->prev->next = node->next;
       } else {
           list->head = node->next;
       }
       if(node->next != NULL) {
           node->next->prev = node->prev;
       } else {
           list->tail = node->prev;
       }

       free(node->data);
       free(node);
       list->size--;
   }

   void freeList (LinkedList *list) {
       Node *curr = list->head;
       while (curr != NULL) {
           Node *next = curr->next;
           free(curr->data);
           free(curr);
           curr = next;
       }
       list->head = NULL;
       list->tail = NULL;
       list->size = 0;
   }

   int getSize (LinkedList *list) {
       return list->size;
   }

   void printList (LinkedList *list) {
       Node *cuur = list->head;
       while (cuur != NULL) {
           printf("%s ",cuur->data);
           cuur = cuur->next;
       }
       printf("\n");
   }

   int main () {
       LinkedList list;
       initList(&list);
       insert(&list, "Hello");
       insert(&list, "Hussein");
       insert(&list, "Hussein2");

       printf("Original List: ");
       printList(&list);

       int size = getSize(&list);
       printf("Size: %d\n" , size);

       Node *foundNode = find(&list, "Hussein2");
       if (foundNode != NULL) {
           printf("Found: %s\n", foundNode->data);
           deleteNode(&list, foundNode);
           printf("After Deletion: ");
           printList(&list);
       } else {
           printf("Not Found.\n");
       }

       freeList(&list);

       return 0;
   }

   ```

   > Compile the program with the following command:

   ```bash
   gcc doubly-linked-list.c
   ```

   > Run the compiled program with:

   ```bash
    .\doubly-linked-list
   ```

## Chapter 2 : A Map of the Territory

### NOTES

1.  This book is about a language’s implementation, which is distinct from the language itself in
    some sort of Platonic ideal form. Things like _stack_, _bytecode_, and _recursive descent_, are nuts and bolts one particular implementation might use.

![overall view](overall.jpg)

2.  A scanner (or lexer) takes in the linear stream of characters and chunks them
    together into a series of something more akin to _words_.

3.  A parser takes the flat sequence of tokens and builds a tree structure that
    mirrors the nested nature of the grammar. These trees have a couple of different
    names—_parse tree_ or _abstract syntax tree_ —depending on how close to the
    bare syntactic structure of the source language they are. In practice, language
    hackers usually call them _syntax trees_, _ASTs_, or often just _trees_.
    The parser’s job also includes letting us know when we do by reporting _syntax errors_.

4.  The language we’ll build in this book (lox) is dynamically typed, so it will do its type
    checking later, at runtime.

5.  The front end of the pipeline is specific to the source language
    the program is written in. The back end is concerned with the final architecture
    where the program will run.

6.  Intermediate representation lets you support multiple source languages and target platforms with less
    effort. Say you want to implement _Pascal_, _C_ and _Fortran_ compilers and you
    want to target _x86_, _ARM_, and, I dunno, _SPARC_. Normally, that means you’re
    signing up to write **_nine full compilers_**: Pascal→x86, C→ARM, and every other
    combination. A shared intermediate representation reduces that dramatically.
    You write one front end for each source language that produces the IR. Then one back end for
    each target architecture. Now you can mix and match those to get every
    combination.

    - Source Languages: C, Fortran, Pascal
    - Target Architectures: x86, ARM, SPARC

      #### Without Intermediate Representation (IR)

      1. C

      - x86
      - ARM
      - SPARC

      2. Fortran

      - x86
      - ARM
      - SPARC

      3. Pascal

      - x86
      - ARM
      - SPARC

      > 9 full compilers

      #### With Intermediate Representation (IR)

      ##### Front-end

      1. C

      - IR

      2. Fortran

      - IR

      3. Pascal

      - IR

      ##### Back-end

      4. IR

      - x86
      - ARM
      - SPARC

      > 3 full compilers

7.  If we generate real _machine code_, we get an executable that the OS
    can load directly onto the chip. _Native code_ is lightning fast, but generating it is
    a lot of work. Today’s architectures have piles of instructions, complex
    pipelines, and enough historical baggage to fill a 747’s luggage bay.

8.  **Execution Environments:**

    - **JVM:**
      - Executes bytecode.
    - **.Net:**
      - Uses CLR (Common Language Runtime).
      - Executes IL (Microsoft Intermediate Language).

9.  **Bytecode Compilation Options:**

    If your compiler produces bytecode, you have two primary options:

    - **Option 1: Native Code Compilation**

      - Write a mini-compiler for each target architecture.
      - Convert the bytecode to native code for the specific machine.

    - **Option 2: Virtual Machine (VM) Execution**
      - Write a virtual machine (VM) program.
      - The VM emulates a hypothetical chip supporting your virtual architecture at runtime.
      - Running bytecode in a VM is slower than translating it to native code ahead of time.
      - Offers simplicity and portability.
      - Implement the VM in a language like C, allowing the language to run on any platform with a C compiler.
      - This approach is employed by the second interpreter developed in this book.

10. If the language is run inside an interpreter or VM, then the runtime lives there. This is how most
    implementations of languages like Java, Python, and JavaScript work.

11. _Syntax-directed translation_ is a structured technique for building these all-at-once compilers. You associate an action with each piece of the grammar, usually one that generates output code. Then, whenever the parser matches that chunk of syntax, it executes the action, building up the target code one rule at a time.

12. The fastest way to execute code is by compiling it to machine code, but you might not know what architecture your end user’s machine supports. What to do?
    > You can do the same thing that the HotSpot _JVM_, _Microsoft’s CLR_ and most
        *JavaScript interpreters* do. On the end user’s machine, when the program is
        loaded—either from source in the case of JS, or platform-independent bytecode
        for the JVM and CLR—you compile it to *native* for the architecture their
        computer supports. Naturally enough, this is called *just-in-time compilation*.
        Most hackers just say "JIT".
13. Compilers and Interpreters
    - _Compiling_ is an implementation technique that involves translating a source language to some other—usually lower-level—form. When you generate bytecode or machine code, you are compiling. When you transpile to another high-level language you are compiling too.
    - When we say a language implementation _is a compiler_, we mean it translates source code to some other form but doesn’t execute it.
    - when we say an implementation “is an interpreter”, we mean it takes in source code and executes it immediately. it runs programs "from source".
14. CPython is an interpreter, and it has a compiler

### CHALLENGES

1. Pick an open source implementation of a language you like. Download the source code and poke around in it. Try to find the code that implements the scanner and parser. Are they hand-written, or generated using tools like Lex and Yacc? (.l or .y files usually imply the latter.)

2. Just-in-time compilation tends to be the fastest way to implement a dynamically typed language, but not all of them use it. What reasons are there to not JIT?

   > Complexity, Portability, and Compilation Overhead.

3. Most Lisp implementations that compile to C also contain an interpreter that lets them execute Lisp code on the fly as well. Why?

   > to provide a more interactive and flexible development environment

## Chapter 3 : The Lox Language

### NOTES

1. Lox is dynamically typed.

2. Automatic memory management

3. There are two main techniques for managing memory: reference counting and tracing garbage collection (usually just called “garbage collection” or “GC”).

4. Data Types:

> Booleans

```c
true; // Not false.
false; // Not *not* false.
```

> Numbers: Lox only has one kind of number: double-precision floating point.

```c
1234; // An integer.
12.34; // A decimal number.

.2; // not allowed in lox
2.; // not allowed in lox
```

> Strings

```c
"I am a string";
""; // The empty string.
"123"; // This is a string, not a number.
```

> Nil

```c
return nil; // similar to returning null in other languages
```

5. Expressions:

> Arithmetic

```c
add + me;
subtract - me;
multiply * me;
divide / me;

-negateMe;
```

> Comparison and equality : 0 in lox is true not false

```c
less < than;
lessThan <= orEqual;
greater > than;
greaterThan >= orEqual;

1 == 2; // false.
"cat" != "dog"; // true.

314 == "pi"; // false.

123 == "123"; // false.
```

- look at this function in the interpreter

```java
  // the 0 in lox is true not false, if U want it be false edit the function below
  // < check-operands
  // > is-truthy
  private boolean isTruthy(Object object) {
    if (object == null)
      return false;
    if (object instanceof Boolean)
      return (boolean) object;
    return true;
  }
```

> Logical operators : The reason and and or are like control flow structures is because they short circuit. Not only does and return the left operand if it is false, it doesn’t even evaluate the right one in that case.

```c
!true; // false.
!false; // true.

true and false; // false.
true and true; // true.

false or false; // false.
true or false; // true.
```

> Precedence and grouping

```js
var average = (min + max) / 2;
```

6. Statements

```c
print "Hello, world!";

"some expression";

{
print "One statement.";
print "Two statements.";
}
```

7. Variables

```js
var imAVariable = "here is my value";
var iAmNil;

var breakfast = "bagels";
print breakfast; // "bagels".
breakfast = "beignets";
print breakfast; // "beignets".
```

8. Control Flow

```js
if (condition) {
print "yes";
} else {
print "no";
}

var a = 1;
while (a < 10) {
print a;
a = a + 1;
}

for (var a = 1; a < 10; a = a + 1) {
print a;
}
```

9. Functions

- An _argument_ is an actual value you pass to a function when you call it.

- A _parameter_ is a variable that holds the value of the argument inside the body of the function.

```c
makeBreakfast(bacon, eggs, toast);

makeBreakfast();

// a and b called parameters
fun printSum(a, b) {
    print a + b;
}

// 1 and 2 called arguments
printSum(1,2);
```

> Closures

```js
fun addPair(a, b) {
return a + b;
}

fun identity(a) {
return a;
}
print identity(addPair)(1, 2); // Prints "3".

fun outerFunction() {
    fun localFunction() {
        print "I'm local!";
    }
    localFunction();
}

fun returnFunction() {
    var outside = "outside";
    fun inner() {
    print outside;
    }
    return inner;
}
var fn = returnFunction();
fn();
```

10. Classes

```c
class Breakfast {
    // var x = 5; // this is not allowed in lox
    cook() {
        print "Eggs a-fryin'!";
    }
    serve(who) {
        print "Enjoy your breakfast, " + who + ".";
    }
}

// Store it in variables.
var someVariable = Breakfast;
// Pass it to functions.
someFunction(Breakfast);

var breakfast = Breakfast();
print breakfast; // "Breakfast instance".
```

> Inheritance: using a less-than (<) operator

```c
class Brunch < Breakfast {
    drink() {
        print "How about a Bloody Mary?";
    }
}

var benedict = Brunch("ham", "English muffin");
benedict.serve("Noble Reader");

class Brunch < Breakfast {
    init(meat, bread, drink) {
        super.init(meat, bread);
        this.drink = drink;
    }
}
```

> The Standard Library : built-in function clock() that returns the number of seconds since the program started.

### CHALLENGES

1. Write some sample Lox programs and run them (you can use the implementations of Lox in my repository). Try to come up with edge case behavior I didn’t specify here. Does it do what you expect? Why or why not?

```js
/*
fun makeCounter() {
  var i = 0;
  fun count() {
    i = i + 1;
    print i;
  }

  return count;
}

//fun scope(a) {
//  print a; // parameter
//  var a = "local";
//  print a; // local
//}

fun thrice(fn) {
  for (var i = 1; i <= 3; i = i + 1) {
    fn(i);
  }
}

thrice(fun (a) {
  print a;
});
// "1".
// "2".
// "3".

var counter = makeCounter();
//counter(); // "1".
//counter(); // "2".

//scope("parameter");

fun scope(a) {
  print a;
  var a = "local";
  print a;
}

print 10;
scope(5);

/*

/*
var a = "global";
{
  fun showA() {
    print a;
  }

  showA();
  var a = "block";
  showA();
  {
    var a = "block2";
    showA();
  }
  showA();
}
*/

/*
var a = 5;

{
  print a;
  var a = a;
  print a;
  a = 6;
  print a;
}
*/



/*
var a = 5;
var a = 6;
print a;
*/



/*
fun bad() {
var a = "first";
var a = "second";
print a;
}

bad();
*/

//break;

/*
while (true) {
  if (5 > 0 ) {
    print 6;
    break;
  }
  var a = 6;
  print a;
}
var a = 5;
print a;
*/

/*
while (true) {
  var a = 6;
  print a;

  if (5 > 0) {
    break;
  }
}

var a = 5;
print a;
*/

/*
if (a > 1) {
  print a;
  break;
}
*/

/*
class DevonshireCream {
  serveOn() {
    return "Scones";
  }
}
print DevonshireCream; // Prints "DevonshireCream".

class Bagel {}
var bagel = Bagel();
print bagel; // Prints "Bagel instance".

class Bacon {
  eat() {
    print "Crunch crunch crunch!";
  }
}

Bacon().eat(); // Prints "Crunch crunch crunch!".
*/

/*
class Doughnut {
  cook() {
    print "Fry until golden brown.";
  }
}
class BostonCream < Doughnut {
  cook() {
    var method = super.cook;
    method();
    super.cook();
    print "Pipe full of custard and coat with chocolate.";
  }
}
BostonCream().cook();
*/

/*
class A {
  method() {
  print "Method A";
  }
}
class B < A {
  method() {
    print "Method B";
  }
  test() {
    super.method();
  }
}
class C < B {}

C().test();
*/

/*
fun fib(n) {
  if (n < 2) return n;
  return fib(n - 1) + fib(n - 2);
}

var before = clock();
print fib(40);
var after = clock();
print after - before;
*/

while (true) {
  if (5 > 0 ) {
    print 6;
    break;
  }
}
```

2. This informal introduction leaves a lot unspecified. List several open questions you have about the language’s syntax and semantics. What do you think the answers should be?

3. Lox is a pretty tiny language. What features do you think it is missing that would make it annoying to use for real programs? (Aside from the standard library, of course.)

## Chapter 4 : Scanning

### NOTES

1. The scanner takes in raw source code as a series of characters and groups it into a series of chunks we call tokens.

2. Lox is a scripting -high level- language, which means it executes directly from source.

3. Each of these blobs of characters is called a lexeme:

![blobs of characters (lexeme)](lexeme.jpg)

4. Note that the ! and = are not two independent operators. You can’t write ! = in Lox and have it behave like an inequality operator.

5. We’ve got another helper:

```java
private char peek() {
  if (isAtEnd()) return '\0';
  return source.charAt(current);
}
```

> It’s sort of like advance(), but doesn’t consume the character. This is called **_lookahead_**. Since it only looks at the current unconsumed character, we have one character of lookahead. **_The smaller this number is, generally, the faster the scanner runs._** The rules of the lexical grammar dictate how much lookahead we need. Fortunately, most languages in wide use only peek one or two characters ahead.

#### Mid Qs : What is the lookahead of the scanner ? is it better to have larger or smaller lookaheads ? What is the lookahead of Lox ?

- Lookahead in Scanners:

Meaning: The lookahead of a scanner refers to the number of characters it can examine ahead of its current position without actually consuming them. It aids in making informed decisions about token formation.

- Optimal Size:

Smaller lookaheads (1-2 characters) are generally preferred for efficiency.
Larger lookaheads might be necessary for certain language constructs but can impact speed.

- Lox's Lookahead

Lox's scanner uses a lookahead of 1 character. This is sufficient for its lexical grammar, as it doesn't have complex constructs that require extensive lookahead.

6. Since we only look for a digit to start a number, that means -123 is not a number literal. Instead, -123, is an expression that applies - to the number literal 123.

```js
print -123.abs();
```

> This prints -123 because negation has lower precedence than method calls. We could fix that by making - part of the number literal. But then consider:

```js
var n = 123;
print - n.abs();
```

> This still produces -123, so now the language seems inconsistent. No matter what you do, some case ends up weird.

7. We don’t allow a leading or trailing decimal point, so these are both invalid:

```c
.1234
1234.
```

8. We don't allow this :

```js
123.sqrt();
```

9. Consider this in lox:

```js
var a = 5;

print -a; // -5
print --a; // 5
print ---a; // -5
```

10. ```java
    case 'o':
      if (peek() == 'r') {
        addToken(OR);
      }
    break;
    ```

Consider what would happen if a user named a variable _orchid_. The scanner would see the first two letters, or, and immediately emit an or keyword token. This gets us to an important principle called **_maximal munch_**. _When two lexical grammar rules can both match a chunk of code that the scanner is looking at, whichever one matches the most characters wins._

11. Maximal munch means we can’t easily detect a reserved word until we’ve reached the end of what might instead be an identifier.

### CHALLENGES

1. The lexical grammars of Python and Haskell are not regular. What does that mean, and why aren’t they?

   > A lexical grammar defines the basic building blocks of a programming language, such as tokens (keywords, identifiers, literals, etc.), and specifies how these tokens are combined to form valid programs. The reason why the lexical grammars of Python and Haskell are not regular can be attributed to the expressive power and flexibility that these languages provide.

2. Aside from separating tokens—distinguishing print foo from printfoo — spaces aren’t used for much in most languages. However, in a couple of dark corners, a space does affect how code is parsed in CoffeeScript, Ruby, and the C preprocessor. Where and what effect does it have in each of those languages?

3. Our scanner here, like most, discards comments and whitespace since those aren’t needed by the parser. Why might you want to write a scanner that does not discard those? What would it be useful for?

4. Add support to Lox’s scanner for C-style /\*..... \*/ block comments. Make sure to handle newlines in them. Consider allowing them to nest. Is adding support for nesting more work than you expected? Why?

- Scanner.java

```java
case '/':
				if (match('/')) {
					// A comment goes until the end of the line.
					while (peek() != '\n' && !isAtEnd())
						advance();
				} else if (match('*')) {
					MultilineComment();
				} else {
					addToken(SLASH);
				}
				break;
```

```java
	private void MultilineComment() {

		int nestLevel = 1;

		while (true) {
			if (isAtEnd()) {
				Lox.error(line, "Unterminated multiline comment.");
				return;
			}

			if (peek() == '/' && peekNext() == '*') {
				// Consume the '*' and '/' characters.
				advance();
				advance();
				nestLevel++;
				return;
			} else if (peek() == '*' && peekNext() == '/') {
				advance();
				advance();
				nestLevel--;

				if (nestLevel == 0) {
					return;

				}
			} else if (peek() == '\n') {
				line++;
			}

			advance();
		}
	}
```
