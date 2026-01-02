---
title: 값을 기준으로 막대그래프 정렬하기
author: Rvinci
description: 막대그래프를 그렸지만 어떤 항목이 중요한지 한눈에 들어오지 않나요? 이 글에서는 값의 크기를 기준으로 막대그래프를 정렬하는 이유와 방법을 설명하며, 비교와 해석이 쉬운 시각화를 만드는 과정을 단계별로 소개합니다.
excerpt: 막대그래프를 그렸지만 어떤 항목이 중요한지 한눈에 들어오지 않나요? 이 글에서는 값의 크기를 기준으로 막대그래프를 정렬하는 이유와 방법을 설명하며, 비교와 해석이 쉬운 시각화를 만드는 과정을 단계별로 소개합니다.
categories: [chart]
tags: [r, ggplot2, tidyverse, reorder]
published: true
---

막대그래프를 만들 때 막대의 순서를 어떻게 배치하느냐에 따라 메시지 전달력이 크게 달라집니다.  
알파벳 순서로 나열된 막대보다는 값의 크기 순서대로 정렬된 막대가 훨씬 직관적이죠.

이번 글에서는 `ggplot2`에서 `reorder` 함수를 활용해 막대그래프를 값에 따라 정렬하는 방법을 알아보겠습니다.

붓꽃(iris) 데이터셋으로 종류별 평균 꽃받침 너비를 비교하면서 실습해볼게요.

## 기본 막대그래프 그리기

먼저 데이터를 준비하고 기본 막대그래프를 그려보겠습니다.  
특별한 정렬을 지정하지 않으면 `ggplot2`는 기본적으로 범주형 변수를 알파벳 순서로 정렬합니다.

그래프를 보면 setosa, versicolor, virginica 순서로 막대가 배치됩니다.  
이는 알파벳 순서일 뿐, 값의 크기와는 무관합니다.  
어느 종류의 꽃받침 너비가 가장 넓은지 한눈에 파악하기 어렵죠.

``` r
# 패키지 로드
library(ggplot2)
library(dplyr)

# 데이터 준비
data <- iris %>%
  group_by(Species) %>%
  summarise(`꽃받침 너비` = mean(Sepal.Width)) %>%
  rename(종류 = Species)

# 기본 그래프
ggplot(data, aes(x = 종류, y = `꽃받침 너비`, label = round(`꽃받침 너비`, 2))) +
  geom_col(width = 0.6, fill = "#56B4E9") +
  geom_text(vjust = -0.5) +
  scale_y_continuous(expand = expansion(mult = c(0, 0.1))) +
  theme_bw()
```

<img src="{{ site.baseurl }}/assets/images/2026-01-02-ordering-bars-by-value/unnamed-chunk-1-1.png" width="70%" style="display: block; margin: auto;" alt="Iris 품종별 꽃받침 및 꽃잎의 길이와 너비 그래프" />

## 내림차순 정렬하기

값이 큰 순서대로 막대를 정렬하면 가장 중요한 정보를 먼저 보여줄 수 있습니다.  
`reorder` 함수에서 정렬 기준이 되는 변수 앞에 마이너스 기호(-)를 붙이면 내림차순으로 정렬됩니다.

`reorder(종류, -꽃받침_너비)`는 '종류'를 '꽃받침_너비'의 역순(마이너스 기호 때문)으로 재배치하라는 의미입니다.  
이렇게 하면 가장 넓은 꽃받침을 가진 종류부터 왼쪽에 배치됩니다.  
연구 보고서에서 상위 지역이나 우수 사례를 강조할 때 유용한 방식입니다.

``` r
ggplot(data, aes(x = reorder(종류, -`꽃받침 너비`), y = `꽃받침 너비`, label = round(`꽃받침 너비`, 2))) +
  geom_col(width = 0.6, fill = "#56B4E9") +
  geom_text(vjust = -0.5) +
  scale_y_continuous(expand = expansion(mult = c(0, 0.1))) +
  theme_bw()
```

<img src="{{ site.baseurl }}/assets/images/2026-01-02-ordering-bars-by-value/unnamed-chunk-2-1.png" width="70%" style="display: block; margin: auto;" alt="막대그래프 순서를 내림차순 정렬한 그래프" />

## 오름차순 정렬하기

반대로 값이 작은 순서부터 보여주고 싶다면 마이너스 기호를 빼면 됩니다.  
개선이 필요한 항목을 먼저 보여주고 싶을 때 효과적입니다.

`reorder(종류, 꽃받침_너비)`는 '꽃받침_너비'의 값이 작은 순서대로 '종류'를 재배치합니다.  

``` r
ggplot(data, aes(x = reorder(종류, `꽃받침 너비`), y = `꽃받침 너비`, label = round(`꽃받침 너비`, 2))) +
  geom_col(width = 0.6, fill = "#56B4E9") +
  geom_text(vjust = -0.5) +
  scale_y_continuous(expand = expansion(mult = c(0, 0.1))) +
  theme_bw()
```

<img src="{{ site.baseurl }}/assets/images/2026-01-02-ordering-bars-by-value/unnamed-chunk-3-1.png" width="70%" style="display: block; margin: auto;" alt="막대그래프 순서를 오름차순 정렬한 그래프" />

`reorder` 함수를 사용하면 x축 제목이 'reorder(종류, -꽃받침_너비)' 같은 코드 형태로 표시되는 문제가 있습니다.  
이는 보고서나 발표 자료에 사용하기에 적합하지 않죠.  
`scale_x_discrete` 함수로 축 제목을 원하는 텍스트로 변경할 수 있습니다.

`scale_x_discrete(name = "붓꽃 종류")`을 추가하면 x축 제목이 '붓꽃 종류'로 표시됩니다.  
필요에 따라 `name = ""`으로 설정해 축 제목을 아예 없앨 수도 있습니다.  
이는 그래프 자체가 충분히 직관적일 때 유용한 옵션입니다.

``` r
ggplot(data, aes(x = reorder(종류, -`꽃받침 너비`), y = `꽃받침 너비`, label = round(`꽃받침 너비`, 2))) +
  geom_col(width = 0.6, fill = "#56B4E9") +
  geom_text(vjust = -0.5) +
  scale_x_discrete(name = "붓꽃 종류") +
  scale_y_continuous(expand = expansion(mult = c(0, 0.1))) +
  theme_bw()
```

<img src="{{ site.baseurl }}/assets/images/2026-01-02-ordering-bars-by-value/unnamed-chunk-4-1.png" width="70%" style="display: block; margin: auto;" alt="x축 제목을 수정한 그래프"/>

`reorder` 함수를 이용하면, 코드 한 줄로 막대그래프의 가독성을 극적으로 향상시킬 수 있습니다.  
특히 여러 범주를 비교하는 보고서나 프레젠테이션에서 메시지를 명확하게 전달하는 데 큰 도움이 됩니다. 

오늘 다룬 내용을 정리하면:
- 내림차순 정렬: `reorder(범주변수, -값변수)`
- 오름차순 정렬: `reorder(범주변수, 값변수)`
- 축 제목 수정: `scale_x_discrete(name = "원하는 제목")`

여러분의 데이터에 맞는 정렬 방식을 선택하여 전달력을 높여보세요!