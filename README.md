---
title: "백화점 유인 효과 분석"
subtitle: "기상 조건을 중심으로"
author: "HAPPY BEAN"
date: '2019 12 26'
output: 
  html_document:
    keep_md: true
    theme: simplex
    # theme: dark
    code_folding: show
    toc: true
    number_section: true
    toc_float:
      collapse: false
      smooth_controll: false
editor_options: 
  chunk_output_type: console
---



***
# 프로젝트 정의

- 미세먼지 비상저감 조치 발령된 2019년 3월, 롯데 백화점 매출 9.1%, 구매 고객 18.8% 증가
- 과연 미세먼지가 많은 날 사람들은 백화점을 더 많이 갈까? 많이 간다면 어느 정도 많이 가는가?
- 미세먼지와 초미세먼지에 대한 반응이 다를까?
- 먼지 이외의 다양한 기상 현상들에 대해서는 어떻게 반응할까?

***
# 수집 데이터
분석을 위해 수집한 데이터셋은 아래의 표에 정리

자료 제공처 | 수집 데이터 
------ | ------
서울열린데이터광장 | 서울 생활인구 내국인
" | 일별 평균 대기오염도
" | 강수량 및 강우일수 통계
" | 지하철 역별 승하차 인원 정보
" | 구별 용지 비율
" | 구별 인구 밀도
" | 구별 유통업체 현황
기상청 기상자료개방포털 | 시간별 강수량, 풍량, 풍속, 적설량
통계청 | 전국 집계 구역
주요 3사 백화점 | 롯데, 현대, 신세계 백화점 주소 (25개소)

각 데이터셋 별로 분석에 사용한 시간 기준은 아래의 표에 명시

종류 | 기간
------ | ------
생활 인구 | 17년 1월 1일 ~ 19년 11월 30일
백화점 주소 | 2019년 11월 30일 기준
백화점 면적 | 2018년 기준
용지 면적 | 2018년 기준
인구 밀도 | 2018년 기준
지하철 역별 하차 인원 | 17년 1월 1일 ~ 19년 11월 30일
일평균 대기 오염 | 17년 1월 1일 ~ 19년 11월 30일
일평균 기상 현황 | 17년 1월 1일 ~ 19년 11월 30일



***
# 데이터 설명 및 전처리
## 내국인 생활 인구 데이터

- KT와 서울시에서 제공
- 19,153개의 집계 구역의 1시간 단위 측정
- KT LTE 가입자 신호 위치 기반 **추정**
- 본 프로젝트에서는 *백화점 포함 집계 구역의 생활 인구를 유동 인구로 가정*
- 출근, 거주 인구 등의 노이즈를 최소화하기 위해 주말 백화점 개점 시간만 분석

# 분석 대상 백화점 선정
## 국내 백화점 상위 3사

- 배제 기준: 대중교통시설 포함 집계 구역 (기차, 공항, 버스)
- 하나의 집계구에 여러 백화점이 있는 경우는 총 면적 합산
- 최종 분석 대상: 16개 집계 구역 식별

<img src="./figure/departments.jpeg" width="100%" />

사명 | 지점명
------ | ------
신세계 | 영등포
현대 | 디큐브시티, 천호, 무역센터(유플렉스), 압구정, 신촌(유플렉스), 목동(유플렉스), 미아
롯데 | 잠실, 명동/본점/에비뉴엘, 노원, 강남, 스타시티, 관악, 에비뉴엘월드타워, 미아

## 일별 평균 대기오염도

- 구별 대기 측정소에서 일별 대기 오염도 측정(이산화탄소, 오존 등의 수치는 제외)

- 백화점이 속해 있는 구를 기준으로 미세먼지 수치 정의

- 미세먼지(PM10) 수치와 초미세먼지(PM2.5) 수치 (단위: ㎍/m³)

- WHO 미세먼지/초미세먼지 8단계 기준 적용

## 강수량 및 기상 조건

- 1시간 단위로 측정된 데이터는 일평균으로 계산

- 종로구 송월동에 있는 서울기상관측소 측정값 기준

- 강수량, 풍속, 적설량 등 포함

## 지하철 역별 승하차 인원 정보

- 백화점 상권의 대중교통 시설 발전 정도를 통제하기 위해 하차 인원을 추가

- 백화점 홈페이지의 오시는 길에 등장하는 지하철 역 중 도보 최단 시간 기준 하차 인원

## 구별 용지 비율 및 인구 밀도

- 백화점이 속한 구의 속성 정보를 추가

- 해당 구의 주거지, 상업지, 녹지의 비율, 인구 밀도(명/㎢) 

## 구별 유통업체 현황

- 백화점 규모에 따른 유동 인구 영향을 고려하기 위해 추가

- 하나의 집계 구역에 여러 백화점이 포함되는 경우, 합산하여 처리

***
# 변수 설정

**종속변수: 백화점이 있는 집계 구역의 일평균 생활 인구(주말/개점 시간)**

**독립변수**

구분 | 설명 | 변수명
----- |------ | -----
백화점 요소 | 백화점 총 면적(m²) | size
지역 환경 요소 | 지하철역 하차 인원(명) | arrival
" | 주거지 비율(%) | residential_area
" | 상업지 비율(%) | commercial_area
" | 인구 밀도(명/㎢) | pop_density
기상 조건 요소 | 일평균 미세먼지(㎍/m³) | fine_dust
" | 일평균 초미세먼지(㎍/m³) | hyper_dust
" | 일평균 강수량(mm) | mean_precipitation
" | 일평균 기온(°C) | mean_temperature
" | 일평균 풍속(m/s) | mean_wind
" | 일평균 적설량(cm) | mean_snow
" | 미세먼지 등급 | fine_dust_grade
" | 초미세먼지 등급 | hyper_dust_grade

***
# 전처리 완료 데이터셋


```r
url = "https://raw.githubusercontent.com/jonghyunlee1993/Multicampus_semi/master/Working/proc/final_df_for_linear_model.csv"
df = read.csv(url, fileEncoding = "UTF-8")[, -1]
df = df %>% select(date, code, mean_pop, size, residential_area, commercial_area, pop_density, arrival, fine_dust, hyper_dust, fine_dust_grade, hyper_dust_grade, mean_temp, mean_precipi, mean_wind, mean_snow) %>% mutate(code = as.factor(code))

summary(df)
```

```
##       date                     code         mean_pop          size       
##  Min.   :20170101   1102052020001: 268   Min.   :   76   Min.   : 18578  
##  1st Qu.:20170824   1105066011201: 268   1st Qu.: 3540   1st Qu.: 70475  
##  Median :20180666   1108068010004: 268   Median : 7681   Median : 90163  
##  Mean   :20180091   1109071040007: 268   Mean   :10442   Mean   :160502  
##  3rd Qu.:20190308   1111066030001: 268   3rd Qu.:13949   3rd Qu.:147194  
##  Max.   :20191130   1113075030010: 268   Max.   :54512   Max.   :807686  
##                     (Other)      :2680                                   
##  residential_area commercial_area    pop_density       arrival      
##  Min.   :0.3290   Min.   :0.01144   Min.   :13618   Min.   :  1024  
##  1st Qu.:0.4957   1st Qu.:0.01532   1st Qu.:16198   1st Qu.: 18762  
##  Median :0.6102   Median :0.03688   Median :18218   Median : 29984  
##  Mean   :0.5769   Mean   :0.05977   Mean   :18258   Mean   : 35142  
##  3rd Qu.:0.6207   3rd Qu.:0.05247   3rd Qu.:19883   3rd Qu.: 49964  
##  Max.   :0.8730   Max.   :0.39218   Max.   :26894   Max.   :143134  
##                                                                     
##    fine_dust        hyper_dust     fine_dust_grade hyper_dust_grade
##  Min.   :  3.00   Min.   :  1.00   Min.   :1.000   Min.   :1.000   
##  1st Qu.: 23.00   1st Qu.: 12.00   1st Qu.:2.000   1st Qu.:2.000   
##  Median : 36.00   Median : 20.00   Median :3.000   Median :3.000   
##  Mean   : 40.92   Mean   : 24.05   Mean   :3.312   Mean   :3.615   
##  3rd Qu.: 54.00   3rd Qu.: 31.00   3rd Qu.:5.000   3rd Qu.:5.000   
##  Max.   :243.00   Max.   :106.00   Max.   :8.000   Max.   :8.000   
##                                                                    
##    mean_temp        mean_precipi      mean_wind        mean_snow      
##  Min.   :-10.429   Min.   : 0.000   Min.   :0.7833   Min.   :0.00000  
##  1st Qu.:  4.749   1st Qu.: 0.000   1st Qu.:1.4698   1st Qu.:0.00000  
##  Median : 15.992   Median : 0.000   Median :1.8104   Median :0.00000  
##  Mean   : 13.896   Mean   : 0.394   Mean   :1.9320   Mean   :0.08191  
##  3rd Qu.: 23.309   3rd Qu.: 0.100   3rd Qu.:2.3021   3rd Qu.:0.00000  
##  Max.   : 31.550   Max.   :12.136   Max.   :3.8500   Max.   :4.62941  
## 
```

***
# 모델 정의 및 검증
## 일평균 유동 인구

정규화를 위해 log 변환을 실시

![](Project_Report_files/figure-html/histogram-1.png)<!-- -->

## 모델 정의

백화점 요소, 지역 환경 요소, 기상 조건 요소를 포함한 선형 모델 생성
해당 모델의 유의 변수 추출(step function: stepwise method)


```r
model = lm(log(mean_pop) ~ size + residential_area + commercial_area + pop_density + arrival + fine_dust_grade * hyper_dust_grade + mean_precipi + mean_temp + mean_snow + mean_wind, data = df)

step(model, direction = "both")
```

```
## Start:  AIC=-1333.8
## log(mean_pop) ~ size + residential_area + commercial_area + pop_density + 
##     arrival + fine_dust_grade * hyper_dust_grade + mean_precipi + 
##     mean_temp + mean_snow + mean_wind
## 
##                                    Df Sum of Sq    RSS      AIC
## - fine_dust_grade:hyper_dust_grade  1      0.06 3122.8 -1335.72
## - mean_wind                         1      0.12 3122.8 -1335.64
## - mean_precipi                      1      0.32 3123.0 -1335.36
## - mean_snow                         1      0.37 3123.1 -1335.29
## - mean_temp                         1      0.55 3123.3 -1335.05
## <none>                                          3122.7 -1333.80
## - arrival                           1     38.90 3161.6 -1282.72
## - residential_area                  1     50.78 3173.5 -1266.64
## - pop_density                       1    161.19 3283.9 -1119.99
## - commercial_area                   1    261.64 3384.4  -990.79
## - size                              1    543.72 3666.4  -647.50
## 
## Step:  AIC=-1335.72
## log(mean_pop) ~ size + residential_area + commercial_area + pop_density + 
##     arrival + fine_dust_grade + hyper_dust_grade + mean_precipi + 
##     mean_temp + mean_snow + mean_wind
## 
##                                    Df Sum of Sq    RSS      AIC
## - mean_wind                         1      0.15 3122.9 -1337.51
## - mean_precipi                      1      0.32 3123.1 -1337.28
## - mean_snow                         1      0.39 3123.2 -1337.18
## - mean_temp                         1      0.54 3123.3 -1336.98
## <none>                                          3122.8 -1335.72
## - hyper_dust_grade                  1      2.13 3124.9 -1334.79
## - fine_dust_grade                   1      2.64 3125.4 -1334.10
## + fine_dust_grade:hyper_dust_grade  1      0.06 3122.7 -1333.80
## - arrival                           1     38.93 3161.7 -1284.59
## - residential_area                  1     50.84 3173.6 -1268.46
## - pop_density                       1    161.28 3284.1 -1121.78
## - commercial_area                   1    261.87 3384.6  -992.42
## - size                              1    543.81 3666.6  -649.33
## 
## Step:  AIC=-1337.51
## log(mean_pop) ~ size + residential_area + commercial_area + pop_density + 
##     arrival + fine_dust_grade + hyper_dust_grade + mean_precipi + 
##     mean_temp + mean_snow
## 
##                                    Df Sum of Sq    RSS      AIC
## - mean_precipi                      1      0.30 3123.2 -1339.09
## - mean_snow                         1      0.39 3123.3 -1338.97
## - mean_temp                         1      0.46 3123.4 -1338.87
## <none>                                          3122.9 -1337.51
## - hyper_dust_grade                  1      2.66 3125.6 -1335.86
## + mean_wind                         1      0.15 3122.8 -1335.72
## + fine_dust_grade:hyper_dust_grade  1      0.10 3122.8 -1335.64
## - fine_dust_grade                   1      3.04 3126.0 -1335.34
## - arrival                           1     38.84 3161.8 -1286.50
## - residential_area                  1     50.85 3173.8 -1270.25
## - pop_density                       1    161.35 3284.3 -1123.49
## - commercial_area                   1    262.12 3385.1  -993.90
## - size                              1    543.67 3666.6  -651.32
## 
## Step:  AIC=-1339.09
## log(mean_pop) ~ size + residential_area + commercial_area + pop_density + 
##     arrival + fine_dust_grade + hyper_dust_grade + mean_temp + 
##     mean_snow
## 
##                                    Df Sum of Sq    RSS      AIC
## - mean_snow                         1      0.33 3123.6 -1340.65
## - mean_temp                         1      0.40 3123.6 -1340.54
## <none>                                          3123.2 -1339.09
## + mean_precipi                      1      0.30 3122.9 -1337.51
## + mean_wind                         1      0.13 3123.1 -1337.28
## + fine_dust_grade:hyper_dust_grade  1      0.09 3123.1 -1337.22
## - hyper_dust_grade                  1      2.84 3126.1 -1337.20
## - fine_dust_grade                   1      3.27 3126.5 -1336.60
## - arrival                           1     38.59 3161.8 -1288.44
## - residential_area                  1     50.92 3174.2 -1271.74
## - pop_density                       1    161.35 3284.6 -1125.10
## - commercial_area                   1    262.44 3385.7  -995.13
## - size                              1    543.43 3666.7  -653.24
## 
## Step:  AIC=-1340.65
## log(mean_pop) ~ size + residential_area + commercial_area + pop_density + 
##     arrival + fine_dust_grade + hyper_dust_grade + mean_temp
## 
##                                    Df Sum of Sq    RSS      AIC
## - mean_temp                         1      0.27 3123.8 -1342.28
## <none>                                          3123.6 -1340.65
## + mean_snow                         1      0.33 3123.2 -1339.09
## + mean_precipi                      1      0.24 3123.3 -1338.97
## + mean_wind                         1      0.14 3123.4 -1338.84
## + fine_dust_grade:hyper_dust_grade  1      0.11 3123.4 -1338.79
## - hyper_dust_grade                  1      2.82 3126.4 -1338.77
## - fine_dust_grade                   1      3.22 3126.8 -1338.24
## - arrival                           1     38.55 3162.1 -1290.06
## - residential_area                  1     50.94 3174.5 -1273.28
## - pop_density                       1    161.36 3284.9 -1126.66
## - commercial_area                   1    262.44 3386.0  -996.70
## - size                              1    543.34 3666.9  -654.96
## 
## Step:  AIC=-1342.28
## log(mean_pop) ~ size + residential_area + commercial_area + pop_density + 
##     arrival + fine_dust_grade + hyper_dust_grade
## 
##                                    Df Sum of Sq    RSS      AIC
## <none>                                          3123.8 -1342.28
## + mean_temp                         1      0.27 3123.6 -1340.65
## + mean_precipi                      1      0.20 3123.6 -1340.55
## + mean_snow                         1      0.19 3123.6 -1340.54
## + mean_wind                         1      0.08 3123.7 -1340.39
## + fine_dust_grade:hyper_dust_grade  1      0.08 3123.7 -1340.39
## - hyper_dust_grade                  1      2.86 3126.7 -1340.35
## - fine_dust_grade                   1      2.97 3126.8 -1340.20
## - arrival                           1     38.52 3162.3 -1291.73
## - residential_area                  1     51.01 3174.8 -1274.82
## - pop_density                       1    161.52 3285.3 -1128.11
## - commercial_area                   1    262.41 3386.2  -998.40
## - size                              1    543.23 3667.1  -656.77
```

```
## 
## Call:
## lm(formula = log(mean_pop) ~ size + residential_area + commercial_area + 
##     pop_density + arrival + fine_dust_grade + hyper_dust_grade, 
##     data = df)
## 
## Coefficients:
##      (Intercept)              size  residential_area   commercial_area  
##        7.652e+00         2.591e-06        -9.435e-01         3.252e+00  
##      pop_density           arrival   fine_dust_grade  hyper_dust_grade  
##        7.058e-05        -5.795e-06         3.181e-02        -2.693e-02
```

아래와 같이 최종 모델 도출

$$
log(Pop) = \beta_1 \cdot Size + \beta_2 \cdot Resid + \beta_3 \cdot Commerce + \beta_4 \cdot Density + \beta_5 \cdot Arrival +
\beta_6 \cdot Fine + \beta_7 \cdot Hyper + \beta_0 + \epsilon
$$

- Pop      : 주말 백화점 개점 시간(10 ~ 20시) 유동 인구의 수
- Size     : 백화점의 총 면적
- Resid    : 백화점이 속한 구의 전체 용지 면적 대비 주거지 비율
- Commerce : 백화점이 속한 구의 전체 용지 면적 대비 상업지 비율
- Density  : 백화점이 속한 구의 인구 밀도 
- Arrival  : 백화점과 가장 가까운 지하철의 일평균 하차 인원 수
- Fine     : 백화점이 속한 구의 일평균 미세먼지의 등급
- Hyper    : 백화점이 속한 구의 일평균 초미세먼지의 등급


```r
model_step = lm(formula = log(mean_pop) ~ size + residential_area + commercial_area + pop_density + arrival + fine_dust_grade + hyper_dust_grade, data = df)

summary(model_step)
```

```
## 
## Call:
## lm(formula = log(mean_pop) ~ size + residential_area + commercial_area + 
##     pop_density + arrival + fine_dust_grade + hyper_dust_grade, 
##     data = df)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -4.1899 -0.4539 -0.0320  0.5766  2.5018 
## 
## Coefficients:
##                    Estimate Std. Error t value Pr(>|t|)    
## (Intercept)       7.652e+00  9.789e-02  78.174  < 2e-16 ***
## size              2.591e-06  9.498e-08  27.282  < 2e-16 ***
## residential_area -9.435e-01  1.129e-01  -8.360  < 2e-16 ***
## commercial_area   3.252e+00  1.715e-01  18.961  < 2e-16 ***
## pop_density       7.058e-05  4.745e-06  14.876  < 2e-16 ***
## arrival          -5.795e-06  7.977e-07  -7.264 4.43e-13 ***
## fine_dust_grade   3.181e-02  1.577e-02   2.017   0.0438 *  
## hyper_dust_grade -2.693e-02  1.361e-02  -1.979   0.0479 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.8543 on 4280 degrees of freedom
## Multiple R-squared:  0.3366,	Adjusted R-squared:  0.3355 
## F-statistic: 310.2 on 7 and 4280 DF,  p-value: < 2.2e-16
```

해당 모델은 데이터의 약 33.6% (Adj. R square)를 설명하고 있으며 미세먼지, 초미세먼지 변수가 모두 유의

종속 변수에 log 변환을 실시하였기 때문에 모델은 **독립 변수가 1단위 증가할 때의 종속 변수의 변화량**으로 해석

백화점의 규모가 클수록, 해당 구의 상업지 비율이 높을수록, 인구 밀도가 높을수록 백화점 유동 인구가 증가

**미세먼지의 경우, 등급이 1 증가할 때 백화점을 방문한 유동 인구는 3.18% 증가하는 반면 초미세먼지의 경우, 2.69% 감소**

## 잔차 분석 및 다중 공선성

잔차의 그래프를 확인하였을 때, 잔차가 정규성을 잘 따르고 있으며 이상치가 없음


```
## `geom_smooth()` using method = 'gam' and formula 'y ~ s(x, bs = "cs")'
## `geom_smooth()` using method = 'gam' and formula 'y ~ s(x, bs = "cs")'
## `geom_smooth()` using method = 'gam' and formula 'y ~ s(x, bs = "cs")'
```

![](Project_Report_files/figure-html/residual_plot-1.png)<!-- -->

다중공선성 여부를 확인하기 위해 분산 팽창 정도를 확인


```r
vif(model_step)
```

```
##             size residential_area  commercial_area      pop_density 
##         1.965084         1.197922         1.388818         1.463388 
##          arrival  fine_dust_grade hyper_dust_grade 
##         1.809195         3.724830         3.719776
```

변수들간의 공선성은 확인되지 않음

## 모델 유의성 검증
### 다항 회귀
모델의 유의성을 검증하기 위해 10겹 교차 검증(10-fold cross validation)을 진행


```r
train_model_lm = function(start, end, mape_res){
  
  # train - test split
  test_data = df[start:end, ]
  train_data = df[-c(start:end), ]
  
  # model fitting
  model = lm(formula = log(mean_pop) ~ size + residential_area + commercial_area + pop_density + arrival + fine_dust_grade + hyper_dust_grade, data = train_data)
  
  # calculate distance
  distPred <- predict(model, test_data)
  actuals_preds <- data.frame(cbind(actuals = log(test_data$mean_pop), predicteds = distPred))
  
  # calculate MAPE 
  mape <- mean(abs((actuals_preds$predicteds - actuals_preds$actuals))/actuals_preds$actuals) * 100
  mape_res = rbind(mape_res, as.data.frame(mape))
  
  return(mape_res)
}

data_num = nrow(df) %/% 10
mape_res = NULL

for (row_idx in 1:10){
  start = 1 + (row_idx - 1) * data_num
  end   = row_idx * data_num

  mape_res = train_model_lm(start, end, mape_res)
}

mean_error_lm = mean(mape_res$mape)
print(mean_error_lm)
```

```
## [1] 7.883955
```

MAPE는 비율 에러를 측정하는 방법으로  $A_t$는 실제 값, $F_t$는 예측 값을 의미

$$
MAPE = \frac{100}{n} \sum_{t=1}^{n}|\frac{A_t - F_t}{A_t}|
$$

해당 모델의 MAPE는 7.88%로 상당 부분 예측을 잘 수행하고 있음

### 인공 신경망
추가적으로 유의 변수들을 이용해 인공 신경망 10겹 교차 검증(10-fold cross validation)을 진행


```r
train_model_nnet = function(start, end, mape_res){
  
  test_pop = data.frame(mean_pop = df[start:end, c(3)])
  train_pop = data.frame(mean_pop = df[-c(start:end), c(3)])
  
  test_data = cbind(test_pop, as.data.frame(sapply(df[start:end, c(4:8, 11:12)], scale)))
  train_data = cbind(train_pop, as.data.frame(sapply(df[-c(start:end), c(4:8, 11:12)], scale)))
  
  nnet_model = nnet(log(mean_pop) ~ ., train_data, size = 50, 
                    linout = T, maxit = 100, trace = F)
  nnet_pred = predict(nnet_model, test_data)
  
    actuals_preds <- data.frame(cbind(log(test_data$mean_pop), nnet_pred))
    names(actuals_preds) = c("actuals", "predicteds")
  
  mape <- mean(abs((actuals_preds$predicteds - actuals_preds$actuals))/actuals_preds$actuals) * 100
  
  mape_res = rbind(mape_res, as.data.frame(mape))
  
  return(mape_res)
}

data_num = nrow(df) %/% 10
mape_res = NULL

for (row_idx in 1:10){
  
  start = 1 + (row_idx - 1) * data_num
  end   = row_idx * data_num
  
  mape_res = train_model_nnet(start, end, mape_res)
}

mean_error_nnet = mean(mape_res$mape)
print(mean_error_nnet)
```

```
## [1] 2.467924
```

이 경우 역시 오차 역시 2.47%로 상당히 잘 예측하고 있음

***
# 논의

본 프로젝트는 서울시 생활인구 데이터를 비롯한 공개 데이터로 기상 현상이 백화점 방문 고객 수에 미치는 영향력을 분석하였다. 본 프로젝트는 기온, 강수량, 적설량, 풍속 등의 기상 조건은 백화점 방문 고객 수의 증감에 유의한 영향을 주지 않으나 미세먼지와 초미세먼지의 경우, 유의한 영향을 미친다는 점을 확인할 수 있었다. 흥미롭게도 백화점 유동 인구 반응은 미세먼지와 초미세먼지에 대해서 다르게 나타났다. 미세먼지 등급과 백화점 유동 인구는 양의 관계가 있으나, 초미세먼지 등급은 음의 관계가 있었다.  

*[미세먼지 1등급 증가당 유동 인구 3.18% 증가 / 초미세먼지 1등급 증가당 유동 인구 2.69% 감소]* 

본 프로젝트에서 수립한 모델은 10-fold 교차 검증에서 MAPE 7.88% 정도로 우수하였으며, 실제 백화점 유동 인구 예측에도 사용이 가능할 것으로 보인다. 향후 연령대별, 성별 반응의 차이를 검증하여 세분화된 고객 세그먼트(Segment)로 확장이 가능할 것이다.^[서울시 생활 인구 데이터는 각 연령별, 성별 생활 인구를 제공한다.]

그러나 본 프로젝트의 한계점 역시 명확하다. 종속변수로 사용한 서울시 생활 인구 데이터셋은 KT의 LTE 통신망 사용자의 수를 바탕으로 추정한 결과이다. 때문에 실제와 차이가 발생할 가능성이 존재한다. 이 검증을 위해서 19년 12월 9일 공개된 SKT 데이터 허브의 유동 인구 데이터와의 비교 검증을 진행할 예정이다. ^[분석 도중 새벽 시간대의 이상 수치를 확인한 바 있다. 예를 들어 약 500명 정도로 유지되던 생활인구가 7개월 정도만 1,500명으로 급증했다가 이후 다시 500명 선으로 복귀되는 경향을 확인하였다.] 또한 본 프로젝트에서는 백화점이 포함된 집계 구역의 생활 인구를 유동 인구로 가정하였다. 이는 거주 인구, 출퇴근 인구 등을 구분하기 어렵다는 문제가 있는데 이를 최소화하기 위해 출근을 하지 않는 주말의 백화점 개점 시간만을 분석 대상으로 포함하였다. 추가적으로 미세먼지와 초미세먼지의 측정소 문제도 발생할 가능성이 있다. 본 프로젝트에서는 백화점이 소재한 구의 미세먼지, 초미세먼지의 수치를 해당 백화점의 미세먼지, 초미세먼지 수준으로 정의하였다. 실제로 대기 오염은 측정소의 위치, 고도 등에 민감하게 반응할 수 있다. 발생 가능한 수치 왜곡을 최소화하기 위해 일평균 미세먼지/초미세먼지 수치를 등급으로 환산하여 적용하였다. 서울의 대기오염 정도를 바탕으로 보간하여 각 행정동별 대기오염 정도를 계산하는 방법을 확인하였으나 해당 방법은 본 프로젝트의 중요 변수인 미세먼지/초미세먼지 값의 신뢰성을 저하시킬 것으로 우려하여 적용하지 않았다.^[내삽법 등의 지리적 특성을 고려한 보간 방법] 향후 대기오염 측정소가 많아져 데이터가 더욱 정교해진다면 좀 더 구체적인 모델을 수립할 수 있을 것이다. 

본 프로젝트는 공개 데이터를 바탕으로 여러 요인들 중 미세먼지와 초미세먼지의 등급이 백화점 유동 인구에 유의한 영향을 미친다는 점을 확인하였다. 흥미롭게도 미세먼지와 초미세먼지에 대한 반응이 달랐으며 이 부분은 향후 깊이있는 탐색이 필요할 것으로 보인다. 

***
# 참고 문헌
[미세먼지에 의한 대기질 악화가 서울시 상권 매출액에 미치는 영향 분석](https://bigdata.seoul.go.kr/noti/selectNoti.do?r_id=P440&bbs_seq=170&sch_type=&sch_text=&currentPage=1)

***
# 프로젝트 github 링크
[Github Repository](https://github.com/jonghyunlee1993/Multicampus_semi)
