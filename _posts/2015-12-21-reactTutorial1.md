---
layout: post
title: React 사용해보기 [1]
categories: react
---

기본 프로젝트 생성

[webjar](http://www.webjars.org/)에서 react관련 js파일 받기

이미지. 빨간색으로 검색, 네모칸

compile 'org.webjars:react:0.14.3' build.gradle에 추가

Gradle refresh

index.html 생성

```html
<!DOCTYPE html>
<html lang="UTF-8">
<head>
    <meta charset="UTF-8">
    <title>Spring Boot</title>
    <script src="/webjars/react/0.13.3/react.min.js"></script>
    <script src="/webjars/react/0.13.3/JSXTransformer.js"></script>
</head>
<body>
Spring Boot View 입니다
<script type="text/jsx" src="index.js"></script>
</body>
</html>
```





