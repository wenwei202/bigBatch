# Train longer, generalize better - Big batch training
## Usage

```
python evaluate.py --model alexnet --dataset imagenet -b 100 --gpus 1 --evaluate TrainingResults/2018-03-17_11-28-33/model_best.pth.tar --ss 0.006 --mode train --no-augment
```

```
python visualize_sharpness.py --model resnet --dataset cifar10 -b 100 --gpus 1 --evaluate TrainingResults/2018-02-20_19-47-52/model_best.pth.tar  --slave-evaluate TrainingResults/2018-02-20_19-47-18/model_best.pth.tar  --no-visualize_train
```

```
python visualize2_sharpness.py --model resnet --dataset cifar10 -b 100 --gpus 1 --evaluate TrainingResults/2018-02-20_19-47-18/model_best.pth.tar  --no-visualize_train --alpha="-0.3:0.02:0.31"
```

```
python measure_sharpness.py --model cifar100_shallow --dataset cifar100 --b 100 --gpus 1 --evaluate TrainingResults/2018-02-20_14-05-28/model_best.pth.tar
```

This is a code repository used to generate the results appearing in ["Train longer, generalize better: closing the generalization gap in large batch training of neural networks"](https://arxiv.org/abs/1705.08741) By Elad Hoffer, Itay Hubara and Daniel Soudry.

It is based off [convNet.pytorch](https://github.com/eladhoffer/convNet.pytorch) with some helpful options such as:
  - Training on several datasets
  - Complete logging of trained experiment
  - Graph visualization of the training/validation loss and accuracy
  - Definition of preprocessing and optimization regime for each model

## Dependencies

- [pytorch](<http://www.pytorch.org>)
- [torchvision](<https://github.com/pytorch/vision>) to load the datasets, perform image transforms
- [pandas](<http://pandas.pydata.org/>) for logging to csv
- [bokeh](<http://bokeh.pydata.org>) for training visualization `conda install bokeh=0.12.0`


## Data
- Configure your dataset path at **data.py**.
- To get the ILSVRC data, you should register on their site for access: <http://www.image-net.org/> and
```
mkdir -p pytorch-gen/imagenet
cd pytorch-gen/imagenet
ln -s ${YOUR_IMAGENET_PATH}/train/ train
ln -s ${YOUR_IMAGENET_PATH}/validation/ val
```

## Experiment examples
```bash
python main_normal.py --dataset cifar10 --model resnet --save cifar10_resnet44_bs2048_lr_fix --epochs 100 --b 2048 --lr_bb_fix;
python main_normal.py --dataset cifar10 --model resnet --save cifar10_resnet44_bs2048_regime_adaptation --epochs 100 --b 2048 --lr_bb_fix --regime_bb_fix;
python main_gbn.py --dataset cifar10 --model resnet --save cifar10_resnet44_bs2048_ghost_bn256 --epochs 100 --b 2048 --lr_bb_fix --mini-batch-size 256;
python main_normal.py --dataset cifar100 --model resnet --save cifar100_wresnet16_4_bs1024_regime_adaptation --epochs 100 --b 1024 --lr_bb_fix --regime_bb_fix;
python main_gbn.py --model mnist_f1 --dataset mnist --save mnist_baseline_bs4096_gbn --epochs 50 --b 4096 --lr_bb_fix --no-regime_bb_fix --mini-batch-size 128;
```
- See *run_experiments.sh* for more examples
## Model configuration

Network model is defined by writing a <modelname>.py file in <code>models</code> folder, and selecting it using the <code>model</code> flag. Model function must be registered in <code>models/\_\_init\_\_.py</code>
The model function must return a trainable network. It can also specify additional training options such optimization regime (either a dictionary or a function), and input transform modifications.

e.g for a model definition:

```python
class Model(nn.Module):

    def __init__(self, num_classes=1000):
        super(Model, self).__init__()
        self.model = nn.Sequential(...)

        self.regime = {
            0: {'optimizer': 'SGD', 'lr': 1e-2,
                'weight_decay': 5e-4, 'momentum': 0.9},
            15: {'lr': 1e-3, 'weight_decay': 0}
        }

        self.input_transform = {
            'train': transforms.Compose([...]),
            'eval': transforms.Compose([...])
        }
    def forward(self, inputs):
        return self.model(inputs)

 def model(**kwargs):
        return Model()
```
