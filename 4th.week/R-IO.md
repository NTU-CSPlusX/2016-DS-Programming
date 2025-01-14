---
title       : "R語言的IO簡介"
author      : "Wush Wu"
job         : 國立台灣大學
framework   : io2012-wush
highlighter : highlight.js
hitheme     : zenburn
widgets     : [mathjax]            # {mathjax, quiz, bootstrap}
mode        : selfcontained # {standalone, draft}
knit        : slidify::knit2slides
--- &vcenter .largecontent

## 大綱

1. 什麼是IO?
1. R 語言一般物件的IO
1. CSV的IO



--- &vcenter .largecontent

## 什麼是IO?

- I: Input
- O: Output
- 一般來說, IO泛指（但不限於）如何讓工具從硬碟中存取資料的技術
- 具體來說, 01-RBasic-07與02-RDataEngineer-01~04 都在教IO的技術

--- &vcenter .largecontent

## IO技術的要點

- 會
    - 會存、會取
- 正確
    - 存取後的資料與原本的資料一致
- 快
    - 含大量數據時怎麼在短時間存取

--- &fullimg bg:url(assets/img/R-dsr-process.png)

--- &vcenter .largecontent

## R物件的存取

- `saveRDS`/`readRDS`


```r
tmp.path <- tempfile()
saveRDS(iris, tmp.path)
iris2 <- readRDS(tmp.path)
all.equal(iris, iris2)
```

```
## [1] TRUE
```

--- &vcenter .largecontent

## `saveRDS`

- 將R物件以binary的方式存入硬碟
    1. binary 格式, 整數用4 bytes, 而非文字。舉例來說: 98765432佔用4 bytes而非8 bytes
    2. 內建壓縮功能, 用CPU換硬碟存取的時間

--- &vcenter .largecontent

## `readRDS`

- 從硬碟中將`saveRDS`的結果讀回R
    - 同電腦的R 一般沒有問題
    - 不同電腦的R 可能會有問題(但是機會不大)

--- .dark .segue

## R 與 CSV格式

--- &vcenter .largecontent

## CSV、TSV或其他類似格式

- CSV: Comma-Seperated Values
- 以分隔符號來儲存明碼
    - 為什麼要分隔符號?


```r
df <- iris[c(1, 51, 101),]
write.csv(df, file = (tmp.path <- tempfile()))
```

```csv
"","Sepal.Length","Sepal.Width","Petal.Length","Petal.Width","Species"
"1",5.1,3.5,1.4,0.2,"setosa"
"51",7,3.2,4.7,1.4,"versicolor"
"101",6.3,3.3,6,2.5,"virginica"
```

- 若沒有分隔符號，第一行變成: `15.13.51.40.2setosa`

--- &vcenter .largecontent

## CSV、TSV或其他類似格式

- R 的存取函數:
    - 存: `write.csv`, `write.table`
    - 取: `read.csv`, `read.table`

--- &vcenter .largecontent

## CSV 的正確性

- 結構化資料
- CSV沒有嚴謹的型態定義，只有經驗構成的判斷
    - 有非數字的即為文字或factor
    - 都是數字的則為數字
    - `"`可以協助判斷，但是不同工具的詮釋會不同
    - excel --> CSV --> R 是簡單的讀取excel資料的方式

```csv
"","Sepal.Length","Sepal.Width","Petal.Length","Petal.Width","Species"
"1",5.1,3.5,1.4,0.2,"setosa"
"51",7,3.2,4.7,1.4,"versicolor"
"101",6.3,3.3,6,2.5,"virginica"
```

--- &vcenter .largecontent

## CSV 的正確性


```r
df <- data.frame(a = as.character(1:3))
write.csv(df, tmp.path <- tempfile(), row.names = FALSE)
cat(readLines(tmp.path), sep = "\n")
```

```
## "a"
## "1"
## "2"
## "3"
```

```r
df2 <- read.csv(tmp.path)
all.equal(df, df2)
```

```
## [1] "Component \"a\": 'current' is not a factor"
```

--- &vcenter .largecontent

## CSV 的正確性

- 在讀取時透過`colClass`參數做校正


```r
df2 <- read.csv(tmp.path, colClasses = "factor")
all.equal(df, df2)
```

```
## [1] TRUE
```

--- &vcenter .largecontent

## CSV 格式的正確性

- 要非常小心才能確認存取的資料的型態正確性，建議使用colClasses比較保險
- 與其他工具可能有不一致的行為
- 文字資料不跨平台

--- &vcenter .largecontent

## 電腦上的文字

- 絕對沒問題的符號ASCII
    - 英數鍵盤上的符號, ex: a ~ z, A ~ Z, 0 ~ 9, 
- 其他的符號都有編碼問題

--- &vcenter .largecontent

## 文字編碼

- 一個文字可能1~3 bytes
    - 一個bytes 包含 8 bits, 所以數字的範圍是 0 ~ 255
    - 我們會用兩個16進位的符號來表示一個byte

--- &vcenter .largecontent

## 文字編碼

<br/>


```r
showBits <- function(r) stats::symnum(as.logical(rawToBits(r)))
showBits(as.raw(0))
```

```
## [1] . . . . . . . .
```

```r
showBits(as.raw(1))
```

```
## [1] | . . . . . . .
```

```r
showBits(as.raw(2))
```

```
## [1] . | . . . . . .
```

```r
showBits(as.raw(255))
```

```
## [1] | | | | | | | |
```

--- &vcenter .largecontent

## 文字編碼

<br/>


```r
showBits(as.raw(0x00))
```

```
## [1] . . . . . . . .
```

```r
showBits(as.raw(0x0a))
```

```
## [1] . | . | . . . .
```

```r
showBits(as.raw(0x12))
```

```
## [1] . | . . | . . .
```

```r
showBits(as.raw(0xff))
```

```
## [1] | | | | | | | |
```

--- &vcenter .largecontent

## 文字編碼


```r
# 不同的電腦，會有不同的值
x <- "中"
charToRaw(x)
```

```
## [1] e4 b8 ad
```

```r
charToRaw(iconv(x, from = "UTF-8", to = "BIG5"))
```

```
## [1] a4 a4
```

```r
charToRaw(iconv(x, from = "UTF-8", to = "UTF-16"))
```

```
## [1] fe ff 4e 2d
```

--- &vcenter .largecontent

## 中文編碼的不一致

- Windows: BIG5, Mac OSX / Linux : UTF-8
- 政府單位多用BIG5, 但是偶爾也會冒出UTF-8
- 產生中文亂碼，大部分的原因都是因為編碼


```r
iconv(x, from = "UTF-8", to = "BIG5")
```

```
## [1] "\xa4\xa4"
```

--- &vcenter .largecontent

## 中文編碼的解決方法

- 用raw vector讀取資料，避開編碼的處理
- 使用`iconv`或`stringi::stri_encode`將編碼轉換成平台的中文編碼
    - Windows 上因為編碼與Open Source Community使用的UTF-8不同, 所以中文有許多出bug的風險
    - 遇到時，一種方式是對character vector做Encoding的註解

--- &vcenter .largecontent

## 中文編碼的解決方法


```r
(x <- "fa\xE7ile")
```

```
## [1] "fa\xe7ile"
```

```r
# 轉換編碼到UTF-8
iconv(x, from = "latin1", to = "UTF-8")
```

```
## [1] "façile"
```

```r
# 註記編碼是latin1讓R能正確顯示
Encoding(x) <- "latin1"
x
```

```
## [1] "façile"
```

--- &vcenter .largecontent

## 練習

- 請完成01-RBasic-07

