---
layout: post
title: "[Paper Review] HDTF: High-Definition Talking Face Dataset (CVPR 2021)" 
description: "Zhang et al., \"Flow-guided One-shot Talking Face Generation with a High-resolution Audio-visual Dataset\" (CVPR 2021) 논문 리뷰"
date: 2022-06-23
tags: [HDTF, lip sync, neural talking face, 3D morphable model, paper review]
comments: true
share: true
---

- Zhang et al., "Flow-guided One-shot Talking Face Generation with a High-resolution Audio-visual Dataset" (CVPR 2021)  [[paper]](https://openaccess.thecvf.com/content/CVPR2021/papers/Zhang_Flow-Guided_One-Shot_Talking_Face_Generation_With_a_High-Resolution_Audio-Visual_Dataset_CVPR_2021_paper.pdf)[[code]](https://github.com/MRzzm/HDTF)[[demo video]](https://www.youtube.com/watch?v=uJdBgWYBTww)

--- 

![title](/assets/posts/lip-sync-synthesis/2022-06-23-reivew-hdtf/title.png)
![main-figure](/assets/posts/lip-sync-synthesis/2022-06-23-reivew-hdtf/main-figure.png)

고해상도의 audio-visual dataset을 찾던 중에 이 논문을 발견했다. 

# Introduction
**왜 고해상도의 talking face를 합성하기 어려울까?**

**첫째, 적절한 데이터셋이 없기 때문이다.** 아래 표에서 확인할 수 있듯이 audio-visual set은 수집 환경에 따라 in-the-wild 또는 in-the-lab으로 구분할 수 있다. 

![dataset-list](/assets/posts/lip-sync-synthesis/2022-06-23-reivew-hdtf/dataset.png) 

Lip-sync synthesis에 사용되는 in-the-wild 데이터셋들은 화질이 아쉽다. Talking face generation을 목적으로 수집된 것이 아니라서 영상의 화질에 크게 신경쓸 이유가 없었기 때문이다. Voxceleb은 speaker identification, LRW는 word recognition을 목적으로 수집되었다. 한편, in-the-lab 데이터셋들은 서로 다른 identity 및 speech content의 개수가 적어서 아쉽다. MEAD만 하더라도 총 60명의 화자와 159개의 문장이 전부이다.

**둘째, 이전에 제안된 모델 구조로는 고해상도 영상을 합성하는데 한계가 있다.** 최근에 발표된 talking head 모델들은 대부분 2-stage로 된 framework를 따른다. 첫번째 module에서 sparse facial landmark를 생성하면, 두번째 module에서 이로부터 영상을 생성하는 식이다. 어떤 모델들은 두번째 module로 하여금 landmark-to-image mapping을 바로 학습하게 한다. 높은 해상도의 결과물을 기대할수록 이 mapping은 복잡해지기 때문에, 학습에 사용하는 데이터셋을 고해상도로 바꾸어주는 것만으로는 기대하는 결과를 낼 수 없다. 아래 그림의 (f), (g)는 원래 256 x 256 영상을 생성하도록 설계된 MakeItTalk (Zhou, 2020) 모델이 512 x 512 영상을 생성한 결과이다. Baseline인 (e)보다 blurry하다. 

![model-comparison](/assets/posts/lip-sync-synthesis/2022-06-23-reivew-hdtf/model-comparison.png)

또한 sparse landmark는 다소 드물게 정의되어 있어서 충분한 얼굴 표정 정보를 표현하지 못한다. 

## Contributions

**본 논문에서는 언급된 문제점들을 해결하기 위해,**
- YouTube의 고해상도 영상들을 가공하여 직접 audio-visual dataset을 제작했다.
- Facial landmark를 3D morphable model(3DMM)로 대체했다. 3DMM은 얼굴에 대한 사전지식 덕분에 noise에 덜 민감하다. 
- 고해상도 영상을 합성하기에 적합한 flow-guided framework를 제안했다. 

**뿐만 아니라,**
- Style-specific한 animation generator를 제안했다. 즉, referece identity에 따라 같은 speech content를 가지고도 다른 움직임이 합성되게끔 했다. 
- 하나의 animation generator가 multi-task를 수행하도록 설계했다. 이 module은 mouth, eyebrow, head pose와 관련된 animation parameter들을 모두 결정한다.

# Datasets
## HDTF Dataset
데이터셋을 제작하기 위해서 먼저 YouTube에서 전체 15.8 시간 분량의 720P 또는 1080P 영상 362개를 선택했다. 다음으로 landmark detector가 얼굴 영역을 감지하면 그 부분을 crop하고 512 x 512로 다시 resize했다. 아래 그림에서 몇 가지 샘플 데이터를 확인할 수 있다.

![hdtf-samples](/assets/posts/lip-sync-synthesis/2022-06-23-reivew-hdtf/hdtf-samples.png)

## Preprocessing with 3DMM

3DMM을 사용해서 crop된 얼굴 이미지를 facial shape 및 facial animation parameter로 나타냈다. 또한 오디오도 저차원의 feature로 바꿔줬다.  

- **3D facial mesh.** 3D facial mesh point $M(c^s, c^e)$는 다음 식과 같이 정의된다. $M_0$는 facial mesh들의 평균을 나타내고, $V_i^s$ 벡터들은 facial shape $V_j^e$ 벡터들은 facial expression을 나타내기 위해 필요한 linear basis를 의미한다. 33개의 $V_j^e$ 벡터 가운데 28개는 mouth basis이고 5개는 eyebrow basis이다. 

$$ M(c^s, c^e) = M_0 + \sum_{i=1}^{60} c_i^s \cdot V_i^s + \sum_{j=1}^{33} c_j^e \cdot V_j^e $$  

- **Facical shape and animation parameters.** 3D facial mesh로부터 아래 parametere들을 얻었다.
  - Face shape parameter $p^s \in \mathbb{R}^{60}$ (모든 프레임에서 공통)
  - Mouth parameter $p_t^{mou} \in \mathbb{R}^{28}$ (매 프레임마다 계산)
  - Eyebrow parameter $p_t^{ebro} \in \mathbb{R}^{5}$ (매 프레임마다 계산)
  - Head pose parameter $p_t^{hed} \in \mathbb{R}^{5}$ (매 프레임마다 계산)

- **Audio representation.** 13-dim MFCC와 2-dim pitch 정보를 연결해서 15-dim의 audio feature $f_t^{audio} \in \mathbb{R}^{15}$를 얻었다.  

# Proposed Method
이 논문에서 제안한 방법의 pipeline은 아래 그림으로 요약할 수 있다. 두 모듈은 각자 학습된다.
- **Audio-to-animation.** Style-specific animation generator가 reference image와 driving audio를 입력받아서 full animation parameter를 예측한다.
- **Animation-to-video.** 3DMM을 사용해서 전달받은 animation parameter를 approximate dense flow로 변환한다. 이 approximate dense flow는 reference image와 함께 flow-guided video generator로 전달되어 talking face 영상을 합성한다.

![pipeline](/assets/posts/lip-sync-synthesis/2022-06-23-reivew-hdtf/pipeline.png)


## Stage 1. Audio-to-animation
### Audio-to-animation generator $\mathbf{G}^{ani}$
Driving audio로부터 추출한 feature $\tilde{f}^{audio}$에 **AdaIN 연산**을 적용해서 speaker identity가 반영된 $\tilde{f}^{audio}_{ref}$를 얻는다. **각 identity마다 다른 speaking style을 가질 수 있으므로 이를 고려해주기 위함이다.** 뒤이어, 세 갈래로 나뉜 decoder branch에서 각각 mouth, eyebrow, head pose에 대응하는 parameter $p^{mou}$, $p^{ebro}$, $p^{hed}$를 예측한다. Eyebrow 및 head branch는 long-time dependency를 반영할 수 있도록 long-time temporal decoder 뒤쪽에 배치한다.

![G-ani](/assets/posts/lip-sync-synthesis/2022-06-23-reivew-hdtf/G-ani.png)

### Loss function
Mouth synthesis에 관해서는 L1 및 LSGAN loss를 사용한다. 이때, $p_t^{mou}$와 $\hat{p}_t^{mou}$는 각각 GT와 predicted mouth parameter를 의미한다.

$$ \mathcal{L}_1^{mou} = \frac{1}{T} \sum_{t=1}^{T} \Vert{p_t^{mou} - \hat{p}_t^{mou}}\Vert_1 $$

$$ \mathcal{L}_{GAN}^{mou} = \min_{\mathbf{G}^{ani}} \max_{\mathbf{D}^{mou}} \mathcal{L}_{GAN}(\mathbf{G}^{ani}, \mathbf{D}^{mou}) $$

Eyebrow와 head pose generation에는 모두 Structural Similarity(SSIM) loss와 LSGAN loss를 사용한다. $\mu_i$, $\hat{\mu}_i$와 $\sigma_i$, $\hat{\sigma}_i$는 각각 $p^{ebro}$, $\hat{p}^{ebro}$ $i$번째 parameter의 평균과 표준편차를 나타낸다. $\{cov}_i$는 둘 사이의 covariance를 의미한다. 

$$ \mathcal{L}_{ssim}^{ebro} = 1 - \frac{1}{5} \sum_{i=1}^{5} \frac{(2\mu_i\hat{\mu}_i + \delta_1)(2{cov}_i + \delta_2)}{(\mu_i^2 + \hat{\mu}_i^2 + \delta_1)(\sigma_i^2 + \hat{\sigma}_i^2 + \delta_2)}$$

$$ \mathcal{L}_{GAN}^{ebro} = \min_{\mathbf{G}^{ani}} \max_{\mathbf{D}^{ebro}} \mathcal{L}_{GAN}(\mathbf{G}^{ani}, \mathbf{D}^{ebro}) $$

전체 loss function은 아래와 같다. 

$$ \mathcal{L}(\mathbf{G}^{ani}) = \mathcal{L}_{GAN}^{mou} + \mathcal{L}_{GAN}^{ebro} + \mathcal{L}_{GAN}^{hed} +
\lambda_{mou}\mathcal{L}_1^{mou} + \lambda_{ebro}\mathcal{L}_{ssim}^{ebro} + \lambda_{hed}\mathcal{L}_{ssim}^{hed}$$

## Stage 2. Animation-to-video

### Approximate dense motion flow $F^{app}$
아래 (a)는 $F^{app}$의 한 예시이다. **3DMM은 연속한 두 animation parameter set을 가지고, 얼굴 안쪽 영역(아래 (b)의 녹색 영역)에 대한 dense motion flow를 정확하게 만들어낼 수 있다.** 한편 머리에 관련된 영역(아래 (b)의 파란색 영역)과 몸통 위쪽 영역(아래 (b)의 주황색 영역)에 대한 dense motion flow는 예측하는 수 밖에 없다.
머리에 관련된 영역에서는 머리카락과 귀, 장신구 등에 집중했다. 따라서 얼굴 경계 부분의 움직임을 rigid하게 따라간다고 가정했다. 몸통 위쪽 영역은 머리와 함께 움직일 것이므로, 얼굴 안쪽 영역의 평균 움직임을 따라간다고 가정했다. 

![F-app](/assets/posts/lip-sync-synthesis/2022-06-23-reivew-hdtf/F-app.png)

그러나 위에서 설명한 방법으로는 background의 움직임을 고려할 수 없다. 따라서 $F^{app}$에는 불가피하게 오류가 섞여있는데, 이는 $\mathbf{G}^{vid}$에서 모두 바로 잡힌다. 

### Flow-guided video generator $\mathbf{G}^{vid}$

- **Dense motion flow $F^{app}$ 수정.** 저자들은 전체 frame동안 background가 변하지 않는다(static)고 가정했다. 따라서 $\mathbf{G}^{vid}$는 $F^{app}$로부터 실제로 움직이는 부분에 대한 정보만 가져오기 위해서 foreground mask $M^f$를 예측한다. $M^f$는 0에서 1 사이의 값을 가지는 soft mask이다. 결국 accurate dense motion flow $F$는 아래와 같이 정의된다.

$$ F = F^{app} * M^f $$ 

- **Talking face 영상 합성.** Feature map space에서 $\hat{f}^{ref}$를 예측한 다음, 이를 CNN-based decoder에 통과시켜서 synthetic image를 얻는 식이다. $\hat{f}^{ref}$는 아래 식처럼 계산되는데, $f^{ref}$($I^{ref}$에 대한 feature map)와 $g$ 사이의 균형이 $M^m$에 의해 조절된다. 이때, $F(\cdot)$는 $F$를 따라 warp하는 연산을 나타낸다. 

$$ \hat{f}^{ref} = F(f^{ref}) * M^m + g * (1 - M^m) $$

![G-vid](/assets/posts/lip-sync-synthesis/2022-06-23-reivew-hdtf/G-vid.png)

아래 그림은 $\mathbf{G}^{vid}$ module의 intermediate output을 보여준다. 

![G-vid-intermediate](/assets/posts/lip-sync-synthesis/2022-06-23-reivew-hdtf/G-vid-intermediate.png)

### Loss function

$\mathbf{G}^{vid}$를 학습시키기 위해서 LSGAN, perceptual 및 feature matching loss를 사용한다. $N(\cdot)$은 VGG-19 네트워크의 $i$번째 layer, $\mathbf{D}_j^{vid}$는 $\mathbf{D}^{vid}$의 $j$번째 layer를 가리킨다. 

$$ \mathcal{L}_{GAN}^{vid} = \min_{\mathbf{G}^{vid}} \max_{\mathbf{D}^{vid}} \mathcal{L}_{GAN}(\mathbf{G}^{vid}, \mathbf{D}^{vid}) $$

$$ \mathcal{L}_{perc}^{vid} = \sum_{i=1}^{n} \frac{1}{W_iH_iC_i} \Vert{N_i(I_t) - N_i(\hat{I}_t)}\Vert_1 $$

$$ \mathcal{L}_{FM}^{vid} = \sum_{j=1}^{n} \frac{1}{W_jH_jC_j} \Vert{\mathbf{D}_j^{vid}(I_t) - \mathbf{D}_j^{vid}(\hat{I}_t)}\Vert_1 $$

전체 loss function은 아래와 같다. 

$$ \mathcal{L}(\mathbf{G}^{vid}) = L_{GAN}^{vid} + \lambda_{perc} \mathcal{L}_{perc}^{vid} + \lambda_{FM} \mathcal{L}_{FM}^{vid} $$

# Experiments
## Style-specificity

Driving audio는 고정해두고, reference image를 달리해가며 animation parameter를 얻었다. **Identity에 따라 eyebrow, head rotation/translation parameter가 아주 다르게 나타나는 것으로부터 style-specificity를 확인할 수 있었다.** 한편, 세 identity가 거의 비슷한 mouth parameter를 공유했다. 이는 입 모양은 style보다는 speech content에 의한 영향을 더 많이 받기 때문이다.  

![animation-parameters](/assets/posts/lip-sync-synthesis/2022-06-23-reivew-hdtf/animation-parameters.png)

## Model comparison

아래 그림의 (a), (b), (c)는 모두 128 x 128 저해상도 결과물이고, (d)는 Wav2Lip으로 합성한 512 x 512 결과물이다. 한 눈에 보기에도 이전에 발표된 모델 가운데 이 모델의 결과 영상(Ours)에 견줄만한 것은 (d) 하나 밖에 없다. 그러나 Wav2Lip은 얼굴 윗쪽 절반에 대해서는 아무런 변화를 줄 수 없다는 분명한 한계가 있다. (Application 관점에서 생각해봤을 때, 나는 사실 이 특징이 한계라고 생각하지 않는다. 불완전하게 합성된 head motion 및 facial expression은 오히려 더 거부감을 유발한다.) 
![model-comparison](/assets/posts/lip-sync-synthesis/2022-06-23-reivew-hdtf/model-comparison.png)

또한 함께 진행된 user study에서도 이 모델이 가장 높은 점수를 받았다. 

## Submodule evaluation
- **Audio-to-animation.** 아래 표는 **각 모델이 landmark 또는 animation parameter를 얼마나 정확하게 예측**하는지 정량적인 metric을 가지고 평가한 결과이다. Face reenactment 모델들을 비교 대상으로 선택했는데, 아무래도 이 모델이 얼굴 안쪽 영역에 특화되어 있다보니 예측 성능이 가장 좋았다. 그리고 밑에서부터 두번째 및 세번째 행은 ablation study 결과를 보여준다. Style-specificity를 더해주는 AdaIN 연산을 생략하거나 각 부위에 대한 animation parameter를 별개로 합성한 경우보다 현재 모델이 더 좋은 결과를 보였다.   

![submodule-evaluation-1](/assets/posts/lip-sync-synthesis/2022-06-23-reivew-hdtf/submodule-evaluation-1.png)

- **Animation-to-video.** 아래 표와 이미지는 **최종 영상의 visual quality**를 평가한 결과이다. 처음부터 고해상도를 염두에 두고 설계된 모델이었기 때문에 이번에도 이 모델이 합성한 영상이 가장 품질이 좋았다. 마찬가지로 여기서도 ablation study 결과가 함께 제시되어 있는데, **static background 가정과 matting 기법을 사용한 것이 품질 향상에 도움이 되었다는 것을 알 수 있다**. 

![submodule-evaluation-2](/assets/posts/lip-sync-synthesis/2022-06-23-reivew-hdtf/submodule-evaluation-2.png)
![submodule-evaluation-3](/assets/posts/lip-sync-synthesis/2022-06-23-reivew-hdtf/submodule-evaluation-3.png)


# Conclusion
- 고해상도 audio-visual dataset을 공개했다. 개인적으로 하고 있는 실험에 이를 사용하려 했는데 official implementation이 굉장히 불친절하다. 결국 AVSpeech dataset을 대신 사용했다.
- 새로운 flow-guided frameworkf를 제안했다. 하나의 오디오에 대해서 다양한 animation이 만들어진다는 점에서 style-specificity를 고려해준다는 아이디어가 좋아보였다.
- 현재는 예측된 dense motion flow의 temporal coherency가 고려되지 않고 있다. 이로 인해서 합성된 고해상도 영상에서 flicker가 관찰되기도 한다.  
- 3DMM에 대해 아는 상태였다면 이 논문을 좀 더 분석적으로 읽을 수 있었을 것 같다. Landmark에 비해서 animation parameter가 정말 강력한지 한 번 고민해볼 생각이다.