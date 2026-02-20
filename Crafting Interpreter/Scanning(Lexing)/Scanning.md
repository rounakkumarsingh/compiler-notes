# Framework
- Either take a whole script and `run`  it
- Make a repl session and `run` each prompt
# Error Handling
- Error handling is important for usability, not theory
	- Helps the programmer understand what went wrong
	- Position info matters (line, column, lexeme)
- Error reporting should be separate from error detection
	- Scanner / parser detect errors
	- A central system decides how to present them
- jlox uses a global `hadError` flag
	- Prevents executing invalid code
	- Enables non-zero exit code
	- Reset in REPL so one error doesn’t kill the session
- Error reporting lives in `Lox` as a pragmatic compromise
	- Avoids over-engineering (no `ErrorReporter` interface)
	- Keeps UI concerns out of scanner/parser
- In a real language, errors could be reported to:
	- stderr
	- IDE UI
	- logs, etc.
# Tokens and Lexemes
- The aim of lexical analysis, i.e of the **lexer** is to take in a whole string of program(be it prompt or a whole file(s)), group them together into smallest sequence that still represent something, these blobs of characters are called **lexemes**.
- Note that these lexemes are nothing but raw strings from the source. In the process of lexing, we bundle these lexemes with additional data(like type, position etc...) to get a **token**.
Following are the the things that can be attached
## Token Type
```java
package com.craftinginterpreters.lox;

enum TokenType {
  // Single-character tokens.
  LEFT_PAREN, RIGHT_PAREN, LEFT_BRACE, RIGHT_BRACE,
  COMMA, DOT, MINUS, PLUS, SEMICOLON, SLASH, STAR,

  // One or two character tokens.
  BANG, BANG_EQUAL,
  EQUAL, EQUAL_EQUAL,
  GREATER, GREATER_EQUAL,
  LESS, LESS_EQUAL,

  // Literals.
  IDENTIFIER, STRING, NUMBER,

  // Keywords.
  AND, CLASS, ELSE, FALSE, FUN, FOR, IF, NIL, OR,
  PRINT, RETURN, SUPER, THIS, TRUE, VAR, WHILE,

  EOF
}
```
## Literal Values
- lexemes for literal values—numbers and strings and the like
## Location Information
- Store location of tokens 
```java
package com.craftinginterpreters.lox;

class Token {
  final TokenType type;
  final String lexeme;
  final Object literal;
  final int line; 

  Token(TokenType type, String lexeme, Object literal, int line) {
    this.type = type;
    this.lexeme = lexeme;
    this.literal = literal;
    this.line = line;
  }

  public String toString() {
    return type + " " + lexeme + " " + literal;
  }
}
```
# Regular Expressions
- The rules that determine how a particular language groups characters into lexemes are called its **lexical grammar**
- Most languages lexical grammar is **regular**
- We can use [flex](https://github.com/westes/flex) or lex
# The Scanner
```
scanTokens():
	while not at end:
		start = current
		scanToken()
	scan EOF
	
scanToken():
	c = advance()
	... matching logic
```
- `current` is the next character to be consumed(not the one being processed)
- `start` is the start of the current lexeme
- If no match then raise error
- When two lexical grammar rules can both match a chunk of code that the scanner is looking at, _whichever one matches the most characters wins_ -> maximal much.s
### Helper functions
1. `advance`
	- consumes a char and increments `current` till `EOF`
2. `isAtEnd()`- checks if current is `EOF`
3. `match(tokentype)` consumes the token if tokentype matches else does not consume anything at all                                                                                                                                                                                           