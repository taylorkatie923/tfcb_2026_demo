 ## Contents of the original workbook
 
| Sheet | Contents |
|---|---|
| `Task` | Instructions for the exercise |
| `Plate 4` | Cq grid, rows A–F × columns 1–9; amplicons GAPDH (A–C) and JUN (D–F) |
| `Plate 7` | Cq grid, rows A–F × columns 2–10; amplicons GAPDH (A–C) and EGR1 (D–F); note below the grid: `F07: outlier, discard` |
 
On both plates, the three column groups are sgRNAs `GCN1-1_2`, `GCN1-3_4` and `FLUC-1_2`, each treated with DMSO, 100 nM or 1 µM HHT for 6 hours in U937 cells. Rows A–C and D–F are technical triplicates.

Excel was used to tidy this data such that this data could be loaded as a DataFrame. 

## Changelog

- **2026-10-13:** Created `tidy_data.csv` from plates 4 and 7 of the Lecture 3 workbook, and added this README and `data/README.md`.

## Derived file
 
- `tidy_data.csv`: one row per well of each plate (108 rows), with columns `plate`, `well`, `amplicon`, `sgrna`, `treatment`, `cq`, `note`
