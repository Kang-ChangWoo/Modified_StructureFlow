# ModifiedStructureFlow
- 본 저장소는 UNIST DGMS 강의 간 진행한 프로젝트에서, 성능 비교를 위해서 재현한 선행연구입니다. 
- 코드 및 논문은 "[StructureFlow: Image Inpainting via Structure-aware Appearance Flow](https://arxiv.org/abs/1908.03852)" (ICCV 2019) 를 참조합니다.



### 예시 결과물 (Inpainting results)
- 참고를 위한 인페인팅 결과물은 아래와 같습니다.
<p align='center'>  
  <img src='https://user-images.githubusercontent.com/30292465/62820141-8e634300-bb92-11e9-9895-570f020edc47.png' width='500'/>
</p>



### 필요한 라이브러리 (Requirements)

( **RTX 3090-24GB** 환경에서 구동하기 위한 설정입니다. )

1. **Pytorch 1.10.1** (modified from Pytorch >= 1.0)
2. **Python 3.9.5** (modified from Python 3)
3. **NVIDIA GPU + CUDA 11.3** (modified from NVIDIA GPU + CUDA 9.0)

그 외 버전에 영향 없는 라이브러리는 필요 시 설치해주시면 됩니다.

4. Tensorboard
5. Matlab
6. Pandas
7. Numpy



### 설치법 (Installation)

1. 본 저장소를 클론한다.

   ```bash
   git clone https://github.com/Kang-ChangWoo/Modified_StructureFlow.git
   ```

2. 가우시안 샘플링(Gaussian Sampling) 목적의 쿠다(CUDA) 패키지를 빌드한다. 

   (c++ 버전은 기존 c++ 11에서 **c++ 14**로 변경했으며, GPU architecture는 **sm_80, sm_86** 으로 설정했습니다.)

   ```bash
   cd ./StructureFlow/resample2d_package
   python setup.py install --user
   ```



### 구동 (Running)

**1.	이미지 준비하기**

기존 선행 연구는 세 가지 공개 데이터셋에서 학습 됐습니다. (Places2, Celeba, and Paris StreetView) 
또한, 불규칙적 마스킹 데이터는 학습 시에 선행연구 [PConv](https://arxiv.org/abs/1804.07723) 에서 활용되었습니다.
하지만 선행연구 비교를 위해서, 주요하게 아래 두 가지 데이터를 참조하면 됩니다.

1. [CelebA](http://mmlab.ie.cuhk.edu.hk/projects/CelebA.html) 
2. [Irregular Masks](http://masc.cs.gmu.edu/wiki/partialconv)
3. [landmark]
4. [smooth CelebA]

데이터셋을 다운로드 받은 이후에, 각 이미지를 가장자리만 남도록 스무딩한 이미지를 [RTV smooth method](http://www.cse.cuhk.edu.hk/~leojia/projects/texturesep/)를 통해 획득해야 합니다. 내장된 [`scripts/matlab/generate_structre_images.m`](scripts/matlab/generate_structure_images.m) 을 matlab을 통해서 실행 시켜서 원하는 데이터를 획득할 수 있습니다. 만약에 celeba 데이터셋의 스무딩 이미지를 얻고 싶다면 아래처럼 입력해야 합니다. (미리 스무딩한 이미지를 위의 (4)번에 공유하니 해당 링크에서 다운로드 해도 됩니다.):

```matlab
generate_structure_images("path to celeba dataset root", "path to output folder");
```

각 이미지를 얻은 다음에는 [`scripts/flist.py`](scripts/flist.py)를 통해 파일 목록을 생성해 학습 및 테스트에 활용해야 합니다.



**2. 학습 (Training)**

모델 학습을 위해서, 설정 파일인 [model_config.yaml](model_config.yaml)를 수정해야 합니다. 데이터셋 경로나 모델의 하이퍼 파라미터를 수정할 수 있습니다. 그 다음엔 아래 코드를 실행시키면 됩니다:

```bash
python train.py \
--name=[the name of your experiment] \
--path=[path save the results] 
```



**3. 테스트 (Testing)**

입력 데이터에 따른 생성 이미지를 얻고 싶으면, [test.py](test.py) 파일을 실행하면 됩니다.  아래 코드를 실행시키면 됩니다:

```bash
python test.py \
--name=[the name of your experiment] \
--path=[path of your experiments] \
--input=[input images] \
--mask=[mask images] \
--structure=[structure images] \
--output=[path to save the output images] \
--model=[which model to be tested]
```


모델 성능을 평가하기 위해선, [./scripts/matric.py](scripts/metrics.py) 파일을 실행하면 됩니다.  해당 스크립트는 PSNR, SSIM and Fréchet Inception Distance ([FID score](https://github.com/mseitzer/pytorch-fid))을 측정합니다.

```bash
python ./scripts/metrics.py \
--input_path=[path to ground-truth images] \ 
--output_path=[path to model outputs] \
--fid_real_path=[path to the real images using to calculate fid]
```

**사전에 학습된 모델 가충치는 다음에서 다운로드 받을 수 있습니다.**

- [Celeba](https://drive.google.com/open?id=1PrLgcEd964etxZcHIOE93uUONB9-b6pI)
- 

(해당 체크포인트를 다운로드 받은 다음에 './path_of_your_experiments/name_of_your_experiment/checkpoints' 위치에 저장합니다.)

(예를 들어, celeba 체크포인트를 다운 받았다면, '.results/celeba/checkpoints'에 저장한 뒤 아래 코드를 실행하면 됩니다:

```bash
python test.py \
--name=places \
--path=results \
--input=./example/places/1.jpg \
--mask=./example/places/1_mask.png \
--structure=./example/places/1_tsmooth.png \
--output=./result_images \
--model=3
```
