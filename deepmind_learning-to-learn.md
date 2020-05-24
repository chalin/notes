deepmind/learning-to-learn

###    README.md

# [(L)](https://github.com/deepmind/learning-to-learn#learning-to-learn-in-tensorflow)[Learning to Learn](https://arxiv.org/abs/1606.04474) in TensorFlow

Compatible with TensorFlow 1.0

## [(L)](https://github.com/deepmind/learning-to-learn#training)Training

	normalpython train.py --problem=mnist --save_path=./mnist
	normal

Command-line flags:

- ` save_path `: If present, the optimizer will be saved to the specified path every time the evaluation performance is improved.
- ` num_epochs `: Number of training epochs.
- ` log_period `: Epochs before mean performance and time is reported.
- ` evaluation_period `: Epochs before the optimizer is evaluated.
- ` evaluation_epochs `: Number of evaluation epochs.
- ` problem `: Problem to train on. See [Problems](https://github.com/deepmind/learning-to-learn#problems) section below.
- ` num_steps `: Number of optimization steps.
- ` unroll_length `: Number of unroll steps for the optimizer.
- ` learning_rate `: Learning rate.
- ` second_derivatives `: If ` true `, the optimizer will try to compute second derivatives through the loss function specified by the problem.

## [(L)](https://github.com/deepmind/learning-to-learn#evaluation)Evaluation

	normalpython evaluate.py --problem=mnist --optimizer=L2L --path=./mnist
	normal

Command-line flags:

- ` optimizer `: ` Adam ` or ` L2L `.
- ` path `: Path to saved optimizer, only relevant if using the ` L2L ` optimizer.
- ` learning_rate `: Learning rate, only relevant if using ` Adam ` optimizer.
- ` num_epochs `: Number of evaluation epochs.
- ` seed `: Seed for random number generation.
- ` problem `: Problem to evaluate on. See [Problems](https://github.com/deepmind/learning-to-learn#problems) section below.
- ` num_steps `: Number of optimization steps.

## [(L)](https://github.com/deepmind/learning-to-learn#problems)Problems

The training and evaluation scripts support the following problems (see` util.py ` for more details):

- ` simple `: One-variable quadratic function.
- ` simple-multi `: Two-variable quadratic function, where one of the variables is optimized using a learned optimizer and the other one using Adam.
- ` quadratic `: Batched ten-variable quadratic function.
- ` mnist `: Mnist classification using a two-layer fully connected network.
- ` cifar `: Cifar10 classification using a convolutional neural network.
- ` cifar-multi `: Cifar10 classification using a convolutional neural network, where two independent learned optimizers are used. One to optimize parameters from convolutional layers and the other one for parameters from fully connected layers.

New problems can be implemented very easily. You can see in ` train.py ` that the ` meta_minimize ` method from the ` MetaOptimizer ` class is given a function that returns the TensorFlow operation that generates the loss function we want to minimize (see ` problems.py ` for an example).

It's important that all operations with Python side effects (e.g. queue creation) must be done outside of the function passed to ` meta_minimize `. The` cifar10 ` function in ` problems.py ` is a good example of a loss function that uses TensorFlow queues.

Disclaimer: This is not an official Google product.