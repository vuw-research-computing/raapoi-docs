# Training

These are longer worked examples.  If you have domain specific training you'd like to provide for your students or peers, contact Andre or Wes, or make a pull request against this repo.

## GPU example with neural style in pytorch

We'll do a quick python example using neural style implimented in pytorch. We'll be using python virtual enviroments to install our own python packages.  Note we could also use conda for this, but python virtualenvs are a bit lighter weight.

The code we use will come from the pytorch example git repo.

### Clone the pytorch example repo

In a sensible location, clone the rep.

```bash
git clone https://github.com/pytorch/examples.git
cd examples/fast_neural_style  # change to the example we will be running.
```

### Setup the virtualenv

```bash
module load python/3.8.1  # load python 3
python3 -m venv env  # create a virtualenv folder called env 
source env/bin/activate # activate the virtualenv
```

Now that we've activated the virtual enviroment, we can install the packages we need - torch and torchvision.

```bash
pip3 install torch torchvision
```

### Download some images to use as content as well as for training.

In your ```examples/fast_neural_style/``` directory.

```bash
# Download an image of an octopus to images/content-images.  ## CC BY-SA 3.0 H. Zell
wget https://upload.wikimedia.org/wikipedia/commons/0/0c/Octopus_vulgaris_02.JPG -P images/content-images/ 

# Download an image of The Great Wave off Kanagawa - public domain
wget https://upload.wikimedia.org/wikipedia/commons/a/a5/Tsunami_by_hokusai_19th_century.jpg -O images/style-images/wave.jpg
```

We need to resize the wave image to 70% so that during style training, it will fit into the GPU memory. We will try with the full size image to see the error.

We will also need to download the pre-trained models for our initial inference runs.
```bash
python download_saved_models.py
```

### Style some images - inference

We'll initially just use pretrained models to generate styled images - this is known as model inference and is much less intensive than training the model, we'll do this on both CPU and GPU. 

submit_cpu.sh
```bash
#!/bin/bash

#SBATCH --job-name=pytorch_test
#SBATCH -o _test.out
#SBATCH -e _test.err
#SBATCH --time=00:15:00
#SBATCH --partition=parallel
#SBATCH --constraint="Intel"
#SBATCH --ntasks=12
#SBATCH --mem=6G

module load python/3.8.1
source env/bin/activate  #activate the virtualenv

# Run our job --cuda 0 means run on the CPU and we'll save the output image as test1.jpg
#
python neural_style/neural_style.py eval --content-image images/content-images/Octopus_vulgaris_02.JPG  --model saved_models/mosaic.pth --output-image ./test1.jpg --cuda 0
```

You can check how long the job took to run with ```vuw-job-history```.  The last lines are your last run job, in my case:

```bash
332281        COMPLETED pytorch_t+              00:02:36 
332281.batch  COMPLETED      batch      0.15G   00:02:36 
332281.exte+  COMPLETED     extern      0.15G   00:02:36
```

the job took 2:36.

Let's run the inference job again on GPU to see the speedup.

submit_gpu.sh
```bash
#!/bin/bash

#SBATCH --job-name=pytorch_test
#SBATCH -o _test.out
#SBATCH -e _test.err
#SBATCH --time=00:15:00
#SBATCH --partition=gpu
#SBATCH --gres=gpu:1
#SBATCH --ntasks=2
#SBATCH --mem=20G

module load python/3.8.1
source env/bin/activate  #activate the virtualenv

# Run our job --cuda 1 means run on the GPU and we'll save the output image as test2.jpg
#
python neural_style/neural_style.py eval --content-image images/content-images/Octopus_vulgaris_02.JPG  --model saved_models/mosaic.pth --output-image ./test2.jpg --cuda 1
```

In this case ```vuw-job-history``` the job took:
```bash
332320        COMPLETED pytorch_t+              00:00:50 
332320.batch  COMPLETED      batch      0.15G   00:00:50 
332320.exte+  COMPLETED     extern      0.15G   00:00:50 
```

but the time varies a lot with short GPU runs, some are nearly 2 min long and some runs are 26s with the same data. The memory usage with pytorch is also hard to estimate, running ```vuw-job-report 332320``` shows:
```bash
Nodes: 1
Cores per node: 2
CPU Utilized: 00:00:08
CPU Efficiency: 16.00% of 00:00:50 core-walltime
Job Wall-clock time: 00:00:25
Memory Utilized: 1.38 MB
Memory Efficiency: 0.01% of 20.00 GB
```

The memory usage is very low, but there is a very brief spike in memory at the end of the run as the image is generated that ```vuw-job-report``` doesn't quite capture. 20G of memory is needed to ensure this completes.

### Train a new style - computationally expensive.

Training a new image style is where we will get the greatest speedup using a GPU.

We will use 13G of training images - [COCO 2014 Training images dataset ](http://cocodataset.org/#download). These images have already been downloaded and are accessable at ```/nfs/home/training/neural_style_data/train2014/```.  Note that training a new style will take about two and a half hours.

```bash
#!/bin/bash

#SBATCH --job-name=pytorch_test
#SBATCH -o _test.out
#SBATCH -e _test.err
#SBATCH --time=03:00:00
#SBATCH --partition=gpu
#SBATCH --gres=gpu:1
#SBATCH --ntasks=2
#SBATCH --mem=20G

module load python/3.8.1
source env/bin/activate  #activate the virtualenv

# Run our job --cuda 1 means run on the GPU                                   
#
python neural_style/neural_style.py train --dataset /nfs/home/training/neural_style_data/train2014/ --style-image images/style-images/wave.jpg --save-model-dir saved_models/ --epochs 2 --cuda 1
```