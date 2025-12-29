---
title: y축 여백을 없애거나 범위 조정하기
author: Rvinci
description: 막대그래프에서 y축 여백이 거슬리거나, 값 차이가 잘 안 보인 적이 있나요? ggplot2로 그래프를 그릴 때 y축 범위를 조정하는 방법을 살펴봅니다.
excerpt: 막대그래프에서 y축 여백이 거슬리거나, 값 차이가 잘 안 보인 적이 있나요? ggplot2로 그래프를 그릴 때 y축 범위를 조정하는 방법을 살펴봅니다.
categories: [chart]
tags: [r, ggplot2, y축, scale_y_continuous, coord_cartesian]
published: true
---

차트를 그리다 보면 y축을 그냥 기본값으로 두는 경우가 많습니다.

막상 그래프를 그리고 나서야  
“어딘가 좀 답답한데?”  
라는 생각이 들기도 하고요.

y축 범위가 적절하지 않으면  
데이터 간 차이가 잘 보이지 않거나,  
쓸데없는 여백이 눈에 띄기도 합니다.  
그래프가 전하고 싶은 메시지도  
그만큼 약해질 수 있고요.

붓꽃(iris) 데이터셋을 예로 들어  
y축 범위를 언제, 어떻게 조정하면 좋은지 살펴봅니다.  
붓꽃 데이터셋은 150개의 붓꽃에 대해  
꽃받침과 꽃잎의 길이와 너비를 측정한 데이터입니다.

이 데이터를 이용해 붓꽃 종류별 평균 값을 계산하면  
아래와 같은 결과를 얻을 수 있습니다.

``` r
head(data)
```

    ## # A tibble: 3 × 5
    ##   종류       `꽃받침 길이` `꽃받침 너비` `꽃잎 길이` `꽃잎 너비`
    ##   <fct>              <dbl>         <dbl>       <dbl>       <dbl>
    ## 1 setosa              5.01          3.43        1.46       0.246
    ## 2 versicolor          5.94          2.77        4.26       1.33 
    ## 3 virginica           6.59          2.97        5.55       2.03

이제 이 데이터를 바탕으로 간단한 시각화를 해보겠습니다.
붓꽃 종류별 평균 꽃받침 길이를 막대그래프로 비교해 봤습니다.

``` r
ggplot(data=data, aes(x=종류, y=`꽃받침 길이`, label=`꽃받침 길이`)) +
  geom_col(width=0.6) +
  geom_text(vjust=-0.5, size=12/.pt) +
  theme_bw(base_size=12)
```

<img src="{{ site.baseurl }}/assets/images/2025-12-29-y-axis-range-adjustment-tips/unnamed-chunk-2-1.png" width="70%" style="display: block; margin: auto;" alt="Iris 품종별 평균 꽃받침 길이 비교 그래프"/>

그래프를 보면 x축과 막대 사이에 여백이 조금 남아 있습니다.

큰 문제는 아니지만, 이 여백 때문에 그래프 아래가 비어 보이면서  
전체적으로 어색한 인상을 줍니다.

이런 경우에는 y축 범위를 조정해  
막대가 자연스럽게 x축에 닿도록 만드는 편이 좋습니다.

## 보통 막대그래프는 0에서 시작합니다

막대그래프 아래 여백을 줄이고 싶다면  
y축을 0에서 시작하도록 설정하면 됩니다.

이를 위해 `scale_y_continuous()` 함수의 `expand` 옵션을 사용합니다.  
`expand`는 축의 시작과 끝에 들어가는 여백을 조절하는 역할을 합니다.

아래 코드를 추가하면 y축이 0에서 시작하고 불필요한 여백이 정리됩니다.
`mult`의 첫 번째 값 `0`은 y축 시작 지점의 여백을 제거하고,  
두 번째 값 `0.1`은 y축 상단에 약간의 여백을 추가하는 역할을 합니다.

``` r
ggplot(data=data, aes(x=종류, y=`꽃받침 길이`, label=`꽃받침 길이`)) +
  geom_col(width=0.6) +
  geom_text(vjust=-0.5, size=12/.pt) +
  scale_y_continuous(expand=expansion(mult=c(0, 0.1))) +
  theme_bw(base_size=12)
```

<img src="{{ site.baseurl }}/assets/images/2025-12-29-y-axis-range-adjustment-tips/unnamed-chunk-4-1.png" width="70%" style="display: block; margin: auto;" alt="y축이 0에서 시작하도록 수정한 그래프"/>

## 차이가 잘 안 보일 땐 y축 범위를 줄입니다

항상 y축이 0에서 시작해야 하는 것은 아닙니다.  
0부터 시작하는 y축에서는 값 사이의 차이가 잘 드러나지 않을 수 있습니다.

이럴 때는 보고 싶은 구간만 보여주는 방법이 있습니다.  
이때 사용하는 함수가 `coord_cartesian()`입니다.  
데이터를 건드리지 않고 보이는 범위만 조정할 수 있습니다.

그래프를 확대해서 중요한 부분에 시선을 집중하는 느낌이라고 보면 됩니다.

``` r
ggplot(data=data, aes(x=종류, y=`꽃받침 길이`, label=`꽃받침 길이`)) +
  geom_col(width=0.6) +
  geom_text(vjust=-0.5, size=12/.pt) +
  scale_y_continuous(expand=expansion(mult=c(0, 0.1))) +
  coord_cartesian(ylim=c(3, 7)) +
  theme_bw(base_size=12)
```

<img src="{{ site.baseurl }}/assets/images/2025-12-29-y-axis-range-adjustment-tips/unnamed-chunk-5-1.png" width="70%" style="display: block; margin: auto;" alt="y축 범위를 조정한 그래프"/>

축 설정 하나만 바꿔도 그래프의 인상과 전달력이 달라집니다.

다음에 그래프가 어딘가 아쉬워 보인다면, y축 범위를 한 번쯤 조정해 보세요.

생각보다 효과가 큽니다.