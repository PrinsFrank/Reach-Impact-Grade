# Reach Impact Grade

The Reach Impact Grade (RIG) is a scoring framework to rate the complexity of a codebase resulting in several scores per type, method, class and for an entire codebase, which can be used in CI/statistics to argue about increased/decreased complexity. 

This grade intentionally doesn't look at cyclomatic complexity (which can still be measured separately). Instead, it focuses on the complexity of distinct units of code by the combination of inputs, effects and outputs. The complexity of these three factors determine how hard it is to _use_ a piece of code. The content of the code with identical effects and signature might be written in hundreds of different ways without changing its 'Reach'.

Given that this is a new grading system, there's no 'good' or 'bad' boundaries given in this document. That might change later on.

The scoring is also not scaled, so larger codebases are inherently scored higher. If you want to compare relative complexity, the score can be divided by the Lines of actual code (excluding comments) or cyclomatic complexity scores, but it's recommended to work with the absolute code when monitoring trends and when using this in CI.

## 0. Objective

Keeping track of RIG deltas in codebases using CI can help enforce focussed PRs.

- Bugfixes should have a RIG Delta close to zero
- Refactors should have negative RIG changes, as refactors should always reduce complexity
- New features will always result in a RIG score increase. Enforcing a hard limit on increases results in PRs being forced to be small

## 1. Type complexity

A single score between that is a positive float

For simple types, the score is as follows:
- 0 points for non/never/void
- 1 points for sub-scalar types. like an integer that's only ever positive or NULL
- 2 points for scalar types, like integers, floats, booleans
- 3 points for sub-compound types, like vectors, strings or lists, or arrays and maps with specific 'shapes'
- 4 points for compounds like arrays and maps
- 10 points for explicitly wide parameters (mixed)

If no type can be deduced/reasoned, the score defaults to 20 (will be calibrated).

### 1A. Union type complexity

A union type increases type complexity. It's calculated as the **sum** of all parts of the union:

- `int|bool` has a score of 4 (2 for int + 2 for bool)
- `int|bool|null|float|array|list|map|vector` has a score of 20 (2 for int + 2 for bool + 1 for null + 2 for float + 4 for array + 3 for list + 4 for map + 3 for vector, capped at 20)

note: If the same type is encountered multiple times in one union type, it is only counted once. `int|int` will get the same score as `int`, so just 2.

### 1B. Intersection type complexity

An intersection narrows the type, so we take the smallest part of that intersection:

- `list&array` has a score of 3, because a list gets 3 points and an array gets 4 points, so the score of the list is more 'specific'

> note: this scoring doesn't handle insensible intersections differently. `int&string` is scored as 2 regardless of if that intersection makes sense at all, as this tool only scores complexity and doesn't check validity.

### 1C. Nested type complexity

Nested types are scored recursively, with each level of nesting halving the effective value of that type.

- `list` has a score of 3
- `list<int>` is scored as 4 (3 for the main list, then the first level of nesting is the score for an integer halved, so 1)
- `list<list<int>>` is scored as 5 (3 for the main list, then 1.5 for the first level list, then .5 for the second level int)

### 1D. Multiple type parameters complexity

It might be possible to specify multiple types within a type. In that case, these types are treated as nested types for each level, and as union types within a level.

`map<string, int>` has a score of 6 (4 points for the map, plus 1 point for the string that's halved from 2 points, and 1 point halved from 2 for int)

## 2. Effect complexity

A piece of code can have multiple effects outside of simple returned values. They're scored like this:

- 1 points for each read of a distinct property in the current class
- 2 points for each write to a distinct property in the current class
- 2 points for each read of a distinct property/variable outside the current class
- 2 points for each distinct exception thrown
- 3 points for each write to a distinct property/variable outside the current class
- 4 points for any code that immediately halts, or pauses execution
- 6 points for any new process spawn

## 3. Method complexity

A method complexity is a set of complexity scores:

- Per parameter score "parameterComplexity" indicating the type complexity of that parameter
- One score "effectComplexity" for the local effect complexity
- One score "transitiveEffectComplexity" for the transitive effect complexity of all methods/functions called from within the method. The "effectComplexity" of all called methods/functions is simply summed up. If the method calls itself, that is counted as 0 as its own score is already included in the effectComplexity. Note that this complexity only uses the _local_ effect complexity of called methods, so won't scale with call depth
- One score "returnTypeComplexity" for the return type complexity

To collapse the method complexity into a single score, all its individual scores are added together.

## 4. Class complexity

A class complexity is the sum of all method complexities, plus the type complexity of any property.

## 5. Codebase complexity

The complexity of a codebase is calculated by adding all the individual class complexity scores together.
