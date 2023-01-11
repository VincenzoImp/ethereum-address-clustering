# Ethereum Address Clustering

Tracing and graph analysis of blockchain addresses linked to *one-day exit scam* accounts on Ethereum and BSC. Starting from a set of seed scam addresses, the pipeline reconstructs the network of accounts that funded them and studies the resulting transaction graph.

## How it works

The project has two stages.

### 1. Collection (`script.py`)

Starting from the seed addresses in `data/one_day_exit_scam_<chain>.csv`, the collector performs a level-by-level breadth-first expansion over on-chain transactions:

- For every block (processed in chunks, in parallel with a multiprocessing `Pool`), it scans transactions whose recipient (`to`) is an address in the current level.
- The corresponding senders (`from`) become the next level — i.e. the accounts that *funded* the current set.
- A per-address `use_untill` block bound keeps the expansion within each address's active window.

Chain data is read through `web3.py`, from either a local Geth node (IPC) or an HTTP provider (Infura for Ethereum, a public dataseed for BSC, with the PoA middleware). The run writes:

- `data/addresses_<chain>.csv` — graph nodes (address, `use_untill`, level)
- `data/transactions_<chain>.csv` — graph edges (from, to, value, gas, hash, input, block, level, status)
- `data/chunk_log_<chain>.csv` — progress log per block chunk

The target expansion depth and chunk size are configurable at the top of the script (`chain`, `query_mode`, `core_number`, `chunk_size`).

### 2. Analysis (`analysis.ipynb`)

The notebook loads the collected nodes and edges, then filters out known labeled service accounts — centralized exchanges, DEXs, NFT/collectibles platforms, and gaming accounts (`data/*_info*.csv`) — so that these high-degree hubs do not act as spurious bridges between otherwise unrelated addresses. It then builds a `networkx` `MultiDiGraph`, computes connected and strongly connected components, and exports the graph to `graph.graphml` for inspection in tools such as Gephi.

## Repository layout

```text
script.py         Transaction-graph collector (BFS over funders)
analysis.ipynb    Filtering, graph construction, component analysis, GraphML export
graph.graphml     Exported transaction graph
data/             Seed scam addresses, collected nodes/edges, labeled-account lists
```

## Requirements

Python 3 with `web3`, `pandas`, `networkx`, and `tqdm`, plus access to an Ethereum/BSC RPC endpoint (local node or HTTP provider). Provider URLs and file paths are set in the `data` config block in both `script.py` and the notebook.

## Status

Research code for a data-analysis study. It expects a funded RPC endpoint and is organized around reproducing the graph, not as a packaged tool.
