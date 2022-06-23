---
layout: post
title: "[Paper Review] LipGAN (ACMMM 2019)" 
description: "Prajwal et al., \"Towards Automatic Face-to-Face Translation\" (ACMMM 2019) 논문 리뷰"
date: 2022-05-10
tags: [LipGAN, lip sync, neural talking face, paper review]
comments: true
share: true
---

- Prajwal et al., "Towards Automatic Face-to-Face Translation" (ACMMM 2019) [[arXiv]](https://arxiv.org/abs/2003.00418)[[code]](https://github.com/Rudrabha/LipGAN)[[project page]](http://cvit.iiit.ac.in/research/projects/cvit-projects/facetoface-translation)

--- 

최근에 Wav2Lip demo를 돌려보고 있다. Wav2Lip을 소개하는 글을 작성하면서 LipGAN과 SyncNet까지 함께 언급하려던게 분량이 넘쳐버렸다. 그래서 LipGAN 내용을 따로 떼어내서 소개하기로 했다. 논문에서 automatic speech recognition(ASR), neural machine translation(NMT), text-to-speech(TTS)와 관련된 부분은 과감하게 건너뛰었다.

# Contribution
![pipeline](/assets/posts/lip-sync-synthesis/2022-05-10-review-lipgan/pipeline.png)

이 논문의 저자들은 어떤 대상이 A 언어로 말하는 영상을 입력받아서 같은 대상이 B 언어로 말하는 영상으로 번역해주는 시스템(**face-to-face translation**)을 만들고 싶어했다. 위의 그림에 그 pipeline이 담겨있다. (1) - (4)단계에서는 이미 개발된 speech-to-speech 번역 시스템을 가져다 사용했고, (5)단계에 있는 visual module을 새로 제안했다.

# Architecture
![architecture](/assets/posts/lip-sync-synthesis/2022-05-10-review-lipgan/architecture.png)

## Generator
LipGAN generator는 세 가지 모듈로 이루어져 있다. 
- **Face encoder**는 입 모양을 참고할 face image와 움직임을 참고할 pose image를 함께 입력받아서 embedding으로 바꿔준다. 이때, pose image의 입 모양을 무시하기 위해서 아래쪽 절반을 masking한다. 
- **Audio encoder**는 mel-frequency cepstral coefficient (MFCC) heatmap을 입력받아서 embedding으로 바꿔준다.
- **Face decoder**는 두 encoder에서 뽑아낸 embedding을 함께 전달 받아서 synced face을 예측한다.  

## Discriminator
LipGAN discriminator는 두 가지 모듈로 이루어져 있다. Generator에서와 비슷하게, 여기에 있는 face encoder와 audio encoder도 각각 target MFCC와 generated face를 embedding으로 바꿔준다. 마지막에는 **두 embedding 사이의 L2 distance**가 출력된다.

# Objectives
모델을 학습시킬 때는 총 세 종류의 데이터를 사용한다. 
- Real synced video와 그에 대응하는 audio $(y_i=0)$
- Predicted video와 이때 입력해준 audio $(y_i=1)$
- 의도적으로 sync가 안맞게 짝지은 video와 audio $(y_i=1)$

## Adversarial Loss
GAN에서 자주 보이는 BCE, L1 loss랑은 다르게 생긴 loss를 사용하는 게 특이했다. **Contrastive loss**는 아래 수식처럼 정의된다. $m$은 margin을 의미하는데, $y_i=0$일 때 두 embedding 사이의 L2 distance가 $m$보다 작으면 봐주는(?) 느낌으로 받아들였다.

$$ L_c(d_i, y_i) = \frac {1}{2N} \sum_{i=1}^{N}{[y_i \cdot d_i^2 + (1 - y_i) \cdot \max(0, m - d_i)^2]} $$

Advarsarial loss는 다음과 같다.  

$$ L_{real} = \mathbb{E}_{z, A} [L_c(D(z, A), y)] $$

$$ L_{fake} = \mathbb{E}_{S', A} [L_c(D(G[S'; S_m], A), A), y = 1)] $$

$$ L_a(G, D) = L_{real} + L_{fake} $$

## L1 Reconstruction Loss
그리고 generator 학습을 돕기 위해서 **L1 reconstruction loss**도 함께 사용한다. 

$$ L_{Re}(G) = \frac{1}{N} \sum_{i=1}^{N} \Vert S-G(S', A) \Vert_1 $$

## Final Objective Function
최종 objective는 이렇다.

$$ G^{*} = \arg \min_G \max_D L_a(G, D) + L_{Re} $$

# Experiments
LipGAN으로 face-to-face translation 한 결과이다. 비교 대상이었던 모델들보다 입 주변이 뚜렷하고 입 모양의 움직임이 커보인다. 

![result](/assets/posts/lip-sync-synthesis/2022-05-10-review-lipgan/result.png)

Resolution이 궁금해서 살펴봤더니, LipGAN으로 전달하기 전에 dlib이라는 toolkit을 사용해서 얼굴 영역을 인식(face detection)하고 그 부분을 **96 x 96 x 3**으로 resize했다고 적혀있었다.

# Conclusion
Lip sync의 목적이 face-to-face translation이라는 점이 신선했다!