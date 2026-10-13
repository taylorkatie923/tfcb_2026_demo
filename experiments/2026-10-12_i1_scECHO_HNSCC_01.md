# scEHCO in HNSCC

2026-10-12

[Issue #1](https://github.com/tayorkatie923/tfcb_2026_demo/issues/1)


## Reagents

For this computational experiment, reagents are software functions and input data. *Vendor* is where each comes from, *catalog number* is the version or file, *stock concentration* is the package default (or the raw input size), and *final concentration* is the setting or input size actually used.

| reagent | vendor | catalog number | stock concentration | final concentration |
|---|---|---|---|---|
| Pt56 RNA object | lab server: `user/cfinkei/nexus-code/work/scecho-manuscript-39/dataset_assembly/hnscc/only_tumor/samples/` | `patient_sample_set_56.h5ad` | 4,613 cells × 16,524 genes, log-normalized | all 4,613 cells; 2,442 HVGs for echo features |
| Pt56 ATAC ArchR export | lab server: `.../only_tumor/archr_out/set_56/export` | `peak_counts/`, `svd.csv`, `cell_metadata.csv` | 4,613 cells × 180,489 peaks; 30 LSI dims | 4,613 cells; LSI with nFrags regressed |
| chromVAR motif deviations | ArchR export (same folder) | `chromvar_motif_scores.csv` | all ArchR motifs (z-scores) | 489 motif TFs matched to gene symbols |
| Cell-cycle gene list (Tirosh/Regev) | lab server: `cfinkbei/` | `regev_lab_cell_cycle_genes.txt` | 97 genes (43 S, 54 G2M) | 43 S, 53 G2M (3 renamed to current symbols) |
| scanpy `tl.rank_genes_groups` | PyPI | 1.12.4 | `method="t-test"` | `method="wilcoxon"`, `layer="lognorm_counts"`, `use_raw=False` |
| scanpy `tl.filter_rank_genes_groups` | PyPI | 1.12.4 | `min_in_group_fraction=0.25`, `max_out_group_fraction=0.5`, `min_fold_change=1` | `0.25`, `0.5`, `min_fold_change=1.5` |
| scanpy `tl.leiden` | PyPI | 1.12.4 | `resolution=1` | `resolution=1`, `flavor="igraph"`, `n_iterations=2`, `directed=False` |
| scanpy `pp.neighbors` (depth-corrected) | PyPI | 1.12.4 | `n_neighbors=15`, all dims | `n_neighbors=15`, `n_pcs=4` |
| scanpy `tl.score_genes_cell_cycle` | PyPI | 1.12.4 | gene lists required | RNA: log-norm counts; ATAC: log1p ArchR gene scores; `random_state=0` |
| scEcho `utils.regress_embedding` | [GitHub](https://github.com/settylab/scEcho) | 0.0.5 | no default (embedding and depth column required) | `X_pca` ~ `total_counts`; `X_lsi` ~ `nFrags` |
| scEcho `utils.compute_diffusion_map` | [GitHub](https://github.com/settylab/scEcho) | 0.0.5 | see `help()` | `t="multiscale"` |
| scEcho `echo_states.dn_comp_obsm` | [GitHub](https://github.com/settylab/scEcho) | 0.0.5 | `pval_threshold=0.05`, `log_fold_change_threshold=2`, `ls_factor=2` | `pval_threshold=0.05`, `log_fold_change_threshold=0.7`, `ls_factor=6` |
| scEcho `echo_features.run_echo_features` | [GitHub](https://github.com/settylab/scEcho) | 0.0.5 | `sigma=0.1`, `ls=None` (auto), `ls_factor=10` | `sigma=0.1`, `ls=10**-0.5` (≈ 0.32) |
| scEcho `plotting.linked_plot` | [GitHub](https://github.com/settylab/scEcho) | 0.0.5 | `downsample_lines_frac=0.3`, `line_alpha=0.5` | `downsample_lines_frac=0.3` (0.1 for gene panels), `line_alpha=0.1` |
| mudata | PyPI | 0.4.1 | n/a | `rna` + `atac`, 4,613 shared cells |
| anndata | PyPI | 0.13.4 | n/a | n/a |

## Procedure

Computational equivalents: *volume* = number of cells/features, *time* = runtime, *temperature* = compute resources. All steps were run in `KT_pt_sample_56.ipynb` on a shared node.

1. Load the Pt56 RNA object (4,613 cells × 16,524 genes) and copy the log-normalized `X` to the `lognorm_counts` layer.
2. Load the ArchR ATAC export (180,489 peaks, 30 LSI dims) and combine it with the RNA object into a MuData of 4,613 shared cells.
3. Add the chromVAR deviations as the `chromVAR` layer, keeping the 489 motifs that match a gene symbol.
4. Cluster the RNA data with HVG selection → PCA → neighbors → Leiden (`resolution=1`, igraph, 2 iterations).
5. Run Wilcoxon differential expression, each Leiden cluster vs the rest, on `lognorm_counts` (key `rank_genes_leiden`).
6. Plot the top 5 DEGs per cluster as a dot plot (`standard_scale="var"`).
7. Run Wilcoxon differential accessibility on the 489 chromVAR motifs per cluster, reporting `mean_diff` in place of log fold change.
8. Regress sequencing depth out of the embeddings (`X_pca` ~ `total_counts`, `X_lsi` ~ `nFrags`).
9. Build depth-corrected UMAPs (`n_pcs=4`) and align the ATAC UMAP to the RNA UMAP.
10. Compute multiscale diffusion maps for RNA and ATAC from the depth-regressed embeddings.
11. Compare RNA vs ATAC cell-state density with `dn_comp_obsm` (p < 0.05, |log FC| ≥ 0.7, `ls_factor=6`), giving a direction call per cell. (56.94s)
12. Plot the direction (RNA-higher / ATAC-higher / neither) on the linked UMAPs and as fractions per Leiden cluster.
13. Plot the desynchronized-state volcano colored by Leiden cluster.
14. Run echo features on the 2,442 HVGs (`sigma=0.1`, `ls=10**-0.5`) per Leiden cluster.
15. Run echo features on the 489 chromVAR motifs, taking cell counts from `lognorm_counts`.
16. Plot echo-feature volcanoes per cluster (≥ 30 cells, 15–25 labels).
17. Plot the top 8 desynchronized genes per cluster (by `var_explained_diff`) on the linked UMAPs.
18. Plot FOXA1 and SOX6 on the linked UMAPs: observed chromVAR, MAGIC expression, accessibility predicted from ATAC and from RNA space, and the predicted residual difference.

## Result

![Linked RNA and ATAC UMAPs colored by desynchronization direction](img/2026-10-12_schEcho_HNSCC_linkedplot.png)

*Figure 1.* Linked RNA (left) and depth-corrected ATAC (right) UMAPs of 4,613 Pt56 tumor cells. Each cell is colored by whether its state density is higher in RNA, higher in ATAC, or not different (scEcho `dn_comp_obsm`, depth-regressed multiscale diffusion maps). Figure produced by Katie Taylor with scEcho 0.0.5 in `KT_pt_sample_56.ipynb`.
