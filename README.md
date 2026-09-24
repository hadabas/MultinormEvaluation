# Multinorm Evaluation
A repository created in order to provide an environment to test trained neural networks against multiple types of attacks, and get certain statistics from it. You can easily expand and modify the code for it to be able to load your own models, and then test it against multiple attacks.

# 1. Prerequisites
To run the evaluation script, you will need a python 3.11 or newer environment. You also need to install the following python packages in said environment: `torch`, `torchvision`, `robustbench`.
In addition, you also need to install the 'autoattack' package: `pip install git+https://github.com/fra31/auto-attack`

# 2. Running an evaluation

First, copy the repository. Then simply run `eval_all_RAMP.py` for the evaluation to start in an environment that has all the prerequisites. The following switch arguments can be used:
```
--model_name <PATH_TO_MODEL> : The path to the model you want to evaluate. Can be relative and absolute.
--data_dir <PATH_TO_DATABASE> : The path to the database you want to test it on, specified by *--dataset*. Can be relative and absolute. Also, if it does not find the database on said path, the script will automatically attempt to download it.
--dataset <cifar10/cifar100/imagenet> : The selected database. Currently, only three are supported.
--run_border : *OPTIONAL* It will run the attack called 'borderattack', and evaluates it.
--run_border_inner: *OPTIONAL* It will run the attack called 'borderattack_inner' and evaluates it.
```
The first three are the most important, and though you could customize the parameters of each individual attack in switch arguments, instead it is strongly recommended to overwrite the default values in the script code from line 159 to line 204 if you desire to do so.

# 3. The attacks

The first part of the evaluation script uses standard autoattack evaluation, as it is described [here](https://github.com/fra31/auto-attack). The default perturbation values of each attack can be read (and overwritten) from line 23 to line 27 for each supported database. These attacks use Linf, L2, and L1 norms as its perturbation metric, and all of them are combined attacks.

The second part of the evaluation script will only run if one (or both) of the optional switches were used. It will evaluate the network's performance on said attacks. More information on these attacks can be found [here](https://github.com/szegedai/BorderAttack).

The last part of the evaluation script will run a combined attack that uses the L0 norm as its perturbation metric, as it is described [here](https://github.com/CityU-MLO/sPGD).

As a result, we get a very thorough check on the model's multinorm performance by running *six* different type of attacks on it.

## Metrics used

After each part, the script automatically calculates the 'union accuracy' (or robust accuracy) of the model using a bitmask. Depending on the database, the bitmask's size is the same as the used database's test pool size (for example, on cifar10 its 10000). Its working principle is very simple, they represent image indices.

For example, if under *ANY* attack, the image on the first index fails to be classified in the correct class, it will permanently be changed from a 1 to a 0. This will not be reset after the first attack, it will be carried over for the next attack until the script ends and there are no more attacks remaining. Before the script ends, it will check how many 1's remain, and will calculate the union accuracy based on that.

This is what gives us our most important metric, since the only 1's remaining in the bitmask represent that neither of the six attacks could fool your network with their perturbations to misclassify the image on said index.

In addition, the script also calculates how well your model did on each attack alone, regardless of all the previous other attacks.

Also, there are partial results recorded for union/robust accuracy after each type of attack, this can be read at the end of the result under "UNION SUMMARY". With these results, you can check which type of attack decreased your model's union accuracy the most.

# 4. An example

Since any model can be loaded which is torch compatible, in the example script we have used a famous model that is Linf, L2 and L1 robust at once. (Or as it is described in their [paper](https://arxiv.org/abs/2402.06827), it tries to maximize union accuracy across these 3 attacks.)

I have trained a model based on their instructions in their [GitHub codebase](https://github.com/uiuc-focal-lab/RAMP), then after the training was done, I have used the following command:

`python ./eval_all_RAMP.py --model_name RAMP_model_ep_80_0.pth -data_dir ../Databases/cifar10 --dataset cifar10 --run_border --run_border_inner`

to evaluate the trained model.

You can check the results in the ramp_result.log file.
