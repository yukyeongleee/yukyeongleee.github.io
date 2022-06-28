---
layout: post
title: "[Demo] PC-AVS (CVPR 2021)로 lip sync video 합성"
description: "PC-AVS (CVPR 2021) official implementation을 사용해서 lip sync video 합성"
date: 2022-05-24
tags: [PC-AVS, pose controllable, lip sync, neural talking face, stylegan2, paper review]
comments: true
share: true
---

- Zhou et al., "Pose-Controllable Talking Face Generation by Implicitly Modularized Audio-Visual Representation" (CVPR 2021)  [[arXiv]](https://arxiv.org/abs/2104.11116)[[code]](https://github.com/Hangz-nju-cuhk/Talking-Face_PC-AVS)[[project page]](https://hangz-nju-cuhk.github.io/projects/PC-AVS)

--- 

Pose Controllable Audio-Visual System (PC-AVS) official implementation을 사용해서 lip sync video를 합성했다. 과정, 결과 및 결과에 대한 내 생각을 공유하려고 한다. 

# 데이터 준비
Input으로 사용할 이미지는 [This Person Does Not Exist](https://this-person-does-not-exist.com/en) 사이트에서 가져왔다. 그리고 [Toonify](https://toonify.photos/) 사이트에서 이 이미지를 만화 주인공같이 바꾼 이미지를 얻었다. Mouth source 및 pose source로 사용할 영상은 VoxCeleb2 dataset에서 가져왔다. 

# 튜토리얼
1. `scripts/prepare_testing_files.py` 파일을 실행한다. 
- `ffmpeg` 커맨드로 입력받은 영상의 각 frame을 추출하고, mouth source(.mp4)로부터 audio source(.mp3)를 추출한다. PC-AVS는 mouth source가 아닌 audio source를 입력받아서 audio-driven lip motion을 합성한다. Mouth source는 마지막에 lip sync가 제대로 합성되었는지 확인할 때 GT처럼 사용한다.
- Audio source를 spectrogram으로 변환한다. 오디오를 처리하는 함수들은 `AudioConfig` 클래스에서 찾아볼 수 있다. 
- PC-AVS로 전달할 파일들의 경로와 길이가 기록된 demo.csv 파일을 생성한다.  

2. (선택사항) `scripts/align_68.py` 파일을 실행한다. 이때, `--folder_path` argument로 VoxCeleb2가 아닌 데이터의 경로를 전달해준다. 
- **Pretrained PC-AVS 모델은 VoxCeleb2처럼 crop 된 사진/영상만 다룰 수 있기 때문에, 내가 원하는 데이터를 입력으로 사용하려면 전처리가 필요하다.**
- 자세한 전처리 방법은 Appendix에 코드와 함께 소개해뒀다.


3. (선택사항) 전처리를 한 경우에는 demo.csv 파일을 수정해야한다. 전처리한 데이터는 파일 경로 뒤에 '_cropped'를 추가한다. 

4. `experiments/demo_vox.sh` 파일을 실행한다. 새로 생성된 결과 폴더의 avconcat.mp4를 확인해보자.

# 데모 샘플
왼쪽에서부터 차례대로 (VoxCeleb2-like cropped) input, generated, pose source, mouth source이다. 처음 두 샘플은 input으로 This Person Does Not Exist 사이트에서 다운받은 이미지를 그대로 사용했고, 다른 두 샘플은 toonify 한 다음 사용했다. 

**샘플 A.** 
![result-1](/assets/posts/lip-sync-synthesis/2022-05-24-demo-pc-avs/result-1.gif)
**샘플 B.**
![result-2](/assets/posts/lip-sync-synthesis/2022-05-24-demo-pc-avs/result-2.gif)
**샘플 C.**
![result-3](/assets/posts/lip-sync-synthesis/2022-05-24-demo-pc-avs/result-3.gif)
**샘플 D.**
![result-4](/assets/posts/lip-sync-synthesis/2022-05-24-demo-pc-avs/result-4.gif)

# 고찰
- Lip sync 결과가 완전 만족스럽다. Wav2Lip으로 합성했던 샘플은 입 모양이 맞는건지 긴가민가했는데 이건 확실하게 맞는 것 같다.
- **합성된 샘플에 StyleGAN2를 학습시킬 때 사용한 데이터셋의 평균 identity가 반영되는 것 같다.** 즉, input의 identity가 유지되지 않는다. 남자 어린이 사진을 input으로 줬을 때는 좀 더 조숙해보이는 샘플(B)이 합성되었고, toonify 한 사진을 input으로 줬음에도 실존 인물같은 샘플(C, D)이 합성되었다.  
- VoxCeleb2처럼 crop하는 과정에서 머리 윗부분이 잘리는 게 아쉽다. 
- 합성된 샘플이 눈을 깜빡이지 않아서 부자연스럽게 느껴진다. 


# Appendix
## 데이터 전처리 방법
사람 얼굴을 포함한 사진/영상의 각 frame마다 68개 facial landmark를 추출한다. 이 부분은 직접 구현하지 않고 오픈소스 [face_alignment](https://github.com/1adrianb/face-alignment) 모델을 사용했다.  

```python
# Line 48 - 49
fa = face_alignment.FaceAlignment(face_alignment.LandmarksType._2D, device=device)
preds = fa.get_landmarks_from_directory(folder_path)
```

특정 frame에 등장하는 얼굴이 하나뿐이고 68개 facial landmark를 모두 얻었다면, `get_eyes_mouth` 함수를 호출해서 **두 눈과 입의 위치**를 찾는다. Brandon Amos가 그린 아래 그림으로부터 36 - 41번 landmark는 오른쪽 눈, 42 - 47번 landmark는 왼쪽 눈, 60 - 67번 landmark는 입 내부를 나타내는 걸 확인할 수 있다. 

```python
# Line 33 - 38
def get_eyes_mouths(landmark):
    three_points = np.zeros((3, 2))
    three_points[0] = landmark[36:42].mean(0)
    three_points[1] = landmark[42:48].mean(0)
    three_points[2] = landmark[60:68].mean(0)
    return three_points
```

![face-landmarks](/assets/posts/lip-sync-synthesis/2022-05-24-demo-pc-avs/face-landmarks.png)

전체 영상에서 두 눈과 입의 평균 위치를 계산한다. 그리고 `get_affine` 함수를 호출해서 이 평균 위치를 (87, 59), (137, 59), (112, 120)로 옮기는 Affine transform `M`을 근사한다.  

```python
# Line 12 -19
def get_affine(src):
    dst = np.array([[87,  59],
                    [137,  59],
                    [112, 120]], dtype=np.float32)
    tform = trans.SimilarityTransform()
    tform.estimate(src, dst)
    M = tform.params[0:2, :]
    return M
```

`M`을 사용해서 각 frame을 warp한 다음 원하는 크기로 crop하면 끝이다. `cv2.warpAffine` 함수는 처음 들어봤는데, 이걸 사용해서 한 방에 해결했다. 자세하게는 `bias`를 정의해서 입의 위치를 살짝 조정해주긴 해야한다. 
```python
# Line 22 - 24
def affine_align_img(img, M, crop_size=224):
    warped = cv2.warpAffine(img, M, (crop_size, crop_size), borderValue=0.0)
    return warped
```