---
title: "Assignment 1 - Data Cleaning - Feedback"
author: "Victor M. Poulsen"
date: "09-08-2020"
output: 
  html_document:
    keep_md: true 
---

### Overview 

**0. Please fill out the blackboard name thing (padlet)** \
**1. How to submit assignments** \
**2. Github (forking)** \
**3. Comments on Assignment 1** \
**4. A few comments about next assignment** \
**5. Work and live-coding** \

### How to submit assignments

* Not in a private repository. \
* Either write which file is the final or put this information in the readme. \
* Consider knitting to markdown (see: https://happygitwithr.com/rmd-test-drive.html) \

### Github 

* Sorry that I confused you last time... \
What you should do is the following: 
1. **Fork** repository to your group (which many of you have done for assignment 1)\
2. **Clone** repository to your computer. 
3. **Push & Pull** like a magnet do. 
4. Today this is done from my github. 

### Assignment 2 

1. For p-values you can use the package lmerTest (summary function). \
That is, if you need them. This is also mentioned in the readings for today.


2. Fitted values against actual values: \
To plot fitted values (predicted by model) \
you could look into the package **broom.mixed**. \
Another option seems to be **sjPlot** or using **predict()**. \
These are just suggestions, and I suggest you to try your own hand. 


3. To obtain **marginal** and **conditional** \
R-squared values I would encourage you to look into \
the function **r.squaredGLMM** from the package **MuMIn**. \
Marginal and conditional R-squared are mentioned \
in the readings for today, and are interesting because \
they tell us about how much variance is explained \
by fixed effects (marginal) and by with random effects (conditional).


4. Remember that you can always get in touch with me, \
either on **facebook** or on **victormoeller@gmail.com** 
or whatever.

### Comments on Assignment 1 

Today we will quickly go through parts of the first assignment. \

This is not something we are likely to do after each \
assignment (because it takes too much time for me, and \
from your working time in class). However, I just want \
to get everyone on the right track. So, what we will cover: \

1. **read_csv()** & **write_csv()**
2. **Tidyverse**, **readability** & **scaleability**
3. **pipes** 
4. **functions**


```r
pacman::p_load(tidyverse,janitor)
setwd("~/Dropbox/MastersSem1/2020MethCP/git-assignments/methods3_A1")
```

### Loading data
We want to use **read_csv()** instead of **read.csv()** 
since it is faster (something like 10x).


```r
#suppress messages: 
options(readr.num_columns = 0)

#read data: 
demo_train <- read_csv("demo_train.csv") 
LU_train <- read_csv("LU_train.csv")
token_train <- read_csv("token_train.csv")
```

2a. Identify which variable names do not match (that is are spelled differently) and find a way to transform variable names.
Pay particular attention to the variables indicating participant and visit.

### Code that I have seen and don't love:

1. names(demo_train)[names(demo_train) == 'Child.ID'] <- 'SUBJ' **(base R)**
2. colnames(demo)[1] <- "SUBJ" **(base R)**
3. demo_train <- demo_train %>% rename(SUBJ = Child.ID, VISIT = Visit) **(readability issues)**

### What I would really recommend: 

1. Use **Tidyverse**. For reference, see: http://r4ds.had.co.nz
2. Make your code **readable** (for you and for me). No long lines. 
3. A lot of you already did this which is great!


```r
#Getting an overview (from the janitor package)
compare_df_cols(demo_train, LU_train, token_train) %>%
  head()
```

```
##   column_name demo_train LU_train token_train
## 1        ADOS    numeric     <NA>        <NA>
## 2         Age    numeric     <NA>        <NA>
## 3        Age2    numeric     <NA>        <NA>
## 4   ASD_check    numeric     <NA>        <NA>
## 5        ASD2    numeric     <NA>        <NA>
## 6   Birthdate  character     <NA>        <NA>
```

```r
#A readable and tidyverse solution. 
demo_train <- demo_train %>%
  rename(SUBJ = Child.ID, 
         VISIT = Visit)
```

2b. Find a way to homogeneize the way "visit" is reported (visit1 vs. 1).

Tip: The stringr package is what you need. str_extract () will allow you to extract only the digit (number) from a string, by using the regular expression \\d.

### 2b. Again I would advocate for the use of Tidyverse

Many of you did this in a one-liner which is good.
However, I am really interested in you writing 
**readable** code, so I would still recommend tidyverse.
Some of you did this smarter than me (in tidyverse),
so I simply show this as a possible way of doing
this task in Tidyverse. 

For this task, note that we are running the 
exact same code on multiple dataframes (or tibbles).
This indicates that a **function** would be useful.


```r
#example solution 1 (some of you did it smarter):

LU_train <- LU_train %>%
  mutate_at("VISIT", str_extract, "\\d") %>%
  mutate(VISIT = as.integer(VISIT)) 

token_train <- token_train %>%
  mutate_at("VISIT", str_extract, "\\d") %>%
  mutate(VISIT = as.integer(VISIT))

demo_train <- demo_train %>%
  mutate(VISIT = as.integer(VISIT))


#example solution 2 (function):

extractor <- function(df){
  
df <- df %>%
  mutate_at("VISIT", str_extract, "\\d") %>%
  mutate(VISIT = as.integer(VISIT))
  
}

#Use the function: 
demo_train <- extractor(demo_train)
LU_train <- extractor(LU_train)
token_train <- extractor(token_train)

#Check that we have the same type: 
compare_df_cols(demo_train, LU_train, token_train) %>%
  tail()
```

```
##              column_name demo_train LU_train token_train
## 47    VinelandExpressive    numeric     <NA>        <NA>
## 48     VinelandReceptive    numeric     <NA>        <NA>
## 49 VinelandStandardScore    numeric     <NA>        <NA>
## 50       VinelandWritten    numeric     <NA>        <NA>
## 51                 VISIT    integer  integer     integer
## 52                     X       <NA>     <NA>     logical
```

2c. We also need to make a small adjustment to the content of the Child.ID column in the demographic data. Within this column, names that are not abbreviations do not end with "." (i.e. Adam), which is the case in the other two data sets (i.e. Adam.). If The content of the two variables isn't identical the rows will not be merged.
A neat way to solve the problem is simply to remove all "." in all datasets.

Tip: stringr is helpful again. Look up str_replace_all
Tip: You can either have one line of code for each child name that is to be changed (easier, more typing) or specify the pattern that you want to match (more complicated: look up "regular expressions", but less typing)

### 2c. Exactly the same points as in 2b


```r
#example solution 1:

demo_train <- demo_train %>%
   mutate_at("SUBJ", str_replace_all, "\\.", "")

LU_train <- LU_train %>%
  mutate_at("SUBJ", str_replace_all, "\\.", "")

token_train <- token_train %>%
  mutate_at("SUBJ", str_replace_all, "[.]", "") #same as "\\."

#example solution 2 (function): 
stringer <- function(df){
  
df <- df %>%
  mutate_at("SUBJ", str_replace_all, "\\.", "") # "\\." because "\" escapes. 
  
}

#using the function: 
demo_train <- stringer(demo_train)
LU_train <- stringer(LU_train)
token_train <- stringer(token_train)
```

2d. Now that the nitty gritty details of the different data sets are fixed, we want to make a subset of each data set only containig the variables that we wish to use in the final data set.
For this we use the tidyverse package dplyr, which contains the function select(). 

### 2d. All fine here 

Again a **function** proves handy,
although many of you also have good solutions. 


```r
# example solution: 
selector2 <- function(df){
  
  df <- df %>%
    select(one_of(c("SUBJ", "VISIT", "Diagnosis", "Ethnicity",
           "Gender", "Age", "ADOS", "MullenRaw", "ExpressiveLangRaw",
           "Socialization", "MOT_MLU", "CHI_MLU", "types_MOT",
           "types_CHI", "tokens_MOT", "tokens_CHI"))) 

}

# run the function: 
demo_train <- selector2(demo_train)
```

```
## Warning: Unknown columns: `MOT_MLU`, `CHI_MLU`, `types_MOT`, `types_CHI`,
## `tokens_MOT`, `tokens_CHI`
```

```r
LU_train <- selector2(LU_train)
```

```
## Warning: Unknown columns: `Diagnosis`, `Ethnicity`, `Gender`, `Age`, `ADOS`,
## `MullenRaw`, `ExpressiveLangRaw`, `Socialization`, `types_MOT`, `types_CHI`,
## `tokens_MOT`, `tokens_CHI`
```

```r
token_train <- selector2(token_train)
```

```
## Warning: Unknown columns: `Diagnosis`, `Ethnicity`, `Gender`, `Age`, `ADOS`,
## `MullenRaw`, `ExpressiveLangRaw`, `Socialization`, `MOT_MLU`, `CHI_MLU`
```

2e. Finally we are ready to merge all the data sets into just one. 

Some things to pay attention to:
* make sure to check that the merge has included all relevant data (e.g. by comparing the number of rows)
* make sure to understand whether (and if so why) there are NAs in the dataset (e.g. some measures were not taken at all visits, some recordings were lost or permission to use was withdrawn)

### 2e. Merging (all true)

Note: I think you need the **all = T** argument to keep 372 obs (which 
we have in *demo_train* but not in *LU_train* and *token_train*). 


```r
#Example solution 1 (although there should be something better): 
merged_data <- merge(demo_train, LU_train, all = T)
merged_data <- merge(merged_data, token_train, all = T) 
```


2f. Only using clinical measures from Visit 1
In order for our models to be useful, we want to minimize the need to actually test children as they develop. In other words, we would like to be able to understand and predict the children's linguistic development after only having tested them once. Therefore we need to make sure that our ADOS, MullenRaw, ExpressiveLangRaw and Socialization variables are reporting (for all visits) only the scores from visit 1.

A possible way to do so:
* create a new dataset with only visit 1, child id and the 4 relevant clinical variables to be merged with the old dataset
* rename the clinical variables (e.g. ADOS to ADOS1) and remove the visit (so that the new clinical variables are reported for all 6 visits)
* merge the new dataset with the old

### 2f. Some issues here

1. Use the **pipe** instead of new lines for each operation. \
You can **filter**, **select** (and **rename**) in one pipeline. \ 
2. Some misunderstanding of the task (which I totally get). 


```r
# example solution: 
new_data <- merged_data %>%
  filter(VISIT == 1) %>%
  select(SUBJ,
         ADOS1 = ADOS, #new_name = old_name. 
         verbalIQ = ExpressiveLangRaw,
         nonVerbalIQ = MullenRaw,
         Socialization1 = Socialization)
  
data <- merge(new_data, merged_data, all = T)
```

2g. Final touches

Now we want to
* anonymize our participants (they are real children!). 
* make sure the variables have sensible values. E.g. right now gender is marked 1 and 2, but in two weeks you will not be able to remember, which gender were connected to which number, so change the values from 1 and 2 to F and M in the gender variable. For the same reason, you should also change the values of Diagnosis from A and B to ASD (autism spectrum disorder) and TD (typically developing). Tip: Try taking a look at ifelse(), or google "how to rename levels in R".
* Save the data set using into a csv file. Hint: look into write.csv()

### 2g. Pipes and write_csv()

Again we want to use **pipes** and **Tidyverse**. \
This can all be done in one pipeline instead of 
on different lines. That is to be preferred. \
We also want to use **write_csv()** and not **write.csv()**. \
In addition, remember to call your file **.csv**. \



```r
#example solution:
data <- data %>%
  mutate(SUBJ = as.numeric(as_factor(SUBJ)),
         VISIT = as.numeric(VISIT),
         Gender = if_else(Gender == "1", "M", "F")) #not clear from instr.

#write csv. 
write_csv(data, "data_cleaned_draft.csv") #check that this matches. 

#read it to check. 
test <- read_csv("data_cleaned_draft.csv")
```


3) BONUS QUESTIONS
The aim of this last section is to make sure you are fully fluent in the tidyverse.
Here's the link to a very helpful book, which explains each function:
http://r4ds.had.co.nz/index.html

1) USING FILTER
List all kids who:
1. have a mean length of utterance (across all visits) of more than 2.7 morphemes.
2. have a mean length of utterance of less than 1.5 morphemes at the first visit
3. have not completed all trials. Tip: Use pipes to solve this


```r
# MLU > 2.7 across visits. 
data %>%
  group_by(SUBJ) %>%
  summarise(MEAN_CHI_MLU = mean(CHI_MLU)) %>%
  filter(MEAN_CHI_MLU > 2.7) %>%
  slice_head(n = 10)
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```
## # A tibble: 9 x 2
##    SUBJ MEAN_CHI_MLU
##   <dbl>        <dbl>
## 1     4         3.02
## 2     5         3.13
## 3    14         2.75
## 4    20         2.84
## 5    28         2.73
## 6    45         2.75
## 7    54         2.94
## 8    58         3.16
## 9    65         3.26
```

```r
# MLU < 1.5 at first visit. 
data %>% filter(VISIT == 1) %>%
  filter(CHI_MLU < 1.5) %>%
  select(SUBJ, VISIT, CHI_MLU, Diagnosis) %>%
  slice_head(n = 10)
```

```
##    SUBJ VISIT  CHI_MLU Diagnosis
## 1     2     1 1.252252         B
## 2     6     1 1.394737         A
## 3     7     1 1.000000         A
## 4     8     1 1.264151         A
## 5    10     1 1.037879         B
## 6    11     1 1.037500         B
## 7    13     1 1.216867         B
## 8    14     1 1.087719         B
## 9    16     1 1.039604         B
## 10   17     1 1.164706         B
```

```r
# Have not completed all trials (could be error prone).
data %>%
  group_by(SUBJ) %>%
  summarize(MAX_VISIT = max(VISIT)) %>%
  filter(MAX_VISIT < 6) %>%
  slice_head(n = 10)
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```
## # A tibble: 6 x 2
##    SUBJ MAX_VISIT
##   <dbl>     <dbl>
## 1     1         1
## 2    12         2
## 3    24         3
## 4    48         5
## 5    49         2
## 6    51         2
```


USING ARRANGE

1. Sort kids to find the kid who produced the most words on the 6th visit
2. Sort kids to find the kid who produced the least amount of words on the 1st visit.


```r
data %>% filter(VISIT == 6) %>%
  arrange(desc(CHI_MLU)) %>%
  select(SUBJ, VISIT, CHI_MLU, Diagnosis) %>%
  slice_head(n = 10)
```

```
##    SUBJ VISIT  CHI_MLU Diagnosis
## 1    16     6 3.811404         B
## 2     4     6 3.710345         B
## 3    15     6 3.701195         B
## 4    54     6 3.595000         B
## 5    14     6 3.504950         B
## 6    65     6 3.441558         A
## 7     3     6 3.413502         A
## 8    29     6 3.364341         A
## 9     5     6 3.278195         A
## 10   58     6 3.243902         B
```

```r
data %>% filter(VISIT == 1) %>%
  arrange(CHI_MLU) %>%
  select(SUBJ, VISIT, CHI_MLU, Diagnosis) %>%
  slice_head(n = 10)
```

```
##    SUBJ VISIT   CHI_MLU Diagnosis
## 1    62     1 0.0000000         A
## 2    22     1 0.1857143         A
## 3    40     1 0.4805825         A
## 4    48     1 0.5584416         B
## 5    33     1 0.9000000         A
## 6     7     1 1.0000000         A
## 7    23     1 1.0000000         A
## 8    32     1 1.0086207         A
## 9    43     1 1.0175439         A
## 10   44     1 1.0262009         B
```

USING SELECT

### Again some unclear writing ### 

1. Make a subset of the data including only kids with ASD, mlu and word tokens
2. What happens if you include the name of a variable multiple times in a select() call?


```r
# 1. subset ASD, MLU, Tokens. 
data %>% filter(Diagnosis == "ASD") %>%
  select(Diagnosis, CHI_MLU, tokens_CHI) %>%
  arrange(desc(CHI_MLU)) %>%
  slice_head(n = 10)
```

```
## [1] Diagnosis  CHI_MLU    tokens_CHI
## <0 rows> (or 0-length row.names)
```

```r
# 2. multiple variable multiple times. (nothing).
test <- data %>% filter(Diagnosis == "ASD") %>%
  select(Diagnosis, CHI_MLU, tokens_CHI)

test2 <- data %>% filter(Diagnosis == "ASD") %>%
  select(Diagnosis, CHI_MLU, tokens_CHI, tokens_CHI)

all.equal(test, test2)
```

```
## [1] TRUE
```

### Some confusion here in word-usage, not clear what is meant ###

USING MUTATE, SUMMARISE and PIPES
1. Add a column to the data set that represents the mean number of words spoken during all visits. ## per child?
2. Use the summarise function and pipes to add an column in the data set containing the mean amount of words produced by each trial across all visits. HINT: group by Child.ID 
3. The solution to task above enables us to assess the average amount of words produced by each child. Why don't we just use these average values to describe the language production of the children? What is the advantage of keeping all the data?


```r
# mutate w. group_by. 
data %>%
  group_by(SUBJ) %>%
  mutate(mean_CHI_MLU = mean(CHI_MLU)) %>%
  select(SUBJ, VISIT, CHI_MLU, mean_CHI_MLU) %>%
  slice_head(n = 15)
```

```
## # A tibble: 372 x 4
## # Groups:   SUBJ [66]
##     SUBJ VISIT CHI_MLU mean_CHI_MLU
##    <dbl> <dbl>   <dbl>        <dbl>
##  1     1     1   NA           NA   
##  2     2     1    1.25         2.03
##  3     2     2    1.01         2.03
##  4     2     3    1.56         2.03
##  5     2     4    2.25         2.03
##  6     2     5    3.24         2.03
##  7     2     6    2.87         2.03
##  8     3     1    2.28        NA   
##  9     3     2    3.45        NA   
## 10     3     3    3.12        NA   
## # … with 362 more rows
```

```r
# is the second question not pretty much the same?
```
