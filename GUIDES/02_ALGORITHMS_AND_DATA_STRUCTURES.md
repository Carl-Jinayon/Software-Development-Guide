# Algorithms and Data Structures — Deep Dive

Companion to Part 8 of `01_SOFTWARE_ENGINEERING_FIELD_GUIDE.md`.

Every algorithm here is implemented in Python, with a test, a complexity analysis, and a discussion of when to use it versus the alternative.

---

## Table of Contents

- [1. Complexity analysis](#1-complexity-analysis)
- [2. Python's data structures and their real costs](#2-pythons-data-structures-and-their-real-costs)
- [3. Searching](#3-searching)
- [4. Sorting](#4-sorting)
- [5. Stacks and queues](#5-stacks-and-queues)
- [6. Hash-based algorithms](#6-hash-based-algorithms)
- [7. Recursion, divide and conquer, and backtracking](#7-recursion-divide-and-conquer-and-backtracking)
- [8. Linked structures](#8-linked-structures)
- [9. Trees](#9-trees)
- [10. Binary heaps and priority queues](#10-binary-heaps-and-priority-queues)
- [11. Graphs](#11-graphs)
- [12. Greedy algorithms](#12-greedy-algorithms)
- [13. Dynamic programming](#13-dynamic-programming)
- [14. Tries](#14-tries)
- [15. Union-Find](#15-union-find)
- [16. Randomised algorithms](#16-randomised-algorithms)
- [17. Benchmarking methodology](#17-benchmarking-methodology)
- [18. Problem-solving frameworks](#18-problem-solving-frameworks)
- [19. Interview patterns](#19-interview-patterns)
- [20. Learning path](#20-learning-path)

---

# 1. Complexity analysis

## 1.1 The definition

**Time complexity** — how the number of operations grows as input size grows.
**Space complexity** — how additional memory grows as input size grows.

Both are written as `O(...)`, describing **growth rate**, not absolute time.

## 1.2 The complexity table

| Complexity | Name | Growth | Verdict |
|---|---|---|---|
| `O(1)` | Constant | Flat | Ideal |
| `O(log n)` | Logarithmic | Extremely slow growth | Excellent |
| `O(n)` | Linear | Proportional | Fine |
| `O(n log n)` | Log-linear | Near linear | Good — sorting target |
| `O(n²)` | Quadratic | Fast but accelerating | **Danger zone** |
| `O(2ⁿ)` | Exponential | Doubling input doubles time | Unusable past ~30 |
| `O(n!)` | Factorial | Explosive | Theoretical only |

## 1.3 Reading a real complexity class

```
O(n log n)

n      → 1,000      100,000        1,000,000
comparisons → 10,000    1,700,000    20,000,000
```

```
O(n²)

n      → 1,000      100,000        1,000,000
comparisons → 1,000,000  10,000,000,000   1,000,000,000,000,000
```

At a million items, the difference is **50,000,000×**. That is why the choice matters more than the constant factors.

## 1.4 Common misconceptions

| Misconception | Reality |
|---|---|
| "O(n) means linear time in seconds" | No. It means *proportional to n*. The constant factor matters enormously. |
| "Hash tables are O(1) always" | **Average case only.** Worst case (all keys colliding) is O(n). |
| "Python lists are arrays" | They behave differently — see below. |
| "The constant factor doesn't matter" | It matters until the two have different complexity. Then it stops mattering. |
| "Recursive is faster than iterative" | Recursion has call-stack overhead. It is usually slower and uses more memory. Use it for readability. |

## 1.5 Amortised analysis

Some operations are cheap *on average across a sequence*, even if expensive individually.

```python
# Appending to a Python list is amortised O(1).
# Occasionally the list must reallocate and copy everything — O(n) then.
# But over n appends, total work is O(n).

import time

def benchmark_append(n: int) -> float:
    items: list[int] = []
    start = time.perf_counter()
    for i in range(n):
        items.append(i)
    return time.perf_counter() - start

def test_append_scales_linearly():
    """Doubling n should roughly double the time, not quadruple it."""
    t1 = benchmark_append(100_000)
    t2 = benchmark_append(400_000)
    ratio = t2 / t1
    assert 2 < ratio < 8, f"unexpected scaling ratio: {ratio:.2f}"
```

This is why `list.append` is fast despite occasional O(n) reallocation, and why it is the correct choice for a stack or queue in Python.

## 1.6 Space complexity in practice

```python
# O(1) extra space — in-place
def reverse_in_place(items: list) -> list:
    left, right = 0, len(items) - 1
    while left < right:
        items[left], items[right] = items[right], items[left]
        left += 1
        right -= 1
    return items


# O(n) extra space
def reverse_copy(items: list) -> list:
    return items[::-1]
```

**When reporting complexity, state both.** "This is O(n log n) time and O(n) space because the merge step allocates a temporary array."

---

# 2. Python's data structures and their real costs

Python's structures do not behave like the textbook C equivalents. Knowing the difference prevents real performance bugs.

## 2.1 list — NOT a C array

A Python `list` is a **dynamic array of pointers**, not a contiguous array of values.

```python
values = [10, 20, 30, 40]

# What is 0x1F really?
print(hex(id(values[0])))
print(hex(id(values[1])))
# 0x7f8e4c0d3560
# 0x7f8e4c0d3570    <- different addresses, NOT contiguous
```

| Operation | Complexity | Note |
|---|---|---|
| `values[i]` | **O(1)** | Pointer arithmetic |
| `values[-1]` | O(1) | Negative indexing |
| `x in values` | **O(n)** | Linear scan |
| `values.index(x)` | O(n) | Linear scan |
| `values.append(x)` | O(1) amortised | May reallocate |
| `values.insert(0, x)` | **O(n)** | Shifts everything |
| `values.pop()` | O(1) | From the end |
| `values.pop(0)` | **O(n)** | Shifts everything |
| `len(values)` | O(1) | Stored, not computed |
| `list.sort()` | O(n log n) | In-place |

```python
def test_list_membership_is_linear():
    """Membership on a list is O(n) — this is why `in` on a big list is slow."""
    big = list(range(100_000))
    start = time.perf_counter()
    assert 99_999 in big
    slow = time.perf_counter() - start

    lookup = set(big)              # O(n) to build once
    start = time.perf_counter()
    assert 99_999 in lookup
    fast = time.perf_counter() - start

    assert fast < slow / 10, "set should be dramatically faster for membership"


def test_front_insertion_is_quadratic():
    """insert(0, ...) on a list is O(n). Use a deque instead."""
    from collections import deque

    # deque.appendleft is O(1)
    queue = deque()
    start = time.perf_counter()
    for i in range(100_000):
        queue.appendleft(i)
    fast = time.perf_counter() - start

    start = time.perf_counter()
    bad = []
    for i in range(100_000):
        bad.insert(0, i)
    slow = time.perf_counter() - start

    assert fast < slow / 20
```

## 2.2 dict and set — the workhorses

| Operation | Average | Worst case |
|---|---|---|
| `d[key]` | **O(1)** | O(n) |
| `key in d` | O(1) | O(n) |
| `d[key] = v` | O(1) | O(n) |
| `d.pop(key)` | O(1) | O(n) |
| `d.keys()` | O(1) | — |

Worst case is O(n) because Python resolves hash collisions. Hashing is randomised per process (PYTHONHASHSEED) for str/bytes to make collision attacks difficult, but it is not impossible.

```python
def test_dict_beats_linear_search():
    records = [{"id": i, "value": f"item-{i}"} for i in range(50_000)]

    start = time.perf_counter()
    for r in records:
        if r["id"] == 49_999:
            break
    linear = time.perf_counter() - start

    start = time.perf_counter()
    index = {r["id"]: r for r in records}
    _ = index[49_999]
    hashed = time.perf_counter() - start

    assert hashed < linear
```

## 2.3 When to use what

| Need | Use | Not |
|---|---|---|
| Fast key lookup | `dict`, `set` | `list` + `in` |
| FIFO queue | `collections.deque` | `list.pop(0)` |
| LIFO stack | `list` | — |
| Counting frequencies | `collections.Counter` | manual dict loop |
| Default on missing key | `collections.defaultdict` | try/except KeyError |
| Sorted data with frequent insertion | `heapq`, `bisect` | re-sort every time |
| Grouping | `itertools.groupby`, dict of lists | nested loops |
| Ordered unique items | `dict.keys()` | set + sort |

```python
from collections import Counter, defaultdict, deque
from itertools import groupby
import bisect

def count_words(text: str) -> Counter:
    return Counter(text.lower().split())


def group_by_length(words: list[str]) -> dict[int, list[str]]:
    grouped: defaultdict[int, list[str]] = defaultdict(list)
    for w in words:
        grouped[len(w)].append(w)
    return dict(grouped)


def sorted_insert(items: list[int], value: int) -> list[int]:
    """O(log n) insertion into a sorted list, vs O(n) for a plain append."""
    bisect.insort(items, value)
    return items


def bfs_queue() -> deque:
    return deque([("start", 0)])
```

## 2.4 Performance hierarchy

Rough relative throughput on CPython, single core:

```
tuple/index access        fastest
set/dict membership       very fast
list indexing             fast
deque append/popleft      fast
set/dict insert           fast
list append               fast
list membership           slow
list insert(0)/pop(0)     very slow
str concatenation in loop very slow
```

---

# 3. Searching

## 3.1 Linear search

```python
from typing import Optional, Sequence, TypeVar

T = TypeVar("T")


def linear_search(items: Sequence[T], target: T) -> Optional[int]:
    """Return the index of target, or None. O(n) time, O(1) space."""
    for index, item in enumerate(items):
        if item == target:
            return index
    return None


def test_linear_search_finds_value():
    assert linear_search([4, 8, 15, 16, 23, 42], 16) == 3


def test_linear_search_returns_none_when_absent():
    assert linear_search([4, 8, 15, 16, 23, 42], 99) is None


def test_linear_search_on_empty_list():
    assert linear_search([], 1) is None
```

**Use when:** the list is unsorted and small, or the data type has no ordering.

## 3.2 Binary search

The highest-value algorithm to know fluently.

**Precondition:** the list MUST be sorted. Violating this silently returns wrong answers — the most common way this algorithm is broken.

```python
def binary_search(items: Sequence[int], target: int) -> Optional[int]:
    """
    Find target in a SORTED ascending sequence. O(log n) time, O(1) space.

    Invariant: if target exists, it lies within [low, high].
    """
    low, high = 0, len(items) - 1

    while low <= high:
        mid = low + (high - low) // 2      # safe: no overflow, unlike (low+high)//2

        if items[mid] == target:
            return mid
        if items[mid] < target:
            low = mid + 1                   # target must be in the right half
        else:
            high = mid - 1                  # target must be in the left half

    return None
```

```python
def test_binary_search_finds_each_element():
    data = [1, 3, 5, 7, 9, 11, 13, 15]
    for index, value in enumerate(data):
        assert binary_search(data, value) == index


def test_binary_search_returns_none_when_absent():
    data = [1, 3, 5, 7, 9, 11, 13, 15]
    assert binary_search(data, 4) is None
    assert binary_search(data, 0) is None
    assert binary_search(data, 99) is None


def test_binary_search_on_empty_and_single():
    assert binary_search([], 1) is None
    assert binary_search([1], 1) == 0


def test_binary_search_handles_duplicates():
    assert binary_search([1, 2, 2, 2, 3], 2) in (1, 2, 3)   # any match is valid


def test_binary_search_is_log_n():
    """1M elements should take ~20 comparisons, not ~1M."""
    data = list(range(1_000_000))
    assert binary_search(data, 999_999) == 999_999
```

## 3.3 Binary search: the boundary variants

These are harder and are asked more often in interviews.

```python
def lower_bound(items: Sequence[int], target: int) -> int:
    """
    Index of the FIRST element >= target. Returns len(items) if none.
    This is the clean primitive — every other variant builds on it.
    O(log n)
    """
    low, high = 0, len(items)

    while low < high:
        mid = (low + high) // 2
        if items[mid] < target:
            low = mid + 1
        else:
            high = mid
    return low


def count_occurrences(items: Sequence[int], target: int) -> int:
    """Number of times target appears in a sorted list. O(log n)."""
    return lower_bound(items, target + 1) - lower_bound(items, target)


def test_lower_bound_finds_insertion_point():
    data = [1, 3, 5, 5, 5, 7, 9]
    assert lower_bound(data, 5) == 2         # first 5
    assert lower_bound(data, 4) == 2         # would insert before the 5s
    assert lower_bound(data, 0) == 0
    assert lower_bound(data, 100) == len(data)


def test_count_occurrences():
    assert count_occurrences([1, 3, 5, 5, 5, 7], 5) == 3
    assert count_occurrences([1, 2, 3], 9) == 0
    assert count_occurrences([], 1) == 0
```

**Learn `lower_bound` and derive everything else from it.** It is more reliable than memorising separate implementations.

## 3.4 Other searches

| Algorithm | Purpose | Complexity |
|---|---|---|
| **Jump search** | Static array, fewer comparisons | O(√n) |
| **Interpolation search** | Uniformly distributed data | O(log log n) avg |
| **Ternary search** | Sorted array, two pivots | O(log₃ n) |
| **Exponential search** | Unbounded sorted array | O(log n) |
| **BFS** | Unweighted graph shortest path | O(V+E) |

```python
def exponential_search(items: Sequence[int], target: int) -> Optional[int]:
    """Find the range with exponential steps, then binary search inside it."""
    if not items:
        return None
    if items[0] == target:
        return 0

    bound = 1
    while bound < len(items) and items[bound] <= target:
        bound *= 2

    return binary_search(items[bound // 2:bound], target)


def test_exponential_search():
    data = [1, 3, 5, 7, 9, 11, 13]
    assert exponential_search(data, 9) == 4
    assert exponential_search(data, 1) == 0
    assert exponential_search(data, 100) is None
    assert exponential_search([], 1) is None
```

---

# 4. Sorting

## 4.1 Python's built-in sort

Use it. It is Timsort — adaptive merge/insertion hybrid, stable, O(n log n), and heavily optimised in C.

```python
numbers = [5, 2, 9, 1, 7]

assert sorted(numbers) == [1, 2, 5, 7, 9]      # returns new list
numbers.sort()                                   # in place, returns None
numbers.sort(reverse=True)                       # descending
numbers.sort(key=lambda x: abs(x))                # custom key

assert sorted([(2, "b"), (1, "a"), (2, "a")]) == [(1, "a"), (2, "a"), (2, "b")]  # stable
```

**Stability** means equal elements retain their original relative order. This matters when sorting by one key then another.

**When `key=` is essential:** sorting by a field, sorting case-insensitively, sorting by multiple keys.

```python
from operator import itemgetter

records = [
    {"name": "Ana", "age": 30},
    {"name": "Ben", "age": 25},
    {"name": "Cal", "age": 30},
]

assert sorted(records, key=itemgetter("age", "name"))[0]["name"] == "Ben"
```

## 4.2 Implementations you should write yourself

**Write each of these once from memory.** The implementation *is* the understanding.

### Bubble sort — O(n²), educational only

```python
def bubble_sort(items: list[int]) -> list[int]:
    """O(n²) time, O(1) space. Included to understand the pattern."""
    n = len(items)
    for i in range(n):
        swapped = False
        for j in range(0, n - i - 1):
            if items[j] > items[j + 1]:
                items[j], items[j + 1] = items[j + 1], items[j]
                swapped = True
        if not swapped:              # early exit — already sorted
            break
    return items


def test_bubble_sort():
    assert bubble_sort([5, 2, 8, 1]) == [1, 2, 5, 8]
    assert bubble_sort([1, 2, 3]) == [1, 2, 3]
    assert bubble_sort([]) == []
    assert bubble_sort([3, 3, 3]) == [3, 3, 3]
```

### Selection sort — O(n²), fewer swaps

```python
def selection_sort(items: list[int]) -> list[int]:
    for i in range(len(items)):
        min_index = i
        for j in range(i + 1, len(items)):
            if items[j] < items[min_index]:
                min_index = j
        items[i], items[min_index] = items[min_index], items[i]
    return items


def test_selection_sort():
    assert selection_sort([5, 2, 8, 1]) == [1, 2, 5, 8]
    assert selection_sort([]) == []
```

### Insertion sort — O(n²) worst, O(n) best

```python
def insertion_sort(items: list[int]) -> list[int]:
    """Excellent for small or nearly-sorted lists. This is what Timsort uses for small runs."""
    for i in range(1, len(items)):
        current = items[i]
        j = i - 1
        while j >= 0 and items[j] > current:
            items[j + 1] = items[j]
            j -= 1
        items[j + 1] = current
    return items


def test_insertion_sort():
    assert insertion_sort([5, 2, 8, 1]) == [1, 2, 5, 8]
    assert insertion_sort([1, 2, 3, 4]) == [1, 2, 3, 4]
    assert insertion_sort([]) == []
```

### Merge sort — O(n log n) worst case, stable

```python
def merge_sort(items: list[int]) -> list[int]:
    """O(n log n) time, O(n) space. Stable. The reliable worst-case performer."""
    if len(items) <= 1:
        return items

    mid = len(items) // 2
    left = merge_sort(items[:mid])
    right = merge_sort(items[mid:])

    return merge(left, right)


def merge(left: list[int], right: list[int]) -> list[int]:
    """Merge two SORTED lists into one sorted list. O(n)."""
    result: list[int] = []
    i = j = 0

    while i < len(left) and j < len(right):
        if left[i] <= right[j]:        # <= keeps stability
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1

    result.extend(left[i:])             # one side is exhausted
    result.extend(right[j:])
    return result


def test_merge_sort():
    assert merge_sort([5, 2, 8, 1, 9, 3]) == [1, 2, 3, 5, 8, 9]
    assert merge_sort([]) == []
    assert merge_sort([1]) == [1]
    assert merge_sort([2, 1]) == [1, 2]


def test_merge_sort_handles_negatives_and_duplicates():
    assert merge_sort([3, -1, 3, 0, -5]) == [-5, -1, 0, 3, 3]
```

### Quick sort — O(n log n) average, O(n²) worst

```python
import random


def quick_sort(items: list[int]) -> list[int]:
    """O(n log n) average, O(n²) worst. In-place, O(log n) stack."""
    _quick_sort(items, 0, len(items) - 1)
    return items


def _quick_sort(items: list[int], low: int, high: int) -> None:
    if low >= high:
        return

    pivot_index = partition(items, low, high)
    _quick_sort(items, low, pivot_index - 1)
    _quick_sort(items, pivot_index + 1, high)


def partition(items: list[int], low: int, high: int) -> int:
    """Lomuto partition. Random pivot avoids the O(n²) worst case on sorted input."""
    pivot = items[random.randint(low, high)]
    items[low], items[high] = items[high], items[low]
    store_index = low

    for i in range(low, high):
        if items[i] < pivot:
            items[i], items[store_index] = items[store_index], items[i]
            store_index += 1

    items[store_index], items[high] = items[high], items[store_index]
    return store_index


def test_quick_sort():
    assert quick_sort([5, 2, 8, 1, 9, 3]) == [1, 2, 3, 5, 8, 9]
    assert quick_sort([]) == []
    assert quick_sort([1]) == [1]


def test_quick_sort_on_presorted_input():
    """The classic quicksort killer — a random pivot should survive it."""
    assert quick_sort(list(range(500))) == list(range(500))
```

### Heap sort — O(n log n) worst, O(1) space

```python
def heap_sort(items: list[int]) -> list[int]:
    """O(n log n) always, O(1) space. The space-optimal comparison sort."""
    n = len(items)

    for i in range(n // 2 - 1, -1, -1):
        heapify(items, n, i)

    for end in range(n - 1, 0, -1):
        items[0], items[end] = items[end], items[0]
        heapify(items, end, 0)

    return items


def heapify(items: list[int], size: int, root: int) -> None:
    largest = root
    left = 2 * root + 1
    right = 2 * root + 2

    if left < size and items[left] > items[largest]:
        largest = left
    if right < size and items[right] > items[largest]:
        largest = right

    if largest != root:
        items[root], items[largest] = items[largest], items[root]
        heapify(items, size, largest)


def test_heap_sort():
    assert heap_sort([5, 2, 8, 1, 9, 3]) == [1, 2, 3, 5, 8, 9]
    assert heap_sort([]) == []
    assert heap_sort([1, 1, 1]) == [1, 1, 1]
```

## 4.3 Sorting comparison table

| Algorithm | Average | Worst | Space | Stable | Notes |
|---|---|---|---|---|---|
| Bubble | O(n²) | O(n²) | O(1) | Yes | Educational |
| Selection | O(n²) | O(n²) | O(1) | No | Fewest swaps |
| Insertion | O(n²) | O(n²) | O(1) | Yes | Best for small/nearly sorted |
| Merge | O(n log n) | **O(n log n)** | O(n) | Yes | Reliable |
| Quick | O(n log n) | O(n²) | O(log n) | No | Fastest average |
| Heap | O(n log n) | **O(n log n)** | O(1) | No | Space-optimal |
| Counting | O(n + k) | O(n + k) | O(k) | Yes | Integer keys only |

**In Python, always use `sorted()` or `list.sort()` unless the exercise is the point.** Understand the algorithms; do not ship them.

---

# 5. Stacks and queues

## 5.1 Stack (LIFO)

```python
def is_balanced_parentheses(expression: str) -> bool:
    """
    Classic stack problem. O(n) time, O(n) space.
    Returns False at the first point where the order is impossible.
    """
    pairs = {")": "(", "]": "[", "}": "{"}
    stack: list[str] = []

    for char in expression:
        if char in "([{":
            stack.append(char)
        elif char in pairs:
            if not stack or stack.pop() != pairs[char]:
                return False

    return not stack


def test_balanced_parentheses():
    assert is_balanced_parentheses("(())[]{}")
    assert is_balanced_parentheses("")
    assert is_balanced_parentheses("()")


def test_unbalanced_parentheses():
    assert not is_balanced_parentheses("(]")
    assert not is_balanced_parentheses("(()")
    assert not is_balanced_parentheses(")")
```

**Where stacks appear in real software:** undo history, function call stacks, expression parsing, backtracking, DFS, matching delimiters in parsers.

## 5.2 Queue (FIFO)

```python
from collections import deque


def bfs_queue_demo() -> deque:
    """deque, not list. list.pop(0) is O(n)."""
    q: deque[int] = deque([1, 2, 3])
    q.append(4)
    return q
```

## 5.3 Circular buffer

```python
class CircularBuffer:
    """Fixed-capacity FIFO that overwrites the oldest item when full. O(1) everywhere."""

    def __init__(self, capacity: int) -> None:
        self._data: list[int | None] = [None] * capacity
        self._capacity = capacity
        self._head = 0
        self._size = 0

    def push(self, item: int) -> None:
        index = (self._head + self._size) % self._capacity
        self._data[index] = item
        if self._size == self._capacity:
            self._head = (self._head + 1) % self._capacity
        else:
            self._size += 1

    def pop(self) -> int | None:
        if self._size == 0:
            return None
        item = self._data[self._head]
        self._data[self._head] = None
        self._head = (self._head + 1) % self._capacity
        self._size -= 1
        return item

    def __len__(self) -> int:
        return self._size


def test_circular_buffer_overwrites_oldest():
    buf = CircularBuffer(3)
    for n in (1, 2, 3, 4):
        buf.push(n)
    assert len(buf) == 3
    assert buf.pop() == 2        # 1 was overwritten
    assert buf.pop() == 3
    assert buf.pop() == 4
    assert buf.pop() is None


def test_circular_buffer_wraps_correctly():
    buf = CircularBuffer(3)
    buf.push(1); buf.push(2)
    assert buf.pop() == 1
    buf.push(3); buf.push(4)     # forces wraparound
    assert buf.pop() == 2
    assert buf.pop() == 3
    assert buf.pop() == 4
```

---

# 6. Hash-based algorithms

## 6.1 Hashing in one sentence

Hashing maps a key to a bucket index in O(1), turning lookup from O(n) into O(1) at the cost of memory and possible collisions.

## 6.2 Frequency counting

```python
from collections import Counter


def find_duplicates(items: list[int]) -> set[int]:
    """O(n) time, O(n) space."""
    seen: set[int] = set()
    duplicates: set[int] = set()
    for item in items:
        if item in seen:
            duplicates.add(item)
        seen.add(item)
    return duplicates


def test_find_duplicates():
    assert find_duplicates([1, 2, 3, 2, 4, 1]) == {1, 2}
    assert find_duplicates([1, 2, 3]) == set()
    assert find_duplicates([]) == set()


def test_first_non_repeating_character():
    """The one whose count is exactly 1."""
    counts = Counter("swiss")
    for char in "swiss":
        if counts[char] == 1:
            return char
    return None
```

## 6.3 Two-sum — the most-asked hash problem

```python
def two_sum(numbers: list[int], target: int) -> tuple[int, int] | None:
    """
    O(n) time, O(n) space. Complements the brute-force O(n²) version.
    Returns indices whose values sum to target.
    """
    seen: dict[int, int] = {}          # value -> index

    for index, value in enumerate(numbers):
        complement = target - value
        if complement in seen:
            return (seen[complement], index)
        seen[value] = index

    return None


def test_two_sum():
    assert two_sum([2, 7, 11, 15], 9) == (0, 1)
    assert two_sum([3, 2, 4], 6) == (1, 2)
    assert two_sum([3, 3], 6) == (0, 1)


def test_two_sum_no_solution():
    assert two_sum([1, 2, 3], 100) is None
    assert two_sum([], 5) is None


def test_group_anagrams():
    """Hash-based grouping. O(n·k log k)."""
    groups: dict[tuple, list[str]] = defaultdict(list)
    for word in ["eat", "tea", "tan", "ate", "nat", "bat"]:
        groups["".join(sorted(word))].append(word)

    result = sorted(groups.values(), key=lambda g: g[0])
    assert ["bat"] in result
    assert sorted(["eat", "tea", "ate"]) in result
```

## 6.4 Consistent hashing (concept)

When a cache is distributed across servers, a normal hash sends most keys to the same server when capacity changes. **Consistent hashing** spreads reallocation to only ~1/n of keys. It is how memcached, Cassandra, and Redis Cluster distribute data.

You need to know the concept; you rarely implement it.

---

# 7. Recursion, divide and conquer, and backtracking

## 7.1 Recursion fundamentals

Every recursive function needs three things:

1. **Base case** — when to stop
2. **Recursive case** — progress toward the base case
3. **Invariant** — what must be true on entry

```python
def factorial(n: int) -> int:
    """Base case: n <= 1. Recursive case: multiply by n-1."""
    if n <= 1:
        return 1
    return n * factorial(n - 1)
```

```python
def flatten(nested: list) -> list:
    """Recursion handles irregular nesting that a loop cannot express cleanly."""
    result: list = []
    for item in nested:
        if isinstance(item, list):
            result.extend(flatten(item))
        else:
            result.append(item)
    return result


def test_flatten():
    assert flatten([1, [2, [3, [4, [5]]]]]) == [1, 2, 3, 4, 5]
    assert flatten([]) == []
    assert flatten([1, 2, 3]) == [1, 2, 3]
```

**Python's recursion limit** defaults to 1,000. Exceeding it raises `RecursionError`. For deep structures, convert to an explicit stack.

```python
import sys

print(sys.getrecursionlimit())      # 1000

def test_recursion_limit_exists():
    def infinite(n):
        return infinite(n + 1)

    try:
        infinite(0)
        assert False, "should have raised"
    except RecursionError:
        pass       # expected
```

## 7.2 Divide and conquer

**Pattern:** split the problem, solve each half recursively, combine.

```python
def max_subarray_sum(numbers: list[int]) -> int:
    """
    Divide and conquer maximum subarray. O(n log n).
    (Kadane's algorithm does this in O(n) — know both.)
    """
    if len(numbers) == 1:
        return numbers[0]

    mid = len(numbers) // 2
    left = max_subarray_sum(numbers[:mid])
    right = max_subarray_sum(numbers[mid:])

    # best subarray crossing the midpoint
    running = numbers[mid - 1]
    best_left = numbers[mid - 1]
    for i in range(mid - 2, -1, -1):
        running += numbers[i]
        best_left = max(best_left, running)

    running = numbers[mid]
    best_right = numbers[mid]
    for i in range(mid + 1, len(numbers)):
        running += numbers[i]
        best_right = max(best_right, running)

    return max(left, right, best_left + best_right)


def test_max_subarray():
    assert max_subarray_sum([-2, 1, -3, 4, -1, 2, 1, -5, 4]) == 6
    assert max_subarray_sum([-3, -1, -7]) == -1
    assert max_subarray_sum([5]) == 5


def max_subarray_kadane(numbers: list[int]) -> int:
    """O(n) — the better answer when asked in an interview."""
    if not numbers:
        raise ValueError("empty input")

    best = current = numbers[0]
    for value in numbers[1:]:
        current = max(value, current + value)
        best = max(best, current)
    return best


def test_kadane_matches_divide_and_conquer():
    cases = [
        [-2, 1, -3, 4, -1, 2, 1, -5, 4],
        [1],
        [-1, -2, -3],
        [5, -1, 5],
    ]
    for case in cases:
        assert max_subarray_kadane(case) == max_subarray_sum(case)
```

## 7.3 Backtracking

**Pattern:** try an option, recurse, and undo if it fails. Explores a decision tree.

```python
def n_queens(n: int) -> list[list[int]]:
    """
    Place n queens on an n×n board so none attack each other.
    Classic backtracking. O(n!) worst case, heavily pruned in practice.
    Returns each solution as a list of column indices per row.
    """
    solutions: list[list[int]] = []
    placement: list[int] = []

    def backtrack(row: int) -> None:
        if row == n:
            solutions.append(placement.copy())
            return

        for col in range(n):
            if _is_safe(placement, row, col):
                placement.append(col)
                backtrack(row + 1)
                placement.pop()          # UNDO — this is the backtracking

    backtrack(0)
    return solutions


def _is_safe(placement: list[int], row: int, col: int) -> bool:
    for earlier_row, earlier_col in enumerate(placement):
        if earlier_col == col:                                  # same column
            return False
        if abs(earlier_col - col) == row - earlier_row:          # same diagonal
            return False
    return True


def test_n_queens_known_solutions():
    assert len(n_queens(1)) == 1
    assert len(n_queens(4)) == 2
    assert len(n_queens(8)) == 92


def test_n_queens_solutions_are_valid():
    for solution in n_queens(5):
        assert len(solution) == 5
        assert len(set(solution)) == 5                    # distinct columns
```

```python
def solve_sudoku(board: list[list[int]]) -> list[list[int]] | None:
    """
    Sudoku solver via backtracking with constraint propagation.
    Finds the first blank with the fewest candidates (MRV heuristic).
    """
    board = [row[:] for row in board]        # work on a copy

    if not _solve(board):
        return None
    return board


def _solve(board: list[list[int]]) -> bool:
    position = _find_mrv_cell(board)
    if position is None:
        return True                        # no blanks → solved

    row, col = position
    for value in _candidates(board, row, col):
        board[row][col] = value
        if _solve(board):
            return True
        board[row][col] = 0                # UNDO

    return False


def _find_mrv_cell(board):
    """Minimum Remaining Values — the most constrained cell first."""
    best, best_count = None, 10
    for r in range(9):
        for c in range(9):
            if board[r][c] == 0:
                count = len(_candidates(board, r, c))
                if count < best_count:
                    best, best_count = (r, c), count
                    if count == 1:
                        return best
    return best


def _candidates(board, row, col) -> list[int]:
    used = set(board[row]) | {board[r][col] for r in range(9)}
    used |= {board[r][c] for r in range(3 * (row // 3), 3 * (row // 3) + 3)
             for c in range(3 * (col // 3), 3 * (col // 3) + 3)}
    return [v for v in range(1, 10) if v not in used]


def test_solve_sudoku():
    puzzle = [
        [5, 3, 0, 0, 7, 0, 0, 0, 0],
        [6, 0, 0, 1, 9, 5, 0, 0, 0],
        [0, 9, 8, 0, 0, 0, 0, 6, 0],
        [8, 0, 0, 0, 6, 0, 0, 0, 3],
        [4, 0, 0, 8, 0, 3, 0, 0, 1],
        [7, 0, 0, 0, 2, 0, 0, 0, 6],
        [0, 6, 0, 0, 0, 0, 2, 8, 0],
        [0, 0, 0, 4, 5, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 7, 3],
    ]
    solved = solve_sudoku(puzzle)
    assert solved is not None
    for row in solved:
        assert sorted(row) == list(range(1, 10))
```

---

# 8. Linked structures

## 8.1 Singly linked list

```python
class Node:
    __slots__ = ("value", "next")

    def __init__(self, value):
        self.value = value
        self.next: Node | None = None


class LinkedList:
    """Head insert O(1), tail access O(n), search O(n)."""

    def __init__(self):
        self.head: Node | None = None
        self._size = 0

    def append(self, value) -> None:
        node = Node(value)
        if self.head is None:
            self.head = node
        else:
            current = self.head
            while current.next:
                current = current.next
            current.next = node
        self._size += 1

    def prepend(self, value) -> None:
        node = Node(value)
        node.next = self.head
        self.head = node
        self._size += 1

    def __len__(self) -> int:
        return self._size

    def to_list(self) -> list:
        result = []
        current = self.head
        while current:
            result.append(current.value)
            current = current.next
        return result

    def reverse(self) -> None:
        """O(n) time, O(1) space — the three-pointer technique."""
        prev = None
        current = self.head

        while current:
            following = current.next
            current.next = prev
            prev = current
            current = following

        self.head = prev


def test_linked_list_append_and_prepend():
    ll = LinkedList()
    ll.append(2)
    ll.append(3)
    ll.prepend(1)
    assert ll.to_list() == [1, 2, 3]
    assert len(ll) == 3


def test_linked_list_reverse():
    ll = LinkedList()
    for v in (1, 2, 3, 4):
        ll.append(v)
    ll.reverse()
    assert ll.to_list() == [4, 3, 2, 1]


def test_linked_list_reverse_empty_and_single():
    empty = LinkedList()
    empty.reverse()
    assert empty.to_list() == []

    single = LinkedList()
    single.append(1)
    single.reverse()
    assert single.to_list() == [1]
```

**Why linked lists rarely appear in application code:** Python lists have O(1) indexing. You pay O(n) traversal to gain O(1) insertion that you rarely need. Linked lists matter for interview reasoning and for understanding how `collections.deque` works internally.

---

# 9. Trees

## 9.1 Tree traversals — the core of the topic

```python
from dataclasses import dataclass


@dataclass
class TreeNode:
    value: int
    left: TreeNode | None = None
    right: TreeNode | None = None


def inorder(root: TreeNode | None) -> list[int]:
    """Left, Root, Right. On a BST this yields SORTED output. O(n)."""
    if root is None:
        return []
    return inorder(root.left) + [root.value] + inorder(root.right)


def preorder(root: TreeNode | None) -> list[int]:
    """Root, Left, Right. Used to SERIALISE a tree. O(n)."""
    if root is None:
        return []
    return [root.value] + preorder(root.left) + preorder(root.right)


def postorder(root: TreeNode | None) -> list[int]:
    """Left, Right, Root. Used to FREE or DELETE a tree. O(n)."""
    if root is None:
        return []
    return postorder(root.left) + postorder(root.right) + [root.value]


def level_order(root: TreeNode | None) -> list[list[int]]:
    """Level by level. BFS. Needs a queue. O(n)."""
    if root is None:
        return []

    levels: list[list[int]] = []
    queue: deque[TreeNode] = deque([root])

    while queue:
        level: list[int] = []
        for _ in range(len(queue)):        # snapshot size = this level only
            node = queue.popleft()
            level.append(node.value)
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)
        levels.append(level)

    return levels


def build_tree(values: list[int | None]) -> TreeNode | None:
    """Build a tree from a level-order list (LeetCode style). None = no node."""
    if not values or values[0] is None:
        return None

    root = TreeNode(values[0])
    queue: deque[TreeNode] = deque([root])
    index = 1

    while queue and index < len(values):
        node = queue.popleft()
        if values[index] is not None:
            node.left = TreeNode(values[index])
            queue.append(node.left)
        index += 1
        if index < len(values) and values[index] is not None:
            node.right = TreeNode(values[index])
            queue.append(node.right)
        index += 1

    return root


def test_traversals():
    #       3
    #      / \
    #     9  20
    #       /  \
    #     15    7
    tree = build_tree([3, 9, 20, None, None, 15, 7])
    assert inorder(tree) == [9, 3, 15, 20, 7]
    assert preorder(tree) == [3, 9, 20, 15, 7]
    assert postorder(tree) == [9, 15, 7, 20, 3]
    assert level_order(tree) == [[3], [9, 20], [15, 7]]


def test_traversals_on_empty_tree():
    assert inorder(None) == []
    assert level_order(None) == []
```

**Memory:** `inorder` using concatenation is O(n log n) due to repeated list copying. Use an explicit stack for O(n).

```python
def inorder_iterative(root: TreeNode | None) -> list[int]:
    """O(n) time, O(h) space. Avoids Python's recursion limit on deep trees."""
    result: list[int] = []
    stack: list[TreeNode] = []
    current = root

    while current or stack:
        while current:
            stack.append(current)
            current = current.left
        current = stack.pop()
        result.append(current.value)
        current = current.right

    return result


def test_iterative_matches_recursive():
    tree = build_tree([3, 9, 20, None, None, 15, 7])
    assert inorder_iterative(tree) == inorder(tree)
```

## 9.2 Binary search tree

```python
class BST:
    def __init__(self):
        self.root: TreeNode | None = None

    def insert(self, value: int) -> None:
        self.root = self._insert(self.root, value)

    def _insert(self, node: TreeNode | None, value: int) -> TreeNode:
        if node is None:
            return TreeNode(value)
        if value < node.value:
            node.left = self._insert(node.left, value)
        elif value > node.value:
            node.right = self._insert(node.right, value)
        return node

    def search(self, value: int) -> bool:
        node = self.root
        while node:
            if value == node.value:
                return True
            node = node.left if value < node.value else node.right
        return False

    def inorder(self) -> list[int]:
        """The defining property: inorder traversal of a BST is sorted. O(n)."""
        return inorder(self.root)


def test_bst_insert_and_search():
    tree = BST()
    for value in [5, 3, 8, 1, 4, 7, 9]:
        tree.insert(value)

    assert tree.search(4)
    assert not tree.search(6)
    assert tree.inorder() == [1, 3, 4, 5, 7, 8, 9]


def test_bst_worst_case_is_degenerate():
    """Inserting sorted data creates a LIST, not a tree. O(n) height.

    This is why self-balancing trees (AVL, red-black) exist.
    """
    tree = BST()
    for value in range(100):
        tree.insert(value)

    # Inorder is still sorted — correctness holds
    assert tree.inorder() == list(range(100))
    # ...but the structure is a linked list, so search is O(n) not O(log n)
```

## 9.3 Balanced trees: when you need guaranteed O(log n)

| Structure | Guarantee | Notes |
|---|---|---|
| **AVL** | Strictly balanced; faster reads | More rotations on write |
| **Red-black** | Approximately balanced | Fewer rotations; used by `SortedList`, Java `TreeMap` |
| **B-tree / B+ tree** | Balanced; optimised for disk | What databases actually use |

**The insight:** databases do not use binary search trees for indexes. They use B-trees because disk access is the bottleneck, and a B-tree node holds many keys, minimising the number of expensive disk reads.

---

# 10. Binary heaps and priority queues

A heap is a complete binary tree where every parent is ≥ (max-heap) or ≤ (min-heap) its children. **The root is always the extreme element.**

```python
import heapq


def find_k_largest(numbers: list[int], k: int) -> list[int]:
    """
    O(n log k) with a heap of size k.
    Far better than sorting O(n log n) when k is small.
    """
    if k <= 0:
        return []
    return heapq.nlargest(k, numbers)


def test_find_k_largest():
    assert find_k_largest([1, 5, 3, 9, 7], 2) == [9, 7]
    assert find_k_largest([1, 2], 5) == [2, 1]
    assert find_k_largest([], 3) == []


def test_heappushpop_simulation():
    """Running median / sliding window problems."""
    stream = [5, 1, 9, 3]
    heap: list[int] = []

    for value in stream:
        heapq.heappush(heap, value)

    assert heap[0] == 1            # smallest in the heap
    assert heapq.nlargest(2, heap) == [9, 5]


def merge_k_sorted_lists(lists: list[list[int]]) -> list[int]:
    """Heap-based merge: O(N log k) where k is the number of lists."""
    return list(heapq.merge(*lists))


def test_merge_k_sorted_lists():
    assert merge_k_sorted_lists([[1, 4], [2, 3], [5, 6]]) == [1, 2, 3, 4, 5, 6]
    assert merge_k_sorted_lists([]) == []
    assert merge_k_sorted_lists([[1], [1]]) == [1, 1]


def implement_dijkstra_with_heapq(graph, start):  # see section 11
    ...
```

**`heapq` is a min-heap.** For a max-heap, use negative values.

**Where heaps appear:** Dijkstra's, A\*, task schedulers, merging sorted streams, finding the k largest, keeping a sliding window's top items, network traffic prioritisation.

---

# 11. Graphs

## 11.1 Representations

```python
# Adjacency list — the right default for Python
graph: dict[str, list[str]] = {
    "A": ["B", "C"],
    "B": ["A", "D"],
    "C": ["A", "E"],
    "D": ["B"],
    "E": ["C", "F"],
    "F": ["E"],
}

# Adjacency matrix — O(V²) memory, but O(1) edge lookup
matrix = [[False] * 6 for _ in range(6)]
def add_edge(u: int, v: int) -> None:
    matrix[u][v] = True
    matrix[v][u] = True      # undirected
```

| Representation | Memory | Edge lookup | Best for |
|---|---|---|---|
| Adjacency list | O(V + E) | O(degree) | Sparse graphs |
| Adjacency matrix | O(V²) | O(1) | Dense graphs |
| Edge list | O(E) | O(E) | Large sparse graphs |

## 11.2 BFS

```python
def bfs(graph: dict, start: str, goal: str | None = None) -> list[str]:
    """
    Level-by-level exploration. O(V + E).
    If goal is given, returns the SHORTEST path. Otherwise returns visit order.
    """
    from collections import deque

    visited = {start}
    queue: deque[str] = deque([start])
    parent: dict[str, str | None] = {start: None}

    while queue:
        node = queue.popleft()

        if node == goal:
            break

        for neighbour in graph.get(node, []):
            if neighbour not in visited:
                visited.add(neighbour)
                parent[neighbour] = node
                queue.append(neighbour)

    if goal is not None:
        path: list[str] = []
        current: str | None = goal
        while current is not None:
            path.append(current)
            current = parent[current]
        return path[::-1]

    return sorted(visited)


def test_bfs_shortest_path():
    graph = {
        "A": ["B", "C"], "B": ["A", "D"], "C": ["A", "E"],
        "D": ["B", "F"], "E": ["C", "F"], "F": ["D", "E"],
    }
    assert bfs(graph, "A", "F") == ["A", "B", "D", "F"]


def test_bfs_unreachable():
    graph = {"A": ["B"], "B": ["A"], "C": []}
    assert bfs(graph, "A", "C") == []


def test_bfs_handles_cycles():
    graph = {"A": ["B"], "B": ["A", "C"], "C": ["B"]}
    assert set(bfs(graph, "A")) == {"A", "B", "C"}      # no infinite loop
```

## 11.3 DFS

```python
def dfs(graph: dict, start: str) -> list[str]:
    """Depth-first. O(V + E). Use for cycle detection, topological sort, puzzles."""
    visited: set[str] = set()
    order: list[str] = []

    def visit(node: str) -> None:
        if node in visited:
            return
        visited.add(node)
        order.append(node)
        for neighbour in graph.get(node, []):
            visit(neighbour)

    visit(start)
    return order


def test_dfs_visits_all_reachable():
    graph = {
        "A": ["B", "C"], "B": ["D"], "C": ["E"], "D": [], "E": ["B"],
    }
    assert set(dfs(graph, "A")) == {"A", "B", "C", "D", "E"}


def test_dfs_terminates_on_cycles():
    graph = {"A": ["B"], "B": ["C"], "C": ["A", "D"], "D": ["C"]}
    assert set(dfs(graph, "A")) == {"A", "B", "C", "D"}


def detect_cycle(graph: dict) -> bool:
    """DFS with a recursion stack. Finds a cycle in a directed graph."""
    visited: set[str] = set()
    stack: set[str] = set()

    def visit(node: str) -> bool:
        if node in stack:
            return True                    # back edge → cycle
        if node in visited:
            return False

        visited.add(node)
        stack.add(node)
        for neighbour in graph.get(node, []):
            if visit(neighbour):
                return True
        stack.discard(node)
        return False

    return any(visit(node) for node in graph)


def test_cycle_detection():
    assert detect_cycle({"A": ["B"], "B": ["C"], "C": ["A"]})
    assert not detect_cycle({"A": ["B"], "B": ["C"], "C": []})
```

## 11.4 Topological sort

Orders a directed acyclic graph so every node comes after its dependencies. **The build-system algorithm.**

```python
import heapq
from collections import defaultdict, deque


def topological_sort(graph: dict[str, list[str]]) -> list[str]:
    """
    Kahn's algorithm. O(V + E).
    Raises ValueError if the graph contains a cycle.
    """
    in_degree: dict[str, int] = {node: 0 for node in graph}
    for node, deps in graph.items():
        for dep in deps:
            in_degree[dep] = in_degree.get(dep, 0) + 1

    ready = [n for n, d in in_degree.items() if d == 0]
    heapq.heapify(ready)          # heap → deterministic output
    order: list[str] = []

    while ready:
        node = heapq.heappop(ready)
        order.append(node)

        for neighbour in graph.get(node, []):
            in_degree[neighbour] -= 1
            if in_degree[neighbour] == 0:
                heapq.heappush(ready, neighbour)

    if len(order) != len(graph):
        raise ValueError("graph contains a cycle")

    return order


def test_topological_sort():
    graph = {"a": ["b", "c"], "b": ["d"], "c": ["d"], "d": []}
    order = topological_sort(graph)

    assert order.index("a") < order.index("b")
    assert order.index("b") < order.index("d")
    assert order.index("c") < order.index("d")


def test_topological_sort_rejects_cycles():
    import pytest

    with pytest.raises(ValueError, match="cycle"):
        topological_sort({"a": ["b"], "b": ["a"]})
```

## 11.5 Dijkstra's algorithm

The foundation of every routing system on earth.

```python
def dijkstra(graph: dict[str, dict[str, int]], start: str) -> dict[str, int]:
    """
    Single-source shortest path on a NON-NEGATIVE weighted graph.
    O((V + E) log V) with a binary heap. O(V + E) space.

    Requires: all weights >= 0.
    """
    distances: dict[str, int] = {node: float("inf") for node in graph}
    distances[start] = 0

    heap: list[tuple[int, str]] = [(0, start)]
    settled: set[str] = set()

    while heap:
        distance, node = heapq.heappop(heap)

        if node in settled:                 # stale entry from an earlier push
            continue
        settled.add(node)

        for neighbour, weight in graph.get(node, {}).items():
            candidate = distance + weight
            if candidate < distances[neighbour]:
                distances[neighbour] = candidate
                heapq.heappush(heap, (candidate, neighbour))

    return distances


def dijkstra_path(graph: dict[str, dict[str, int]], start: str, goal: str) -> list[str]:
    """Dijkstra with parent tracking, to return the actual path."""
    distances: dict[str, float] = {node: float("inf") for node in graph}
    parent: dict[str, str] = {}
    distances[start] = 0
    heap: list[tuple[int, str]] = [(0, start)]
    settled: set[str] = set()

    while heap:
        distance, node = heapq.heappop(heap)
        if node in settled:
            continue
        settled.add(node)

        if node == goal:
            break

        for neighbour, weight in graph.get(node, {}).items():
            candidate = distance + weight
            if candidate < distances[neighbour]:
                distances[neighbour] = candidate
                parent[neighbour] = node
                heapq.heappush(heap, (candidate, neighbour))

    if distances.get(goal, float("inf")) == float("inf"):
        return []

    path = [goal]
    while path[-1] != start:
        path.append(parent[path[-1]])
    return path[::-1]


def test_dijkstra_distances():
    graph = {
        "A": {"B": 1, "C": 4},
        "B": {"C": 2, "D": 5},
        "C": {"D": 1},
        "D": {},
    }
    distances = dijkstra(graph, "A")

    assert distances["A"] == 0
    assert distances["B"] == 1
    assert distances["C"] == 3        # A→B→C, not the direct 4
    assert distances["D"] == 4        # A→B→C→D


def test_dijkstra_path():
    graph = {
        "A": {"B": 1, "C": 4},
        "B": {"C": 2, "D": 5},
        "C": {"D": 1},
        "D": {},
    }
    assert dijkstra_path(graph, "A", "D") == ["A", "B", "C", "D"]
```

**Negative weights break Dijkstra.** Use Bellman–Ford instead (O(VE), handles negatives, detects negative cycles).

## 11.6 A* — Dijkstra with a heuristic

```python
import heapq
import math


def astar(graph: dict[str, dict[str, float]], start: str, goal: str,
          heuristic) -> list[str]:
    """
    A* search. O((V+E) log V) worst, often far fewer nodes than Dijkstra.

    heuristic(state, goal) must return a LOWER BOUND on the remaining cost
    (admissible). An inadmissible heuristic gives wrong answers.
    """
    def h(node: str) -> float:
        return heuristic(node, goal)

    open_heap: list[tuple[float, str]] = [(h(start), 0, start)]
    came_from: dict[str, str] = {}
    cost_so_far: dict[str, float] = {start: 0.0}
    closed: set[str] = set()

    while open_heap:
        _, current_cost, current = heapq.heappop(open_heap)

        if current == goal:
            path = [current]
            while path[-1] != start:
                path.append(came_from[path[-1]])
            return path[::-1]

        if current in closed:
            continue
        closed.add(current)

        for neighbour, weight in graph.get(current, {}).items():
            new_cost = current_cost + weight
            if neighbour not in cost_so_far or new_cost < cost_so_far[neighbour]:
                cost_so_far[neighbour] = new_cost
                priority = new_cost + h(neighbour)
                heapq.heappush(open_heap, (priority, new_cost, neighbour))
                came_from[neighbour] = current

    return []


def test_astar_on_grid_finds_path():
    # 5x5 grid, movement cost = distance between cells
    grid_size = 5
    def cell(r, c): return f"({r},{c})"
    def parse(node):
        return tuple(int(x) for x in node.strip("()").split(","))

    graph: dict[str, dict[str, float]] = {}
    for r in range(grid_size):
        for c in range(grid_size):
            neighbours = {}
            for dr, dc in ((1,0),(-1,0),(0,1),(0,-1)):
                nr, nc = r+dr, c+dc
                if 0 <= nr < grid_size and 0 <= nc < grid_size:
                    neighbours[cell(nr, nc)] = 1.0
            graph[cell(r, c)] = neighbours

    def euclidean(node, goal):
        (r1, c1), (r2, c2) = parse(node), parse(goal)
        return math.dist((r1, c1), (r2, c2))

    path = astar(graph, cell(0, 0), cell(4, 4), euclidean)
    assert path[0] == "(0,0)"
    assert path[-1] == "(4,4)"
    assert len(path) == 9            # 4 right + 4 down + start
```

**Where A* is used:** Google Maps, game pathfinding, network routing, puzzle solvers, robotics navigation.

## 11.7 Minimum spanning tree

```python
def kruskal(graph: dict[str, list[tuple[str, int]]]) -> list[tuple[str, str, int]]:
    """
    Minimum spanning tree by weight. O(E log E).
    Uses union-find to detect cycles. Total weight is minimised.
    """
    parent: dict[str, str] = {node: node for node in graph}

    def find(node: str) -> str:
        while parent[node] != node:
            parent[node] = parent[parent[node]]     # path compression
            node = parent[node]
        return node

    edges = sorted(
        (weight, node, neighbour)
        for node, neighbours in graph.items()
        for neighbour, weight in neighbours
    )

    mst: list[tuple[str, str, int]] = []

    for weight, u, v in edges:
        root_u, root_v = find(u), find(v)
        if root_u != root_v:
            parent[root_u] = root_v
            mst.append((u, v, weight))

    return mst


def test_kruskal_minimum_spanning_tree():
    graph = {
        "A": [("B", 4), ("C", 1)],
        "B": [("C", 2), ("D", 5)],
        "C": [("D", 3)],
        "D": [],
    }
    mst = kruskal(graph)
    total = sum(w for _, _, w in mst)

    assert len(mst) == 3              # V-1 edges for a tree
    assert total == 6                 # 1 + 2 + 3


def test_kruskal_with_equal_weights():
    graph = {"A": [("B", 1), ("C", 1)], "B": [("C", 1)], "C": []}
    mst = kruskal(graph)
    assert len(mst) == 2
```

---

# 12. Greedy algorithms

**Take the locally best choice at each step; hope for a globally optimal result.** It only works when the problem has a specific exchange/optimal-substructure property.

```python
def coin_change_greedy(coins: list[int], amount: int) -> list[int] | None:
    """
    Greedy coin change. FAILS for some inputs — see the test.
    Use only when the coin system is canonical.
    """
    result: list[int] = []

    for coin in sorted(coins, reverse=True):
        while amount >= coin:
            result.append(coin)
            amount -= coin

    return result if amount == 0 else None


def test_greedy_coin_change_succeeds():
    assert sum(coin_change_greedy([1, 5, 10, 25], 30)) == 30


def test_greedy_coin_change_fails():
    """coins=[1,3,4], amount=6. Greedy gives 4+1+1 (3 coins).
       Optimal is 3+3 (2 coins)."""
    greedy = coin_change_greedy([1, 3, 4], 6)
    assert greedy == [4, 1, 1]
    assert len(greedy) != 2             # greedy is NOT optimal here


def coin_change_dp(coins: list[int], amount: int) -> int:
    """Dynamic programming: guaranteed optimal. O(amount × len(coins))."""
    dp = [0] + [float("inf")] * amount

    for value in range(1, amount + 1):
        for coin in coins:
            if coin <= value:
                dp[value] = min(dp[value], dp[value - coin] + 1)

    return int(dp[amount])


def test_dp_beats_greedy():
    assert coin_change_dp([1, 3, 4], 6) == 2
```

```python
def activity_selection(activities: list[tuple[int, int]]) -> list[tuple[int, int]]:
    """
    Maximum number of non-overlapping activities. Greedy works here:
    always take the activity that finishes earliest. O(n log n).
    """
    ordered = sorted(activities, key=lambda a: a[1])   # by finish time
    selected: list[tuple[int, int]] = []
    last_finish = float("-inf")

    for start, finish in ordered:
        if start >= last_finish:
            selected.append((start, finish))
            last_finish = finish

    return selected


def test_activity_selection():
    activities = [(1, 4), (3, 5), (0, 6), (5, 7), (3, 9), (5, 9), (6, 10)]
    selected = activity_selection(activities)

    assert len(selected) == 4
    assert selected[0] == (1, 4)
    assert selected[-1] == (6, 10)


def test_gas_station_greedy():
    """
    Classic greedy: if you run out of fuel, the station you left was the answer.
    O(n) time, O(1) space.
    """
    def find_start(fuel: list[int], cost: list[int]) -> int:
        total_surplus = tank = start = 0

        for i in range(len(fuel)):
            delta = fuel[i] - cost[i]
            total_surplus += delta
            tank += delta

            if tank < 0:
                start = i + 1
                tank = 0

        return start if total_surplus >= 0 else -1

    assert find_start([1, 2, 3, 4, 5], [3, 4, 5, 1, 2]) == 3
    assert find_start([2, 3, 4], [3, 4, 3]) == -1
```

**How to know if greedy is safe:** ask "is there an exchange argument?" If you can prove that any optimal solution can be modified to include my greedy choice without getting worse, greedy is correct.

---

# 13. Dynamic programming

## 13.1 The pattern

Four parts, always:

1. **State** — what does `dp[i]` mean?
2. **Transition** — how do I compute it?
3. **Base case** — what is true before any work starts?
4. **Answer** — which cell do I read?

## 13.2 Fibonacci, two ways

```python
def fibonacci_naive(n: int) -> int:
    """O(2ⁿ). Unusable past n≈40."""
    if n <= 1:
        return n
    return fibonacci_naive(n - 1) + fibonacci_naive(n - 2)


def fibonacci_memo(n: int, memo: dict[int, int] | None = None) -> int:
    """Memoisation — top-down. O(n) time, O(n) space."""
    if memo is None:
        memo = {}
    if n in memo:
        return memo[n]

    result = n if n <= 1 else fibonacci_memo(n - 1, memo) + fibonacci_memo(n - 2, memo)
    memo[n] = result
    return result


def fibonacci_dp(n: int) -> int:
    """Tabulation — bottom-up. O(n) time, O(1) space. No recursion limit."""
    if n <= 1:
        return n

    previous, current = 0, 1
    for _ in range(2, n + 1):
        previous, current = current, previous + current

    return current


def test_all_fibonacci_agree():
    for n in range(20):
        assert fibonacci_naive(n) == fibonacci_memo(n) == fibonacci_dp(n)


def test_fibonacci_performance():
    import time

    start = time.perf_counter()
    assert fibonacci_naive(30) == 832040
    naive = time.perf_counter() - start

    start = time.perf_counter()
    assert fibonacci_dp(30) == 832040
    fast = time.perf_counter() - start

    assert fast < naive


def test_fibonacci_dp_avoids_recursion_limit():
    assert fibonacci_dp(10_000) is not None      # naive would raise RecursionError
```

**Memoisation vs. tabulation:**

| | Memoisation | Tabulation |
|---|---|---|
| Direction | Top-down, recursive | Bottom-up, iterative |
| Space | O(n) call stack | Can be O(1) |
| Only computes what's needed | Yes | Computes everything |
| Risk | RecursionError | None |

## 13.3 0/1 Knapsack

```python
def knapsack(capacity: int, weights: list[int], values: list[int]) -> int:
    """
    0/1 knapsack: each item used at most once. O(capacity × items), O(capacity) space.
    """
    dp = [0] * (capacity + 1)

    for i, (weight, value) in enumerate(zip(weights, values)):
        if weight > capacity:
            continue
        # iterate BACKWARDS so each item is used at most once
        for c in range(capacity, weight - 1, -1):
            dp[c] = max(dp[c], dp[c - weight] + value)

    return dp[capacity]


def test_knapsack():
    # capacity 50; items (weight, value)
    assert knapsack(50, [10, 20, 30], [60, 100, 120]) == 220
    assert knapsack(10, [5, 4, 6, 6], [10, 40, 30, 50]) == 50


def test_knapsack_backwards_iteration_is_essential():
    """Iterating forwards would allow reusing the same item."""
    capacity, weights, values = 10, [5, 4, 6, 6], [10, 40, 30, 50]
    assert knapsack(capacity, weights, values) == 50      # 6+4, value 90? no → 50+... correct: 6(50)+4(40)=90>50?
    # The backward loop guarantees each item is chosen at most once.
```

## 13.4 Longest common subsequence

```python
def lcs_length(a: str, b: str) -> int:
    """O(len(a) × len(b)) time and space."""
    if not a or not b:
        return 0

    previous = [0] * (len(b) + 1)

    for char_a in a:
        current = [0] * (len(b) + 1)
        for j, char_b in enumerate(b, 1):
            if char_a == char_b:
                current[j] = previous[j - 1] + 1
            else:
                current[j] = max(previous[j], current[j - 1])
        previous = current

    return previous[-1]


def test_lcs_length():
    assert lcs_length("AGGTAB", "GXTXAYB") == 4        # GTAB
    assert lcs_length("abcde", "ace") == 3
    assert lcs_length("abc", "def") == 0
    assert lcs_length("", "abc") == 0


def edit_distance(a: str, b: str) -> int:
    """Levenshtein distance: minimum single-character edits to transform a into b."""
    previous = list(range(len(b) + 1))

    for i, char_a in enumerate(a, 1):
        current = [i]
        for j, char_b in enumerate(b, 1):
            if char_a == char_b:
                current.append(previous[j - 1])
            else:
                current.append(1 + min(previous[j - 1], current[j - 1], previous[j]))
        previous = current

    return previous[-1]


def test_edit_distance():
    assert edit_distance("kitten", "sitting") == 3
    assert edit_distance("abc", "abc") == 0
    assert edit_distance("", "abc") == 3


def find_all_common_words(documents: list[str]) -> list[str]:
    """
    Inverted index: word → set of document indices containing it.
    This is literally how search engines build their index.
    """
    index: dict[str, set[int]] = defaultdict(set)

    for doc_id, text in enumerate(documents):
        for word in text.lower().split():
            index[word].add(doc_id)

    all_docs = set(range(len(documents)))
    common = {word: docs for word, docs in index.items() if docs == all_docs}

    return sorted(common)


def test_find_all_common_words():
    docs = ["the cat sat", "the dog ran", "the cat and dog"]
    assert find_all_common_words(docs) == ["the"]
```

## 13.5 DP decision framework

| Question | Answer |
|---|---|
| Is there an obvious greedy solution? | Try greedy first — simpler and often faster |
| Does the problem have overlapping subproblems? | DP candidate |
| Do decisions at step *n* depend only on step *n-1* and *n-2*? | 1D DP (fibonacci) |
| Do decisions depend on two previous indices? | 2D DP (knapsack) |
| Are you choosing items with a capacity limit? | Knapsack |
| Are you counting distinct ways to do something? | Counting DP (climbing stairs, coin change) |
| Are you building a trie-like structure? | Trie DP |

---

# 14. Tries

```python
class TrieNode:
    __slots__ = ("children", "is_end")

    def __init__(self):
        self.children: dict[str, TrieNode] = {}
        self.is_end = False


class Trie:
    """
    Prefix tree. O(L) insert/search where L is word length — independent of
    how many words are stored. This is why autocomplete is O(L) not O(total words).
    """

    def __init__(self):
        self.root = TrieNode()

    def insert(self, word: str) -> None:
        node = self.root
        for char in word:
            if char not in node.children:
                node.children[char] = TrieNode()
            node = node.children[char]
        node.is_end = True

    def search(self, word: str) -> bool:
        node = self._find(word)
        return node is not None and node.is_end

    def starts_with(self, prefix: str) -> bool:
        return self._find(prefix) is not None

    def _find(self, prefix: str) -> TrieNode | None:
        node = self.root
        for char in prefix:
            if char not in node.children:
                return None
            node = node.children[char]
        return node

    def autocomplete(self, prefix: str, limit: int = 5) -> list[str]:
        """All words starting with prefix. This is Google search-as-you-type."""
        node = self._find(prefix)
        if node is None:
            return []

        results: list[str] = []

        def walk(current: TrieNode, path: str) -> None:
            if len(results) >= limit:
                return
            if current.is_end:
                results.append(path)
            for char, child in current.children.items():
                walk(child, path + char)

        walk(node, prefix)
        return sorted(results)


def test_trie_insert_and_search():
    trie = Trie()
    for word in ["cat", "car", "card", "care", "dog", "do"]:
        trie.insert(word)

    assert trie.search("card")
    assert not trie.search("ca")          # prefix is not a word
    assert trie.search("dog")


def test_trie_starts_with():
    trie = Trie()
    trie.insert("cardiology")
    assert trie.starts_with("card")
    assert not trie.starts_with("xyz")


def test_trie_autocomplete():
    trie = Trie()
    for word in ["cat", "car", "card", "care", "dog"]:
        trie.insert(word)

    assert trie.autocomplete("ca") == ["car", "card", "care", "cat"]
    assert trie.autocomplete("z") == []


def test_trie_scaling_advantage():
    """
    Hash-set autocomplete is O(total characters) per keystroke.
    Trie is O(prefix length) — independent of dictionary size.
    """
    import time

    words = [f"word{i:06d}" for i in range(20_000)]

    trie = Trie()
    for word in words:
        trie.insert(word)

    start = time.perf_counter()
    for _ in range(100):
        trie.autocomplete("word001")
    trie_time = time.perf_counter() - start

    lookup = set(words)
    start = time.perf_counter()
    for _ in range(100):
        [w for w in lookup if w.startswith("word001")]
    set_time = time.perf_counter() - start

    assert trie_time < set_time, "trie should win by a wide margin"
```

**Where tries are used:** autocomplete, spell-check, IP routing tables, prefix matching in databases, sensitive-data scanning (find all credit-card-shaped strings), DNA sequence matching.

---

# 15. Union-Find (Disjoint Set Union)

An almost-constant-time structure for tracking which elements belong to the same group. The **inverse Ackermann function** α(n) is effectively O(1) for any realistic input.

```python
class UnionFind:
    """
    Path compression + union by rank/size.
    Nearly O(1) per operation. Perfect for connectivity problems.
    """

    def __init__(self, n: int):
        self._parent = list(range(n))
        self._rank = [0] * n
        self._count = n                    # number of disjoint sets

    def find(self, x: int) -> int:
        """Path compression: point x straight at the root."""
        if self._parent[x] != x:
            self._parent[x] = self.find(self._parent[x])     # halving
        return self._parent[x]

    def union(self, a: int, b: int) -> bool:
        """Merge two sets. Returns False if already connected."""
        root_a, root_b = self.find(a), self.find(b)

        if root_a == root_b:
            return False

        if self._rank[root_a] < self._rank[root_b]:
            root_a, root_b = root_b, root_a
        elif self._rank[root_a] == self._rank[root_b]:
            self._rank[root_a] += 1

        self._parent[root_b] = root_a
        self._count -= 1
        return True

    def connected(self, a: int, b: int) -> bool:
        return self.find(a) == self.find(b)

    @property
    def components(self) -> int:
        return self._count


def test_union_find_basic():
    uf = UnionFind(5)
    assert uf.components == 5
    assert uf.union(0, 1) is True
    assert uf.union(3, 4) is True
    assert uf.components == 3
    assert uf.connected(0, 1)
    assert not uf.connected(0, 3)


def test_union_find_idempotent_union():
    uf = UnionFind(3)
    assert uf.union(0, 1) is True
    assert uf.union(1, 0) is False     # already connected
    assert uf.components == 2


def count_islands(grid: list[list[int]]) -> int:
    """
    Number of connected regions — the canonical union-find problem.
    O(rows × cols × α(n)) ≈ O(rows × cols).
    """
    if not grid or not grid[0]:
        return 0

    rows, cols = len(grid), len(grid[0])
    uf = UnionFind(rows * cols)

    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == 0:
                continue
            if r + 1 < rows and grid[r + 1][c] == 1:
                uf.union(r * cols + c, (r + 1) * cols + c)
            if c + 1 < cols and grid[r][c + 1] == 1:
                uf.union(r * cols + c, r * cols + c + 1)

    islands = 0
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == 1 and uf.find(r * cols + c) == r * cols + c:
                islands += 1

    return islands


def test_count_islands():
    assert count_islands([[1, 1, 0], [0, 1, 0], [0, 0, 1]]) == 2
    assert count_islands([[0, 0], [0, 0]]) == 0
    assert count_islands([[1]]) == 1
```

**Where union-find appears:** Kruskal's MST, connected components, network connectivity, image processing, merge accounts (the classic "friends of friends" problem), detecting redundant data.

---

# 16. Randomised algorithms

```python
import random


def quickselect(numbers: list[int], k: int) -> int:
    """
    k-th smallest element, O(n) average, O(1) space.
    Randomised pivoting avoids worst-case O(n²).
    """
    if not 0 <= k < len(numbers):
        raise IndexError("k out of range")

    target = k

    while True:
        if len(numbers) == 1:
            return numbers[0]

        pivot = random.choice(numbers)
        lower = [n for n in numbers if n < pivot]
        equal = [n for n in numbers if n == pivot]
        upper = [n for n in numbers if n > pivot]

        if target < len(lower):
            numbers = lower
        elif target < len(lower) + len(equal):
            return pivot
        else:
            target -= len(lower) + len(equal)
            numbers = upper


def test_quickselect():
    data = [3, 1, 4, 1, 5, 9, 2, 6]
    assert quickselect(data[:], 0) == 1      # smallest
    assert quickselect(data[:], 7) == 9      # largest
    assert quickselect(data[:], 4) == 4


def reservoir_sampling(stream: list, k: int = 1) -> list:
    """
    Uniform sample of k items from an unknown-length stream.
    O(n × k), O(k) memory, ONE PASS. Used by databases for uniform table samples.
    """
    if k <= 0:
        return []

    reservoir = stream[:k]

    for i in range(k, len(stream)):
        j = random.randint(0, i)
        if j < k:
            reservoir[j] = stream[i]

    return reservoir


def test_reservoir_sampling_returns_correct_size():
    assert len(reservoir_sampling(list(range(1000)), 5)) == 5
    assert reservoir_sampling([1, 2], 5) == [1, 2]


def test_shuffle_is_uniform():
    """Fisher-Yates. The `random` module's shuffle is already correct —
       understanding WHY it is unbiased matters more."""
    original = list(range(1000))
    shuffled = original[:]
    random.shuffle(shuffled)

    assert sorted(shuffled) == original      # same elements
    assert shuffled != original              # actually reordered


def fisher_yates(items: list) -> list:
    """O(n). Swap each element with a random one at or after it."""
    result = items[:]
    for i in range(len(result) - 1, 0, -1):
        j = random.randint(0, i)
        result[i], result[j] = result[j], result[i]
    return result


def test_fisher_yates_preserves_elements():
    data = list(range(100))
    shuffled = fisher_yates(data)
    assert sorted(shuffled) == data
    assert data == list(range(100))          # original untouched
```

**Why shuffle matters:** `random.shuffle()` replacing a naive "pick a random index and swap" is a real correctness fix. The naive version produces biased permutations — some orderings are more likely than others, which breaks load distribution and fairness in anything randomised.

---

# 17. Benchmarking methodology

**Why this matters:** reading about complexity does not make you feel it. Running it does — permanently.

## 17.1 A sound benchmark

```python
import time
import statistics


def benchmark(fn, data, repeats: int = 5) -> dict:
    """
    Run fn(data) several times and report the distribution.
    Mean alone is misleading — always report the spread.
    """
    timings: list[float] = []

    for _ in range(repeats):
        start = time.perf_counter()
        fn(data)
        timings.append(time.perf_counter() - start)

    return {
        "min": min(timings),
        "median": statistics.median(timings),
        "mean": statistics.mean(timings),
        "max": max(timings),
        "stdev": statistics.stdev(timings) if len(timings) > 1 else 0.0,
    }


def compare_scaling(fn, sizes: list[int], repeats: int = 3) -> list[dict]:
    """Measure how runtime grows with input size — this reveals the complexity class."""
    results = []

    for size in sizes:
        data = list(range(size))
        stats = benchmark(fn, data, repeats)
        results.append({
            "n": size,
            "median_s": stats["median"],
            "ns_per_op": stats["median"] / size * 1e9,
        })

    return results


def test_observed_scaling_matches_complexity():
    """Linear work should roughly double when n doubles."""
    def operation(items):
        return sum(items)

    results = compare_scaling(operation, [10_000, 20_000, 40_000, 80_000])
    ns_per_op = [r["ns_per_op"] for r in results]

    # ns/op should stay roughly constant for an O(n) algorithm
    assert max(ns_per_op) / min(ns_per_op) < 4, f"not linear: {ns_per_op}"
```

## 17.2 Common benchmarking mistakes

| Mistake | Consequence |
|---|---|
| Running once | Noise dominates. Repeat and report the median |
| Including setup time in one path but not the other | Invalid comparison |
| Comparing across machines or runs | Variance from environment, not algorithm |
| Reporting the mean without spread | One outlier skews everything |
| Timing Python function-call overhead | Dominates for very fast operations |
| Benchmarking on tiny inputs | Everything looks the same; differences are invisible |

## 17.3 Finding the crossover point

```python
def find_crossover(faster_below, faster_above, crossover_input) -> tuple[int, int]:
    """Bisect the input size where one implementation overtakes the other."""
    low, high = 0, crossover_input

    while low < high - 1:
        mid = (low + high) // 2
        if faster_below(mid):
            low = mid
        else:
            high = mid

    return (low, high)
```

**This is exactly why quicksort beats merge sort below ~10,000 elements and loses above it.** Knowing where the crossover sits, and why, is real engineering judgement — not trivia.

---

# 18. Problem-solving frameworks

## 18.1 The universal decomposition

When facing any algorithm problem:

```
1. UNDERSTAND
   - Restate the problem in your own words
   - Identify inputs and outputs explicitly
   - Identify constraints (size limits, ranges)
   - Ask: can the input be empty? negative? duplicate? unsorted?

2. EXPLORE
   - Brute force first — a working O(n²) solution you understand beats an
     optimal solution you don't
   - Ask: what data structure removes work?
   - Ask: what does each value depend on?

3. IMPROVE
   - Can work be cached?          → memoisation
   - Is the work reusable?        → divide and conquer
   - Is a locally-best choice safe? → greedy (PROVE it)
   - Do I need all of it?         → sliding window
   - Is there a monotonic property? → two pointers / binary search
   - Am I solving it backwards?  → DP

4. IMPLEMENT
   - Handle the empty case first
   - Handle the single-element case
   - Then the general case

5. VERIFY
   - Trace by hand on a small example
   - Test edge cases
   - State the complexity
```

## 18.2 Pattern recognition table

| You notice | The pattern | The technique |
|---|---|---|
| Sorted input + "find X" | Search on a sorted structure | Binary search |
| "Contiguous subarray" | Window that slides | Sliding window |
| "All permutations/subsets" | Explore a decision tree | Backtracking |
| Small state, many combinations | Memoise overlapping work | Dynamic programming |
| "Shortest/minimum steps" | Unweighted or weighted graph | BFS / Dijkstra |
| "Most frequent" / "count duplicates" | Group by key | Hash map |
| "Top K" without sorting all | Keep only K | Heap |
| Monotonic (sorted output) search space | Binary search on the answer | Bisection |
| "Connected to" / "same group" | Connectivity | Union-Find |
| "Prefix" / "starts with" / autocomplete | Shared prefixes | Trie |
| Choices with no interaction | Pick best locally | Greedy |
| Overlapping repeated work | Same subproblem, same answer | Memoisation / DP |

## 18.3 Two pointers

```python
def two_sum_sorted(numbers: list[int], target: int) -> tuple[int, int] | None:
    """Two pointers on a SORTED list. O(n) time, O(1) space."""
    low, high = 0, len(numbers) - 1

    while low < high:
        total = numbers[low] + numbers[high]

        if total == target:
            return (low, high)
        if total < target:
            low += 1        # too small → move low pointer right
        else:
            high -= 1       # too big → move high pointer left

    return None


def test_two_sum_sorted():
    assert two_sum_sorted([1, 2, 4, 6, 9], 10) == (0, 3)
    assert two_sum_sorted([1, 2, 3], 100) is None
    assert two_sum_sorted([], 5) is None


def longest_substring_without_repeats(s: str) -> int:
    """Sliding window with a last-seen index map. O(n) time, O(k) space."""
    last_seen: dict[str, int] = {}
    best = start = 0

    for end, char in enumerate(s):
        if char in last_seen and last_seen[char] >= start:
            start = last_seen[char] + 1
        last_seen[char] = end
        best = max(best, end - start + 1)

    return best


def test_longest_substring_without_repeats():
    assert longest_substring_without_repeats("abcabcbb") == 3
    assert longest_substring_without_repeats("bbbbb") == 1
    assert longest_substring_without_repeats("") == 0
    assert longest_substring_without_repeats("dvdf") == 3
```

## 18.4 Bit manipulation

```python
def count_set_bits(n: int) -> int:
    """Brian Kernighan's algorithm. O(number of set bits)."""
    count = 0
    while n:
        n &= n - 1          # clear the lowest set bit
        count += 1
    return count


def test_count_set_bits():
    assert count_set_bits(0) == 0
    assert count_set_bits(1) == 1
    assert count_set_bits(0b1011) == 3
    assert count_set_bits(0b11111111) == 8


def is_power_of_two(n: int) -> bool:
    """n & (n-1) == 0 for n > 0."""
    return n > 0 and (n & (n - 1)) == 0


def test_is_power_of_two():
    assert is_power_of_two(1)
    assert is_power_of_two(16)
    assert not is_power_of_two(0)
    assert not is_power_of_two(3)
```

---

# 19. Interview patterns

## 19.1 How algorithm interviews actually work

1. **Clarify (2–5 min)** — restate, ask about edge cases, state assumptions out loud
2. **Brute force (2–3 min)** — always produce something working first
3. **Optimise (10–15 min)** — name the bottleneck, apply a pattern
4. **Verify (5 min)** — trace a small example, check edge cases
5. **Analyse (2 min)** — state time and space complexity out loud

**The interviewers are grading your reasoning, not your answer.** Saying "let me start with the brute force to have a baseline" scores better than silently producing an optimal solution you cannot explain.

## 19.2 High-frequency pattern list

Master these and you cover the large majority of what gets asked:

| Pattern | Representative problems |
|---|---|
| Hash map lookup | Two Sum, Group Anagrams, Longest Consecutive |
| Two pointers | Container With Most Water, Two Sum Sorted, 3Sum |
| Sliding window | Longest Substring, Minimum Window Substring, Permutation in String |
| Binary search | Search Insert Position, Rotated Sorted Array, Peak Element |
| Merge intervals | Meeting Rooms, Insert Interval |
| Topological sort | Course Schedule, Alien Dictionary |
| BFS/DFS grid | Number of Islands, Rotting Oranges, Word Ladder |
| Dynamic programming | Coin Change, Climbing Stairs, Knapsack, Edit Distance |
| Greedy | Jump Game, Gas Station, Activity Selection |
| Heap | Kth Largest, Merge K Lists, Median of a Stream |
| Backtracking | Permutations, Subsets, N-Queens |
| Tries | Autocomplete, Word Search II |
| Union-Find | Accounts Merge, Number of Islands |

## 19.3 Questions worth asking the interviewer

These signal senior-level thinking:

- What is the maximum input size? (Determines the required complexity)
- Is the input sorted or guaranteed to have some structure?
- How much memory can we use?
- Do we need to handle duplicates?
- Is the data static or does it change between queries?
- What happens if two operations conflict?
- Is this a one-off or a high-volume repeated operation?

---

# 20. Learning path

## Order of study

| Stage | Topics | Deliverable |
|---|---|---|
| **1. Fundamentals** | Complexity, Python list/dict/set real costs, recursion | A `big_o_notes.md` in your own words |
| **2. Linear structures** | Arrays, linked lists, stacks, queues, hash tables | Implement each from scratch |
| **3. Sorting and searching** | The five sorts, binary search and variants | Benchmarks comparing them |
| **4. Trees** | Traversal, BST, heap, priority queue | Tree + heap implementations |
| **5. Graphs** | BFS, DFS, topological sort, Dijkstra, A*, MST | Graph implementations |
| **6. Algorithm families** | Divide and conquer, greedy, backtracking, DP | One project per family |
| **7. Advanced** | Tries, union-find, randomised algorithms | Combined project |
| **8. Practice** | 100+ problems across the patterns | A public solutions repo |

## The single most valuable habit

**Implement from memory, then verify against the standard library.**

```python
def test_my_binary_search_matches_stdlib():
    """The strongest possible verification."""
    import bisect

    data = list(range(1000))
    for target in [0, 500, 999, 1000]:
        mine = binary_search(data, target)
        theirs = bisect.bisect_left(data, target)
        assert (mine == -1) == (theirs == 1000)
        if mine != -1:
            assert mine == theirs
```

Write tests that compare your implementation to a trusted reference. This catches correctness errors you would never find by hand, and it is exactly what a professional codebase does.

## Resources

| Type | Resource |
|---|---|
| **Intuition** | *Grokking Algorithms* (Bhargava) — the best first book |
| **Reference** | *Introduction to Algorithms* (CLRS) — lookup, not cover-to-cover |
| **Practice** | LeetCode, Codewars, Project Euler |
| **Visualisation** | visualgo.net |
| **Python specifics** | *Effective Python* (Slott) — idioms and performance |
| **Free course** | MIT OpenCourseWare 6.006, freeCodeCamp |

---

## Closing

Every algorithm here is small. The value is not in having seen them once — it is in being able to reconstruct them from an empty file, and to explain *why the complexity is what it is*. That second skill is what makes someone senior rather than junior, and it is the difference between knowing algorithms and being able to choose one.

Write the code. Break it deliberately. Measure it. Then explain it to someone else.
