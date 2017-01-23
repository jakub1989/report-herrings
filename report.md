# Herrings length report
Jakub Kwitowski  
22 stycznia 2017  

# ANALIZA PROBLEMU
W związku z różnego rodzaju czynnikami zauważono stopniowy spadek rozmiaru śledzia oceanicznego wyławianego w Europie. Analizie został poddany zbiór danych opisujacy pomiary śledzi i warunki w jakich żyły z ostatnich 60 lat. Celem raportu jest próba odnalezienia zależności, które mają wpływ na rozmiar śledzia.


# ANALIZA DANYCH
Dane były pobierane z połowów komercyjnych jednostek. W ramach połowu jednej jednostki losowo wybierano od 50 do 100 sztuk trzyletnich śledzi. Zbiór składa się z 52583 wierszy ułożonych chronologicznie.

Opisy kolumn występujących w zbiorze

| Kolumna        | Opis                                                                 |
| ------------- |:---------------------------------------------------------------------|
| length      | długość złowionego śledzia [cm] |
| cfin1      | dostępność planktonu [zagęszczenie Calanus finmarchicus gat. 1];      |
| cfin2 | dostępność planktonu [zagęszczenie Calanus finmarchicus gat. 2];      |
| chel1 | dostępność planktonu [zagęszczenie Calanus helgolandicus gat. 1];      |
| chel2 | dostępność planktonu [zagęszczenie Calanus helgolandicus gat. 2];      |
| lcop1 | dostępność planktonu [zagęszczenie widłonogów gat. 1];     |
| lcop2 | dostępność planktonu [zagęszczenie widłonogów gat. 2];      |
| fbar | natężenie połowów w regionie [ułamek pozostawionego narybku];      |
| recr | roczny narybek [liczba śledzi];      |
| cumf | łączne roczne natężenie połowów w regionie [ułamek pozostawionego narybku];      |
| totaln | łączna liczba ryb złowionych w ramach połowu [liczba śledzi];     |
| sst | temperatura przy powierzchni wody [°C];      |
| sal | poziom zasolenia wody [Knudsen ppt]; |
| xmonth | miesiąc połowu [numer miesiąca];     |
| nao |oscylacja północnoatlantycka [mb]. |



#BIBLIOTEKI

Analiza danych została wykonana w oparciu o poniższe biblioteki 

```r
library(readr)
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
library(knitr)
library(shiny)
library(ggplot2)
library(corrplot)
library(caret)
```

```
## Loading required package: lattice
```

```r
library(randomForest)
```

```
## randomForest 4.6-12
```

```
## Type rfNews() to see new features/changes/bug fixes.
```

```
## 
## Attaching package: 'randomForest'
```

```
## The following object is masked from 'package:ggplot2':
## 
##     margin
```

```
## The following object is masked from 'package:dplyr':
## 
##     combine
```

oraz została zapewniona powtarzalność wyników

```r
set.seed(1)
```

#CZYSZCZENIE ZBIORU
Zbiór został wczytany a brakujące dane zostały zastąpione przez symbol "NA" w celu łatwiejszego późniejszego przetwarzania

```r
herrings <- read_csv("C:/Users/Jacob/Desktop/Raport/sledzie.csv", col_names = TRUE, na = c("?", "NA"))
```

```
## Parsed with column specification:
## cols(
##   X1 = col_integer(),
##   length = col_double(),
##   cfin1 = col_double(),
##   cfin2 = col_double(),
##   chel1 = col_double(),
##   chel2 = col_double(),
##   lcop1 = col_double(),
##   lcop2 = col_double(),
##   fbar = col_double(),
##   recr = col_integer(),
##   cumf = col_double(),
##   totaln = col_double(),
##   sst = col_double(),
##   sal = col_double(),
##   xmonth = col_integer(),
##   nao = col_double()
## )
```

Wszystkie pola NA zostały zastąpione średnią wartością z danej kolumny w której wystąpiła brakująca wartość

```r
herringsReplaceByMean <- herrings
columnMean <- colMeans(herringsReplaceByMean, na.rm=TRUE)
index <- which(is.na(herringsReplaceByMean), arr.ind=TRUE)
herringsReplaceByMean[index] <- columnMean[index[,2]]
```

#PODSUMOWANIE ZBIORU
Zbiór danych historycznych jest pokaźny. Zawiera szczegółowe dane statystyczne z dla każdego śledzia.

1. Liczba wierszy w zbiorze

```r
knitr::kable(nrow(herringsReplaceByMean))
```



------
 52582
------

2. Liczba kolumn w zbiorze

```r
knitr::kable(ncol(herringsReplaceByMean))
```



---
 16
---

3. Padsumowanie statystyczne:
Tabela zawiera wartości minimalne, maksymalne, mediany, średnie dla wszyskich zmienny w zbiorze

```r
knitr::kable(summary(herringsReplaceByMean))
```

           X1            length         cfin1             cfin2             chel1            chel2            lcop1              lcop2             fbar             recr              cumf             totaln             sst             sal            xmonth            nao         
---  --------------  -------------  ----------------  ----------------  ---------------  ---------------  -----------------  ---------------  ---------------  ----------------  ----------------  ----------------  --------------  --------------  ---------------  -----------------
     Min.   :    0   Min.   :19.0   Min.   : 0.0000   Min.   : 0.0000   Min.   : 0.000   Min.   : 5.238   Min.   :  0.3074   Min.   : 7.849   Min.   :0.0680   Min.   : 140515   Min.   :0.06833   Min.   : 144137   Min.   :12.77   Min.   :35.40   Min.   : 1.000   Min.   :-4.89000 
     1st Qu.:13145   1st Qu.:24.0   1st Qu.: 0.0000   1st Qu.: 0.2778   1st Qu.: 2.469   1st Qu.:13.589   1st Qu.:  2.5479   1st Qu.:17.808   1st Qu.:0.2270   1st Qu.: 360061   1st Qu.:0.14809   1st Qu.: 306068   1st Qu.:13.63   1st Qu.:35.51   1st Qu.: 5.000   1st Qu.:-1.89000 
     Median :26291   Median :25.5   Median : 0.1333   Median : 0.7012   Median : 6.083   Median :21.435   Median :  7.1229   Median :25.338   Median :0.3320   Median : 421391   Median :0.23191   Median : 539558   Median :13.86   Median :35.51   Median : 8.000   Median : 0.20000 
     Mean   :26291   Mean   :25.3   Mean   : 0.4458   Mean   : 2.0248   Mean   :10.006   Mean   :21.221   Mean   : 12.8108   Mean   :28.419   Mean   :0.3304   Mean   : 520367   Mean   :0.22981   Mean   : 514973   Mean   :13.87   Mean   :35.51   Mean   : 7.258   Mean   :-0.09236 
     3rd Qu.:39436   3rd Qu.:26.5   3rd Qu.: 0.3603   3rd Qu.: 1.9973   3rd Qu.:11.500   3rd Qu.:27.193   3rd Qu.: 21.2315   3rd Qu.:37.232   3rd Qu.:0.4560   3rd Qu.: 724151   3rd Qu.:0.29803   3rd Qu.: 730351   3rd Qu.:14.16   3rd Qu.:35.52   3rd Qu.: 9.000   3rd Qu.: 1.63000 
     Max.   :52581   Max.   :32.5   Max.   :37.6667   Max.   :19.3958   Max.   :75.000   Max.   :57.706   Max.   :115.5833   Max.   :68.736   Max.   :0.8490   Max.   :1565890   Max.   :0.39801   Max.   :1015595   Max.   :14.73   Max.   :35.61   Max.   :12.000   Max.   : 5.08000 


#ANALIZA WARTOŚCI ZMIENNYCH
Dla każdej zmiennej (oprócz porządkowej) możemy dokonać analizy histogramów i gęstości rozkładu wartości. Dzięki temu zaobserwowałem iż zbiór nie posiada wartości odstających dlatego nie ma konieczności podejmowania prób ich eliminacji. 


```r
for(i in names(herringsReplaceByMean)){
  ggplot(herringsReplaceByMean, aes(x=herringsReplaceByMean[i])) + geom_histogram(aes(fill = ..count..)) + scale_fill_gradient("Count", low = "#132b44", high = "#55b1f7") + labs(x=i, y="count")
  
  ggplot(herringsReplaceByMean, aes(x=herringsReplaceByMean[i])) + geom_density() + labs(x=i, y="density")
  
 }
```

#KORELACJA MIĘDZY ZMIENNYMI
Dzięki heatmapie czyli graficznemu przedstawienie korelacji między zmiennymi możemy zaobserować ciekawe zależności, które występują między zmiennymi w zbiorze.


```r
corrplot(cor(herringsReplaceByMean), type = "upper", order = "hclust", tl.col = "black", tl.srt = 45)
```

![](report_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

Wykres pokazuje dość silne trywialne korelacje dodatnie.

Zmiana natężenia połowów powoduje zmianę łącznego rocznego natężenie połowów
1.natężenie połowów w regionie [ułamek pozostawionego narybku
2.łączne roczne natężenie połowów w regionie

Zmiana zagęszczenia planktonu chel1 powoduje zmianę zagęszczenia planktonu lcop1
1.chel1: dostępność planktonu [zagęszczenie Calanus helgolandicus gat. 1];
2.lcop1: dostępność planktonu [zagęszczenie widłonogów gat. 1];

Zmiana zagęszczenia planktonu chel2 powoduje zmianę zagęszczenia planktonu lcop2	 
1.chel2: dostępność planktonu [zagęszczenie Calanus helgolandicus gat. 2];
2.chel2: dostępność planktonu [zagęszczenie widłonogów gat. 2];

Istnieje również korelacja ujemna.
Wraze ze wzrostem łącznego rocznego natężenia połowów w regionie [ułamek pozostawionego narybku], spada łączna liczba ryb złowionych w ramach połowu [liczba śledzi]

#WYKRES PREZENTUJĄCY ZMIANĘ ROZMIARU ŚLEDZIA W CZASIE
W związku z tym iż zbiór danych nie posiada dokładnego znacznika czasowego względem roku połowu w grę wchodzi założenie o chronologiczności zbioru. Zmienna czasowa jest przedstawiona jako przyrostowa liczba rekordów (obserwacji).


```r
ggplot(herringsReplaceByMean, aes(X1, length)) + geom_point() + geom_smooth(se=TRUE) + ggtitle("Herrings length") + labs(x="dataset of 52584 surveys (past 60 years)", y="Herrings length in cm")
```

```
## `geom_smooth()` using method = 'gam'
```

![](report_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

#REGRESJA
Zastosowanie regresji pozwoli przewidzieć wielkość wyłowianych śledzi w przyszłości. Ze zbioru eleminuję kolumnę porządową X1, która nie powinna być brana pod uwagę podczas analizy. Zbiór został podzielony na uczący i testowy.


```r
fit<-select(herringsReplaceByMean,-X1)
fit<-lm(length ~ ., fit)
summary(fit)
```

```
## 
## Call:
## lm(formula = length ~ ., data = fit)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -7.6451 -0.9372  0.0042  0.9290  6.9724 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  5.709e+01  6.331e+00   9.018  < 2e-16 ***
## cfin1        1.399e-01  6.910e-03  20.248  < 2e-16 ***
## cfin2        2.245e-02  3.008e-03   7.465 8.48e-14 ***
## chel1       -2.490e-03  1.272e-03  -1.958   0.0502 .  
## chel2       -2.802e-03  1.724e-03  -1.626   0.1040    
## lcop1        1.410e-02  1.187e-03  11.880  < 2e-16 ***
## lcop2        9.098e-03  1.412e-03   6.442 1.19e-10 ***
## fbar         6.185e+00  8.298e-02  74.539  < 2e-16 ***
## recr        -3.547e-07  2.784e-08 -12.739  < 2e-16 ***
## cumf        -1.029e+01  1.571e-01 -65.491  < 2e-16 ***
## totaln      -5.571e-07  5.167e-08 -10.781  < 2e-16 ***
## sst         -1.249e+00  1.995e-02 -62.574  < 2e-16 ***
## sal         -3.997e-01  1.792e-01  -2.231   0.0257 *  
## xmonth       8.605e-03  2.172e-03   3.962 7.45e-05 ***
## nao          2.482e-02  4.076e-03   6.090 1.14e-09 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.367 on 52567 degrees of freedom
## Multiple R-squared:  0.3165,	Adjusted R-squared:  0.3163 
## F-statistic:  1739 on 14 and 52567 DF,  p-value: < 2.2e-16
```

```r
summary(fit)$r.squared
```

```
## [1] 0.3164881
```

```r
rmse<-function(num) {
  sqrt(mean(num^2))
}

rmse(fit$residuals)
```

```
## [1] 1.366484
```

```r
n<-nrow(herringsReplaceByMean)
trainIndex<-sample(1:n, size = round(0.7*n), replace=FALSE)
training<-herringsReplaceByMean[trainIndex ,]
test<-herringsReplaceByMean[-trainIndex ,]

training_ctrl<-trainControl(
    method = "cv",
    number=5
)

fit<-train(length ~ .,
    data = training,
    trControl = training_ctrl,
    method = "rf",
    ntree = 2
)

predi <- predict(fit, newdata = test)
```

#Ważność atrybutów


```r
randfor_fit<-select(herringsReplaceByMean,-X1)
randfor<-randomForest(length ~ .,randfor_fit)
print(randfor)
```

```
## 
## Call:
##  randomForest(formula = length ~ ., data = randfor_fit) 
##                Type of random forest: regression
##                      Number of trees: 500
## No. of variables tried at each split: 4
## 
##           Mean of squared residuals: 1.314559
##                     % Var explained: 51.88
```

#Wnioski
W WYNIKU PRZEPROWADZONYCH OBLICZEŃ
ISTANIEJE POWAŻNA PODSTAWA ABY TWIERDZIĆ, ŻE DŁUGOŚĆ ŚLEDZIA JEST ZWIĄZA Z TEMPERATURĄ PRZY POWIERZCHNI WODY. ROZMIAR ŚLEDZIA JEST DŁUŻSZY W PRZYPADKU GDY TEMPERATURA WODY JEST NIŻSZA
