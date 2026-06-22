# Sorting Algorithms & Big-O Notation - Complete Study Guide

# Table of Contents

1. What is Big O?
2. Why Time Complexity Matters
3. Common Big O Complexities
4. Time Complexity Cheat Sheet
5. Bubble Sort
6. Selection Sort
7. Insertion Sort
8. Merge Sort
9. Quick Sort
10. Heap Sort
11. Counting Sort
12. Radix Sort
13. Bucket Sort
14. Comparison of All Sorting Algorithms
15. Stability in Sorting
16. In-Place Sorting
17. Which Sorting Algorithm is Used in Real Systems?
18. Interview Questions
19. Final Cheat Sheet

---

# 1. What is Big O?

Big O Notation describes:

```text
How an algorithm grows
as input size grows.
```

It measures:

```text
Time Complexity
Space Complexity
```

---

Example:

```java
for(int i=0;i<n;i++){
    System.out.println(i);
}
```

If:

```text
n = 10
```

10 operations.

If:

```text
n = 1000
```

1000 operations.

Complexity:

```text
O(n)
```

---

# Why Not Measure Seconds?

Because:

```text
Different CPUs
Different Memory
Different Machines
```

produce different timings.

Big O measures:

```text
Growth Rate
```

which remains constant across machines.

---

# 2. Why Time Complexity Matters

Suppose:

```text
1 Million Records
```

---

Algorithm A

```text
O(n)
```

Operations:

```text
1,000,000
```

---

Algorithm B

```text
O(n²)
```

Operations:

```text
1,000,000,000,000
```

(1 trillion)

Huge difference.

---

# 3. Common Big O Complexities

---

## O(1)

Constant Time

```java
arr[5];
```

Always one operation.

---

Example:

```text
HashMap Lookup
Array Access
```

---

## O(log n)

Logarithmic

Example:

```text
Binary Search
```

Each step halves search space.

---

Example:

```text
1024 elements

1024
512
256
128
64
32
16
8
4
2
1
```

Only:

```text
10 steps
```

---

## O(n)

Linear

```java
for(int i=0;i<n;i++)
```

Examples:

```text
Linear Search
Array Traversal
```

---

## O(n log n)

Most efficient comparison sorts.

Examples:

```text
Merge Sort
Quick Sort (Average)
Heap Sort
```

---

## O(n²)

Nested loops.

```java
for(...)
   for(...)
```

Examples:

```text
Bubble Sort
Selection Sort
Insertion Sort (Worst)
```

---

## O(2ⁿ)

Exponential

Example:

```text
Recursive Fibonacci
```

Very slow.

---

## O(n!)

Factorial

Example:

```text
Travelling Salesman Brute Force
```

Extremely expensive.

---

# 4. Time Complexity Growth Chart

```text
Best → Worst

O(1)
O(log n)
O(n)
O(n log n)
O(n²)
O(n³)
O(2ⁿ)
O(n!)
```

---

# 5. Bubble Sort

---

## Idea

Repeatedly swap adjacent elements.

---

Example

```text
5 3 8 4
```

Pass 1

```text
3 5 8 4
3 5 4 8
```

Pass 2

```text
3 4 5 8
```

Sorted.

---

## Complexity

| Case | Time |
|--------|--------|
| Best | O(n) |
| Average | O(n²) |
| Worst | O(n²) |

Space:

```text
O(1)
```

---

## Advantages

Simple.

---

## Disadvantages

Very slow.

Rarely used.

---

# 6. Selection Sort

---

## Idea

Find minimum element.

Place it at beginning.

---

Example

```text
64 25 12 22 11
```

Pass 1

```text
11 25 12 22 64
```

Pass 2

```text
11 12 25 22 64
```

---

## Complexity

| Case | Time |
|--------|--------|
| Best | O(n²) |
| Average | O(n²) |
| Worst | O(n²) |

Space:

```text
O(1)
```

---

## Advantage

Minimum swaps.

---

## Disadvantage

Still O(n²).

---

# 7. Insertion Sort

---

## Idea

Build sorted portion gradually.

Like sorting playing cards.

---

Example

```text
5 3 8 4
```

Insert:

```text
3 into [5]
```

Result

```text
3 5
```

Continue.

---

## Complexity

| Case | Time |
|--------|--------|
| Best | O(n) |
| Average | O(n²) |
| Worst | O(n²) |

Space:

```text
O(1)
```

---

## Advantage

Excellent for:

```text
Small Data
Nearly Sorted Data
```

---

# 8. Merge Sort

---

## Divide and Conquer

Split array.

Sort both halves.

Merge.

---

Example

```text
8 3 5 4
```

Split

```text
8 3

5 4
```

Sort

```text
3 8

4 5
```

Merge

```text
3 4 5 8
```

---

## Complexity

| Case | Time |
|--------|--------|
| Best | O(n log n) |
| Average | O(n log n) |
| Worst | O(n log n) |

Space:

```text
O(n)
```

---

## Advantage

Guaranteed performance.

Stable.

---

## Disadvantage

Extra memory.

---

# 9. Quick Sort

Most important interview sorting algorithm.

---

## Idea

Choose Pivot.

Partition.

Recursively sort.

---

Example

```text
8 3 5 4
```

Pivot:

```text
5
```

Partition

```text
3 4 | 5 | 8
```

Recursively sort.

---

## Complexity

| Case | Time |
|--------|--------|
| Best | O(n log n) |
| Average | O(n log n) |
| Worst | O(n²) |

Space:

```text
O(log n)
```

---

## Why Worst O(n²)?

Bad pivot.

Example

```text
1 2 3 4 5
```

always choosing first element.

---

## Advantage

Very fast in practice.

Cache friendly.

---

# 10. Heap Sort

Uses Heap Data Structure.

---

## Steps

Build Max Heap.

Extract maximum repeatedly.

---

Example

```text
50
/ \
30 40
```

Max element always at root.

---

## Complexity

| Case | Time |
|--------|--------|
| Best | O(n log n) |
| Average | O(n log n) |
| Worst | O(n log n) |

Space:

```text
O(1)
```

---

## Advantage

Guaranteed O(n log n).

In-place.

---

## Disadvantage

Less cache friendly than Quick Sort.

---

# 11. Counting Sort

Non-comparison sort.

---

Input

```text
1 4 1 2 7 5 2
```

Count occurrences.

---

Complexity

```text
O(n + k)
```

where:

```text
k = range of numbers
```

---

Example

```text
0-100
```

Excellent.

---

Example

```text
0-1 Billion
```

Bad.

---

# 12. Radix Sort

Sort digit by digit.

---

Example

```text
170
45
75
90
802
```

Sort:

```text
Units Digit
Tens Digit
Hundreds Digit
```

---

Complexity

```text
O(nk)
```

k = digits.

---

Used for:

```text
Integers
IDs
Phone Numbers
```

---

# 13. Bucket Sort

Distribute values into buckets.

Sort each bucket.

Merge.

---

Example

```text
0.12
0.45
0.23
0.99
```

Bucket:

```text
0.0-0.1
0.1-0.2
...
```

---

Complexity

Average:

```text
O(n)
```

Worst:

```text
O(n²)
```

---

# 14. Comparison of All Sorting Algorithms

| Algorithm | Best | Average | Worst | Space |
|------------|------------|------------|------------|------------|
| Bubble | O(n) | O(n²) | O(n²) | O(1) |
| Selection | O(n²) | O(n²) | O(n²) | O(1) |
| Insertion | O(n) | O(n²) | O(n²) | O(1) |
| Merge | O(n log n) | O(n log n) | O(n log n) | O(n) |
| Quick | O(n log n) | O(n log n) | O(n²) | O(log n) |
| Heap | O(n log n) | O(n log n) | O(n log n) | O(1) |
| Counting | O(n+k) | O(n+k) | O(n+k) | O(k) |
| Radix | O(nk) | O(nk) | O(nk) | O(n+k) |
| Bucket | O(n) | O(n+k) | O(n²) | O(n) |

---

# 15. Stable vs Unstable Sorting

Stable means:

```text
Equal elements preserve order.
```

Example

```text
A(100)
B(100)
```

After sorting:

```text
A(100)
B(100)
```

order remains same.

---

## Stable Sorts

```text
Bubble Sort
Insertion Sort
Merge Sort
Counting Sort
Radix Sort
```

---

## Unstable Sorts

```text
Quick Sort
Heap Sort
Selection Sort
```

---

# 16. In-Place Sorting

Uses little extra memory.

---

In-place:

```text
Bubble
Selection
Insertion
Quick
Heap
```

---

Not In-place:

```text
Merge Sort
Counting Sort
Radix Sort
```

---

# 17. Which Sorting Algorithms Are Used In Real Systems?

---

## Java Arrays.sort()

Primitive Arrays

```text
Dual Pivot Quick Sort
```

Complexity:

```text
O(n log n)
```

---

Objects

```text
TimSort
```

Hybrid of:

```text
Merge Sort
Insertion Sort
```

---

## Python sort()

```text
TimSort
```

---

## Database ORDER BY

Usually:

```text
Quick Sort
Merge Sort
External Merge Sort
```

depending on memory.

---

# 18. Common Interview Questions

---

## Why Quick Sort Faster Than Merge Sort?

Answer:

```text
Better cache locality.
Less memory usage.
```

---

## Why Merge Sort Used For Linked List?

Answer:

```text
No random access required.
```

---

## Which Sort Is Stable?

Answer:

```text
Merge Sort
Insertion Sort
Bubble Sort
```

---

## Which Sort Has Guaranteed O(n log n)?

Answer:

```text
Merge Sort
Heap Sort
```

---

## Why Quick Sort Worst Case O(n²)?

Answer:

```text
Bad pivot selection creates
highly unbalanced partitions.
```

---

# 19. Final Interview Cheat Sheet

## Big O Order

```text
O(1)
O(log n)
O(n)
O(n log n)
O(n²)
O(n³)
O(2ⁿ)
O(n!)
```

---

## Best General Purpose Sort

```text
Quick Sort
```

---

## Guaranteed O(n log n)

```text
Merge Sort
Heap Sort
```

---

## Best For Nearly Sorted Data

```text
Insertion Sort
```

---

## Best For Large Distributed Data

```text
Merge Sort
External Merge Sort
```

---

## Stable Sorts

```text
Merge Sort
Insertion Sort
Bubble Sort
```

---

## In-place Sorts

```text
Quick Sort
Heap Sort
Insertion Sort
Selection Sort
Bubble Sort
```

---

# Golden Interview Answer

> Big O notation describes how the running time or memory usage of an algorithm grows as input size increases. Among sorting algorithms, Quick Sort offers O(n log n) average performance and is widely used in practice, Merge Sort provides guaranteed O(n log n) performance with extra memory, and Heap Sort offers O(n log n) performance while remaining in-place.
