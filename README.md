# CANU-MMT: A Conditional Adversarial Network for Unsupervised person Re-IDentification for MMT 
Implementation of [CANU-ReID: A Conditional Adversarial Network for Unsupervised person Re-IDentification](https://arxiv.org/abs/1904.01308), ICPR 2020 (Oral),
based on [Mutual Mean-Teaching: Pseudo Label Refinery for Unsupervised Domain Adaptation on Person Re-identification](https://openreview.net/forum?id=rJlnOhVYPS).

![Illustration of CANU-ReID.](pipeline.png)
## Running the experiments

### Setup data and weights
Please refer to the original [implementation of MMT](https://github.com/yxgeee/MMT) for setup details.
We use the same pretrained models as provided by MMT.

### Singularity Container
We provide the [singularity recipe](https://singularity.lbl.gov/docs-build-container#building-containers-from-singularity-recipe-files)
 to build the container used for those experiments singularity/recipe.txt

Once built, the container can be launched interactively with :
```shell
singularity shell --nv --bind /HOST_PATH/:/CONTAINER_PATH/ pytorchcuda100.sif
```
### Train
We use 1 or 2 GTX-TITAN X for training (instead of 4 as in the original MMT), and divide the batch in 4 to do 4 independent forwards in order to cope with BN.

training command for CANU-MMT and Market1501 to DukeMTMC:
```shell
python examples/mmt_train_dbscan_adcam.py \
	-ds market1501\
	-dt dukemtmc\
	-a resnet_ibn50a\
	--num-instances 4\
	--lr 0.00035\
	--iters 400\
	-b 64 --epochs 40\
	--soft-ce-weight 0.5\ 
	--soft-tri-weight 0.8\
	--dropout 0\ 
	--lambda-value 0\
	--init-1 logs/pretrained_MMT/m2d_resnet_ibn50a_pretrained_1.tar\
	--init-2 logs/pretrained_MMT/m2d_resnet_ibn50a_pretrained_2.tar\
	--logs-dir logs/EXP_NAME\
	--ncamera 8\
	--mu 0.1\
	--data-dir=data\
	--print-freq 100\
	--cond_gan\
	--multi_disc\
	--centroid\
	--nosampler
```
To run baseline (MMT only) experiment, set mu to 0.0.
Dropping --cond_gan removes the conditioning of the camera discriminator (Table IV: MMT+Adv).

To run experiments with msmt17 as target, we drop the --nosampler argument.

### Evaluate
To evaluate the weights on tgt dataset (here dukemtmc):
```shell
python examples/mmt_train_dbscan_adcam.py \
	-ds market1501\
	-dt dukemtmc\
	-a resnet_ibn50a\
	--num-instances 4\
	--lr 0.00035\
	--init-1 logs/pretrained_MMT/m2d_resnet_ibn50a_pretrained_1.tar\
	--init-2 logs/pretrained_MMT/m2d_resnet_ibn50a_pretrained_2.tar\
	--logs-dir DIR_PATH_OF_MODEL_TO_EVAL\
	--evaluate
```
## Results

Weights are available [here](http://perception.inrialpes.fr/Free_Access_Data/DELORME_ICPR2020/CANU_MMT.zip).
<!-- markdownlint-disable MD033 -->
<table>
    <tr>
        <th rowspan="2">SRC --&gt; TGT</th>
        <th colspan="2">Adaptation by MMT(dbscan)</th>
        <th colspan="2">Adaptation by Adv+MMT</th>
        <th colspan="2">Adaptation by CANU+MMT</th>
    </tr>
    <tr>
        <td>Rank-1</td>
        <td>mAP</td>
        <td>Rank-1</td>
        <td>mAP</td>
        <td>Rank-1</td>
        <td>mAP</td>
    </tr>
    <tr><td>Market1501 --&gt; DukeMTMC</td><td> 80.2</td><td>67.2</td><td>82.6</td><td>70.3</td><td>83.3</td><td>70.3</td></tr>
    <tr><td>DukeMTMC --&gt; Market1501</td><td>91.7</td><td>79.3</td><td>93.6</td><td>82.2</td><td>94.2</td><td>83.0</td></tr>
    <tr><td>Market1501 --&gt; MSMT17 </td><td>51.6</td><td>26.6</td><td>-</td><td>-</td><td>61.7</td><td>34.6</td></tr>
    <tr><td>DukeMTMC --&gt; MSMT17 </td><td>59.0</td><td>32.0</td><td>-</td><td>-</td><td>66.9</td><td>38.3</td></tr>

</table>

## Acknowledgement

Our code is based on [MMT](https://github.com/yxgeee/MMT)
