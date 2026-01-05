---
title: 차트 색상 바꾸기
author: Rvinci
description: 그래프를 그렸는데 색상이 마음에 들지 않았던 적 있나요? 이 글에서는 ggplot2의 기본 색상 설정부터 RColorBrewer 팔레트, 직접 색상을 지정하는 방법까지 정리합니다.
excerpt: 그래프를 그렸는데 색상이 마음에 들지 않았던 적 있나요? 이 글에서는 ggplot2의 기본 색상 설정부터 RColorBrewer 팔레트, 직접 색상을 지정하는 방법까지 정리합니다.
categories: [chart]
tags: [r, tidyverse, ggplot2, RColorBrewer, scale_fill_brewer, scale_fill_manual]
published: true
---

값의 차이뿐 아니라 범주를 함께 비교해야 할 때 색상은 특히 중요한 역할을 합니다. 어떤 막대가 같은 그룹에 속하는지는 색상이 있어야 빠르게 인식할 수 있습니다. 같은 모양의 막대가 반복될수록 색상은 시선을 분산시키지 않으면서 정보를 구분해 주는 기준이 됩니다.

또한 색상은 그래프를 읽는 순서에도 영향을 미칩니다. 강조된 색은 먼저
보이고, 덜 눈에 띄는 색은 자연스럽게 배경으로 물러납니다. 따라서 어떤
값을 강조할지, 어떤 값은 비교 대상으로 남길지에 따라 색상 설정이
달라져야 합니다.

`ggplot2`는 기본 색상을 자동으로 지정해 주기 때문에 빠르게 그래프를
그리기에는 충분합니다. 하지만 메시지를 분명하게 전달해야 하는 경우에는
색상을 직접 조정하는 것이 좋습니다. 이럴 때 팔레트를 활용하거나 색상을
직접 지정하면, 그래프의 가독성과 전달력을 함께 높일 수 있습니다.

이번 글에서는 `ggplot2`로 막대그래프를 그릴 때 색상을 설정하는 기본
흐름을 살펴봅니다. 먼저 기본 색상이 어떻게 적용되는지 확인하고,
`RColorBrewer` 팔레트를 사용하는 방법과 직접 색상을 지정하는 방법을
차례로 정리합니다.

## 데이터 준비하기

이번 예제에서는 `iris` 데이터를 사용합니다. 품종별로 꽃받침과 꽃잎의
평균 길이를 계산한 뒤, 막대그래프로 표현합니다. 이후 색상을 바꿔 가며
결과가 어떻게 달라지는지 살펴봅니다.

``` r
data <- iris %>% 
  rename(`꽃받침 길이` = Sepal.Length, 
         `꽃받침 너비` = Sepal.Width,
         `꽃잎 길이` = Petal.Length, 
         `꽃잎 너비` = Petal.Width, 
         종류 = Species) %>% 
  group_by(종류) %>% 
  summarise(across(where(is.numeric), mean))

# 데이터 확인
head(data)
```

    ## # A tibble: 3 × 5
    ##   종류       `꽃받침 길이` `꽃받침 너비` `꽃잎 길이` `꽃잎 너비`
    ##   <fct>              <dbl>         <dbl>       <dbl>       <dbl>
    ## 1 setosa              5.01          3.43        1.46       0.246
    ## 2 versicolor          5.94          2.77        4.26       1.33 
    ## 3 virginica           6.59          2.97        5.55       2.03

막대그래프를 그리기 쉬운 형태로 데이터를 변환합니다.

``` r
data2 <- data %>% 
  pivot_longer(
    cols = `꽃받침 길이`:`꽃잎 너비`, 
    names_to = "구분",
    values_to = "값"
  )

# 데이터 확인
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

## 기본 색상 적용된 그래프 그리기

그래프를 그리기 전에 차트에서 사용할 글꼴과 텍스트 크기를 먼저
설정합니다. 한글이 포함된 그래프를 그릴 때는 한글을 안정적으로 지원하는
글꼴을 사용하는 편이 좋습니다. 여기서는 `showtext` 패키지를 사용해 KoPub
돋움체 글꼴을 등록하고, 그래프 출력 시 해당 글꼴이 적용되도록
설정합니다.

`showtext_auto()`는 그래프를 그릴 때 자동으로 `showtext`를 적용하도록
지정합니다. `showtext_opts(dpi = 300)`은 고해상도 출력에서도 글자가
또렷하게 보이도록 설정합니다. 이후 차트 전체에서 사용할 기본 글자 크기를
`theme.size`로 정하고, 이를 `ggplot`에서 사용하는 단위로 변환해
`text.size`로 저장합니다. 이렇게 설정해 두면 이후 그래프에서 텍스트
크기를 일관되게 조절할 수 있습니다.

``` r
# 패키지 로드
library(showtext)

# 글꼴 설정
font_add("kopub", "C:/Users/.../appdata/local/microsoft/windows/fonts/kopub dotum medium.ttf")

showtext_auto()
showtext_opts(dpi=300)

theme.size = 14
text.size = theme.size / .pt
```

먼저 색상을 따로 지정하지 않은 상태에서 그래프를 그립니다.
`fill = 종류`를 지정하면 `ggplot2`가 기본 색상을 자동으로 적용합니다. 기본 색상만 사용해도 품종 간 차이는 충분히 구분됩니다. 하지만 보고서나 발표자료에서는 특정 색상으로 직접 바꿔줘야 하는 경우가 있습니다.

``` r
# 차트 그리기
base <- ggplot(data = data2, 
       aes(x = 구분, y = 값, group = 종류, fill = 종류)) +
  geom_col(position=position_dodge(width = 0.7), width = 0.6) +
  geom_text(aes(label=scales::comma(값, accuracy = .1)), 
            position=position_dodge(width = 0.7), 
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

base
```

<img src="{{ site.baseurl }}/assets/images/2026-01-05-setting-colors/unnamed-chunk-4-1.png" width="100%" style="display: block; margin: auto;" alt="기본 색상 적용된 그래프" />

## RColorBrewer 팔레트 사용하기

`RColorBrewer`는 미리 정리된 색상 조합을 제공하는 패키지로, 데이터의
성격과 시각화 목적에 맞는 색상을 비교적 안전하게 선택할 수 있도록
돕습니다. 색상을 직접 고르지 않아도 되기 때문에, 색상 선택에 익숙하지
않은 경우에도 그래프의 가독성을 유지하기 쉽습니다.

`RColorBrewer`에서 제공하는 팔레트는 크게 세 가지 유형으로 나뉩니다.
- 연속형 팔레트는 값이 낮은 곳에서 높은 곳으로 자연스럽게 이어지는
데이터를 표현할 때 사용합니다. 온도, 판매량, 인구 수처럼 크기의 흐름이
중요한 경우에 적합합니다.
- 발산형 팔레트는 특정 기준값을 중심으로 양쪽
방향의 차이를 강조할 때 사용합니다. 예를 들어 0을 기준으로 음수와 양수를 구분하거나, 기준 대비 증감 여부를 보여줄 때 활용합니다.
- 범주형 팔레트는 순서가 없는 범주를 구분할 때 사용합니다. 꽃의 종류, 제품 유형,
지역 구분처럼 크기나 방향보다 구분 자체가 중요한 경우에 적합합니다.

이번 예제에서는 iris 데이터의 품종처럼 범주 간 순서가 없는 데이터를
다룹니다. 따라서 연속형이나 발산형 팔레트보다는 범주형 팔레트를 사용하는
것이 적합합니다.

어떤 팔레트가 있는지 감이 오지 않는다면 `display.brewer.all()` 함수를
사용해 확인할 수 있습니다. 이 함수는 `RColorBrewer`에서 제공하는 모든
팔레트를 한 화면에 보여주며, 각 팔레트의 이름과 색상 구성을 함께 확인할
수 있게 해줍니다. 팔레트 이름만 보고 선택하기 어려울 때 특히 유용합니다.
또한 [colorbrewer2 웹사이트](https://colorbrewer2.org)를 참고하면,
데이터 유형별로 추천 팔레트를 직관적으로 살펴볼 수 있습니다.

먼저 사용 가능한 팔레트를 확인합니다.

``` r
# 패키지 로드
library(RColorBrewer)

# 팔레트 보기
display.brewer.all()
```

<img src="{{ site.baseurl }}/assets/images/2026-01-05-setting-colors/unnamed-chunk-5-1.png" width="100%" style="display: block; margin: auto;" alt="RColorBrewer 팔레트 종류" />

이제 범주형 팔레트 중 하나인 `Pastel2`를 적용합니다.
`scale_fill_brewer()`는 범주형 데이터를 색상으로 구분할 때 사용하는
함수입니다. 팔레트 이름만 바꾸면 전체 색상을 쉽게 바꿀 수 있습니다.

``` r
base +
  scale_fill_brewer(palette = "Pastel2")
```

<img src="{{ site.baseurl }}/assets/images/2026-01-05-setting-colors/unnamed-chunk-6-1.png" width="100%" style="display: block; margin: auto;" alt="팔레트 적용한 그래프" />

## 직접 색상 지정하기

디자인 가이드나 발표자료 테마에 맞춰 색상을 직접 지정해야 할 때도
있습니다. 이제 색상 팔레트를 사용하는 대신, 각 범주에 원하는 색상을 직접
지정해 봅니다. 이때 사용하는 함수가 `scale_fill_manual()`입니다. 이
함수는 범주형 변수의 값과 색상을 1:1로 연결해 설정할 때 사용합니다.

`values` 인자에는 범주 이름과 색상 코드를 쌍으로 지정합니다. 여기서는
꽃의 종류인 setosa, versicolor, virginica에 각각 보라색, 황토색,
민트색을 할당합니다. 범주 이름은 데이터에 실제로 들어 있는 값과 정확히
같아야 하며, 이름이 다르면 색상이 적용되지 않습니다.

``` r
base +
  scale_fill_manual(values=c("setosa" = "#C072ED", 
                             "versicolor" = "#EDC272", 
                             "virginica" = "#72EDAC"
                             )
                    )
```

<img src="{{ site.baseurl }}/assets/images/2026-01-05-setting-colors/unnamed-chunk-7-1.png" width="100%" style="display: block; margin: auto;" alt="직접 색상 지정한 그래프" />

이번 글에서는 `ggplot2`로 그래프를 그릴 때 색상을 설정하는 방법을
살펴봤습니다. 먼저 기본 색상이 어떻게 적용되는지 확인하고,
`RColorBrewer` 팔레트를 사용해 범주형 데이터를 안정적으로 구분하는
방법을 정리했습니다. 이어서 `scale_fill_manual()`을 사용해 색상을 직접
지정하는 방법도 함께 살펴봤습니다.

이 방법은 막대그래프에만 국한되지 않습니다. 선그래프, 점그래프, 영역
그래프처럼 `color`나 `fill`을 사용하는 다른 차트에서도 같은 방식으로
적용할 수 있습니다. 차트의 종류가 달라져도 색상을 설정하는 원리는 크게
달라지지 않습니다.

빠르게 결과를 확인할 때는 기본 색상이나 팔레트를 활용하고, 보고서나 발표
자료처럼 색상 규칙이 필요한 경우에는 직접 색상을 지정해 보세요!