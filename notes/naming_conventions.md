# Naming Conventions

## Count vs NumberOf

When naming variables that represent quantities, the recommended convention is to use `Count` suffix rather than `NumberOf` or similar alternatives.

### Recommended Naming: Count

- Clarity: Count is concise and clearly conveys the meaning of a quantity.
- Consistency: Aligns with .NET framework standards (e.g., List.Count, Dictionary.Count).
- Readability: Widely recognized and intuitive for developers.

```cs
// Example
int itemCount; // ✅ Preferred
```

### Less Preferred: NumberOf, NumItems

- Verbosity: Longer and less readable.
- Ambiguity: "Number" can imply different meanings (e.g., numeric value, index).

ref: In "Code Complete", by Steve McConnell

> "But, because using Number so often creates confusion, it's probably best to sidestep the whole issue by using Count to refer to a total number of sales and Index to refer to a specific sale."

source: https://stackoverflow.com/questions/6358588/how-to-name-a-variable-numitems-or-itemcount
