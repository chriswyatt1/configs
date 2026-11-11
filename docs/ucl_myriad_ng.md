# nf-core/configs: Myriad-ng Configuration

All nf-core pipelines have been successfully configured for use on UCL's myriad cluster [University College London](https://www.rc.ucl.ac.uk/docs/Clusters/Myriad/).

> [!NOTE]
> This profile targets **Myriad-ng**, the upgraded part of Myriad that runs RHEL9 with the Slurm scheduler. For the original part of Myriad, which still runs SGE, use the [`ucl_myriad`](ucl_myriad.md) profile instead. Log in to `myriad-ng.rc.ucl.ac.uk` (or `ssh -J ssh-gateway.ucl.ac.uk myriad-ng.rc.ucl.ac.uk`) before you run pipelines. Software installed in your home directory on old Myriad will probably need reinstalling on Myriad-ng.

## Using Nextflow on Myriad

Before running an nf-core pipeline you will need to set up the requirements and install Nextflow.

### Requirements

You will need Java 17 or later to install and run Nextflow. Myriad-ng's software stack is loaded through the `ucl-stack/2026-07` module, which is loaded by default. Load Java with:

```bash
module load openjdk/17.0.11_9/gcc-12.3.0-q2gfpzn
```

Use `module avail openjdk` to see other versions. Your home directory is shared with the old half of Myriad, so if you put this line in your `.bashrc`, wrap it in a check for RHEL9 so that old Myriad keeps its own Java module:

```bash
if grep -q 'release 9' /etc/redhat-release 2>/dev/null; then
    module load openjdk/17.0.11_9/gcc-12.3.0-q2gfpzn
else
    module load java/temurin-17/17.0.2_8
fi
```

Apptainer is installed system-wide on Myriad-ng (`/usr/bin/apptainer`), so it does not need a module.

Slurm copies your login environment into each job, so modules loaded when you launch Nextflow are also available to the jobs it submits.

### Install Nextflow

Download the latest release of nextflow. Warning: the self-update line should update to the latest version, but sometimes not, so please check which is the latest release (https://github.com/nextflow-io/nextflow/releases), you can then manually set this by entering (`NXF_VER=XX.XX.X`).

```bash
## Download Nextflow-all
curl -s https://get.nextflow.io | bash
NXF_VER=XX.XX.X
nextflow -self-update
chmod a+x nextflow
mv nextflow ~/bin/nextflow

```

Then make sure that your bin PATH is executable, by placing the following line in your `.bash_profile`:

```bash
export PATH=$PATH:$HOME/bin

```

## Running an nf-core pipeline

To run the pipeline, make sure to add `-profile ucl_myriad_ng`. This will download and launch the [`ucl_myriad_ng.config`](../conf/ucl_myriad_ng.config) which has been pre-configured with a setup suitable for Myriad-ng, as well as use the correct container engine.

The profile submits jobs through Slurm to the `myriad` partition and caps each job at 36 CPUs, 1483 GB of memory (B-type nodes) and 48 hours. Single-core jobs run under the `1cpu72h` QOS and multi-core jobs under `highcpu48h`. Each job requests 10 GB of local `$TMPDIR` space. Without it, Slurm gives a job only 100 MiB. Processes labelled `process_gpu` go to the `gpu` partition and request one GPU (`--gres=gpu:1`) under the `freegpu` QOS, on an E (V100) or L (A100) node.

```bash
nextflow run nf-core/your-pipeline -profile ucl_myriad_ng --outdir your/output/directory

```

You can check your set up is correct by using `-profile test` before running the pipeline with your own data.

```bash
nextflow run nf-core/your-pipeline -profile test,ucl_myriad_ng --outdir your/output/directory

```
