---
title: 지역명에 윤곽선 두르는 법
author: Rvinci
description: 지도에 지역명이 잘 보이지 않아 고민한 적 있나요? 글자 주위에 윤곽선을 둘러, 어떤 지도에서도 글씨가 또렷하게 보이는 법을 알려드립니다.
excerpt: 지도에 지역명이 잘 보이지 않아 고민한 적 있나요? 글자 주위에 윤곽선을 둘러, 어떤 지도에서도 글씨가 또렷하게 보이는 법을 알려드립니다.
categories: [map]
tags: [r, tidyverse, sf, geom_text_repel, geom_shadowtext]
published: true
---

## 지도의 디테일, 왜 윤곽선이 중요할까?

지도를 그리다 보면 산이나 바다처럼 어두운 배경 위에는 밝은 글씨를, 밝은
배경 위에는 어두운 글씨를 써야 할 때가 있습니다. 하지만 하나의 지도 안에
다양한 색상이 섞여 있다면 어떤 색의 글씨를 써도 가독성이 떨어지는 경우가
있습니다.

이때 효과적인 해결책이 바로 텍스트 윤곽선입니다. 글자 테두리에 배경과
대비되는 색상을 얇게 입혀주면, 어떤 복잡한 배경 위에서도 텍스트가
선명하게 보입니다.

지난번 [지도 지역명 겹칠 때
해결하기](https://rlogrim.github.io/map/handling-overlapping-labels-in-maps/)
글에서는 수도권 지도를 그리고 지역명을 겹치지 않게 배치하는 방법을
알아봤습니다. 텍스트를 분산 배치하거나, 면적이 작은 일부 지역의 명칭을
생략하는 방법으로 문제를 해결했죠. 이번에는 지도의 가독성을 한 단계 더
높이기 위해 텍스트에 윤곽선을 추가하는 방법을 살펴보겠습니다.

## 데이터 준비하기

본격적인 시각화에 앞서 수도권 행정구역 데이터를 불러오고 폰트를
설정합니다. 지리 데이터(sf 객체)를 정제하고 각 구의 면적을 계산하는 기초
전처리를 먼저 진행합니다. 구체적인 데이터 처리 및 지도 작성 방법은 [지도
지역명 겹칠 때
해결하기](https://rlogrim.github.io/map/handling-overlapping-labels-in-maps/)
글을 참고하세요.

``` r
# 패키지 로드
library(tidyverse)
library(sf)
library(showtext)
library(ggrepel)

# 글꼴 설정
font_add("kopub", "C:/Users/.../AppData/Local/Microsoft/Windows/Fonts/KoPub Dotum Medium.ttf")
showtext_auto()
showtext_opts(dpi=300)

theme.size = 10
text.size = theme.size / .pt

# 데이터 불러오기
sdg <- st_read("아웃풋/수도권 행정구 병합 지도.shp",
           quiet = TRUE) %>% 
    separate(col = SGG_NM,
           into = c("SD_NM", "SGG_NM"),
           sep = " ",
           fill = "right")

# 폴리곤 면적 계산
sdg <- sdg %>% 
  mutate(면적 = as.numeric(st_area(sdg)))
```

## 방법 1: `geom_text_repel`로 텍스트 윤곽선 추가하기

첫 번째 방법은 텍스트 겹침 방지 패키지로 유명한 `ggrepel`을 사용하는
것입니다. 이 패키지의 `geom_text_repel` 함수는 글자끼리 겹치지 않게
밀어내면서, 동시에 `bg.color(윤곽선 색상)`와 `bg.r(윤곽선 두께)`이라는
인자를 제공합니다. 텍스트가 많은 복잡한 지도를 만들 때 가장 추천하는
방법입니다.

``` r
# 지도 작성하기
ggplot(data = sdg) +
  geom_sf(colour = "gray40", fill = "#eaeaea", linewidth = 0.5) +
  geom_text_repel(aes(label = SGG_NM, geometry = geometry),
                  stat = "sf_coordinates",
                  family = "kopub",
                  size = text.size,
                  force = 0.0001,
                  force_pull = 1000,
                  color = "white",
                  bg.color = "black",
                  bg.r = 0.1) +
  theme_void()
```

<img src="{{ site.baseurl }}/assets/images/2026-01-16-adding-text-halos/unnamed-chunk-2-1.png" alt="geom_text_repel로 지역명에 윤곽선 추가한 지도" width="100%" style="display: block; margin: auto;" />

## 방법 2: `geom_shadowtext`로 텍스트 윤곽선 추가하기

두 번째는 `shadowtext` 패키지를 활용하는 방법입니다. 이 패키지는
텍스트에 그림자나 윤곽선을 입히는 데 특화되어 있습니다. 다만, sf 객체의
좌표를 인식할 수 있도록 `geometry`와 `stat` 옵션을 꼭 지정해 주어야
합니다.

``` r
# 패키지 로드
library(shadowtext)

# 지도 작성하기
ggplot(data = sdg) +
  geom_sf(colour = "gray40", fill = "#eaeaea", linewidth = 0.5) +
  geom_shadowtext(aes(label = ifelse(sdg$면적 < median(sdg$면적), "", SGG_NM),
                      geometry = geometry),
                  stat = "sf_coordinates",
                  family = "kopub",
                  size = text.size,
                  color = "white",
                  bg.color = "black",
                  bg.r = 0.1) +
  theme_void()
```

<img src="{{ site.baseurl }}/assets/images/2026-01-16-adding-text-halos/unnamed-chunk-3-1.png" alt="geom_shadowtext로 지역명에 윤곽선 추가한 지도" width="100%" style="display: block; margin: auto;" />

이번 글에서는 `geom_text_repel`과 `geom_shadowtext`로 텍스트에 윤곽선을
추가하는 두 가지 방법을 살펴봤습니다. 텍스트 주위에 둘러진 얇은 윤곽선
하나가 여러분의 지도를 더 친절하게 만들어 줄 것입니다. 지역명에
윤곽선을 입혀, 전하고자 하는 메시지를 더 효과적으로 전달해 보시길
바랍니다!
