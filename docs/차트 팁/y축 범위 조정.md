---
layout: post_comments
title: y축 범위 조정
parent: 차트 팁
published: true
output:
  md_document:
    variant: markdown_github
    preserve_yaml: true
knit: (function(inputFile, encoding) {
  rmarkdown::render(inputFile, 
  encoding = encoding, 
  output_file=paste0(Sys.Date(), "-", sub(".Rmd", ".md",inputFile)), 
  output_dir = "C:/Users/MINYOUNG/projects/ggsave/docs/차트 팁") })
---

## y축 범위, 왜 신경 써야 할까?

데이터를 시각화할 때 y축 범위는 중요하지 않은 요소로 보일 수 있지만,
사실 그래프의 가독성과 메시지 전달력에 큰 영향을 미칩니다. 적절하지 않은
y축 범위는 데이터의 주요 차이를 흐리거나, 불필요한 여백을 만들어 시각적
집중도를 떨어뜨릴 수 있기 때문입니다.

이 글에서는 붓꽃 데이터셋을 예로 들어, y축 범위 조정이 필요한 상황과
이를 해결하는 방법을 구체적으로 살펴보겠습니다. 붓꽃 데이터셋은 150개의
붓꽃에 대해 꽃받침과 꽃잎의 길이와 너비를 측정한 데이터입니다. 이를
활용해 각 붓꽃 종류별로 꽃받침 길이, 꽃받침 너비, 꽃잎 길이, 꽃잎 너비의
평균을 계산하면 아래와 같은 결과를 얻을 수 있습니다.

``` r
head(data)
```

    ## # A tibble: 3 × 5
    ##   종류       `꽃받침 길이` `꽃받침 너비` `꽃잎 길이` `꽃잎 너비`
    ##   <fct>              <dbl>         <dbl>       <dbl>       <dbl>
    ## 1 setosa              5.01          3.43        1.46       0.246
    ## 2 versicolor          5.94          2.77        4.26       1.33 
    ## 3 virginica           6.59          2.97        5.55       2.03

이 데이터를 기반으로 간단한 시각화를 진행해 보겠습니다. 붓꽃 종류별 평균
꽃받침 길이를 비교하는 막대그래프를 그려보았습니다.

``` r
ggplot(data=data, aes(x=종류, y=`꽃받침 길이`, label=`꽃받침 길이`)) +
  geom_col(width=0.6) +
  geom_text(vjust=-0.5, size=12/.pt) +
  theme_bw(base_size=12)
```

<img src="{{ site.baseurl }}/assets/images/y축-범위-조정/unnamed-chunk-2-1.png" width="70%" style="display: block; margin: auto;" />

그런데 x축과 막대그래프 사이에 공간이 생겨 시각적으로 불필요한 여백이
발생했습니다. 이 공간을 없애려면 어떻게 해야 할까요?

## y축을 0에서 시작하도록 설정

x축과 막대그래프 사이의 여백을 없애려면 y축의 범위를 0에서 시작하도록
설정해야 합니다. 이를 위해 `scale_y_continuous` 함수의 `expand` 옵션을
활용할 수 있습니다.

`expand` 옵션은 그래프의 축 끝에 추가되는 여백을 설정하는 데 사용됩니다.
이 여백은 데이터를 더 명확히 표현하거나 그래프의 가독성을 높이는 데
도움을 줄 수 있지만, 불필요한 여백은 제거하는 것이 좋습니다.

다음 코드를 기존 그래프 코드에 추가하면, y축이 정확히 0에서 시작하고
여백이 최소화됩니다.

여기서 `mult` 매개변수는 축 끝에 추가할 상대적 여백을 설정합니다. 첫
번째 값 `0`은 y축의 시작점에서 여백을 0으로 설정합니다. 즉, y축이 0에서
시작하게 됩니다. 두 번째 값 `0.1`은 y축의 끝점에서 총 데이터 범위의
10%에 해당하는 여백을 추가하는 것을 의미합니다.

``` r
ggplot(data=data, aes(x=종류, y=`꽃받침 길이`, label=`꽃받침 길이`)) +
  geom_col(width=0.6) +
  geom_text(vjust=-0.5, size=12/.pt) +
  scale_y_continuous(expand=expansion(mult=c(0, 0.1))) +
  theme_bw(base_size=12)
```

<img src="{{ site.baseurl }}/assets/images/y축-범위-조정/unnamed-chunk-4-1.png" width="70%" style="display: block; margin: auto;" />

## y축이 데이터 중간에서 시작하도록 설정

경우에 따라 y축을 데이터 중간부터 시작하고 싶을 때가 있습니다. 예를
들어, 비교하고자 하는 값들이 큰 차이가 없는 경우, y축이 0부터 시작하면
중요한 차이를 제대로 드러내기 어렵습니다. 이럴 때는 y축의 범위를
데이터의 중간값이나 원하는 구간으로 조정하면 됩니다. 이를 위해
`coord_cartesian` 함수를 사용할 수 있습니다.

``` r
ggplot(data=data, aes(x=종류, y=`꽃받침 길이`, label=`꽃받침 길이`)) +
  geom_col(width=0.6) +
  geom_text(vjust=-0.5, size=12/.pt) +
  scale_y_continuous(expand=expansion(mult=c(0, 0.1))) +
  coord_cartesian(ylim=c(3, 7)) +
  theme_bw(base_size=12)
```

<img src="{{ site.baseurl }}/assets/images/y축-범위-조정/unnamed-chunk-5-1.png" width="70%" style="display: block; margin: auto;" />

## 마무리하며

그래프의 작은 설정 하나가 시각화의 가독성과 전달력을 크게 바꿀 수
있습니다. 오늘 다룬 y축 범위 조정 팁을 활용해 데이터를 더 명확히 전달할
수 있는 그래프를 만들어 보세요 😊
