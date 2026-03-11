# push_swap

![Language](https://img.shields.io/badge/Language-C-blue)
![School](https://img.shields.io/badge/School-Hive%20Helsinki-green)
![Type](https://img.shields.io/badge/Type-Algorithm-blueviolet)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

A sorting algorithm that arranges integers using **two stacks** and a **limited set of operations** — optimized for the minimum number of moves. Built at [Hive Helsinki](https://www.hive.fi/) (42 Network).

---

## Demo

```bash
$ ./push_swap 4 67 3 87 23
ra
pb
ra
pb
ra
pa
pa

$ ./push_swap 2 1 3 6 5 8
sa
pb
pb
ra
pa
pa

$ ARG=$(shuf -i 1-100 -n 100 | tr '\n' ' '); ./push_swap $ARG | wc -l
587
```

---

## Available Operations

| Operation | Description |
|-----------|-------------|
| `sa` | Swap top two elements of stack A |
| `sb` | Swap top two elements of stack B |
| `ss` | `sa` + `sb` simultaneously |
| `pa` | Push top of B onto A |
| `pb` | Push top of A onto B |
| `ra` | Rotate A up (top → bottom) |
| `rb` | Rotate B up (top → bottom) |
| `rr` | `ra` + `rb` simultaneously |
| `rra` | Reverse rotate A (bottom → top) |
| `rrb` | Reverse rotate B (bottom → top) |
| `rrr` | `rra` + `rrb` simultaneously |

---

## Algorithm: Chunk-Based Sorting

### Step 1: Normalize to Indices

```
  Input:    42  -7  100  3  55
  Indices:   2   0    4  1   3

  → Work with ranks (0..N-1) instead of raw values
    so chunks divide evenly regardless of input distribution
```

### Step 2: Push to B in Chunks

```
  Stack A          Stack B         Chunk Range
  ┌─────┐          ┌─────┐
  │  4  │          │     │        Push indices 0~1
  │  2  │    pb    │  0  │        to B first,
  │  0  │  ────►   │  1  │        then 2~3,
  │  3  │          │     │        then 4.
  │  1  │          │     │
  └─────┘          └─────┘

  For each number in A:
    Is index within current chunk range?
    ├─ YES → pb (push to B)
    │        If below chunk midpoint → rb (rotate to bottom of B)
    └─ NO  → ra (rotate A, try next)

  After all pushed: B has numbers roughly sorted in descending chunks
```

### Step 3: Push Back to A (Greedy)

```
  Stack B              Stack A
  ┌─────┐              ┌─────┐
  │  3  │              │     │
  │  4  │   find max   │  4  │     Find the maximum in B,
  │  1  │   ────────►  │  3  │     calculate cheapest rotation
  │  2  │   push to A  │  2  │     (rotate or reverse-rotate),
  │  0  │              │  1  │     then pa.
  └─────┘              │  0  │
                       └─────┘     Repeat until B is empty.

  Cost optimization:
  ┌──────────────────────────────────────────┐
  │ If both stacks need rotation in the same │
  │ direction → use rr or rrr to save moves  │
  └──────────────────────────────────────────┘
```

---

## Performance

| Input Size | Target (Full Score) | Target (Pass) | Achieved |
|-----------|--------------------|--------------|---------:|
| 3 | ≤ 2 | ≤ 3 | ≤ 2 |
| 5 | ≤ 12 | ≤ 12 | ≤ 12 |
| 100 | ≤ 700 | ≤ 900 | ~580 |
| 500 | ≤ 5500 | ≤ 7000 | ~5200 |

---

## Build & Run

### Prerequisites

- GCC or Clang
- GNU Make

### Build

```bash
git clone https://github.com/Hyeon-coder/push_swap.git
cd push_swap
make
```

### Run

```bash
# Sort specific numbers
./push_swap 4 67 3 87 23

# Generate random input and count operations
ARG=$(shuf -i 1-500 -n 500 | tr '\n' ' '); ./push_swap $ARG | wc -l

# Verify correctness (pipe to checker if available)
ARG=$(shuf -i 1-100 -n 100 | tr '\n' ' '); ./push_swap $ARG | ./checker $ARG
```

### Clean

```bash
make clean    # Remove object files
make fclean   # Remove object files and binary
make re       # Rebuild from scratch
```

---

## Key Challenges & What I Learned

### 1. Failed First Attempt with Quicksort
My initial approach was a quicksort variant adapted for stack operations. It worked well on average but had **worst-case scenarios** where operation counts exploded far beyond the limits. The chunk-based approach trades theoretical elegance for consistent, predictable performance.

### 2. Chunk Size Tuning
Chunk size directly impacts total operations: too small means expensive B→A transfers (many rotations to find the max), too large means B becomes poorly ordered. Through empirical testing across thousands of random inputs, I found **chunk sizes of 20–25 for 100 elements** and proportionally larger for 500 hit the sweet spot.

### 3. Index Normalization
Working with raw values makes chunk boundaries uneven — one chunk might contain 50 numbers while another has 5. Normalizing inputs to ranks `0..N-1` guarantees each chunk contains exactly the same number of elements, making the algorithm uniformly efficient regardless of input distribution.

### 4. Combined Rotation Cost Optimization
When both stacks need rotating in the same direction, using `rr` or `rrr` saves one operation per combined move. Calculating the **true cost** considering these combined operations (instead of treating each stack independently) shaved hundreds of operations off the 500-element case.

---

## License

This project was developed as part of the 42 curriculum at Hive Helsinki.
