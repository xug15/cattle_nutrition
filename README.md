# cattle_nutrition
##
```r
install.packages('Rglpk')
library(Rglpk)
#min(z=2a+4b+3c)
#3a+4b+2c<=60
#2a+b+2c<=40
#a+3b+2c<=80
#Rglpk_solve_LP((obj, mat, dir, rhs, bounds = NULL, types = NULL, max = FALSE,control = list(), ...))
#obj	规划目标系数
#mat	约束向量矩阵
#dir	约束方向向量，有’>’、’<’、’=’构成
#rhs	约束值
#bounds	上下限的约束，默认0到INF
#type	限定目标变量的类型,’B’指的是0-1规划,’C’代表连续,’I’代表整数，默认是’C’
#control	包含四个参数verbose、presolve、tm_limit、canonicalize_status。

obj <- c(2, 4, 3)
mat <- matrix(c(3, 2, 1, 4, 1, 3, 2, 2, 2), nrow = 3)
#       [,1] [,2] [,3]
# [1,]    3    4    2
# [2,]    2    1    2
# [3,]    1    3    2
dir <- c("<=", "<=", "<=")
rhs <- c(60, 40, 80)
max <- TRUE

Rglpk_solve_LP(obj, mat, dir, rhs, max = max)
# $optimum
# [1] 76.66667
# $solution
# [1]  0.000000  6.666667 16.666667
# $status
# [1] 0
# $solution_dual
# [1] -1.833333  0.000000  0.000000
# $auxiliary
# $auxiliary$primal
# [1] 60.00000 40.00000 53.33333
# $auxiliary$dual
# [1] 0.8333333 0.6666667 0.0000000
```
## Load data.
```r
setwd('D:/a-document/github/cattle_nutrition')
library("readxl")
# read Beef_cattle_nutritional_requirements
Beef_cattle_nutritional_requirements <- read_excel("data/Beef_cattle_nutritional_requirements_standards.xlsx")
# read nutrion_value
nutrion_value <- read_excel("data/nutrion.value.xlsx")
#display data.
head(Beef_cattle_nutritional_requirements)
head(nutrion_value)

```

## Load the beet cattle weight is 300kg, and daily weight gain 1.2 kg per day.
```r
#Load the beet cattle weight is 300kg, and daily weight gain 1.2 kg per day.
nutrition_require_300kg_gain1.2=Beef_cattle_nutritional_requirements[Beef_cattle_nutritional_requirements$Weight_kg==300 & Beef_cattle_nutritional_requirements$Daily_weight_gain_kg_day==1.2,]
nutrition_require_300kg_gain1.2

```
## Extract feed nutrition
```r
# Extract corn nutrition
nutrition_corn_value=nutrion_value[nutrion_value$Raw_materials=="corn",]
nutrition_corn_value
#Extract wheat straw
nutrition_wheat_straw_value=nutrion_value[nutrion_value$Raw_materials=="wheat straw",]
nutrition_wheat_straw_value
#Extract cottonseed cake
nutrition_cottonseed_cake_value=nutrion_value[nutrion_value$Raw_materials=="cottonseed cake",]
nutrition_cottonseed_cake_value

```
##  set requirement
```r
# set requirement
#concentrated feed,
#roughage feed
## set order is nutrition_corn_value, nutrition_cottonseed_cake_value, nutrition_wheat_straw_value.
nutrition_array=c()
requirement_array=c()

#1. dry matter part
# the frist thing is calculate dry matter.
nutrition_array=append(nutrition_array,nutrition_corn_value$Dry_matter)
nutrition_array=append(nutrition_array,nutrition_cottonseed_cake_value$Dry_matter)
nutrition_array=append(nutrition_array,nutrition_wheat_straw_value$Dry_matter)
#dry matter requirement
requirement_array=append(requirement_array,nutrition_require_300kg_gain1.2$Dry_matter_intake_kg_day)
#2. Crude protein
# the frist thing is calculate crude protein.
nutrition_array=append(nutrition_array,nutrition_corn_value$Crude_protein)
nutrition_array=append(nutrition_array,nutrition_cottonseed_cake_value$Crude_protein)
nutrition_array=append(nutrition_array,nutrition_wheat_straw_value$Crude_protein)
#dry matter requirement
requirement_array=append(requirement_array,nutrition_require_300kg_gain1.2$Crude_protein_g_day)
#3. energy
# the frist thing is calculate crude protein.
nutrition_array=append(nutrition_array,nutrition_corn_value$Energy_Unit_RND)
nutrition_array=append(nutrition_array,nutrition_cottonseed_cake_value$Energy_Unit_RND)
nutrition_array=append(nutrition_array,nutrition_wheat_straw_value$Energy_Unit_RND)
#dry matter requirement
requirement_array=append(requirement_array,nutrition_require_300kg_gain1.2$Energy_Unit_RND)

nutrition_array
requirement_array
```
## Calculate the formular
```r
obj <- c(10, 1, 1)
mat <- t(matrix(as.numeric(nutrition_array), nrow = 3))
mat
dir <- c("==", "==", "==")
rhs <- as.numeric(requirement_array)
rhs
max <- TRUE
forumla_result=Rglpk_solve_LP(obj, mat, dir, rhs, max = max)
forumla_result
feed_formula=forumla_result$solution

```
## Output the formulate.
```r
names(feed_formula)=c('corn kg/day',"cotton seed kg/day","wheat straw kg/day")
cost=forumla_result$optimum
feed_formula
cost
```



