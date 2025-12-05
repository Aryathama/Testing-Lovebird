# Refactoring Summary: K-Drone Notebook

## Perubahan Utama

### 1. **Data Loading & Distance Matrix** (Cell: Initial Setup)
**Sebelum:** Banyak baris yang terpisah, logic tidak jelas
**Sesudah:** 
- Struktur clear: load → mapping → vectorized matrix
- Added comments untuk setiap step
- Efficient: pre-compute distance matrix sekali (O(N²))

```python
# Helper functions diperluas:
- id2idx: id → index di locations array
- idx: node ID → index di distance matrix D
- coords: matrix N×2 sesuai urutan nodes
- D: pre-computed distance matrix N×N
```

### 2. **Mutation Operators** (Cell: Lovebird Initial Algorithm)
**Sebelum:** Functions terpisah tanpa dokumentasi konsisten
**Sesudah:**
- Consistent naming & documentation
- Setiap operator diberi docstring singkat
- Komentar color-coded untuk identifiction cepat

```python
Operators:
- swap_two():     Tukar dua segment
- flip():         Reverse segment (2-OPT)
- interchange():  Swap dua nodes
- slide():        Move node ke posisi lain
- guided_swap():  Hybrid flip + interchange
```

### 3. **K-Means Clustering** (Cell: K Clustering)
**Sebelum:** Logic elbow method kompleks, banyak nested loop
**Sesudah:**
- Separated concerns:
  - `elbow_analysis()`: Cari K optimal
  - `run_kmeans()`: Jalankan K-Means
  - `plot_clusters()`: Visualisasi
- Better error handling & input validation
- Enhanced visualization: dual-axis plot (inertia + silhouette)

### 4. **Local Search (2-OPT)** (Cell: Local Search Algorithm)
**Sebelum:** Kode panjang ~150 lines, logic bercampur
**Sesudah:**
- Split menjadi logical blocks:
  1. Time management
  2. Solution initialization
  3. Single-route improvement (nested function)
  4. Multi-route outer loop
- Better variable naming (ops_count, check_every)
- Clearer docstring dengan Parameters/Returns section
- Added throttling untuk time budget check

### 5. **Helper Functions** (Cell: Helper functions untuk Lovebird)
**Baru:**
- `roulette_wheel()`: Selection operator (reusable)
- `_clusters_from_labels()`: Convert labels → cluster dict
- `_apply_mutation()`: Factory pattern untuk operator selection
- `_calculate_multi_route_cost()`: Consistent cost calculation

### 6. **OR-Tools Wrapper** (Cell: OR-Tools for Benchmarking)
**Sebelum:** Banyak helper function tersebar, redundan
**Sesudah:**
- Consolidated menjadi satu logical module
- Clear function hierarchy:
  - Low-level: `_build_distance_matrix()`, `_remap_labels_to_vehicles()`
  - Mid-level: `create_data_model_clustered()`
  - High-level: `solve_vrp_clustered()`, `solve_vrp_random_starts()`
- Better error messages & docstrings

## Code Quality Improvements

### Documentation
- Added docstrings ke semua major functions
- Setiap function memiliki Parameters/Returns section
- Inline comments untuk logic yang kompleks

### Efficiency
- Distance matrix pre-computed (tidak repeated calculation)
- Vectorized NumPy operations untuk koordinat
- Time budget throttling: check setiap N operasi (bukan setiap operasi)

### Maintainability
- Consistent naming convention
- Helper functions dipisah ke cell tersendiri
- Logical grouping per algorithm/technique

### Readability
- Reduced line count in complex functions
- Better variable names (e.g., `ops_count` vs `op`)
- Clear separation: logic vs visualization

## Usage Example

```python
# 1. Load data & clustering
info = elbow_analysis(coords, k_min=1, k_max=10, show_plot=True)
chosen_k = info["best_k"]
labels, clusters, centroids = run_kmeans(coords, nodes, chosen_k)
plot_clusters(coords, labels, centroids)

# 2. Optimize dengan 2-OPT
best_sol, best_cost, history, per_costs = two_opt_local_search(
    maxLoop=100,
    labels=labels,
    nodes=nodes,
    time_budget=20.0
)

# 3. Benchmark dengan OR-Tools
res = solve_vrp_random_starts(nodes, labels, timeout_s=20)
print(f"OR-Tools cost: {res['total_cost']:.2f}")
print(f"Local Search cost: {best_cost:.2f}")

# 4. Visualisasi
plot_clustered_routes_matplotlib(best_sol, title=f"Final Routes • Cost={best_cost:.2f}")
plt.plot(history)
plt.show()
```

## Next Steps (untuk lanjutan)

1. **Implement Lovebird Algorithm Clustered**: GA dengan mutation operators
2. **Implement ILS (Iterated Local Search)**: Perturbation + local search loop
3. **Add benchmarking suite**: Compare methods (OR-Tools vs Lovebird vs ILS)
4. **Adaptive parameters**: Tune pop_size, generations, time_budget otomatis

## Files Modified

- `k_drone_fix.ipynb`: Main notebook dengan refactored code
- `REFACTORING_NOTES.md`: Dokumentasi ini
