# EgoVLP: Egocentric Video-Language Pretraining

[project page](https://qinghonglin.github.io/EgoVLP/) | [arXiv](https://arxiv.org/pdf/2206.01670.pdf)


<img src="/figures/egovlp_framework.jpg" alt="EgoVLP" style="zoom:67%;" />

**TL;DR:** We pioneer Egocentric Video-Language Pretraining from pretraining dataset, model and development benchmark; the resulted pretrained model exhibits strong performance on six downstream tasks across three egocentric datasets.

## 📢 News

- [2022.6.3] We release the arXiv paper.
- [2022.6.10] We release the EgoClip pretraining dataset.
- [2022.6.20] Our EgoVLP won [**1st place** in OSCC](https://eval.ai/web/challenges/challenge-page/1627/overview) & [**2nd place** in NLQ](https://eval.ai/web/challenges/challenge-page/1629/overview) & [**3rd place** in PNR](https://eval.ai/web/challenges/challenge-page/1622/overview) @ [Ego4D  Challenge 2022](https://ego4d-data.org/docs/challenge/), and [**1st place** in Multi-Instance Retrieval](https://codalab.lisn.upsaclay.fr/competitions/617#learn_the_details) @ [EPIC-Kitchens Challenge 2022](https://epic-kitchens.github.io/2022), hosted by CVPR 2022.
- [2022.6.30] We release the first version of the EgoVLP codebase.

## 📝 Preparation
> You may skip this step if pretraining is not required.
### Ego4D videos and metadata
1. Follow the guideline [here](https://ego4d-data.org/docs/start-here/#cli-download), download the following to  `{PATH_TO_EGO4D}`
   - Ego4D source videos (nearly 7 TB).
   - Ego4D videos metadata `manifest.csv` and benchmark metadata, e.g., `nlq_train.json` for NLQ.
   - Create the dir `./dataset` and add a soft link by `ln -s {PATH_TO_EGO4D} ./dataset/ego4d`.

2. For effectively pretraining, we compress videos in the following way:
   - Resize the source videos with a short size equal to 256 by script  `./utils/video_resize.py`.
   - Chunk the resized videos to multiple segments (up to 600 sec) by script `./utils/video_chunk.py`.

### EgoClip
- Download the EgoClip metadata from [here](https://drive.google.com/file/d/1-aaDu_Gi-Y2sQI_2rsI2D1zvQBJnHpXl/view?usp=sharing) and put it to `./dataset/egoclip.csv`.

- For the usage of EgoClip, please refer to `./data_loader/EgoClip_EgoMCQ_dataset.py`. The data format of EgoClip is:
  ```python
  import pandas as pd
  
  metadata = pd.read_csv('./dataset/egoclip_metadata.csv', sep='\t', error_bad_lines=False)
  print(metadata.shape[0])
  print(metadata.iloc[0])
  
  # Out:
  3847723                                                         # Num of clips for EgoClip
  
  clip_idx                                                     0  # the idx of clip
  video_uid                 001e3e4e-2743-47fc-8564-d5efd11f9e90  # the uid of source video
  video_dur                                           128.033333  # the duration of source video
  narration_source                              narration_pass_1  # the source of annotator
  narration_ind                                                0  # the idx of narration
  narration_time                                          3.3445  # the narration timestamp
  clip_start                                            2.967651  # the start timestamp of clip
  clip_end                                              3.721266  # the end timestamp of clip
  clip_text           #C C picks a bag of clothes from the floor  # the narration of clip
  tag_verb                                                  [93]  # the verb idx of the narration
  tag_noun                                        [192, 115, 12]  # the noun idx of the narration
  ```

### EgoMCQ

- Download the EgoMCQ metadata from [here](https://drive.google.com/file/d/1-5iRYf4BCHmj4MYQYFRMY4bhsWJUN3rW/view?usp=sharing) and put it to `./dataset/egomcq.json`.
- For the usage of EgoMCQ, please refer to `./data_loader/EgoClip_EgoMCQ_dataset.py`.

## 🏋️‍️ Pretraining
- We pretrain EgoVLP on 4 nodes, each with 8 A100 GPUs (10 epochs in about two days).

`python3 -m torch.distributed.launch 
  --nnodes=$HOST_NUM 
  --node_rank=$INDEX 
  --master_addr $CHIEF_IP 
  --nproc_per_node $HOST_GPU_NUM 
  --master_port 8081 
  ./run/train_egoclip.py --config ./configs/pt/egoclip.json`
  
- You can monitor the EgoMCQ performance of the VLP model by tensorboard:

`tensorboard --logdir ./results  --bind_all`
