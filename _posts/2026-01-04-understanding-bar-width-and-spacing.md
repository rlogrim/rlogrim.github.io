---
title: 막대그래프 너비와 간격 조절하기
author: Rvinci
description: 막대그래프를 그렸는데 막대가 너무 붙어 보이거나 답답하게 느껴진 적 있나요? 이 글에서는 geom_col(width)와 position_dodge(width)가 실제로 무엇을 의미하는지 설명하고, 막대 너비와 간격을 조절하는 방법을 알려드립니다.
excerpt: 막대그래프를 그렸는데 막대가 너무 붙어 보이거나 답답하게 느껴진 적 있나요? 이 글에서는 geom_col(width)와 position_dodge(width)가 실제로 무엇을 의미하는지 설명하고, 막대 너비와 간격을 조절하는 방법을 알려드립니다.
categories: [chart]
tags: [r, tidyverse, ggplot2, geom_col, position_dodge, width]
published: true
---

이번 글에서는 `ggplot2` 패키지를 사용해 막대그래프의 막대 너비와 막대
사이 간격을 조절하는 방법을 정리해 보겠습니다.

막대그래프는 자주 사용하는 차트 중 하나입니다. `ggplot2` 패키지를
이용하면 기본 설정만으로도 바로 그릴 수 있습니다. 다만 아무 설정 없이
그리면 막대가 서로 너무 붙어 보이거나, 전체적으로 답답한 느낌이 들 때가
있습니다. 이럴 때는 `geom_col(width)`와 `position_dodge(width)` 두 가지 옵션을 조절하면 됩니다. 각 옵션이 어떤 역할을 하는지, 그리고 어떻게 조합하면 좋은지 살펴보겠습니다.

[여러 변수를 하나의 그래프로
표현하기](https://rlogrim.github.io/chart/visualizing-multiple-variables/)
글에서 그린 차트를 예제로 활용하겠습니다. `geom_col()`을 이용해 아무 옵션도 주지 않고 기본 막대그래프를
그려봅니다. `position_dodge()`를 사용하면 같은 x위치에 있는 막대들을 옆으로
나란히 배치해 줍니다. 완성된 차트를 보면, 막대들이 서로 붙어 있습니다.

``` r
# 패키지 로드
library(showtext)

# 글꼴 설정
font_add("kopub", "C:/Windows/Fonts/KoPubDotumMedium.ttf")

showtext_auto()
showtext_opts(dpi=300)

theme.size = 14
text.size = theme.size / .pt

# 차트 그리기
ggplot(data = data2, 
       aes(x = 구분, y = 값, group = 종류, fill = 종류)) +
  geom_col(position=position_dodge()) +
  geom_text(aes(label=scales::comma(값, accuracy = .1)), 
            position=position_dodge(width = 0.9), 
            vjust = -0.5,
            family = "kopub",
            size = text.size) +
  scale_y_continuous(name = "길이(단위: cm)",
                     expand = expansion(mult = c(0, 0.3))) +
  theme_bw(base_family = "kopub", base_size = theme.size) +
  theme(
    axis.title.x = element_blank(),
    legend.title = element_blank(),
    legend.key.height = unit(theme.size, "pt"),
    legend.key.spacing.y = unit(3, "pt")
  )
```

<img src="{{ site.baseurl }}/assets/images/2026-01-04-understanding-bar-width-and-spacing/unnamed-chunk-1-1.png" width="70%" style="display: block; margin: auto;" alt="막대그래프 간격과 너비 기본 설정 차트" />

## 막대 너비와 간격을 조절하는 두 가지 옵션

막대그래프의 모양은 아래 두 개의 `width`로 결정됩니다.
`geom_col(width)`로 막대 하나하나의 두께를 조절할 수 있습니다.
`position_dodge(width)`로는 막대들이 얼마나 떨어져 배치될지를 결정할 수
있습니다. 같은 x위치에 속한 막대 묶음 전체가 차지할 가로 폭을 정하기
때문입니다.

다음 규칙을 기억해두면 좋습니다. `geom_col(width)`가
`position_dodge(width)`보다 작으면, 막대 사이에 여백이 생깁니다. 두 값이
같으면, 막대가 딱 붙습니다. `geom_col(width)`가 더 크면, 막대가 서로
겹치게 됩니다.

## 막대 두께는 얇게 만들고 간격은 없애기

막대를 조금 얇게 만들면서 막대 사이 간격은 없애고 싶을 때는 아래와 같이
설정하면 됩니다. 기본값보다 작은 값을 사용해 막대를 얇게 만들고, 두 같을
같게 맞춰 막대들이 붙어서 정렬되도록 합니다.

``` r
ggplot(data = data2, 
       aes(x = 구분, y = 값, group = 종류, fill = 종류)) +
  geom_col(position=position_dodge(width=0.5), width = 0.5) +
  geom_text(aes(label=scales::comma(값, accuracy = .1)), 
            position=position_dodge(width=0.5), 
            vjust = -0.5,
            family = "kopub",
            size = text.size) +
  scale_y_continuous(name = "길이(단위: cm)",
                     expand = expansion(mult = c(0, 0.3))) +
  theme_bw(base_family = "kopub", base_size = theme.size) +
  theme(
    axis.title.x = element_blank(),
    legend.title = element_blank(),
    legend.key.height = unit(theme.size, "pt"),
    legend.key.spacing.y = unit(3, "pt")
  )
```

<img src="{{ site.baseurl }}/assets/images/2026-01-04-understanding-bar-width-and-spacing/unnamed-chunk-2-1.png" width="70%" style="display: block; margin: auto;" alt="막대 두께를 조절한 차트" />

## 막대 두께는 그대로 두고 간격만 넓히기기

이번에는 막대 두께는 그대로 유지한 채 막대 사이 간격만 넓혀보겠습니다.
`geom_col()`의 `width`를 따로 지정하지 않으면 기본값 0.9가 적용됩니다.
`position_dodge(width = 1)`로 막대 사이 공간을 조금 더 벌려줍니다.
막대가 여전히 두껍기 때문에 경우에 따라서는 답답해 보일 수도 있습니다.

``` r
ggplot(data = data2, 
       aes(x = 구분, y = 값, group = 종류, fill = 종류)) +
  geom_col(position=position_dodge(width=1)) +
  geom_text(aes(label=scales::comma(값, accuracy = .1)), 
            position=position_dodge(width=1), 
            vjust = -0.5,
            family = "kopub",
            size = text.size) +
  scale_y_continuous(name = "길이(단위: cm)",
                     expand = expansion(mult = c(0, 0.3))) +
  theme_bw(base_family = "kopub", base_size = theme.size) +
  theme(
    axis.title.x = element_blank(),
    legend.title = element_blank(),
    legend.key.height = unit(theme.size, "pt"),
    legend.key.spacing.y = unit(3, "pt")
  )
```

<img src="{{ site.baseurl }}/assets/images/2026-01-04-understanding-bar-width-and-spacing/unnamed-chunk-3-1.png" width="70%" style="display: block; margin: auto;" alt="막대 간격을 넓힌 차트" />

## 막대그래프에서 가장 많이 쓰는 너비와 간격 설정하기

앞의 예제들은 막대 너비만 바꾸거나, 간격만 바꾸는 방식이었습니다. 하지만
실제로 그래프를 그릴 때는 이 두 가지를 함께 조절하는 경우가 가장
많습니다. 막대가 너무 두꺼우면 답답해 보이고, 반대로 너무 얇으면 값의
차이가 눈에 잘 들어오지 않기 때문입니다. 아래 설정은 이런 점을 적당히
균형 잡아 주는 조합입니다.

이 설정에서는 `geom_col(width = 0.7)`로 막대의 두께를 기본값보다 약간
줄여 막대 하나하나가 가볍게 보이도록 합니다. 동시에
`position_dodge(width = 0.8)`를 사용해 같은 그룹 안에 있는 막대들이 서로
너무 붙지 않도록 간격을 확보합니다. 이렇게 하면 막대는 얇아져 답답함이
줄고, 막대 사이에는 자연스러운 여백이 생깁니다.

``` r
ggplot(data = data2, 
       aes(x = 구분, y = 값, group = 종류, fill = 종류)) +
  geom_col(position=position_dodge(width=0.8), width=0.7) +
  geom_text(aes(label=scales::comma(값, accuracy = .1)), 
            position=position_dodge(width=0.8), 
            vjust = -0.5,
            family = "kopub",
            size = text.size) +
  scale_y_continuous(name = "길이(단위: cm)",
                     expand = expansion(mult = c(0, 0.3))) +
  theme_bw(base_family = "kopub", base_size = theme.size) +
  theme(
    axis.title.x = element_blank(),
    legend.title = element_blank(),
    legend.key.height = unit(theme.size, "pt"),
    legend.key.spacing.y = unit(3, "pt")
  )
```

<img src="{{ site.baseurl }}/assets/images/2026-01-04-understanding-bar-width-and-spacing/unnamed-chunk-4-1.png" width="70%" style="display: block; margin: auto;" alt="막대 너비와 간격 둘 모두 조정한 차트" />

앞으로 막대그래프를 그릴 때 “왜 이렇게 답답해 보이지?”, “막대가 너무
붙어 보이는 것 같은데?”라는 생각이 든다면, 이 글에서 정리한 내용을
떠올려 보세요. `geom_col(width)`와 `position_dodge(width)` 값을 조금만 바꿔보는
것만으로도 그래프의 완성도가 한 단계 올라가는 경험을 할 수 있습니다.