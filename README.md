# Topological Data Analysis (TDA) for Limit Order Books

## 📉 Detecting Market Fragility with Topology

This project is a Proof-of-Concept (PoC) demonstrating how **Topological Data Analysis (TDA)**—specifically Persistent Homology using the **Gudhi** library—can be used to detect structural fragility in financial markets (Limit Order Books) *before* a price crash occurs.

### 🚀 Overview
Standard financial metrics (volatility, volume, spread) often look at aggregate statistics. However, they may miss the geometric "shape" of liquidity. This project treats the Limit Order Book (LOB) as a **Point Cloud** in 2D space (Price vs. Volume) and extracts topological features (Betti numbers) to identify:
1.  **Fragmentation (Betti-0):** How scattered the orders are.
2.  **Liquidity Voids (Betti-1):** Large "holes" in the order book structure that indicate a lack of support, foreshadowing a crash.

---

## 🛠️ Prerequisites & Installation

The code relies on `Gudhi` for topological computations and standard Python data science libraries.

```bash
pip install gudhi numpy pandas matplotlib
```

---

## 🧬 The Experiment: Synthetic Data Phases

The code generates a synthetic dataset simulating a "Flash Crash" over 9 distinct time steps. The simulation moves through four specific phases:

1.  **Stable Market (t=0, 1):**
    *   High density of orders.
    *   Tight spread.
    *   **Topology:** Low Betti-1 persistence (no holes), stable Betti-0.

2.  **Fragile Market (t=2, 3, 4):**
    *   **The Setup:** Orders immediately behind the best bid are removed (hollowed out), creating a "void."
    *   *Crucially:* The current mid-price remains roughly the same. Standard metrics might look normal, but the structure is weak.
    *   **Topology:** Spike in **Betti-1 Persistence** (a large hole appears).

3.  **The Crash (t=5, 6):**
    *   Selling pressure hits. Because the support orders were removed in Phase 2, the price plummets through the void to find the next cluster of orders.
    *   **Topology:** The hole collapses as the price adjusts to the new level.

4.  **Recovery (t=7, 8):**
    *   Liquidity rebuilds at the lower price level.
    *   **Topology:** Betti numbers return to stable baselines.

---

## 📊 How to Interpret the Results

The script produces two visualization outputs.

### 1. Point Cloud Inspection
The script visualizes the LOB as a geometric shape.
*   **X-Axis:** Price Delta (distance from mid-price).
*   **Y-Axis:** Scaled Volume (Square root of volume).
*   *Observation:* You will visually see the "gap" between the best price and the rest of the market during the **Fragile** phase.

### 2. The "Definitive Proof" Plot
This is the main result dashboard containing two subplots:

*   **Top Plot (Price Evolution):**
    *   Shows the synthetic price over time. Note how the price is relatively stable until it suddenly drops at `t=5`.

*   **Bottom Plot (Topological Indicators):**
    *   **🔴 Red Line (Betti-1 Persistence):** represents **Liquidity Voids**.
    *   **🔵 Blue Line (Betti-0 Count):** represents **Fragmentation**.

**The Key Takeaway:**
> **Look at Time Step 2-4.** The Red Line (Betti-1) spikes *before* the price crashes. This demonstrates that TDA provides an **early warning signal** of structural weakness that purely price-based metrics would miss.

---

## 📂 Code Structure

*   **`synthetic_lob_snapshots`**: A manually defined dictionary simulating the order book state (Bids/Asks) at every timestamp.
*   **`run_tda_on_snapshot`**:
    *   Converts LOB data into a Point Cloud.
    *   Constructs an **Alpha Complex**.
    *   Computes the **Simplex Tree** and **Persistence**.
    *   Returns Betti-0 counts and the Maximum Persistence of Betti-1 (the "size" of the biggest hole).
*   **`process_and_analyze`**: Iterates through the timeline, normalizing volume using `np.sqrt` to ensure the X and Y axes are geometrically comparable.

---

## 🏃 Usage

Simply run the script in a Python environment (Jupyter Notebook or terminal):

```python
python tda_flash_crash.py
```

*Note: Ensure your Python environment supports `gudhi` (C++ backend required, pre-compiled wheels are available for Linux, Mac, and Windows).*
