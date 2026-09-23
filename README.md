# Typed Effective Complexity Index

A framework to rate the complexity of a codebase resulting in one absolute score, which can be used in CI/statistics to argue about increased/decreased complexity. It intentionally omits cyclomatic complexity as that can be measure seperately.

## Type complexity

A score between 0 and 100. An ideal value is 10 or lower.

typeComplexity is the sum of distinct type scores for a variable. In case of union types, both sides of the union get added. In case of intersection types, the score for the smallest part of that intersection gets taken. This score is capped at 100.

In PHP (scores generated based on testing in representative codebases):
0: never
5 points for types more specific than scalars: class-strings, standalone false/true, bounded floats/integers
10 point for scalars (int, float, string, bool), NULL, and typed lists, iterables and maps, or concrete classes/enums
30 points for untyped arrays
100 points for mixed

## Method complexity

A score between 0 and 100. An ideal value is 10 or lower.

A method complexity is calculated like the following:

```md
complexityMethod = inputComplexity
    + weightedTransitiveComplexity
    + exceptionComplexity
    + outputComplexity
```

where inputComplexity = the sum of all individual parameters' typeComplexity
where weightedTransitiveComplexity = the sum of all distinctive method/function call complexity, divided by 10. When any transitive dependency relies back onto the method that's currently being analysed, its score is counted as 100 to prevent circular dependencies.
where exceptionComplexity = The number of distinct exceptions that can be thrown and that are not handled. Includes uncaught exceptions from children.
where outputComplexity = the typeComplexity of the return type

## Class/file complexity

A score between 0 and 100. An ideal value is 10 or lower.

A classes complexity is the sum of all the method complexity scores in the class

## Codebase complexity

A score between 0 and 100. An ideal value is 10 or lower.

The codebase complexity is calculated by taking the sum of all the scores of individual files/classes;
