- From lexer we got a stream of tokens, now we need a richer, complex representation
- It should be simple for the parser to produce and easy for the interpreter to consume
- One common one is [[Abstract Syntax Trees]]
- Another one is [[Bytecode]]
- We use **Syntax Grammar** here
- Alphabet is the Token Set
# Context Free Grammar

| Terminology | Lexical grammar | Syntactic grammar    |
| ----------- | --------------- | -------------------- |
| Alphabet    | characters      | tokens               |
| String      | token           | Expression/statement |
## Rules for Grammars
- We can't write the infinite number of generated strings from a grammar
- We use rules and use them to *generate* the strings in the grammar
- Strings created this way are called **derivations** because each is _derived_ from the rules of the grammar
- Rules are called **productions** because they _produce_ strings in the grammar
- Each production in a context-free grammar has a **head**—its name—and a **body**, which describes what it generates. In its pure form, the body is simply a list of symbols. Symbols come in two delectable flavors:
	1.  A **terminal** is a letter from the grammar’s alphabet. It's a literal value. These are kind of endpoints.
	2. A **nonterminal** is a named reference to another rule in the grammar. In this way, the grammar composes.
- Multiple rules with same name will lead to any of the rules
- Tracking the number of required trailing parts is beyond the capabilities of a regular grammar. Regular grammars can express _repetition_, but they can’t _keep count_ of how many repetitions there are, but Context Free Grammars can.

# Ambiguity and Parsing Game
- Since any of the rules can be used to generate string from multiple rules with same nonterminal name, and where different choices of productions can lead to the same string, that grammar would be ambiguous.
Take the following grammar, for example,
```bnf
expression     → literal
               | unary
               | binary
               | grouping ;

literal        → NUMBER | STRING | "true" | "false" | "nil" ;
grouping       → "(" expression ")" ;
unary          → ( "-" | "!" ) expression ;
binary         → expression operator expression ;
operator       → "==" | "!=" | "<" | "<=" | ">" | ">="
               | "+"  | "-"  | "*" | "/" ;
```
A valid string of  the grammar is, ![[Pasted image 20260216114006.png]]
and two parse trees can be used for this string production,![[Pasted image 20260216114146.png]]
In other words, the grammar allows seeing the expression as `(6 / 3) - 1` or `6 / (3 - 1)`. When parsing, ambiguity means the parser may misunderstand the user’s code. Ambiguity in parsing means a grammar allows multiple valid parse trees for the same input. This problem is resolved using precedence and associativity rules
- Precedence defines which operator binds stronger
- Associativity defines grouping order when operators have the same precedence(`=` is right associative and almost rest are left)
In C,

| Name       | Operators         | Associates |
| ---------- | ----------------- | ---------- |
| Equality   | `==`, `!=`        | Left       |
| Comaprison | `>` `>=` `<` `<=` | Left       |
| Term       | `-` `+`           | Left       |
| Factor     | `/` `*`           | Left       |
| Unary      | `!` `-`           | Right      |

Right now, the grammar stuffs all expression types into a single `expression` rule. That same rule is used as the non-terminal for operands, which lets the grammar accept any kind of expression as a subexpression, regardless of whether the precedence rules allow it.
Fix: Stratify the grammar into separate rules for each precedence level, like
```
expression → ...
equality → ...
comparison → ...
term → ...
factor → ...
unary → ...
primary → ...
```
Each rule here only matches expressions at its precedence level or higher. For example, `unary` matches a unary expression like `!negated` or a primary expression like `1234`. And `term` can match `1 + 2` but also `3 * 4 / 5`. The final `primary` rule covers the highest-precedence forms—literals and parenthesized expressions.
Let us do that one by one(easiest ones first)
Since equality has lowest precedence, so if we match it, we cover everything, therefore,
```expression     → equality```.
Also `primary` is also simple.
```
primary       → NUMBER | STRING | "true" | "false" | "nil"
               | "(" expression ")" ;
```
Now let us do `unary`, 
```
unary → ("-" | "!") unary
```
It looks right, but it does not terminate, so we terminate it(by adding something of higher precedence(enough foreshadowing its `primary`)).
```
unary → ("-" | "!") unary | primary;
```
Let's do binary operators,
```
factor → factor ("*" | "/") unary | unary;
```
This is correct but not optimal for parsers like Recursive Descent, LL Parsers (Top down) which can't parse such grammars which have left recursion(reasons discussed later).
```
factor → unary(("*" | "/")unary)*;
```
and same for `term` and `equality`
```
term → factor (("+" | "-") factor)*;
```
```
equality → comparison (("!=" | "==") comparsion)*;
```
```
comparison → term (("!=" | "==") term)*;
```
# Parsing Techniques
- Multiple Parsing Techniques exist, like LL(k), LR, LALR, parser combinators, Earley Parsers, the shunting yard algorithm, packrat parsing etc
- We shall discuss [[Recursive Descent Parsing]] and [[Pratt Parsing]] in detail.
# Syntax Error
Roles of parser:
1. Convert sequence of tokens into structured code representation
2. Given invalid sequence of tokens, detect and report errors
The second job is important(don't ignore/underestimate). Modern IDEs and editors(I use neovim btw) the parser is constantly reparsing code, when the programmer is editing the source code, for syntax highlighting and support like autocomplete, which means parser will encount incomplete code most of the times -- so yeah, its important.
Requirements for error  handling
- Must detect and report the error(should not miss as incorrect code will be sent further in the pipeline)
- Avoid crashing or hanging - In case we get errors, things like segfault or getting stuck in infinite loop is not acceptable.
Optional
- Be fast
- Report as many distinct errors as there are
- Minimize *cascaded* errors
The last 2 ones are quite cumbersome.
**Error Recovery** - the way parser responds to an error and it keeps going to look for later errors.
## Panic mode error recovery
- Most common recovery technique -> panic mode.
- When parser detects an error, it enters **Panic Mode** -- atleast one token does not make sense given its current state in grammar production
- It has to **synchronize**, it means that it has to align its state and forthcoming sequence of token so that the next token does match the rule being parsed.
How do we synchronize?
1. We select some rule in grammar to mark as synchronization point.
2. State Fix is done by jumping out any nested production until it gets back to the synchronization rule.
3. We discard the stream of tokens until we match tokens that can mark the start of new rule.
Trade off in this method: Real syntax errors in discarded stream are not reported but it also means that any misatken cascaded errors that are side effects of initial errors aren't *falsely* reported either.

Traditional Place to synchronize: between statements.
# Error Productions
- Way to handle common syntax errors
- We augment grammar with rule that matches *erroneous* syntax.
- Parser matches it but then reports it as an error instead of producing syntax tree.
For example, if language does not support `+123`, so we augment unary rule to 
```
unary → ("-" | "!" | "+") unary | primary;
```
- parse consumes `+` without going into panic mode or leaving parser in weird state
- It is good as parser author knows what went wrong and what the source programmar might wanna do.
