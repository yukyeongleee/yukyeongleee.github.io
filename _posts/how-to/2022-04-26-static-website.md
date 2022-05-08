---
layout: post
title: "[Github Blog] Jekyll로 정적 웹사이트(Static Website) 생성"
description: "Github Blog 구성을 변경하거나 포스팅을 추가할 때 로컬에서 미리보며 작업하는 방법 소개"
date: 2022-04-26
tags: [github blog, how to, jekyll]
comments: false
share: true
---

# Jekyll, Gem, Bundler
- **Jekyll**은 Ruby 언어로 개발된 프레임워크이다. HTML/Markdown 등 마크업 언어로 글을 작성하면 미리 정의해둔 레이아웃 규칙에 따라 **정적 웹사이트(static website)**를 만들어준다. 정적 웹사이트는 웹 서버를 사용하지 않고 미리 저장된 파일 만으로 생성한 웹사이트를 의미하는데, 빠르고 가벼운 장점이 있다. 
- **Gem**은 Ruby 언어의 여러 플러그인을 말한다. 
- **Bundler**는 Ruby 언어를 위한 gem 패키지 매니저로, Ruby 프로그램에 필요한 gem을 가져올 때 필요하다.


# Jekyll 설치 방법 (Windows/Linux)
## Windows
Windows는 공식적으로 Jekyll이 지원되는 플랫폼은 아니지만, RubyInstaller를 통해 설치할 수 있다. 

1. [다운로드 페이지](https://rubyinstaller.org/downloads/)에서 **Ruby+Devkit** (버전 2.4 이상) 설치 프로그램을 다운로드하고 실행한다. 설치 마법사에서 아래 두 가지 체크박스는 꼭 선택해야 한다.   
  - _Add Ruby executables to your PATH_  
  - _Run 'ridk install' to setup MSYS2 and development toolchain_ 

2. Ruby 설치가 완료되면, MSYS2를 설치하는 검정 화면이 나온다. _'Which components shall be installed?'_ 에 대한 답으로 1, 2, 3 모두를 입력한다. 

3. Ruby가 잘 설치되었는지 확인한다. 설치한 Ruby 버전이 출력되면 성공이다. 
```powershell
ruby -v
```

4. **Jekyll** 및 **Bundler**를 설치한다.
```powershell
gem install jekyll bundler
```

5. Jekyll 이 잘 설치되었는지 확인한다. 마찬가지로 설치한 Jekyll 버전이 출력되면 성공이다. 
```powershell
jekyll -v
```

## Ubuntu (Linux)
1. Jekyll을 설치하는데 필요한 모든 dependency를 설치한다. 
```bash
sudo apt-get install ruby-full build-essential zlib1g-dev
```

2. Ruby gem 설치 경로에 관한 환경변수들을 `~/.bashrc` 파일에 추가한다.
```bash
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

3. Windows 항목의 3 - 5번을 따라한다. 

# 정적 웹사이트 호스팅 및 접속
0. [username].github.io 디렉토리로 이동한다.

1. `Gemfile`에 명시된 gem을 모두 설치한다. 디렉토리 내에 `Gemfile`이 없는 경우, `README.md`를 유심히 살펴보자. 그곳에서 힌트를 얻을지도 모른다.
```bash
bundle install
```

2. 정적 웹사이트를 호스팅한다. 위의 단계를 똑같이 따라했음에도 이 과정에서 에러가 뜰 수 있다. 보통 현재 사용중인 템플릿이 필요로 하는 gem이 설치되지 않았거나, 그 버전이 유효하기 않을 때 에러가 발생한다. 그럴 때는 구글링을 해서 Gemfile을 적절히 수정하면 된다. 
```bash
bundle exec jekyll serve 
```   
마지막에 아래와 같은 출력이 나오면 성공이다.
```
Server address: http://127.0.0.1:4000
Server running... press ctrl-c to stop.
```

3. 인터넷 브라우저를 열고 `localhost:4000`에 접속하면 내 블로그가 보인다! 새로고침할 때마다 문서의 수정사항이 반영되는 것을 확인할 수 있다.  

# 참고한 포스팅
- jekyllrb-ko, "Jekyll 설치" [[Link](http://jekyllrb-ko.github.io/docs/installation)]
- cheersHena, "Jekyll이란?" [[Link](https://cheershennah.tistory.com/214)]