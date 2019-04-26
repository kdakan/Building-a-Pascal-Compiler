# BUILDING A PASCAL COMPILER WITH C, YACC & LEX

- [ 1. Compilation stages and parts of a compiler](#1-compilation-stages-and-parts-of-a-compiler)
- [ 2. Grammar, production, alphabet, language](#2-grammar-production-alphabet-language)
- [ 3. Classification of grammars](#3-classification-of-grammars)
- [ 4. Equivalence and ambiguity in grammars](#4-equivalence-and-ambiguity-in-grammars)
- [ 5. Attributes and extended grammar](#5-attributes-and-extended-grammar)
- [ 6. Using Lex and YACC](#6-using-lex-and-yacc)
- [ 7. Lex file format](#7-lex-file-format)
- [ 8. Lex regular expressions and operators](#8-lex-regular-expressions-and-operators)
- [ 9. Lex definitions](#9-lex-definitions)
- [10. YACC file format](#10-yacc-file-format)
- [11. YACC definitions](#11-yacc-definitions)
- [12. Calculator example](#12-calculator-example)
- [13. Local variable storage and nested block scope](#13-local-variable-storage-and-nested-block-scope)
- [14. Symbol table](#14-symbol-table)
- [15. P-code](#15-p-code)
- [16. P-code in detail](#16-p-code-in-detail)
- [17. P-code templates for basic Pascal structures](#17-p-code-templates-for-basic-pascal-structures)
- [18. Data structures used in the compiler](#18-data-structures-used-in-the-compiler)
- [19. Types and functions used in the compiler](#19-types-and-functions-used-in-the-compiler)

I don't have much left from the Pascal compiler I had built in 1997 when I was a student at the university. 
I remember at that time I had written some 10-15 thousand lines of C code, but in the early 2000s, the code was lost as its floppy disk went dead. 
Fortunately, in 1998, I had created an article on fortunecity.com as an introduction to this compiler, and I was able to find a copy of that article from archive.org. 
I reformatted the article and put it on GitHub. 
The techniques I was using at the time also apply to the present, as nothing much has changed in compiler design since the 1960s.

## 1. Compilation stages and parts of a compiler:
The compilation of a source program consists of the following stages:

1. Lexer/scanner (lexical analysis, produces the token table)
2. Parser (syntax analysis & semantic analysis, generates symbol table and uses it while checking for semantic errors or generating code)
3. Code generator (code generation, generates machine code or intermediate code)
4. Code optimizer (code optimization, generates optimized machine code in terms of speed and memory usage)

- The purpose of lexical analysis is to recognize the words (tokens) that make up the language and pass them to the parser for syntax analysis.
- The parser loads syntactic meaning to the words (tokens) with the help of reserved words and signs of the language. 
This is similar to loading syntactic meaning such as subject, object, verb to words in a sentence.
- As words (tokens) are recognized, they are registered together with some additional information (variable types, addresses, sizes, etc.) to the symbol table. 
When the same words (tokens) are encountered again, the information is stored in the symbol table is used to check for semantic errors.
- The last step is to produce machine code that matches the meaning and meaning of these words in the sentence. 
If necessary, the generated code can be optimized to reduce memory or increase speed.
- In order to make more efficient optimizations and/or to generate code for different machines, 
first, a more convenient generalized intermediate code is generated, 
then this intermediate code is optimized and converted to real machine code for the desired machine.
- Some compilers produce fast interpretable code for a virtual machine and run this program on different machines by interpreting this code.

## 2. Grammar, production, alphabet, language:
The grammar of a language (G) is as in the simple example shown below:
```
sentence -> subject object verb
subject -> I | name
object -> the book | home
verb -> went | took
```
The template is determined by the production rules.

- Here the "|" symbol means "or". 
Each of these rules, which derives the right-hand symbols from the left-hand symbol, is called a production. 
The set of production rules is denoted by P.
- Symbols that are not found on the left side of any production, are called terminal symbols. 
In the above example, they are symbols I, name, the book, home, went, and took.
- The set of terminal symbols is indicated by T. 
For example, the sentence "Michael took the book" is transmitted to the parser by the scanner in the following manner: name(Michael), took, the book.
The thing in parentheses, "Michael", is the attribute of the terminal symbol "name".
- These attributes, which are transmitted to the parser together with the terminal symbols, 
are processed by the parser when the productions are applied.
- In this example, the parser registers the information that "Michael" is a name for future use in the symbol table. 
Then it applies productions to the symbols to obtain the sentence symbol.
- Symbols other than terminal symbols are called non-terminal symbols. 
In this example, the non-terminal symbols are "sentence", "subject", "object", and "verb". 
The set of non-terminal symbols is indicated by N.
- The special symbol, which derives all symbols from itself, is called a sentencial symbol and is denoted by S. 
In this example, the sentencial symbol is "sentence".
- S, P, N, T quadruplet is called grammar and is denoted by G = {S, P, N, T}.
- V=N∪T (N union T) is called an alphabet.
- The ordered symbol group consisting of terminal symbols, resulting from subsequent application of productions of P, to the S symbol is called a sentence.
- The whole set of sentences that can be derived from a grammar is called a language and is denoted by L(G).
- The special terminal symbol, indicated by EPSILON, corresponds to nothing (not to be mixed with the space character).

## 3. Classification of grammars:
According to the Chomsky classification, grammars are placed into four classes, from the most general to the most specific as follows:

1. Free grammars:
Productions are of the form: u->v. Here u,v∈V ve u≠EPSILON.

2. Context-sensitive grammars:
Productions are of the form: uxv->uvw. Here u,v,w∈V, u≠EPSILON and x∈N.
That is, the intermediate symbol x can only be derived with the symbol group v if it is surrounded by groups of u and w symbols.

3. Context-free grammars:
Productions are of the form: x->v. Where v∈V and x∈N. Programming languages ​​are usually produced from such grammars. 
Such languages ​​are always parsable.

4. Finite (regular) grammars:
Productions are of the form: x->a or x->ay. Here x,y⊂N and a⊂T. 
Such grammars are used to describe the words of the language, that is, a grammar that is used by the scanner during word analysis. 
Finite grammars can be effectively parsed by finite state machines.

The reason why word analysis and syntax analysis are done separately, is that they belong to different grammar classes. 
Word analysis is done using much more effective algorithms.

## 4. Equivalence and ambiguity in grammars:
If the sets L(G) and L(H) are equal, the grammars G and H are called equivalent grammars. 


When productions are applied in a different order and the same sentence can be derived in different ways, 
this kind of grammar is called an ambiguous grammar. 
The parser must be able to recognize sentences unambiguously in any way, 
otherwise, the results will be different each time the sentence is parsed. 
We frequently encounter ambiguous grammars in arithmetic operations, 
in which case ambiguity can be resolved by defining the right or left association and priority rules for arithmetic operations. 

For example in the following grammar:
```
s -> aa
a -> x | xx
```
The sentence "xxx" can be parsed in two different ways:

<table>
    <tbody><tr>
        <td align="center"></td>
        <td align="center"></td>
        <td align="center"><strong>s</strong></td>
        <td align="center"></td>
        <td align="center"></td>
        <td align="center"></td>
        <td align="center"></td>
        <td align="center"></td>
        <td align="center"><strong>s</strong></td>
        <td align="center"></td>
        <td align="center"></td>
    </tr>
    <tr>
        <td align="center"></td>
        <td align="right">/</td>
        <td align="center"></td>
        <td align="left">\</td>
        <td align="center"></td>
        <td align="center"><strong>veya</strong></td>
        <td align="center"></td>
        <td align="right">/</td>
        <td align="center"></td>
        <td align="left">\</td>
        <td align="center"></td>
    </tr>
    <tr>
        <td align="center"></td>
        <td align="center"><strong>a</strong></td>
        <td align="center"></td>
        <td align="center"><strong>a</strong></td>
        <td align="center"></td>
        <td rowspan="3"></td>
        <td align="center"></td>
        <td align="center"><strong>a</strong></td>
        <td align="center"></td>
        <td align="center"><strong>a</strong></td>
        <td align="center"></td>
    </tr>
    <tr>
        <td align="right">/</td>
        <td align="center"></td>
        <td align="right">/</td>
        <td align="center"></td>
        <td align="left">\</td>
        <td align="right">/</td>
        <td align="center"></td>
        <td align="left">\</td>
        <td align="center"></td>
        <td align="left">\</td>
    </tr>
    <tr>
        <td align="center"><strong>x</strong></td>
        <td align="center"></td>
        <td align="center"><strong>x</strong></td>
        <td align="center"></td>
        <td align="center"><strong>x</strong></td>
        <td align="center"><strong>x</strong></td>
        <td align="center"></td>
        <td align="center"><strong>x</strong></td>
        <td align="center"></td>
        <td align="center"><strong>x</strong></td>
    </tr>
</tbody></table>

One of these parse trees should be preferred by the parser.

## 5. Attributes and extended grammar:
Each symbol has its own attributes. 
In addition to production, the grammar obtained from the processes related to these qualities is called expanded grammar. 
The nature of the symbol on the left is if the attributes of the symbols on the right are generated 
by processing the attributes of the symbols on the right:

x -> y1 y2 y3 ... yn and x.a = f(y1.a, y2.a, y3.a, ..., yn.a)

Here, x.a means the attribute of x, and it is synthesized from attributes of y1, y2, ..., yn.
This is called a synthesized attribute.

## 6. Using Lex and YACC:
Lex and YACC are programs available on the UNIX system, and which provide a great convenience in building compilers. 
Lex generates a scanner that can parse a given finite grammar. 
Similarly, YACC generates a parser that accepts an extended LALR grammar and performs actions and operations using the parsed symbols and their attributes, defined in the production rules. 
They generate the scanner and the parser programs in C language.

## 7. Lex file format:
```
% {
C expressions (optional)
%}
lex definitions (optional)
%%
lex regular-expressions and operations
%% (optional)
C statements (user functions)
```

## 8. Lex regular expressions and operators:
The regular expressions used in Lex include lex operators and the characters that we want to be recognized by the scanner.
```
.     single character other than \n            .a              a and its previous character that is not beginning of a line
*     zero or more characters                   a[a-z]*         lower case words beginning with a (including the word "a")
+     one or more characters                    a[a-z]+         lower case words beginning with a (excluding the word "a")
?     optional character                        XY?Z            XZ or XYZ
|     or (either lefthand or righthand)         a|b             a or b
xy    concatenation (x and y concatenated)      abc             concatenated a, b and c ("abc")
( )   grouping                                  (UNIX)|(Unix)   UNIX or Unix
" "   cancels the meaning of operators          c"++"           the word "c++"
\     1. cancels the meaning of an operator     c\+\+           the word "c++"
      2. C escape character                     \n              end of line character
      3. octet representation                   \176            ~ character (ASCII=176)
[ ]   character group or range                  [A-Z]           an upper case letter (from A to Z)
[^ ]  exclusion of characters                   [^XYZ]          a character other than X,Y or Z
^     beginning of line                         ^(abc)          the word "abc" at the beginning of a line
$     end of line                               $a              "a" at the end of a line
{m,n} minimum m, maximum n number of            a{1,5}          up to 5 "a" characters (a, aa, aaa, aaaa, aaaaa)
```

## 9. Lex Definitions:
A Lex definition consists of a name and a regular-expression pair. The name on the left can be used in other expressions in exchange for the corresponding regular expression on the right.

Sample:
```
integer [0-9]+
%%
integer          {sscanf(yytext, "%d", &yylval.ival); return INTEGER;}
integer\.integer {sscanf(yytext, "%f", &yylval.fval); return REAL;}
```
In this example, the statements inside `{}` are C statements. 
`yytext` is the generated string variable, which stores the parsed word (an integer such as 15, 6349, or a real number such as 0.359, 1368.4). 
`yylex() is the generated function, which is called by the parser, and returns the recognized (parsed) token from the scanner to the parser.
In addition, the attributes of the token are also passed by the global variable `yylval`. 
Since the expressions in `{}` are included in the C block, it is also possible to define the local C variable here. 
The type of `yylval` is defined in the parser generated by YACC, as YYSTYPE.

## 10. YACC file format:
```
%{
C expressions (optional)
%}
YACC definitions
%%
YACC production rules and operations
%%                 
C expressions (user functions and the main() function)
```

## 11. YACC definitions:
`%union{}`: defines `yylval` as a `union` C type. If we don't want a union type, we should place an expression such as `#define YYSTYPE type` inside the `% {%}`. We should place the fields that belong to the union type, inside `%union{}`.

`%token`: terminal symbols should be defined as`%token symbol`. If `% union {}` is used, the type of the token, `the union` in the name of that type of field t 'token <t>` symbol should be specified as.

`%left`,`%right': if an operator has left or right association, we can define it using `%left operator` or `%right operator`. 
These are used to eliminate ambiguity in the grammar.

`%nonassoc`: indicates that the operator does not have a left or right association.

`%type`: this is similar to the type definition `%union{}`, but instead used for non-terminal symbols.

The precedence of the operator inversely depends on the row where the operator is defined with `%left`,`%right`, or `%nonassoc`. 
Operator defined in lower rows has higher precedence.

## 12. Calculator example:
```
%{
#define  YYSTYPE int /* type of yylval and the internal YACC stack */
%}
%token   INTEGER
%left    '+' '-'
%left    '*' '/'
%right   '^'
%left    MINUS
%%
exprs    : exprs '+' exprs       {$$=$1+$3;}
         | exprs '-' exprs       {$$=$1-$3;}
         | exprs '*' exprs       {$$=$1*$3;}
         | exprs '/' exprs       {$$=$1/$3;}
         | exprs '^' exprs       {$$=power($1,$3);}
         | '-' exprs %prec MINUS {$$=-$2;}
         | INTEGER               {$$=$1;}
         ;
%%
...
```
In this grammar, the term `1 + 2 * 3 * 4 ^ 5` is parsed bottom up in the following order:

<table border="0">
  <tbody><tr>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td align="center"><b><font face="Courier New">e</font></b></td>
    <td align="center"><b><font face="Courier New">x</font></b></td>
    <td align="center"><b><font face="Courier New">p</font></b></td>
    <td align="center"><b><font face="Courier New">r</font></b></td>
    <td align="center"><b><font face="Courier New">s</font></b></td>
  </tr>
  <tr>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">/</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">/</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">\</font></b></td>
  </tr>
  <tr>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">/</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">/</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td align="center"><b><font face="Courier New">e</font></b></td>
    <td align="center"><b><font face="Courier New">x</font></b></td>
    <td align="center"><b><font face="Courier New">p</font></b></td>
    <td align="center"><b><font face="Courier New">r</font></b></td>
    <td align="center"><b><font face="Courier New">s</font></b></td>
  </tr>
  <tr>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">/</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">/</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">/</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">|</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">\</font></b></td>
  </tr>
  <tr>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">/</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">/</font></b></td>
    <td align="center"><b><font face="Courier New">e</font></b></td>
    <td align="center"><b><font face="Courier New">x</font></b></td>
    <td align="center"><b><font face="Courier New">p</font></b></td>
    <td align="center"><b><font face="Courier New">r</font></b></td>
    <td align="center"><b><font face="Courier New">s</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">|</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td align="center"><b><font face="Courier New">e</font></b></td>
    <td align="center"><b><font face="Courier New">x</font></b></td>
    <td align="center"><b><font face="Courier New">p</font></b></td>
    <td align="center"><b><font face="Courier New">r</font></b></td>
    <td align="center"><b><font face="Courier New">s</font></b></td>
  </tr>
  <tr>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">/</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">/</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">/</font></b></td>
    <td><b><font face="Courier New">|</font></b></td>
    <td><b><font face="Courier New">\</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">|</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">/</font></b></td>
    <td><b><font face="Courier New">|</font></b></td>
    <td><b><font face="Courier New">\</font></b></td>
  </tr>
  <tr>
    <td><b><font face="Courier New">1</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">+</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">2</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">*</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">3</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">*</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">4</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">^</font></b></td>
    <td><b><font face="Courier New">&nbsp;</font></b></td>
    <td><b><font face="Courier New">5</font></b></td>
  </tr>
</tbody></table>

- Bottom-up derivation/production follows the post-order traversal of this tree structure. 
In other words, for each node, all its sub-branches are traversed from left to right, and finally, the node itself is traversed.
- Operations are carried out by the C statements (C blocks) inside { } next to the production rules. 
Here, the pseudo-variables $n correspond to the attributes of the symbols. 
The attribute of the non-terminal symbol on the left is indicated by $0 or $$, 
and the attribute of the k.th symbol on the right is represented by $k. 
Inside {}, the result obtained by processing $1, $2, ..., $n, is assigned to $$.
- The attributes along with the symbols, move upward on the stack used by the parser, there is no real tree data structure used in the parser.
- The symbol on the left of the top rule, or another symbol defined by %start, is the sentencial symbol.
Starting from the terminal symbols, all symbols are derived from right to left of the production row, and from bottom row to top row.
- Parsing/compilation ends when the sentencial symbol is reached.
- A special symbol, the error symbol, is used to derive any syntax errors encountered during parsing/compilation. This allows the parser to go on parsing next statements and help gather more information about the errors in the source program.

## 13. Local variable storage and nested block scope:
- The Pascal source code is compiled to intermediate code (p-code) and executed on the virtual stack-machine called p-machine.
- The p-machine is a virtual machine that processes all its commands using its runtime stack. The commands are called p-code.
- This virtual machine contains only a stack and two registers that address the stack. 
All data, including pointer variables, is kept on the stack. 
Results of arithmetic and logic operations are also placed on the stack.
- Since Pascal language is a  block-structured language (unlike C, BASIC, or FORTRAN), nested blocks (nested functions and procedures) can be defined.
- A symbol can be accessed through the block in which it is defined and all the nested blocks within it, and is inaccessible from the outer blocks. 
Storage duration, or the life-span of a local symbol, is the same as the life span of the container block.
- Functions and procedures can call themselves recursively. Temporary variables can also be defined. 
All these can be achieved by using the runtime stack of the p-machine.

During the course of the execution of a p-code program, the stack frame is as follows:
<table><tbody>
  <tr>
    <td align="center"><b><font face="Courier New">variable<sub>N</sub></font></b></td>
    <td align="left"><-</td>
    <td align="left"><b><font face="Courier New">T</font></b></td></tr>
  <tr>
    <td align="center"><b><font face="Courier New">...</font></b></td>
  </tr>
  <tr>
    <td align="center"><b><font face="Courier New">variable<sub>2</sub></font></b></td>
  </tr>
  <tr>
    <td align="center"><b><font face="Courier New">variable<sub>1</sub></font></b></td>
  </tr>
  <tr>
    <td align="center"><b><font face="Courier New">return address</font></b></td>
  <tr>
    <td align="center"><b><font face="Courier New">dynamic link</font></b></td>
  </tr>
  <tr>
    <td align="center"><b><font face="Courier New">static link</font></b></td>
    <td align="left"><-</td>
    <td align="left"><b><font face="Courier New">B</font></b></td></tr>
  </tr>
  <tr>
    <td align="center"><b><font face="Courier New">parameter<sub>M</sub></font></b></td>
  </tr>
  <tr>
    <td align="center"><b><font face="Courier New">...</font></b></td>
  </tr>
  <tr>
    <td align="center"><b><font face="Courier New">parameter<sub>2</sub></font></b></td>
  </tr>
  <tr>
    <td align="center"><b><font face="Courier New">parameter<sub>1</sub></font></b></td>
  </tr>
  <tr>
    <td align="center"><b><font face="Courier New">return value</font></b></td>
  </tr>
</tbody></table>

- Here, B and T are the two registers of the p-machine where the compiled intermediate code (p-code) is executed. 
T is the pointer which always shows the top of the stack, and B is the pointer we use to address the stack.
- All addresses are specified by adding a positive or negative offset to the base address B holds. 
Parameter1, ..., parameterM is used for parameters of the current function/procedure, variable1, ..., variableN is used for the local variables defined in the current function/procedure, return value indicates the area reserved for the return value of the current function.
- Static link is the base address of the lexical outer block, which is used for accessing the variables of this lexical outer block (function/procedure) that contains the current block (function/procedure or procedure).
- The dynamic link is the base address of the calling function/procedure's stack frame, to be used for switching back to the calling function/procedure's local scope (local variables, parameters, etc.)
- The return address is the address of the command to be executed after the current function/procedure ends or returns.

## 14. Symbol table:
- The symbols defined in a block can only be seen in this block and within the blocks in this block. Thanks to the above stack structure, this is provided when the program is processed.
- The symbol table used in the compiler is in the stack structure to ensure that this is achieved during the recovery phase and the symbols in the different blocks are not mixed with each other.

For example, if block B and C are located in block A, the status of the symbol stack is:

When compiling block A:
```
Symbols of block A
```

When compiling block B:
```
Symbols of block B
Symbols of block A
```

When compiling block C:
```
Symbols of block C
Symbols of block A
```

## 15. P-code:
```
p-code    opcode   what it does
LIT 0,N     00     push value N to the stack
OPR 0,N     01     execute arithmetic operation N on the stack
LOD L,D     02     push a variable to the stack
LODX L,D    12     push an array variable to the stack
STO L,D     03     store the popped value in a variable
STOX L,D    13     store the popped value in an array variable
CAL L,A     04     call a procedure/function
INT 0,N     05     add a positive or negative constant to the T register
JMP 0,A     06     jump to address
JPC C,A     07     jump to address conditionally
CSP 0,N     08     call a standard procedure/function
```
Note that the opcodes are in hexadecimal format.

## 16. P-code in detail:
```
p-code      what it does                                     algorihmic expression
LIT 0,N     push value N to the stack                        PUSH N
OPR 0,0     return from procedure/function                   return (exit current block)
OPR 0,1     negate                                           POP A, PUSH (-A)
OPR 0,2     add                                              POP A, POP B, PUSH (B+A)
OPR 0,3     subtract                                         POP A, POP B, PUSH (B-A)
OPR 0,4     multiply                                         POP A, POP B, PUSH (B*A)
OPR 0,5     divide                                           POP A, POP B, PUSH (B/A)
OPR 0,6     lowest bit                                       POP A, PUSH (A AND 1)
OPR 0,7     mod                                              POP A, POP B, PUSH (B MOD A)
OPR 0,8     is equal to                                      POP A, POP B, PUSH (B =? A)
OPR 0,9     is not equal to                                  POP A, POP B, PUSH (B ≠? A)
OPR 0,10    is less than                                     POP A, POP B, PUSH (B <? A)
OPR 0,11    is greater than or equal to                      POP A, POP B, PUSH (B >=? A)
OPR 0,12    is greater than                                  POP A, POP B, PUSH (B >? A)
OPR 0,13    is less than or equal to                         POP A, POP B, PUSH (B <=? A)
OPR 0,14    or                                               POP A, POP B, PUSH (B OR A)
OPR 0,15    and                                              POP A, POP B, PUSH (B AND A)
OPR 0,16    not                                              POP A, PUSH (NOT A)
OPR 0,17    shift left                                       POP A, POP B, PUSH (B shift left (A times))
OPR 0,18    shift right                                      POP A, POP B, PUSH (B shift right (A times))
OPR 0,19    increment                                        POP A, PUSH (A+1)
OPR 0,20    decrement                                        POP A, PUSH (A-1)
OPR 0,21    copy                                             POP A, PUSH A, PUSH A
LOD L,D     push value from address                          load A from (base of level offset L)+D, PUSH A
LOD 255,0   push value from address popped from the stack    POP address, load A with byte from address, PUSH A
LODX L,D    push value from indexed address                  POP index, load A from (base of level offset L)+D+index, PUSH A
STO L,D     store popped value at address                    POP A, store A at (base of level offset L)+D
STO 255,0   store popped value at popped address             POP A, POP address, store low byte of A at address
STOX L,D    store popped value at indexed address            POP index, POP A, STORE A at (base of level offset L)+D+index
CAL L,A     call procedure/function                          call procedure/function at p-code location A, with base at level offset L
CAL 255,0   call address popped from the stack               POP address, PUSH return address(=PC), jump to address
INT 0,N     add N to T                                       T=T+N
JMP 0,A     jump                                             jump to p-code at location A
JPC 0,A     jump if popped value is true                     POP A, if (A and 1) = 0 then jump to p-code at location A
CSP 0,0     input one character                              INPUT A, PUSH A
CSP 0,1     output one character                             POP A, OUTPUT A
CSP 0,2     input an integer                                 INPUT A#, PUSH A
CSP 0,3     output an integer                                POP A, OUTPUT A#
CSP 0,8     output a character string                        POP A, FOR I:=1 to A DO BEGIN POP B; OUTPUT B; END
```
Note that true=1, false=0

POP X means to remove the top element of the stack and load it into X (the
stack is now one smaller). In other words, copy the value pointed by the T register into X, and decrement T.

PUSH X means to place the value of X onto the top of the stack (the stack is
now one bigger). In other words, increment the T register, and copy the value pointed by the T register into X.

## 17. P-code templates for basic Pascal structures:
```
Pascal expression              equivalent p-code
x+10*y[5]                      LOD x
                               LIT 10
                               LIT 5
                               LODX Y
                               OPR *
                               OPR +
      
a:=expr;                       (expr)
                               STO a
      
p^=expr;                       (expr)
                               STO 255 p
      
if expr then stmt1 else stmt2; (expr)
                               JPC 0,1b1
                               (stmt1)
                               JMP label2
                               label1: (stmt2)
                               label2: ...
      
for i=expr1 to expr2 do stmt;  (expr1)
                               STO I
                               (expr2)
                               label1: OPR CPY
                               LOD I
                               OPR >=
                               JPC 0,label2
                               (stmt)
                               LOD I
                               OPR INC
                               STO I
                               JMP label1
                               label2: INT -1
      
while expr do stmt             label1: (expr)
                               JPC 0,label2
                               (stmt)
                               JMP label1
                               label2: ...
      
case expr of                   (expr)
const1,const2: stmt1;          OPR CPY
const3 : stmt2;                LIT const1
else stmt3 end;                OPR =
                               JPC 1,label1
                               OPR CPY
                               LIT const2
                               OPR =
                               JPC 0,label2
                               label1: (stmt1)
                               JMP 1abel4
                               label2: OPR CPY
                               LIT const3
                               OPR =
                               JPC 0,1abel3
                               (stmt2)
                               JMP 1abel4
                               1abel3: (stmt3)
                               1abel4: INT -1
      
repeat stmt until expr;        label1: (stmt)
                               (expr)
                               JPC 0,label1
      
i=func1(expr1,expr2);          INT 1
                               (expr1)
                               (expr2)
                               CAL func1
                               INT -2
```
## 18. Data structures used in the compiler:
- Linked lists are used to create dynamic arrays.
- For each new node of the list, dynamic memory must be allocated with the malloc() function in C and then the previous node should be connected to this new node.

- Single linked list:
```
head->d
      n->d
         n->d
            n->d
               n=0
```
Here, d holds the data and n is the pointer to the next node. 
The address of the first node is kept in head. The value of n in the last node is NULL, or 0.
It is declared in C like this:
```
struct snode {
    int d;
    struct snode *n; /* next */
} *head;
```
- Double linked list:
```
      p=0 
head->d<-p
      n->d<-p
         n->d<-p
            n->d<-tail
               n=0
```
Here, d holds the data, n is the pointer to the next node, and p is the pointer to the previous node. 
The address of the first node is kept in head, and the address of the last node is kept in tail. 
The value of n in the last node is NULL, or 0. Likewise, the value of p in the first node is NULL, or 0.
It is declared in C like this:
```
struct snode {
    int d;
    struct snode *p; /* previous */
    struct snode *n; /* next */
} *head, *tail;
```
- Dynamic stack:

When the double linked list is used as a stack, tail pointer, which points to the last node, also points to the top of the stack. 
Thus we obtain a stack data structure which can be resized dynamically.

## 19. Types and functions used in the compiler:

- `symblocktop`: Pointer addressing the top block on the symbol table (stack).
- `pushsymblock()`: Adds (pushes) a new block on top of the symbol table (stack).
- `popsymblock()`: Removes (pops) the top block from the symbol table (stack).
- `instconst()`, `instlabel()`, `insttype()`, `instvar()`: Respectively, adds a new constant, label, type, or variable declaration inside the top block on the symbol table (stack).
- `makeptrtype()`, `makeenumtype()`, `makerangetype()`, `makeidtype()`, `makerectype()`, `makearraytype()`, `makeuniontype()`, `makesettype()`, `makefiletype()`: Creates various type nodes. Type of a variable is stored on the symbol table in a special variant data structure, which varies for each type.

Type node is declared in C like this:
```
struct stype {
    char metatype; /* TENUM, TID, TREC, TUNION, TFILE, TSET, TRANGE, TARR */
    void *restptr; 
};
```
Here, metatype can hold one of the constants `TENUM`, `TID`, `TREC`, `TUNION`, `TARR`, `TFILE`, `TSET` ve `TRANGE`. 
restptr points to a different data structure for each different metatype. 
These data structures are:
<table>
    <tr>
        <td>metatype</td>
        <td>restptr</td>
    </tr>
    <tr>
        <td>TENUM</td>
        <td>-->id0-->id1-->...-->idN</td>
    </tr>
    <tr>
        <td></td>
        <td>(linked list, id's are strings, which are also registered in the symbol table as constants from o to N)</td>
    </tr>
    <tr>
        <td>TRANGE</td>
        <td>-->min,max,type (type is either character or number)</td>
    </tr>
    <tr>
        <td>Example:</td>
        <td>VAR r:(A..Z); (min=A, max=Z, tip=character)</td>
    </tr>
    <tr>
        <td>TID</td>
        <td>-->id (id: name of the equivalent type)</td>
    </tr>
    <tr>
        <td>Example:</td>
        <td>TYPE numerictype = INTEGER; (type that is equivalent to INTEGER)
    </tr>
    <tr>
        <td></td>
        <td>VAR i:numerictype; (id="numerictype")</td></td>
    </tr>
    <tr>
        <td>TARRAY</td>
        <td>-->type,indextype1-->...-->indextypeN</td>
    </tr>
    <tr>
        <td></td>
        <td>(linked list, types of indexes in an array of N elements)</td>
    </tr>
    <tr>
        <td>TREC and TUNION</td>
        <td>-->fields1-->...-->fieldsN</td>
    </tr>
    <tr>
        <td></td>
        <td>(linked list, each one of fieldsN is a tuple of a linked list of the names of the fields of same type and the field type)</td>
    </tr>
    <tr>
        <td>Example:</td>
        <td>VAR u: UNION
f1,f2,f3: REAL; (fields1->(f1->f2->f3,REAL))
i1,i2: INTEGER; (fields2->(i1->i2,INTEGER))
END;</td>
    </tr>
    <tr>
        <td>TPTR</td>
        <td>-->type (type of the variable addressed by this pointer type)</td>
    </tr>
    <tr>
        <td>Example:</td>
        <td>VAR p: ^ INTEGER; (type=INTEGER)</td>
    </tr>
    <tr>
        <td>TFILE</td>
        <td>-->type</td>
    </tr>
    <tr>
        <td>Example:</td>
        <td>VAR f: FILE OF CHAR; (type=CHAR)</td>
    </tr>
    <tr>
        <td>TSET</td>
        <td>-->type</td>
    </tr>
    <tr>
        <td>Example:</td>
        <td>VAR s: SET OF INTEGER; (type=INTEGER)td>
    </tr>
</table>


`instproc()`, `instfunc()`: Adds a new procedure/function to the top block in the symbol table (stack).

`addXnode()`: Adds a new node to the linked list of X's.
These type of functions are: `addidnode()`, `addtypenode()`, `addfieldsnode()`, `addfparsec()`, `addexprnode()`, `addclinenode()`, `addconstnode()`, `addvarnode()`

`linkNcodes()`: Concatenates linked lists containing p-code nodes, into one large linked list.

`codeXXX()`: Generates a block of p-code. 
The generated code is actually a single linked list with p-code as data. 
The blocks (linked lists) of p-code produced at different times during parsing, are combined with `linkNcodes ()` at different stages and finally converted into one large block. 
The address of this final linked list is assigned to the variable named program, later used for the execution of the program.
```
