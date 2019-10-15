# PyTorch
[![https://www.singularity-hub.org/static/img/hosted-singularity--hub-%23e32929.svg](https://www.singularity-hub.org/static/img/hosted-singularity--hub-%23e32929.svg)](https://singularity-hub.org/collections/1266)

Singularity container running PyTorch.

# Pytorch with Visdom Job
```
#!/bin/bash
#PBS -N pytorch_test
#PBS -l nodes=1:ppn=1:gpus=1
#PBS -l mem=5gb
#PBS -l walltime=1:00:00
#PBS -j oe

# get tunneling info
port=$(shuf -i8000-9999 -n1)
node=$(hostname -s)
user=$(whoami)
submit_host=$(echo ${PBS_O_HOST%%.*}.rcc.mcw.edu)
cd $PBS_O_WORKDIR

# print tunneling instructions
echo -e "
SSH tunnel from your workstation using the following command:
   
   ssh -L ${port}:${node}:${port} ${user}@${submit_host}
   
and point your web browser to http://localhost:${port}.
"

# load modules
module load pytorch/1.3.0

# start visdom monitor(required)
pytorch python -m visdom.server -port ${port} &
visdomps=`ps ax | grep visdom | awk 'NR==1{print $1}'`

# start training
pytorch python train.py --display_port ${port} --dataroot ./datasets/maps --name maps_cyclegan --model cycle_gan --no_dropout

# cleanup visdom server
kill $visdomps
```

## Start the PyTorch Job
1. Copy contents of pytorch.pbs (example above) to a file in your home directory.

2. Open terminal on cluster login node and submit the job script:

```
$ qsub pytorch.pbs
```

## Connect to Visdom
1. Check output file (*jobname*.o*JOBNUM*) for details.

Example output file: pytorch_test.o166283 (Port numbers will be different for your job)
```
SSH tunnel from your workstation using the following command:

   ssh -L 8844:node01:8844 tester@loginnode

and point your web browser to http://localhost:8844.
```

2. Open second terminal and run tunneling command from the output file:
```
$ ssh -L 8844:node01:8844 tester@loginnode
```
3. Open a browser and enter the URL from the output file:
```
http://localhost:8844
```

You should now be connected to your Visdom monitor. It is not necessary to connect, but it will show useful realtime information about training. To close Visdom, close browser window. If you need to reconnect, repeat steps.

## PyTorch with TensorBoard Job Script
```
#!/bin/bash
#PBS -N pytorch_tensorboard
#PBS -l nodes=1:ppn=1:gpus=1
#PBS -l mem=5gb
#PBS -l walltime=1:00:00
#PBS -j oe

# get tunneling info
port=$(shuf -i8000-9999 -n1)
node=$(hostname -s)
user=$(whoami)
submit_host=$(echo ${PBS_O_HOST%%.*}.rcc.mcw.edu)

# add your own log directory (required)
logdir="~/path/to/your/logdir"

cd $PBS_O_WORKDIR

# print tunneling instructions
echo -e "
1. SSH tunnel from your workstation using the following command:

   ssh -L ${port}:${node}:${port} ${user}@${submit_host}

   and point your web browser to http://localhost:${port}.

2. Copy/paste the token below when you connect for the first time.
"

# load modules
module load pytorch/1.3.0

# start training
pytorch python train.py --display_port ${port} --dataroot ./datasets/maps --name maps_cyclegan --model cycle_gan --no_dropout

# start tensorboard
tensorboard-gpu

```
1. Copy contents of tensorboard.pbs (example above) to a file in your home directory. Modify the logdir variable.

2. Open terminal on cluster login node and submit the job script:

```
$ qsub tensorboard.pbs
```

### Connect to TensorBoard
1. Check output file (*jobname*.o*JOBNUM*) for details.

Example output file: tensorboard.o178936
```
SSH tunnel from your workstation using the following command:

   ssh -L 8754:node01:8754 tester@loginnode

and point your web browser to http://localhost:8754.
```

2. Open second terminal and run tunneling command from the output file:
```
$ ssh -L 8754:node01:8754 tester@loginnode
```
3. Open a browser and enter the URL from the output file:
```
http://localhost:8754
```

You should now be connected to TensorBoard. To close TensorBoard, close browser window. If you need to reconnect, repeat steps.
