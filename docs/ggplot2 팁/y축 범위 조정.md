---
layout: post
title: y축 범위 조정
parent: ggplot2 팁
published: true
---

# {{ page.title }}
{: .no_toc }

## 목차
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## y축 범위를 조정하고 싶을 때

붓꽃 데이터셋을 이용해 그래프를 그려보겠습니다.

붓꽃 데이터셋은 150개 붓꽃을 조사하여 붓꽃의 종류, 꽃받침와 꽃잎의 길이, 너비를 측정하여 기록해 놓은 데이터셋입니다.

그래프를 그리기 전, 사전 작업으로 붓꽃 종류별로 꽃받침 길이, 꽃받침 너비, 꽃잎 길이, 꽃잎 너비의 평균값을 계산해 놓았습니다.

``` r
head(data)
```

    ## # A tibble: 3 × 5
    ##   종류       `꽃받침 길이` `꽃받침 너비` `꽃잎 길이` `꽃잎 너비`
    ##   <fct>              <dbl>         <dbl>       <dbl>       <dbl>
    ## 1 setosa              5.01          3.43        1.46       0.246
    ## 2 versicolor          5.94          2.77        4.26       1.33 
    ## 3 virginica           6.59          2.97        5.55       2.03

이 데이터를 사용해 붓꽃 종류별 평균 꽃받침 길이를 비교하는 막대그래프를 그려보겠습니다.

x축은 '꽃의 종류', y축은 '평균 꽃받침 길이'입니다.

막대그래프는 ```geom_col```로, 막대그래프 위의 텍스트는 ```geom_text```로 그렸습니다.

그리고 Complete themes 중 ```theme_bw```를 이용해 그래프 테마를 기본 테마보다 더 깔끔한 버전으로 바꿔 주었습니다.

``` r
ggplot(data=data, aes(x=종류, y=`꽃받침 길이`, label=`꽃받침 길이`)) +
  geom_col(width=0.6) +
  geom_text(vjust=-0.5, size=12/.pt) +
  theme_bw(base_size=12)
```

<img src="/assets/images/y축-범위-조정/unnamed-chunk-2-1.png" width="70%" style="display: block; margin: auto;" />

위 그래프를 보면, x축과 막대그래프 사이에 간격이 벌어져 있습니다.

이 빈 공간을 어떻게 하면 없앨 수 있을까요?

## y축이 0에서 시작하도록 수정하기

y축이 0에서 시작하도록 수정해 주면, x축과 막대그래프 사이의 빈 공간을 없앨 수 있습니다.

기존 코드에 `scale_y_continuous(expand=expansion(mult=c(0, 0.1)))`를 한 줄 추가하면 해결됩니다.

``` r
ggplot(data=data, aes(x=종류, y=`꽃받침 길이`, label=`꽃받침 길이`)) +
  geom_col(width=0.6) +
  geom_text(vjust=-0.5, size=12/.pt) +
  scale_y_continuous(expand=expansion(mult=c(0, 0.1))) +
  theme_bw(base_size=12)
```

<img src="/assets/images/y축-범위-조정/unnamed-chunk-4-1.png" width="70%" style="display: block; margin: auto;" />

## y축이 데이터 중간에서 시작하도록 수정하기

종종 막대그래프가 0이 아니라 데이터의 중간 값에서 시작하도록 바꾸고 싶을 때도 있습니다.

비교하려는 값들 사이에 큰 차이가 없는 경우 막대그래프가 0에서 시작하면 어떤 값이 더 큰 지 한 눈에 확인하기 어려운 경우가 있기 때문입니다.

평균 꽃받침 길이를 더 빠르게 비교할 수 있도록 위 그래프를 수정해 보겠습니다.

기존 코드에 `coord_cartesian(ylim=c(3, 7))`를 추가해 주기만 하면 됩니다.

그래프 상 표현되는 y축 범위가 3과 7사이로 변경되고 막대그래프 간 높이 차이가 더 커져서, 이전 그래프보다 값 비교가 더 쉬워집니다.

``` r
ggplot(data=data, aes(x=종류, y=`꽃받침 길이`, label=`꽃받침 길이`)) +
  geom_col(width=0.6) +
  geom_text(vjust=-0.5, size=12/.pt) +
  scale_y_continuous(expand=expansion(mult=c(0, 0.1))) +
  coord_cartesian(ylim=c(3, 7)) +
  theme_bw(base_size=12)
```

<img src="/assets/images/y축-범위-조정/unnamed-chunk-5-1.png" width="70%" style="display: block; margin: auto;" />
