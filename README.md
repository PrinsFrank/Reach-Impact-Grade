# Typed Effective Complexity Index

A framework to rate the complexity of a codebase resulting in several weighted scores from 0 to 100 per type, method, class and for an entire codebase, which can be used in CI/statistics to argue about increased/decreased complexity. It intentionally omits cyclomatic complexity as that can be measured separately.

All individual scores are within the range from 0 to 100. There are several ranges:
- 0-10: good
- 11-30: warning
- 31+: bad

Intermediate scores can be floats, but method/class/total scores are rounded to integers.

## 1. Type complexity

A single score between 0 and 100

For simple types, the score is as follows:
- 5 points for sub-scalar types. like an integer that's only ever positive or NULL
- 10 points for scalar types, like integers, floats, booleans
- 15 points for sub-compound types, like vectors, strings or lists
- 20 points for compounds like arrays and maps

If no type can be deduced/reasoned, the score defaults to the maximum score of a 100.

### 1A. Union type complexity

A union type increases type complexity. It's calculated as the **sum** of all parts of the union:

`int|bool` has a score of 20 (10 for int + 10 for bool)
`int|bool|null|float|array|list|map|vector` has a score of 100 (10 for int + 10 for bool + 5 for null + 10 for float + 20 for array + 15 for list + 20 for map + 15 for vector, capped at 100)

note: If the same type is encountered multiple times in one union type, it is only counted once. `int|int` will get the same score as `int`, so just 10.

### 1B. Intersection type complexity

An intersection narrows the type, so we take the smallest part of that intersection:

`list&array` has a score of 15, because a list gets 15 points and an array gets 20 points, so the score of the list is more 'specific'

note: this scoring doesn't handle insensible intersections differently. `int&string` is scored as 10 regardless of if that intersection makes sense at all, as this tool only scores complexity and doesn't check validity.

### 1C. Nested type complexity

Nested types are scored recursively, with each level of nesting halving the effective value of that type.

`list` has a score of 15
`list<int>` is scored as 20 (15 for the main list, then the first level of nesting is the score for an integer halved, so 5)
`list<list<int>>` is scored as 25 (15 for the main list, then 7.5 for the first level list, then 2.5 for the second level int)

### 1D. Multiple type parameters complexity

It might be possible to specify multiple types within a type. In that case, these types are treated as nested types for each level, and as union types within a level.

`map<string, int>` has a score of 30 (20 points for the map, plus 5 points for the string that's halved from 10 points, and 5 points halved from 10 for int)

## 2. Method complexity

A method complexity is a compound of 4 scores, each also indicated independently:

- Per parameter score "parameterComplexity" (0-100) indicating the type complexity of that parameter
- One score "exceptionComplexity" (0-100) for the type complexity of all the uncaught exceptions, including those thrown by children
- One score "transitiveComplexity" (0-100) for the transitive complexity of this method, calculated as the sum of all the transitive complexity scores of distinctive methods/functions called from within this method **halved**. In case of (int) direct recursion, a score of 10 is used for any ultimately recursing call.
- One score "returnComplexity" (0-100) for the return type complexity

To collapse the method complexity into a single score, the 4 scores are sorted from high to low, and then multiplied like this:
`MIN(100, n0 + n1 / 2 + n2 / 4 + n3 / 8)`

So for complexity scores of 80, 40, 20 and 100, the final score is 100 (the score is capped at 100). For 10, 20, 30, 40, the final score is 61, rounded from 61.25

## 3. Class complexity

// TODO

## 4. Codebase complexity

// TODO
