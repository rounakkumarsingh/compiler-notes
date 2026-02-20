# The Role of Lexical Analyzer
Main role: Read input chars from source files, group them into lexemes and produce a sequence of tokens for Praser to consume for Syntax Analysis.
- Common for it to interact with The Symbol Table -- when lexers identifies an identifier, enters lexeme into the Symbol Table.
![[Pasted image 20260219225127.png]]
Other tasks/things that lexers sometimes do:
1. Stripping comments and whitespaces
2. Corelating Lexical Errors with source program
	a. Associate Line number with errors
	b. Some compilers make a copy of sources programs with error inserted at appropriate positions.
3. Maybe use for macroprocessor expansion
---
Sometimes this phase is divided into two parts:
1. Scanning: Simple, No tokenisation here like comments deletion, stripping whitspaces
2. Lexical Analysis: Produce tokens here
## Lexer vs Parser
Why these portions are separated
1. Simplicity of design - imagine a parser where we would have to handle whitespaces and comments
2. Compiler Efficiency Improved - Let us apply specialized techniques that serves only lexical task not parser
3. Portability - only single layers deals with Input device
## Tokens Patterns Lexemes
- A token = (token name, attribute)
	- Token Name - Abstract symbol representing type/class, for eg, **id**, **literal**, **operator**. Typed in **boldface**. 
	- Attributes will be discussed later
- Pattern - Description of form that lexeme of token may take. For example, for keywords -> keyword itself is pattern, for identifiers->something complex *matched* by many strings
- Lexeme - Sequence of characters in the source program that matches pattern for a token and identified by lexer as an instance of that token.
Common classes of tokens that cover most/all of the tokens that a PL might need:
1. One for each keyword
2. For operators
3. One for identifiers
4. One for literals
5. One for each punctuation symbol
## Attribute for Tokens
- We assume at most one attribute - that attribute itself can be a struct
- Something that describes lexeme of the token instance
For example,
- **id** -> entry in symbol table
- **literals** -> actual value/lexeme in source
## Lexical Errors
Possible Causes: 
1. No patter matched any prefix of remaining input
2. Scanned character not in the alphabet
Most common error recovery strategy - **Panic Mode**. Delete successive input character until lexer matches a pattern.
---
Other more complex recovery stratergies(Often not worth it):
1. Delete one character from the remaining input.
2. Insert a missing character into the remaining input.
3. Replace a character by another character.
4. Transpose two adjacent characters.
# Input Buffering
- Optimizing the task of "reading file"
- Problem: The unknown lookahead
## Two Buffer
- Two buffers of same size, loaded alternatively when we run of out of space
![[Pasted image 20260219235048.png]]
- One `read()` syscall fills $N$ characters in these buffers(We dont do one syscall per char)
- When less than $N$ chars are left, then a special character **eof**, marking the end of teh source file.
- Two pointers
	- `lexemeBegin` - pointing the start of current lexeme
	- `forward` - scans ahead until pattern matches
## Sentinels
- Everytime `forward` is advanced, we have to check if buffer has ended or not, if yes the reload another buffer
- For each char lead we make two tests
	- Buffer end test
	- Character test(is actually multiway instead if single but get the vibe)
- We can merge these checks into one, by extending buffers to hold a sentinel character--a character that can't be part of source programs, a good candidate $\rightarrow$ **eof**-- at the ends.
![[Pasted image 20260220004130.png]]
- Any **eof** char read other than at end of buffers means end of input else end of buffers
Algorithm:
```c
switch ( *forward++ ) {
	case eof:
		if (forward at end of first buffer) {
			reload second buffer;
			forward = start of second buffer;
		} else (forward at end of second buffer) {
			reload first buffer;
			forward = start of first buffer;
		} else {
			// end of input
			terminate
		}
		break;
	Cases for other chars
}
```
- Notice how we only read the input char to check end of buffers
- A `switch` on characters is compiled into a **jump table (array of addresses indexed by the character)**, so dispatch happens in **O(1) time regardless of case order**.
# Specification of Tokens
## Strings and Languages
- Alphabet: Set of symbols. Eg, ASCII, Unicode
- String over an alphabet: Finite sequence of symbols drawn from that alphabet. $s$ has length $|s|$. Empty String $\rightarrow$ $\epsilon$.
- Language: Countable Set of strings over some fixed alphabet. Abstract languages $\rightarrow$ $\phi$ , $\{\epsilon\}$.
Parts of strings
- Prefix- Formed by deleting symbols from end of string
- Suffix- Formed by deleting symbols from start of string
- Substring- Formed by deleting symbols from end and start of string
- proper prefix/suffix/substring =  prefix/suffix/substring -  $\{\epsilon\}$
- Subsequence: FOrmed by deleting one or more symbols from string(not necessarily consecutive)
### Concatenation
For strings $x$ and $y$, $xy$ is the concatenation of $x$ and $y$ - $y$ appended to $x$
$\epsilon$ is identity under concatenation. $\epsilon = \epsilon s = s \epsilon$ 
$s^0 = \epsilon$ , $\forall i \gt 0, s^i = s^{i-1}s$ 
## Operation on Languages

| OPERATION                    | NOTATION and DEFINITION                             |
| ---------------------------- | --------------------------------------------------- |
| Union of $L$ and $M$         | $L \cup M = \{s \mid s \in L \text{ or } s \in M\}$ |
| Concatenation of $L$ and $M$ | $LM = \{st \mid s \in L \text{ and } t \in M\}$     |
| Kleen Closure of $L$         | $L^* = \bigcup_{i = 0}^{\infty} L^i$                |
| Positive Closure of $L$      | $L^+ = \bigcup_{i = 1}^{\infty} L^i$                |
## Regular Expressions
- Defined using induction.
- A regular expression $r$ represents a language $L(r)$.
---
**BASIS**:
1. $\epsilon$ is a regular expression and $L(\epsilon) = \{\epsilon\}$.
2. If $a \in \text{ alphabet}$, $\mathbf{a}$ is a regular expression, and $L(\mathbf{a}) = \{a\}$.
---
**INDUCTION**:
If $r$ and $s$ are two regular expressions, then
1. $(r) | (s)$ is a regular expression representing $L(r) \cup L(s)$.
2. $(r)(s)$ is a regular expression representing $L(r)L(s)$.
3. $(r)^*$ is a regular expression representing $(L(r))^*$.
4. $(r)$ is a regular expression representing $L(r)$.
Last one can be removed by adding precedence and associativity:
a. The unary operator $*$ has highest precedence and is left associative.
b. Concatenation has second highest precedence and is left associative.
c. $|$ has lowest precedence and is left associative.
- - -
Some rules
![[Pasted image 20260220012603.png]]
## Regular Definitions
- For notational convenience, we may wish o give names to certain regular expressions and use them later.
- If $\Sigma$ is an alphabet, then a *regular definition* is a sequence of definitions $$\begin{align}
d_1 &\rightarrow r_1 \\
d_2 &\rightarrow r_2 \\
&\dots \\
d_n &\rightarrow r_n \\
\end{align}$$
where:
1. $d_i \notin \Sigma$ and $d_i$ is unique in $\{d_i\}_{i = 1}^n$
2. $r_i$ is regular expression over $\Sigma \cup \{d1, d2, \dots , d_{i-1}\}$ 
![[Pasted image 20260220014006.png]]
## Extensions of Regular Expressions
1. One or more instance - Positive Closure
2. Zero or one instance - $r? = r | \epsilon$ 
3. Character classes- $a_1 | a_2 | \dots | a_n = [a_1a_2\dots a_n]$ and for a sequence we can write $[a_1-a_n]$![[Pasted image 20260220014424.png]]
# Recognition of Token
- This section is about how to take a set of patterns and build a code/program that recognizes it from input strings(one of the pattern matches the input's prefix) to find the lexemes of tokens. 
## The Setup
In this entire section,we shall talk about the following example
$$\begin{align}
stmt &\rightarrow \mathbf{if} \space expr \space \mathbf{then} \space stmt \\ &\space \mid \space \mathbf{if} \space expr \space \mathbf{then} \space stmt \space \mathbf{else} \space stmt \\ &\space \mid \space \epsilon
\\
\\
expr &\rightarrow term \space \mathbf{relop} \space term \\ &\space \mid \space term \\
\\
term &\rightarrow \mathbf{id} \\ &\space \mid \mathbf {number}
\end{align}$$
The terminals
$$\begin{align}
digit &\rightarrow [0-9] \\ 
digits &\rightarrow digit^+ \\
number &\rightarrow  digit \space (. \space digits)? (\space\mathrm{E} \space [+-]? \space digits \space)? \\
letter &\rightarrow [\mathrm{A} -\mathrm{Z}\mathrm{a}-\mathrm{z}] \\
id &\rightarrow letter (letter \mid digit)^* \\
if &\rightarrow \mathbf{if} \\
then &\rightarrow \mathbf{then} \\
else &\rightarrow \mathbf{else} \\
relop &\rightarrow \space < \space \mid \space > \space \mid \space <= \space \mid \space >= \space \mid \space = \space \mid \space <> \space \\
ws &\rightarrow (\mathbf{blank} \space \mid \space \mathbf{tab} \space| \space \mathbf{newline})^+
\end{align}$$
![[Pasted image 20260220223643.png]]
## Transition Diagrams
- Intermediate step in construction of lexical analyzer
- We convert regular expression(patterns) to "transition diagrams"
Side note: Just like drawing Automatons
- Transition diagrams have a collection of nodes or circle called *states*
- They represent a condition that could occur while scanning the input, looking for a lexeme that matches one of the patterns
- States summarizes all we need to know about what characters we have seen between `lexemeBegin` and `forward`
- There are *edges* directed from one of the states to any other state, including the source state itself, labelled by a transition symbol/set of symbols.
- When we are at a state $s$ and next input symbol is $a$ then, we look for an edge out of state $s$ labelled $a$. If we find such edge, we advance `forward` and then enter the stage where the edge leads to
- We shall assume(for now) that transition diagrams are deterministic, i.e, from a state $s$ there is at most one edge for a symbol $a$.
Important Conventions:
1. Certain states are *accepting* or *final*, denoting that a lexeme has been found, although, entire lexeme **might not** be between `lexemeBegin` and `forward`.  These states are denoted by double circles and if there is any action to be taken -- typically returning a token -- we attach that action to the state
2. In case we need to retract the `forward` pointer on position, we put a $*$ near accepting state(number of $*$ = number of positions to retract) -- the lexeme does not include the symbol that got us to the accepting
3. Start State /initial state is indicated by a edge starting from nowhere labeled "start"
Example:
![[Pasted image 20260220231412.png]]
- Notice how we return token (token name, attribute)
## Recognition of 