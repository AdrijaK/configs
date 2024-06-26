# nf-core/configs: configuration for KU_SUND_DANHEAD
All nf-core pipelines have been successfully configured for use on the DANHEAD server at the Novo Nordisk Foundation Center for Stem Cell Medicine (reNEW) at the University of Copenhagen.

To use the institution profile, run the pipeline with `-profile ku_sund_danhead`. This will download and launch the `ku_sund_danhead.config` which has been pre-configured with a setup suitable for the DANHEAD.

## What you need to know before running nf-core pipleines on DANHEAD

**Danhead** is organized into three compute nodes. When running a nf-core pipeline, SLURM jobs are randomly assigned to these three possible compute nodes. The node priority is as follows:

**dancmp02fl > dancmp01fl > dangpu01fl**

If your job is small, it's likely that all the jobs will be completed within **dancmp02** and there will be no errors. However, when the jobs get distributed across the node lists, the nf-core pipeline may crash.

The error message that indicates that the job was sent to an incorrect node list looks like this:

```
nf-core/rnaseq execution completed unsuccessfully!

The exit status of the task that caused the workflow execution to fail was: null.

... blablabla ...

Caused by:
  Process ` ... blabla ... ` terminated for an unknown reason -- Likely it has been terminated by the external system
```

To prevent the pipelines from crashing, you need to explicitly configure all the SLURM jobs to land on the same nodelist. Currently we have configured the profile **ku_sund_danhead** to use **dancmp02fl** for nf-core computations.
To use the **ku_sund_danhead** profile, remember to explicitly state the nodelist when starting a new SLURM interactive session:

```bash
tmux new-session -s my_slurm_session
srun -c 2 --mem=8gb --time=3-00:00:00 --pty bash
nextflow run nf-core/rnaseq -profile ku_sund_danhead [MORE OPTIONS HERE]
```


## If you want to manually choose which node to use on DANHEAD

Alternatively, if you want to choose yourself which node list you will use for your nf-core jobs, use these three steps:

**Step 1**: Start a new SLURM interactive session. Explicitly state which node list you are launching the nf-core pipeline from:

```bash
srun -c 2 --nodelist=dancmpn01fl --mem=8gb --time=0-01:00:00 --pty bash #available options: dancmpn02fl, dancmpn02fl, dangpu
```

**Step 2**: Configure the `my.conf` file to send all the jobs to the same node list. The `my.conf` file should include `$SLURMD_NODENAME`: this is an environmental variable that stores on which node the slurm jobs are launched from (in this case it is `--nodelist=dancmpn01fl`). 

```bash
process {
  //available options: dancmpn01fl, dancmpn02fl, dangpu
  //dangpu is not recommended for running nf-core pipelines
  clusterOptions = '-w $SLURMD_NODENAME' 
}
```

**Step 3**: Run nf-pipelines with the `my.conf` file included. The command should look like this:

```bash
nextflow run nf-core/rnaseq \
  -profile ku_sund_dangpu \
  -c my.conf \
  [MORE OPTIONS HERE]
```

This will ensure that all the jobs will be sent to the same node list.

