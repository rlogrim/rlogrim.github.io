---
title: 여러 변수를 하나의 그래프로 표현하기
author: Rvinci
description: 여러 변수를 동시에 비교해야 할 때 어떤 방식으로 시각화해야 할지 고민하고 있나요? 이 글에서는 ggplot2를 활용해 여러 속성을 한눈에 비교하는 방법부터, 속성별 분할과 정렬까지 실무에 바로 적용할 수 있는 시각화 기법을 단계별로 소개합니다.
excerpt: 여러 변수를 동시에 비교해야 할 때 어떤 방식으로 시각화해야 할지 고민하고 있나요? 이 글에서는 ggplot2를 활용해 여러 속성을 한눈에 비교하는 방법부터, 속성별 분할과 정렬까지 실무에 바로 적용할 수 있는 시각화 기법을 단계별로 소개합니다.
categories: [chart]
tags: [r, ggplot2, tidyverse, position_dodge, facet_wrap, reorder_within]
published: true
---

데이터 분석을 하다 보면 하나의 데이터셋에 여러 변수가 섞여 있어서  어떻게 표현해야 할지 막막할 때가 있습니다.  
마치 여러 과목 성적표를 한 장의 차트로 정리해야 하는 것처럼 말이죠.   이번 글에서는 `ggplot2`와 `tidyverse`를 활용해 여러 속성을  효과적으로 시각화하는 두 가지 방법을 소개하겠습니다.

예제로 사용할 데이터는 데이터 과학계의 'Hello World'라 불리는 붓꽃(iris) 데이터셋입니다.  
이 데이터에는 세 종류의 붓꽃(setosa, versicolor, virginica)별로  꽃받침 길이, 꽃받침 너비, 꽃잎 길이, 꽃잎 너비가 기록되어 있습니다.

## 데이터 준비: Wide에서 Long 형태로 변환하기

여러 속성을 효율적으로 시각화하려면 먼저 데이터를 'long' 형태로 변환해야 합니다.  
이는 넓게 펼쳐진 테이블을 세로로 길게 쌓는 과정이라고 생각하면 됩니다.  
`pivot_longer` 함수가 이 작업을 간단하게 처리해줍니다.

```r
# 필요한 패키지 불러오기
library(tidyverse)

# 데이터 불러오기 및 전처리
data <- iris %>% 
  rename(`꽃받침 길이` = Sepal.Length, 
         `꽃받침 너비` = Sepal.Width,
         `꽃잎 길이` = Petal.Length, 
         `꽃잎 너비` = Petal.Width, 
         종류 = Species) %>% 
  group_by(종류) %>% 
  summarise_if(is.numeric, ~sum(.)/n())

# Long 형태로 데이터 변환
data2 <- data %>% 
  pivot_longer(cols = `꽃받침 길이`:`꽃잎 너비`, 
               names_to = "구분",
               values_to = "값")

# 변환된 데이터 확인
head(data2)
```

    ## # A tibble: 6 × 3
    ##   종류       구분           값
    ##   <fct>      <chr>       <dbl>
    ## 1 setosa     꽃받침 길이 5.01 
    ## 2 setosa     꽃받침 너비 3.43 
    ## 3 setosa     꽃잎 길이   1.46 
    ## 4 setosa     꽃잎 너비   0.246
    ## 5 versicolor 꽃받침 길이 5.94 
    ## 6 versicolor 꽃받침 너비 2.77

이렇게 변환하면 4개의 속성(꽃받침 길이, 너비, 꽃잎 길이, 너비)이  '구분'이라는 하나의 열로 통합되고,  
각 값은 '값' 열에 담기게 됩니다.

## 방법 1: position_dodge로 그룹별 막대 그래프 그리기

종류별로 색상을 달리하고 막대를 나란히 배치하면 차이를 직관적으로 파악할 수 있습니다.  
`position_dodge`를 사용해 막대를 옆으로 나란히 배치하고,  `geom_text`로 각 막대 위에 수치를 표시합니다.  
또한 `expand`를 조정해 막대 위쪽에 여백을 확보함으로써  
텍스트가 잘리지 않도록 처리합니다.

이 방법은 전체 패턴을 빠르게 파악하기 좋지만,  
속성이 많아지면 그래프가 복잡해질 수 있다는 단점이 있습니다.

```r
ggplot(data = data2, 
       aes(x = 구분, y = 값, group = 종류, fill = 종류)) +
  geom_col(position = position_dodge(width = 0.7), width = 0.6) +
  geom_text(aes(label = scales::comma(값, accuracy = .1)), 
            position = position_dodge(width = 0.7), vjust = -0.5) +
  scale_y_continuous(expand = expansion(mult = c(0, 0.1))) +
  theme_bw()
```

<img src="{{ site.baseurl }}/assets/images/2026-01-02-visualizing-multiple-variables/unnamed-chunk-2-1.png" width="70%" style="display: block; margin: auto;" alt="Iris 품종별 꽃받침 및 꽃잎의 길이와 너비 그래프" />

## 방법 2: facet_wrap으로 속성별 화면 분할하기

정보가 많을 때는 화면을 나누는 게 더 명확할 수 있습니다.  `facet_wrap`을 사용하면 각 속성을 독립된 패널로 분리해서 보여줄 수 있습니다.
마치 4개의 작은 액자를 나란히 걸어두는 것처럼 말이죠.

여기서는 `facet_wrap(vars(구분), nrow = 2)`을 사용해  
'구분' 변수를 기준으로 화면을 2행으로 배열합니다.  
`scale_x_discrete(name = "")`로 x축 제목을 제거해서 깔끔하게 만들고,  
각 패널 제목으로 속성명이 자동으로 표시되도록 합니다.

이 방식은 각 속성의 패턴을 독립적으로 파악하기 좋습니다.

```r
ggplot(data = data2, aes(x = 종류, y = 값, group = 종류, fill = 종류)) +
  geom_col(width = 0.6) +
  geom_text(aes(label = scales::comma(값, accuracy = .1)), vjust = -0.5) +
  scale_x_discrete(name = "") +
  scale_y_continuous(name = "",
                     expand = expansion(mult = c(0, 0.2))) +
  facet_wrap(vars(구분), nrow = 2) +
  theme_bw()
```

<img src="{{ site.baseurl }}/assets/images/2026-01-02-visualizing-multiple-variables/unnamed-chunk-3-1.png" width="70%" style="display: block; margin: auto;" alt="속성별 화면을 분할한 그래프" />

앞선 그래프에서는 x축이 항상 setosa, versicolor, virginica 순서로 고정되어 있습니다.  
각 속성 내에서 값의 크기 순서대로 정렬하면 데이터를 더 쉽게 비교할 수 있습니다.  
`tidytext` 패키지의 `reorder_within` 함수가 이를 해결해줍니다.

`reorder_within(종류, 값, 구분)`을 사용해 각 패널(구분) 내에서 값 기준으로 재정렬합니다.  
`scale_x_reordered()`는 `reorder_within`과 함께 사용되어 정렬된 축을 올바르게 표시해줍니다.  
`scales = "free_x"` 옵션을 통해 각 패널마다 x축이 독립적으로 정렬되도록 설정합니다.

이 방법을 사용하면 "꽃받침 길이는 virginica가 가장 크지만,  
꽃받침 너비는 setosa가 가장 크다"는 식의 인사이트를 한눈에 파악할 수 있습니다.

```r
# tidytext 패키지 불러오기
library(tidytext)

# 각 속성 내에서 값 기준으로 오름차순 정렬
ggplot(data = data2, 
       aes(x = reorder_within(종류, 값, 구분), 
           y = 값, group = 종류, fill = 종류)) +
  geom_col(width = 0.6) +
  geom_text(aes(label = scales::comma(값, accuracy = .1)), vjust = -0.5) +
  scale_x_reordered(name = "") +
  scale_y_continuous(name = "",
                     expand = expansion(mult = c(0, 0.2))) +
  facet_wrap(vars(구분), nrow = 2, scales = "free_x") +
  theme_bw()
```

<img src="{{ site.baseurl }}/assets/images/2026-01-02-visualizing-multiple-variables/unnamed-chunk-4-1.png" width="70%" style="display: block; margin: auto;" alt="x축 막대그래프 순서를 오름차순 정렬한 그래프" />

이번 글에서는 여러 속성을 비교하기 위한 시각화 방법으로  
`position_dodge`로 그룹별 막대 그래프를 그리는 방법과  
`facet_wrap`을 이용해 화면을 분할하는 방법을 살펴봤습니다.

어떤 방법이 가장 좋다기보다는,  
데이터의 특성과 강조하고 싶은 메시지에 따라  
적절한 방법을 선택하는 게 중요합니다.  
위 코드를 여러분의 데이터에 맞게 변형해서 사용해보세요.