## Simple tensorflow example (using new module system)


In a sensible location create an example python script - this is basically copied verbatim from the tensorflow docs: ps://www.tensorflow.org/tutorials/quickstart/beginner

example.py
```python
import tensorflow as tf
print("TensorFlow version:", tf.__version__)


# Load and prepare the MNIST dataset. Convert the sample data from integers to floating-point numbers
mnist = tf.keras.datasets.mnist

(x_train, y_train), (x_test, y_test) = mnist.load_data()
x_train, x_test = x_train / 255.0, x_test / 255.0


# Build a machine learning model

model = tf.keras.models.Sequential([
  tf.keras.layers.Flatten(input_shape=(28, 28)),
  tf.keras.layers.Dense(128, activation='relu'),
  tf.keras.layers.Dropout(0.2),
  tf.keras.layers.Dense(10)
])


# The model returns a vector of log-odds scores, one for each class
predictions = model(x_train[:1]).numpy()
predictions

# The tf.nn.softmax function converts these log odds to probabilities for each class

tf.nn.softmax(predictions).numpy()

# Define a loss function for training.
loss_fn = tf.keras.losses.SparseCategoricalCrossentropy(from_logits=True)

# This untrained model gives probabilities close to random 
loss_fn(y_train[:1], predictions).numpy()

# Configure and compile the model using Keras Model.compile
model.compile(optimizer='adam',
              loss=loss_fn,
              metrics=['accuracy'])

# Train and evaluate the model - use Model.fit to adjust parameters and minimize loss
model.fit(x_train, y_train, epochs=5)

# Check model performance
model.evaluate(x_test,  y_test, verbose=2)

# Return a probability - wrap the trained model and attach softmax
probability_model = tf.keras.Sequential([
  model,
  tf.keras.layers.Softmax()
])
probability_model(x_test[:5])
```

Next create a submission script
submit.sh
```bash
#!/bin/bash

#SBATCH --job-name=tensoflow_test
#SBATCH -o _test.out
#SBATCH --time=00:10:00
#SBATCH --partition=gpu
#SBATCH --ntasks=6
#SBATCH --mem=50G
#SBATCH --gres=gpu:1

# Use the new module system
module use /home/software/tools/eb_modulefiles/all/Core

#to load tf 2.6.0 you'll first need the compiler set it was built with
module load foss/2021a

#load tf
module load TensorFlow/2.6.0-CUDA-11.3.1

# Run the simple tensorflow example - taken from the docs: https://www.tensorflow.org/tutorials/quickstart/beginner
python example.py
```

Submit your job to the queue and then observe in the queue
```bash
sbatch submit.sh
squeue -u <username>
```

Possible errors - Tensorflow jobs on the gpu nodes can be a bit dicey
I'd suggest always choosing more memory than the GPU has (40GB) the gpu nodes have a lot of memory so I'd suggest asking for 50GB of ram minimum.

There is also a relationship between cpu's allocated and memory used - the errors are not always obvious.  If you're running into issues try increasing the requested memory or reducing the requested CPUs

Example errors due to requesting many cpus while requesting only 50GB ram
Note std::bad_alloc - this suggests a problem allocating memory
```
terminate called after throwing an instance of 'std::bad_alloc'
  what():  std::bad_alloc
/var/lib/slurm/slurmd/job1125851/slurm_script: line 21: 46983 Aborted                 (core dumped) python example.py
```

Note this example is also in our example git repo: https://github.com/vuw-research-computing/raapoi-examples

In the *tensorflow-simple* directory