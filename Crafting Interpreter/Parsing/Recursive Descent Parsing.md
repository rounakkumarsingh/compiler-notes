- Top Down Parser -- starts from top grammar rule(with lowest precedence) and works its way down to nested subexpressios
- Simplest way to build parser yet effective
A recursive descent parser is a literal translation of the grammar’s rules straight into imperative code. Each rule becomes a function. The body of the rule translates to code roughly like:

| Grammar notation | Code representation               |
| ---------------- | --------------------------------- |
| Terminal         | Code to match and consume a token |
| Nonterminal      | Call to that rule’s function      |
| `\|`             | `if` or `switch` statement        |
| `*` or `+`       | `while` or `for` loop             |
| `?`              | `if` statement                    |

The descent is described as “recursive” because when a grammar rule refers to itself—directly or indirectly—that translates to a recursive function, that's why no left recursion else will lead to non-stop recursion.
Some utility helpers
- `match(TokenTypes)` if next token to be consumed matches any of the types it consumes and returns True else return False(no consumption)
- `check(Token)` only checks no consumer
- `advance()` - consumes and returns
- `peek()` returns but no consume
- `previous()` returns the last consumed