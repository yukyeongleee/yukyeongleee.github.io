---
layout: post
title: "[Github blog] Jekyll 템플릿 폰트 수정" 
description: "Github blog 한글 문서 가독성 향상을 위해 글꼴을 커스터마이즈 하는 방법 소개"
date: 2022-04-27
tags: [github blog, how to, jekyll]
comments: false
share: true
---

[Blueface](http://jekyllthemes.org/themes/blueface/) 템플릿을 사용해서 회사 Github blog를 제작했다. 만들고 보니 한글 문서에 대한 가독성이 별로였다. 글꼴을 커스터마이즈했던 과정을 소개하려고 한다.

# 글꼴 커스터마이즈
1. [Google Fonts](https://fonts.google.com/?subset=korean)에서 마음에 드는 글꼴을 선택한다. 

2. Styles에서 필요한 항목에 대해서만 `+ Select this style`을 클릭한다.

3. 오른쪽 상단에 있는 `View selected family` 아이콘을 클릭한다.<br>
Review를 보면 내가 선택했던 폰트 스타일이 모여있다. 
![review](/assets/posts/2022-04-27-customize-font/googlefont-review.png)
또한 Use on the web에서 웹 폰트를 불러오는데 필요한 HTML 코드를 확인할 수 있다.  
![usage](/assets/posts/2022-04-27-customize-font/googlefont-usage.png)

4. **(Blueface 템플릿 기준)** `/css/main.css` 파일을 수정해야한다. 다른 템플릿을 사용중이라면 `/_sass/_layout.scss`에서 비슷한 내용을 찾을 수 있을 것이다. <br>
먼저, 웹 폰트 파일을 불러오는 코드를 삽입한다. 
```css
@import url(https://fonts.googleapis.com/css2?family=Gowun+Dodum&display=swap);
@import url(https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@300;400;500;700&display=swap);
```
다음으로, 각 스타일의 `font-family`를 불러온 웹 폰트로 수정한다.
```css
body {
    color: #586161;
    font-family: 'Gowun Dodum', sans-serif;
    font-weight: 600;
    text-rendering: optimizeLegibility;
}
``` 
5. 각 스타일마다 `font-family` 외에도 `color`, `font-weight`, `font-size` 등의 속성을 수정해서 효과를 극대화시킬 수 있다. 문자열을 대문자 또는 소문자로 바꿔주는 `text-transform`이라는 흥미로운 속성도 있다. 제목이 모두 대문자로 출력돼서 보기에 불편했는데 `text-transform: uppercase` 부분을 주석 처리해서 해결했다. 

# 참고한 포스팅
- S_ih_yun, "[Jekyll] Jekyll와 Github로 만드는 깃허브 블로그 - (3) 글씨체 변경" [[Link](https://codesyun.tistory.com/116)]
- Coding Factory, "CSS/text-transform/대문자로 또는 소문자로 바꾸는 속성" [[Link](https://www.codingfactory.net/10656)]