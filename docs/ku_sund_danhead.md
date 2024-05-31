

**Danhead** is organized into three compute nodes. When running a nf-core pipeline, SLURM jobs are randomly assigned to these three possible compute nodes. The node priority is as follows:

**dancmp02 > dancmp01 > dangpu**

If your job is small, it's likely that all the jobs will be completed within **dancmp02** and there will be no errors. However, when the jobs get distributed across the node lists, the nf-core pipeline may crash.

The error message that indicates that the job was sent to an incorrect node list looks like this:

```
nf-core/rnaseq execution completed unsuccessfully!

The exit status of the task that caused the workflow execution to fail was: null.

... blablabla ...

Caused by:
  Process ` ... blabla ... ` terminated for an unknown reason -- Likely it has been terminated by the external system
```

To prevent the pipelines from crashing, you need to explicitly configure all the SLURM jobs to land on the same nodelist. Since **dancmp01** has the largest available memory, we have configured the profile **ku_sund_danhead** to use that node for nf-core computations.
To use the **ku_sund_danhead** profile, remember explicitly state which node list you are launching the nf-core pipeline from when you start a new SLURM interactive session:

```bash
tmux new-session -s my_slurm_session
srun -c 2 --nodelist=dancmpn02fl --mem=8gb --time=0-01:00:00 --pty bash
nextflow run nf-core/rnaseq -profile ku_sund_danhead [MORE OPTIONS HERE]
```


If you want to determine yourself which node list you will use for your nf-core jobs, use these three steps:

**Step 1**: Start a new SLURM interactive session. Explicitly state which node list you are launching the nf-core pipeline from:

```bash
srun -c 2 --nodelist=dancmpn01fl --mem=8gb --time=0-01:00:00 --pty bash #available options: dancmpn02fl, dancmpn02fl, dangpu
```

**Step 2**: Configure the `my.conf` file to send all the jobs to the same node list. The `my.conf` file should include this bit:

```bash
process {
  //available options: dancmpn01fl, dancmpn02fl, dangpu
  clusterOptions = '-w dancmpn01fl' 
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

