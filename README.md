# ModifiedStructureFlow
- 본 저장소는 UNIST DGMS 강의 간 진행한 프로젝트에서, 성능 비교를 위해서 재현한 선행연구입니다. 
- 코드 및 논문은 "[StructureFlow: Image Inpainting via Structure-aware Appearance Flow](https://arxiv.org/abs/1908.03852)" (ICCV 2019) 를 참조합니다.
- 원 문서에 대한 설명은 [`origin_repo/README.md`](origin_repo/README.md) 파일에 저장되어 있다.


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

3. 앞서 언급한, 필요한 라이브러리를 설치한다.


### 구동 (Running)

**1.	이미지 준비하기**

기존 선행 연구는 세 가지 공개 데이터셋에서 학습 됐습니다. (Places2, Celeba, and Paris StreetView)  하지만, 본 프로젝트에서는 하단 데이터셋만을 활용하여 테스트하고자 합니다.  필요한 데이터는 `evaluation_dataset/input/` 위치에 저장되어 있습니다. 미리 스무딩한 이미지도 `evaluation_dataset/pred_structure/`에 저장되어 있으니 그대로 사용하면 됩니다.  (해당 방식은 각 이미지를 가장자리만 남도록 스무딩한 이미지를 [RTV smooth method](http://www.cse.cuhk.edu.hk/~leojia/projects/texturesep/)를 통해 획득했으며, 내장된 [`scripts/matlab/generate_structre_images.m`](scripts/matlab/generate_structure_images.m) 을 matlab을 통해서 실행 시켜서 획득했습니다.)

- [DGSM 평가 데이터](evaluation_dataset/input/000.png)


각 이미지를 얻은 다음에는 [`scripts/flist.py`](scripts/flist.py)를 통해 파일 목록을 생성해 학습 및 테스트에 활용해야 합니다.  각 파일을 생성한 다음엔 `flist/`경로에 각각 저장해주어야 합니다.



**2. 테스트 (Testing)**
사전에 학습된 모델 가중치는 다음에서 다운로드 받을 수 있습니다. 

- [Celeba](https://drive.google.com/open?id=1PrLgcEd964etxZcHIOE93uUONB9-b6pI)


해당 체크포인트를 다운로드 받은 다음에 './path_of_your_experiments/name_of_your_experiment/checkpoints' 위치에 저장합니다.  예를 들어, celeba 체크포인트를 다운 받았다면, '.results/celeba/checkpoints'에 저장한 뒤 아래 코드를 실행하면 됩니다:

```bash
python test.py \
--name=celeba \
--path=results \
--input=./evaluation_dataset/input/ \
--mask=./evaluation_dataset/masks/ \
--structure=./evaluation_dataset/pred_structure/ \
--output=./results/celeba/result/ \
--model=3
```


또한 `config.yaml`파일의 최하단 경로 다섯 개를 생성한 .flist의 경로에 맞게 수정해주어야 합니다.
- DATA_VAL_GT
- DATA_TRAIN_GT
- DATA_TRAIN_STRUCTURE
- DATA_TRAIN_GT
- DATA_TRAIN_STRUCTURE
