---
layout: post
title:  "Markdown 간단하게 정리하기"
categories: github
---

# Markdown?
Plain text로 작성된 파일을 HTML로 변경해서 보여주기 위한 문법이다.

# GFM?
표준 Markdown에 조금 변경한 GitHub 버젼 Markdown 이라고 보면 된다.

현재 이 블로그는 GitHub Pages(GitHub에서 제공하는 무료 웹사이트 서비스)에서 작성되고 있으므로, Markdown Basic과 GFM에 대해서 간단하게 정리하고 넘어가려고 한다.

# Markdown Basics

## Paragraphs
Markdown 에서 문단(줄바꿈)은 하나 이상의 enter가 필요하다.

예를 들어 plain text로 아래와 같이 쓴다면

```text
이것은
줄바꿈이 되지 않습니다.
```

결과는 "이것은
     줄바꿈이 되지 않습니다." 와 같이 한 줄로 나온다.

실제로 문단을 나누려면(새로운 줄로 글을 쓰려면) 아래와 같이 하나 이상의 blank line이 필요하다.

```text
이것은 

줄바꿈이 됩니다.
```

## Headings

하나 이상의 `#` 심볼을 추가하여 사용하면, heading text가 된다.
 
```text
# 가장 큰 heading (<h1> tag)
## 두번째로 큰 heading (<h2> tag)
...
###### 6번째로 큰(가장 작은) heading (<h6> tag)
```

## Blockquotes

참조에 대해서는 아래와 같이 사용한다.

```text
Docker 컨테이너를 실행 시키기 위해서는 run 명령어를 사용합니다.
> 포트포워딩을 위해서는 -p 옵션을 함께 사용합니다.  
```

Docker 컨테이너를 실행 시키기 위해서는 run 명령어를 사용합니다.
> 포트포워딩을 위해서는 -p 옵션을 함께 사용합니다.

## Styling text

bold나 italic 으로 만들 수 있다.

```text
*여기는 이텔릭체로 써집니다.*
**여기는 볼드체로 써집니다.**
```
*여기는 이텔릭체로 써집니다.*
**여기는 볼드체로 써집니다.**

**bold**와 *italic*을 함께 쓰기 위해서는 `*` 대신에 '_'을 사용할 수도 있다.

```text
**볼드 *이텔릭* 입니다**
**볼드 _이텔릭_ 입니다**
```

**볼드 *이텔릭* 입니다**

**볼드 _이텔릭_ 입니다**

## Lists

**Unordered lists**

순서가 없는 lis를 만들기 위해서는 `*` 또는 `-`를 사용하면 된다.

```text
* item1
* item2

- item1
- item2
```

* item1
* item2
- item1
- item2

**Ordered lists**
순서가 있는 리스트를 위해서는 숫자와 함께 사용하면 된다.
```text
1. Item 1
2. Item 2
3. Item 1
```
1. Item 1
2. Item 2
3. Item 1

**Nested lists**
두 개의 space로 들여 쓰기를 하게 되면, 중첩된 목록을 만들 수 있다.

```text
1. 아키텍쳐
  1. MSA
  2. SOA
2. Java
  1. Java 8
    * 스트림
    * 람다
  2. 자바 성능
```

1. 아키텍쳐
  1. MSA
  2. SOA
2. Java
  1. Java 8
    * 스트림
    * 람다
  2. 자바 성능

## Code formatting
**Inline formats**

하나의 따옴표 안에 작성된 plain text들은 HTML로 변환되지 않고 그대로 plain text로 출력된다. 
```text
`**여기에 존재하는 내용은 볼드가 아니라 그대로 출력된다.**` 는 내용이다.
```

`**여기에 존재하는 내용은 볼드가 아니라 그대로 출력된다.**` 는 내용이다.

**Multiple lines**

세 개의 따옴표 안에 있는 plain text는 멀티라인을 지원한다.

## Links
`[링크텍스트](링크URL)`을 사용해서 링크를 생성할 수 있다.

[eom's blog 바로가기](http://blog.eomdev.com)

# GitHub Flavored Markdown (GFM)

## Markdown Basic과 다른점

**취소선**

```text
~~이렇게 하면 취소선이 됩니다.~~
```
~~이렇게 하면 취소선이 됩니다.~~

**Tables**

row를 분리하기 위해서 `-` 를 사용하고, column을 분리하기 위해서 `|`를 사용한다.
좌측, 가운데, 우측정렬을 위해서는 `:`를 사용한다.

```text
|헤더1|헤더2|
|----|----|
|셀1  | 셀2|
|셀3  | 셀3|

|좌측정렬  |가운데정렬  |우측정렬  |
|:-------|:-------:|-------:|
|1       |2        |3       |
```

|헤더1|헤더2|
|----|----|
|셀1  | 셀2|
|셀3  | 셀3|

|좌측정렬  |가운데정렬  |우측정렬  |
|:-------|:-------:|-------:|
|1       |2        |3       |

# 정리하며

더 자세한 정보는 [GitHub Help](https://help.github.com/categories/writing-on-github/) 에서 확인할 수 있다.