# FastLog

## Project Goal
**FastLog** is a high-performance C++ log processing engine designed to handle massive datasets efficiently. The primary objective is to build a baseline implementation and then iteratively optimize it to achieve a **4x speedup** through advanced techniques in concurrency, zero-copy parsing, and chunked I/O.

Implemented systems programming concepts:
- Zero-copy parsing (pointer arithmetic & `std::string_view`)
- Multithreading and concurrency (thread-local accumulation & range splitting)
- High-performance buffered chunked I/O
- Rapid log filtering (by level, user, or latency threshold)
- Fast statistics aggregation (level counts, top-k users via min-heap, average latency, p95)

## Log Format
The engine processes ISO8601-timestamped log files with the following structure:
```text
<TIMESTAMP> <LEVEL> <USER> <LATENCY> "<MESSAGE>"
```

**Example:**
```text
2025-08-25T11:33:35Z WARN user2920 152ms "User authenticated"
2026-10-25T03:13:06Z ERROR user8469 66ms "Permission denied"
```

## Directory Structure

```text
.
├── .github/             # GitHub configuration (CI workflows)
├── docs/                # Project documentation and track details
├── include/             # C++ header files
│   └── fastlog/         # Core headers (types.h)
├── log generator/       # Test log generator utility (generator.cpp)
├── src/                 # Source code
│   └── main.cpp         # Main application entry point (optimized engine)
├── base.cpp             # Baseline single-threaded implementation
├── final.cpp            # Optimized multithreaded engine with filtering
├── task2/               # Task 2 benchmarks and contributions (Siddharth)
└── README.md
```

## Code Structure

### `base.cpp`
- Implements the **baseline version** of the system  
- Single-threaded execution using standard stream reading  
- Parses, filters, and aggregates metrics (counts, average, top 10 users, 95th percentile)  
- Serves as the reference point for measuring optimization gains  

---

### `final.cpp`
- Implements the **optimized version** of the system  
- Employs zero-copy string scanning with `std::string_view`  
- High-throughput 8MB buffered binary reads with `memchr`  
- Multithreaded execution across split file ranges with thread-local accumulators  
- Fast filtering by log level, user ID, and latency  
- Min-heap (`std::priority_queue`) for top-10 user extraction and `std::nth_element` for 95th percentile  

---

### `src/main.cpp`
- Production entrypoint containing the optimized engine and CLI interface  

---

### `log generator/generator.cpp`
- Utility script to generate customizable synthetic log files (e.g. 100MB, 1GB)  

## How to Run

### Prerequisites
- C++17 compliant compiler (GCC/Clang/MSVC)

### Build
```bash
g++ -Iinclude src/main.cpp -o fastlog
```

### Run
```bash
./fastlog "log generator/tc1.log"
```
