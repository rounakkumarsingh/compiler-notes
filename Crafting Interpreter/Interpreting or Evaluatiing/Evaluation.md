- We assume we got parse trees of expression
# Representation of Values
- We need a representation of values in target PL
- In lox, values -> created by expressions and stored in variables.
- User sees Lox Objects, but we implement it in our underlying language our interpreter is written in
- We need to bridge between Lox Dynamic Types and our interpreter/compiler source PL types(it maybe static/dynamic)
- In dynamic langs like pyhton, lox, lua, a single variable has hold value of any type, so we need something like Java's `Object` to represent everything.
- Given a object we can find if runtime value is number or string or etc... using `instanceof`

| Lox type      | Java representation |
| ------------- | ------------------- |
| Any Lox value | Object              |
| `nil`         | `null`              |
| Boolean       | Boolean             |
| number        | Double              |
| string        | String              |
- Define truthiness and falsiness
- Define Target PL notion of *equality*
# Runtime Errors
- Errors like subtracting numbers from strings
- Runtime errors are failures that the language semantics demand we detect and report while the program is running (hence the name).
