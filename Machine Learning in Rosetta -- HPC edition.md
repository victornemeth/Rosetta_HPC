**This file will help you follow the tutorial on [ml_in_rosetta.pdf](https://meilerlab.org/wp-content/uploads/2025/11/ml_in_rosetta.pdf) for the HPC.**

This file just contains the commands that needs to be run differently on the HPC.
## A few notes:
- `pymol` commands you should run on your own pc, so download the output and open in pymol on your own pc.
- Step 2 `export ROSETTA=/PATH/TO/YOUR/ROSETTA/FOLDER/` **is not needed** on the HPC if you follow the commands below.
- `gedit` is a file editor, but is not available on the HPC, use `cat` instead to print the contents of a file

## Getting started

First you need to request some resources for this example we will request 48CPU cores and 120GB of RAM and 3 hours on the doduo cluster.

```bash
module swap cluster/doduo
qsub -I -l nodes=1:ppn=48 -l mem=120gb -l walltime=03:00:00
```

Then we load the module for CPU parallelization.

```bash
module load OpenMPI/4.1.5-GCC-12.3.0
```

Now you can make your folder and copy the files to there. Tip you should do this in your VO (e.g. `cd /kyukon/scratch/gent/vo/002/gvo00214/vsc12345`)

```bash
mkdir my_files
```

Now we use a containerized version of Rosetta and bind our folder to the workspace, so that it is available in the container. We also bind the models to avoid future re-downloads of the models.

**Make sure you are outside your `my_files` folder!** (e.g. `ls` => my_files)

```bash
apptainer shell \
  --bind ./my_files/:/workspace/ \
  --bind $VSC_SCRATCH_KYUKON_VO/rosetta/rosetta_models/:/usr/local/database/protocol_data/tensorflow_graphs/tensorflow_graph_repo_submodule/ \
  --bind $VSC_SCRATCH_KYUKON_VO/rosetta/mifst/:/usr/local/database/protocol_data/inverse_folding/mifst/ \
  $VSC_SCRATCH_KYUKON_VO/rosetta/rosetta-configured.sif
```

Now inside the container you can go inside your folder.

```
cd my_files
```

**NOTE:** This folder exists both in the container and on the HPC, so you can still download files to that folder e.g. on a different terminal, vscode or dashboard.

## HPC commands (adapted version of those in the tutorial)

3.2:

```bash
python /usr/local/tools/protein_tools/scripts/clean_pdb.py 8BRB A
```

3.3:

```bash
/usr/local/bin/relax.cxx11threadmpiserializationtensorflowtorch.linuxgccrelease -s 8BRB_A.pdb -nstruct 1 -ex1 -ex2 -constrain_relax_to_start_coords -out:suffix _relax -beta
```

4.2

```bash
/usr/local/bin/rosetta_scripts.cxx11threadmpiserializationtensorflowtorch.linuxgccrelease \
    -s 8BRB_A_relax_0001.pdb \
    -parser:protocol predict_probs.xml \
    -auto_download \
    -out:file:score_only score.sc
```

5.2

```bash
/usr/local/bin/rosetta_scripts.cxx11threadmpiserializationtensorflowtorch.linuxgccrelease \
-s 8BRB_A_relax_0001.pdb \
-parser:protocol current_probs.xml \
-out:suffix _mpnn_probs \
-parser:script_vars filename=mpnn_probs.weights
```

5.3

```bash
/usr/local/bin/rosetta_scripts.cxx11threadmpiserializationtensorflowtorch.linuxgccrelease \
-s 8BRB_A_relax_0001.pdb \
-parser:protocol current_probs.xml \
-out:suffix _esm_probs \
-parser:script_vars filename=esm_probs.weights
```

5.4

```bash
/usr/local/bin/rosetta_scripts.cxx11threadmpiserializationtensorflowtorch.linuxgccrelease \
-s 8BRB_A_relax_0001.pdb \
-parser:protocol current_probs.xml \
-out:suffix _mifst_probs \
-parser:script_vars filename=mifst_probs.weights
```

6.2

```bash
/usr/local/bin/rosetta_scripts.cxx11threadmpiserializationtensorflowtorch.linuxgccrelease \
-s 8BRB_A_relax_0001.pdb \
-parser:protocol best_mutations.xml \
-beta-out:file:score_only best_mutations.sc
```

7.2

```bash
/usr/local/bin/rosetta_scripts.cxx11threadmpiserializationtensorflowtorch.linuxgccrelease \
-s 8BRB_A_relax_0001.pdb \
-parser:protocol sample_sequences.xml \
-beta \
-out:suffix _design-nstruct
```

8.2

```bash
/usr/local/bin/rosetta_scripts.cxx11threadmpiserializationtensorflowtorch.linuxgccrelease \
-s 8BRB_A_relax_0001.pdb \
-parser:protocol constrain_energy.xml \
-beta \
-ex1 \
-ex2aro \
-nstruct 20 \
-out:suffix _constraint_design
```

9.2

```bash
/usr/local/bin/rosetta_scripts.cxx11threadmpiserializationtensorflowtorch.linuxgccrelease \
-s 1KX5_chAB.pdb \
-parser:protocol IFA_perplexity.xml \
-beta \ 
-out:file:score_only score_interface.sc
```

10.2

```bash
/usr/local/bin/rosetta_scripts.cxx11threadmpiserializationtensorflowtorch.linuxgccrelease \
-s 1KX5_chAB.pdb \
-parser:protocol current_probs_mifst_dimer.xml \
-out:suffix _mifst_probs
```