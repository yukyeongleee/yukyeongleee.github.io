---
layout: post
title: "[Linux Server] Anaconda 설치 및 가상환경 관리" 
description: "Linux Server에 Ancaconda 설치 및 가상환경 관리하는 방법 소개"
date: 2022-05-08
tags: [anaconda, linux server, how to]
comments: false
share: true
---

# Anaconda 설치
1. [Anaconda Installers](https://www.anaconda.com/products/distribution#Downloads)에서 Linux 최신 release에 커서를 놓고 우클릭 > [Copy Link Address]를 선택한다.
![release](/assets/posts/how-to/2022-05-08-anaconda-install/release.png)

2. 원하는 디렉토리로 이동한 다음, .sh 파일(1에서 복사한 주소)을 다운로드한다.
```bash
curl -O {Copied Address}
# eg. curl -O https://repo.anaconda.com/archive/Anaconda3-2021.11-Linux-x86_64.sh
```

3. .sh 파일을 실행한다. 
```bash
bash {Anaconda filename}.sh
# eg. bash Anaconda3-2021.11-Linux-x86_64.sh
```

4. 아래 명령어로 잘 설치되었는지 확인한다. 
```bash
conda
```
`usage: conda [-h] [-V] command ...`가 출력되면 성공이다. 반면 `conda: command not found`라는 메시지가 뜨는 경우에는 아래 명령어를 실행해서 환경변수(PATH)를 설정해줘야한다.
```
export PATH=~/anaconda3/bin:$PATH
```

# 가상환경 관리
## Anaconda 기본 명령어
- 가상환경 생성
```bash
conda create -n {env name} python={version}
# conda create -n my_env python=3.7
```

- 가상환경 삭제
```bash
conda remove -n {env name} --all
```

- 가상환경 목록 확인
```bash
conda env list
```

- 가상환경 활성화
```bash
conda activate {env name}
```

- 현재 활성화된 가상환경의 패키지 목록 확인
```bash
conda list
```

## 가상환경에 PyTorch 설치
1. [PyTorch get started](https://pytorch.org/get-started/locally/)에서 내 세팅에 맞는 명령어를 확인한 다음 복사한다. 사용중인 CUDA 버전을 모른다면 터미널에 `nvidia-smi`를 입력해서 알아낼 수 있다.
![start-locally](/assets/posts/how-to/2022-05-08-anaconda-install/start-locally.png)

2. 터미널에서 복사해온 명령어를 실행한다. 
```bash
conda install pytorch torchvision torchaudio cudatoolkit=11.3 -c pytorch
```

# 참고한 포스트
- 덴싸, "[linux] 아나콘다 리눅스 서버에 설치하기 install anaconda3 in linux" [[Link](https://datainsider.tistory.com/33)]