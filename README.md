<!--
 * @Author:  Kanbe 
 * @Date: 2025-08-22 23:52:50
 * @LastEditors:  Kanbe 
 * @LastEditTime: 2025-08-23 00:16:57
 * @FilePath: /undefined/home/hungn/Desktop/Task/2025iDEC/3C/PockDesign/PKG/generate/LigandMPNN/README.md
 * @Description: 
-->
# Modifications to LigandMPNN

This is a functionally extended version of the official LigandMPNN, designed to solve a core workflow : **flexibly designating any protein residue as a "ligand" or "environmental context" for design, without needing to manually edit the PDB file.**

The original LigandMPNN relies on standard records within the PDB file to identify ligands. This means if you want to use a peptide chain, a partner protein at an interaction interface, or even a non-standard PDB file (e.g., exported from VMD) as the design context, you must manually modify the PDB file. This process is both tedious and prone to errors.

## ✨ Core Features

*   **Dynamic Ligand Definition**: A new command-line argument, `--ligand_residues`, allows you to specify in real-time any number of residues that are originally part of the protein (e.g., `"B194 B195 B196"`). The program will then separate them from the "protein" during runtime and treat them as a fixed environmental context for your design.

*   **PDB Compatibility**: This version includes built-in compatibility for PDB files that lack the element information column. If the script cannot read atomic elements (like `C`, `N`, `O`) from the PDB file, it will automatically guess them based on atom names (like `CA`, `N`, `OXT`).
  
> If element information is missing, it will be inferred from first atom names.

*   **Flexible Workflows**: Tasks such as designing an antibody-antigen interface, optimizing residues around an enzyme's active site, or even redesigning one chain of a protein complex while treating the others as a fixed environment have now become incredibly simple.

## 🚀 How to Use

When running `run.py`, in addition to all the standard LigandMPNN arguments, you can now use the newly added `--ligand_residues` argument.

```bash
# argparse definition
argparser.add_argument(
    "--ligand_residues",
    type=str,
    default="",
    help="Specify which residues should be treated as a ligand, e.g., 'B194 B195 B196'. These will be excluded from the protein and treated as context atoms."
)
```

**Argument Format**:
Provide a list of residues as a space-separated string. Each residue is identified by its **chain ID** and **residue number**.
For example: `--ligand_residues "B194 B195 B196 A32"`

## 💡 Example Use Cases

#### Use Case 1: Designing with a Peptide as a Ligand

This was the initial problem we solved. Imagine in the PDB file `5a10_0.pdb`, you want to enhance the interaction between some residues on chain A and residues 194-196 on chain B. However, these chain B residues are marked with standard `ATOM` records. You want to redesign the pocket on chain A around this peptide.

**The New Way**:
Simply add one argument to your command. **No PDB modification is needed.**

```bash
python run.py \
 --pdb_path "./demo/5a10_0.pdb" \
 --out_folder "./motif_design" \
 --redesigned_residues "A105 A106 A107 A108 A109 A110 A111 A112 A113" \
 --model_type "ligand_mpnn" \
 --ligand_residues "B194 B195 B196"
 ... [other arguments]
```

#### Use Case 2: Protein-Protein Interface Design

Suppose you have a protein complex with two chains, A and B. You want to redesign the binding interface on chain A while keeping **the entire chain B** as the fixed environmental context.

**The New Way**:
Pass all residues of chain B to the `--ligand_residues` argument.

```bash
python run.py \
 --pdb_path "complex_AB.pdb" \
 --out_folder "./interface_design" \
 --chains_to_design "A" \
 --model_type "ligand_mpnn" \
 --ligand_residues "B1 B2 B3 B4 ... B150" # List all residues of chain B
 ... [other arguments]
```
The program will automatically treat all atoms of chain B as the ligand context, providing precise environmental information for the design of chain A.

## 🔧 Brief Implementation Details

These modifications to LigandMPNN were achieved by coordinated changes to `data_utils.py` and `run.py`:
1.  **`run.py`**: A new `argparse` argument, `--ligand_residues`, was added. The parsed list of residues is then passed to the `parse_PDB` function.
2.  **`data_utils.py`**:
    *   The `parse_PDB` function now accepts this list of residues.
    *   **Before** classifying atoms, it dynamically builds a ProDy selection string to "intercept" and separate the user-specified ligand residues.
    *   It then excludes these intercepted residues from the standard `"protein"` selection.
    *   Finally, it safely merges these residues with any pre-existing `HETATM` (non-protein atoms) into the final `other_atoms` set.
    *   When processing `other_atoms`, a check and a fallback mechanism for guessing element information were added to ensure the robustness of the process.

## 🙏 Acknowledgements

This work is built upon the original publications of ProteinMPNN and LigandMPNN. If you use this tool in your research, please be sure to cite the original authors' papers.

```
@article{dauparas2023atomic,
  title={Atomic context-conditioned protein sequence design using LigandMPNN},
  author={Dauparas, Justas and Lee, Gyu Rie and Pecoraro, Robert and An, Linna and Anishchenko, Ivan and Glasscock, Cameron and Baker, David},
  journal={Biorxiv},
  pages={2023--12},
  year={2023},
  publisher={Cold Spring Harbor Laboratory}
}

@article{dauparas2022robust,
  title={Robust deep learning--based protein sequence design using ProteinMPNN},
  author={Dauparas, Justas and Anishchenko, Ivan and Bennett, Nathaniel and Bai, Hua and Ragotte, Robert J and Milles, Lukas F and Wicky, Basile IM and Courbet, Alexis and de Haas, Rob J and Bethel, Neville and others},
  journal={Science},
  volume={378},
  number={6615},  
  pages={49--56},
  year={2022},
  publisher={American Association for the Advancement of Science}
}
```