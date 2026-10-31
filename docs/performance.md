# Performance & Complexity

Atlas is optimized for medium-sized in-memory datasets up to hundreds of thousands of rows, leveraging Nox native code generation.

## 1. Algorithmic Complexity

| Operation | Time Complexity | Auxiliary Space |
| :--- | :--- | :--- |
| **Row Count / Col Count** | $O(1)$ | $O(1)$ |
| **Cell Access (`cell`)** | $O(1)$ | $O(1)$ |
| **Column Projection (`select`/`drop`)** | $O(N \cdot K)$ | $O(N \cdot K)$ |
| **Row Filtering (`filter`)** | $O(N)$ | $O(N_{\text{matched}})$ |
| **Multi-key Sort (`sort_by`)** | $O(N \log N)$ (stable merge sort) | $O(N)$ |
| **Summary Stats (`mean`/`variance`)** | $O(N)$ (Welford's algorithm) | $O(1)$ |
| **Median (`median`)** | $O(N \log N)$ (sorting) | $O(N)$ |
| **Grouping (`group_by`)** | $O(N \cdot G)$ where $G$ is unique groups | $O(N)$ |
| **CSV Export (`to_csv`)** | $O(N \cdot K)$ | $O(N \cdot K)$ |

## 2. Benchmark Results (10,000 Records)

Measured on macOS (Apple Silicon, `noxc 2.0.86`):
- **Generation of 10,000 rows**: ~2 ms
- **Filtering 10,000 rows**: ~1 ms
- **Mean, Median, StdDev**: ~2 ms
- **GroupBy (4 categories, 4 aggregates)**: ~3 ms
- **Multi-key Stable Merge Sort**: ~11 ms
