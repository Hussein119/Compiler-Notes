# Compiler Nots

_This compilation highlights key points from each chapter that I deem essential for memorization. Following each chapter, you'll find solutions to the challenges presented, providing a comprehensive understanding of the material._

![Static Badge](https://img.shields.io/badge/Processing-8A2BE2%2CProcessing?label=Status)

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
