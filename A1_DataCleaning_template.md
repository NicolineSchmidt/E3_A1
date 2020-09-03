---
title: "Assignment 1 - Data Cleaning - Instructions"
author: "Victor M. Poulsen"
date: "03-08-2020"
output: 
  html_document:
    keep_md: true 
---

# Assignment 1, Language development in Autism Spectrum Disorder (ASD) - Brushing up your code skills

In this first part of the assignment we will brush up your programming skills, and make you familiar with the data sets you will be analysing for the next parts of the assignment.

In this warm-up assignment you will:
1) Create a Github (or gitlab) account, link it to your RStudio, and create a new repository/project
2) Use small nifty lines of code to transform several data sets into just one. The final data set will contain only the variables that are needed for the analysis in the next parts of the assignment
3) Warm up your tidyverse skills (especially the sub-packages stringr and dplyr), which you will find handy for later assignments.


## Learning objectives:

- Become comfortable with tidyverse (and R in general)
- Test out the git integration with RStudio
- Build expertise in data wrangling (which will be used in future assignments)


## 0. First an introduction on the data

# Language development in Autism Spectrum Disorder (ASD)

Reference to the study: https://www.ncbi.nlm.nih.gov/pubmed/30396129

Background: Autism Spectrum Disorder (ASD) is often related to language impairment, and language impairment strongly affects the patients ability to function socially (maintaining a social network, thriving at work, etc.). It is therefore crucial to understand how language abilities develop in children with ASD, and which factors affect them (to figure out e.g. how a child will develop in the future and whether there is a need for language therapy).
However, language impairment is always quantified by relying on the parent, teacher or clinician subjective judgment of the child, and measured very sparcely (e.g. at 3 years of age and again at 6). 

In this study the researchers videotaped circa 30 kids with ASD and circa 30 comparison kids (matched by linguistic performance at visit 1) for ca. 30 minutes of naturalistic interactions with a parent. They repeated the data collection 6 times per kid, with 4 months between each visit. Then the researchers transcribed the data and counted: 
i) the amount of words that each kid uses in each video. Same for the parent.
ii) the amount of unique words that each kid uses in each video. Same for the parent.
iii) the amount of morphemes per utterance (Mean Length of Utterance) displayed by each child in each video. Same for the parent. 

Different researchers involved in the project provide you with different datasets: 
1) demographic and clinical data about the children (recorded by a clinical psychologist)
2) length of utterance data (calculated by a linguist)
3) amount of unique and total words used (calculated by a jack-of-all-trade, let's call him RF)

Your job in this assignment is to double check the data and make sure that it is ready for the analysis proper (Assignment 2), in which we will try to understand how the children's language develops as they grow as a function of cognitive and social factors and which are the "cues" suggesting a likely future language impairment.

## 1. Let's get started on GitHub

In the assignments you will be asked to upload your code on Github and the GitHub repositories will be part of the portfolio, therefore all students must make an account and link it to their RStudio (you'll thank us later for this!).

Follow the link to one of the tutorials indicated in the syllabus: 
* Recommended: https://happygitwithr.com/
*	Alternative (if the previous doesn't work): https://support.rstudio.com/hc/en-us/articles/200532077-Version-Control-with-Git-and-SVN
*	Alternative (if the previous doesn't work): R.version.string
N.B. Create a GitHub repository for the Assignment 1 and link it to a project on your RStudio.

## 2. Now let's take dirty dirty data sets and make them into a tidy one

If you're not in a project in Rstudio, make sure to set your working directory here.
If you created an RStudio project, then your working directory (the directory with your data and code for these assignments) is the project directory.


```r
pacman::p_load(tidyverse,janitor)
setwd("~/Dropbox/MastersSem1/2020MethCP/git-assignments/methods3_A1")
```

Load the three data sets, after downloading them from dropbox and saving them in your working directory:
* Demographic data for the participants: https://www.dropbox.com/s/lol8f5m4pgrpmto/demo_train.csv?dl=0
* Length of utterance data: https://www.dropbox.com/s/usyauqm37a76of6/LU_train.csv?dl=0
* Word data: https://www.dropbox.com/s/8ng1civpl2aux58/token_train.csv?dl=0


```r
demo_train <- read_csv("demo_train.csv") #read_csv() faster than read.csv()
```

```
## Parsed with column specification:
## cols(
##   .default = col_double(),
##   Child.ID = col_character(),
##   Ethnicity = col_character(),
##   Diagnosis = col_character(),
##   Birthdate = col_character()
## )
```

```
## See spec(...) for full column specifications.
```

```r
LU_train <- read_csv("LU_train.csv")
```

```
## Parsed with column specification:
## cols(
##   SUBJ = col_character(),
##   VISIT = col_character(),
##   MOT_MLU = col_double(),
##   MOT_LUstd = col_double(),
##   MOT_LU_q1 = col_double(),
##   MOT_LU_q2 = col_double(),
##   MOT_LU_q3 = col_double(),
##   CHI_MLU = col_double(),
##   CHI_LUstd = col_double(),
##   CHI_LU_q1 = col_double(),
##   CHI_LU_q2 = col_double(),
##   CHI_LU_q3 = col_double()
## )
```

```r
token_train <- read_csv("token_train.csv")
```

```
## Parsed with column specification:
## cols(
##   SUBJ = col_character(),
##   VISIT = col_character(),
##   types_MOT = col_double(),
##   types_CHI = col_double(),
##   types_shared = col_double(),
##   tokens_MOT = col_double(),
##   tokens_CHI = col_double(),
##   X = col_logical()
## )
```

Explore the 3 datasets (e.g. visualize them, summarize them, etc.). You will see that the data is messy, since the psychologist collected the demographic data, the linguist analyzed the length of utterance in May 2014 and the fumbling jack-of-all-trades analyzed the words several months later. 
In particular:
- the same variables might have different names (e.g. participant and visit identifiers)
- the same variables might report the values in different ways (e.g. participant and visit IDs)
Welcome to real world of messy data :-)

Before being able to combine the data sets we need to make sure the relevant variables have the same names and the same kind of values.

So:

2a. Identify which variable names do not match (that is are spelled differently) and find a way to transform variable names.
Pay particular attention to the variables indicating participant and visit.

Tip: look through the chapter on data transformation in R for data science (http://r4ds.had.co.nz). Alternatively you can look into the package dplyr (part of tidyverse), or google "how to rename variables in R". Or check the janitor R package. There are always multiple ways of solving any problem and no absolute best method.



```r
#Getting an overview: compare_df_cols()
str(demo_train) #Child.ID, Visit. (makes sense to change this one).  
```

```
## tibble [372 × 36] (S3: spec_tbl_df/tbl_df/tbl/data.frame)
##  $ Child.ID                  : chr [1:372] "A.A." "A.D." "A.D." "A.D." ...
##  $ Visit                     : num [1:372] 1 1 2 3 4 5 6 1 2 3 ...
##  $ Ethnicity                 : chr [1:372] "White" "White" "White" "White" ...
##  $ Diagnosis                 : chr [1:372] "B" "B" "B" "B" ...
##  $ ASD_check                 : num [1:372] 0 0 0 0 0 0 0 0 0 0 ...
##  $ ASD2                      : num [1:372] 0 0 0 0 0 0 0 0 0 0 ...
##  $ Gender                    : num [1:372] 1 1 1 1 1 1 1 2 2 2 ...
##  $ Birthdate                 : chr [1:372] "18/02/09" "20/12/04" "20/12/04" "20/12/04" ...
##  $ Age                       : num [1:372] 18.1 19.8 23.9 27.7 32.9 ...
##  $ Total..Understands...Says.: num [1:372] 0 16 NA NA NA NA NA 28 NA NA ...
##  $ Total..Understands.       : num [1:372] 183 245 NA NA NA NA NA 266 NA NA ...
##  $ Total.of.Both             : num [1:372] 183 261 133 563 76 95 94 294 303 570 ...
##  $ Age2                      : num [1:372] 18.1 19.8 23.9 27.7 32.5 ...
##  $ ADOS                      : num [1:372] 15 0 NA NA NA 0 NA 1 NA NA ...
##  $ CARS                      : num [1:372] 29 16 NA NA NA 15 NA 15.5 NA NA ...
##  $ CDI1                      : num [1:372] 183 261 NA NA NA NA NA 294 NA NA ...
##  $ VinelandStandardScore     : num [1:372] 90 100 100 103 107 110 106 100 103 103 ...
##  $ VinelandReceptive         : num [1:372] 20 26 27 26 29 29 31 24 25 25 ...
##  $ VinelandExpressive        : num [1:372] 19 19 31 57 73 80 87 30 60 70 ...
##  $ VinelandWritten           : num [1:372] NA NA NA NA NA NA 6 NA NA NA ...
##  $ DailyLivingSkills         : num [1:372] 121 119 113 109 102 107 109 91 95 93 ...
##  $ Socialization             : num [1:372] 104 108 110 109 102 107 100 88 89 91 ...
##  $ MotorSkills               : num [1:372] 111 111 113 114 105 114 107 84 88 88 ...
##  $ MullenRaw                 : num [1:372] NA 28 NA NA 33 NA 42 29 NA NA ...
##  $ MullenTScore              : num [1:372] NA 66 NA NA 53 NA 57 59 NA NA ...
##  $ MullenAge                 : num [1:372] NA 26 NA NA 33 NA 46 27 NA NA ...
##  $ FineMotorRaw              : num [1:372] NA 22 NA NA NA NA 39 21 NA NA ...
##  $ FineMotorTScore           : num [1:372] NA 52 NA NA NA NA 59 36 NA NA ...
##  $ FIneMotorAge              : num [1:372] NA 21 NA NA NA NA 45 20 NA NA ...
##  $ ReceptiveLanguageRaw      : num [1:372] NA 28 NA NA NA NA 40 27 NA NA ...
##  $ ReceptiveLanguageTScore   : num [1:372] NA 72 NA NA NA NA 61 59 NA NA ...
##  $ ReceptiveLanguageAge      : num [1:372] NA 30 NA NA NA NA 49 28 NA NA ...
##  $ ExpressiveLangRaw         : num [1:372] NA 14 NA NA NA NA 44 18 NA NA ...
##  $ ExpressiveLangTScore      : num [1:372] NA 33 NA NA NA NA 68 38 NA NA ...
##  $ ExpressiveLangAge         : num [1:372] NA 14 NA NA NA NA 55 18 NA NA ...
##  $ EarlyLearningComposite    : num [1:372] NA 112 NA NA NA NA 122 96 NA NA ...
##  - attr(*, "spec")=
##   .. cols(
##   ..   Child.ID = col_character(),
##   ..   Visit = col_double(),
##   ..   Ethnicity = col_character(),
##   ..   Diagnosis = col_character(),
##   ..   ASD_check = col_double(),
##   ..   ASD2 = col_double(),
##   ..   Gender = col_double(),
##   ..   Birthdate = col_character(),
##   ..   Age = col_double(),
##   ..   Total..Understands...Says. = col_double(),
##   ..   Total..Understands. = col_double(),
##   ..   Total.of.Both = col_double(),
##   ..   Age2 = col_double(),
##   ..   ADOS = col_double(),
##   ..   CARS = col_double(),
##   ..   CDI1 = col_double(),
##   ..   VinelandStandardScore = col_double(),
##   ..   VinelandReceptive = col_double(),
##   ..   VinelandExpressive = col_double(),
##   ..   VinelandWritten = col_double(),
##   ..   DailyLivingSkills = col_double(),
##   ..   Socialization = col_double(),
##   ..   MotorSkills = col_double(),
##   ..   MullenRaw = col_double(),
##   ..   MullenTScore = col_double(),
##   ..   MullenAge = col_double(),
##   ..   FineMotorRaw = col_double(),
##   ..   FineMotorTScore = col_double(),
##   ..   FIneMotorAge = col_double(),
##   ..   ReceptiveLanguageRaw = col_double(),
##   ..   ReceptiveLanguageTScore = col_double(),
##   ..   ReceptiveLanguageAge = col_double(),
##   ..   ExpressiveLangRaw = col_double(),
##   ..   ExpressiveLangTScore = col_double(),
##   ..   ExpressiveLangAge = col_double(),
##   ..   EarlyLearningComposite = col_double()
##   .. )
```

```r
str(LU_train) #SUBJ, VISIT. 
```

```
## tibble [352 × 12] (S3: spec_tbl_df/tbl_df/tbl/data.frame)
##  $ SUBJ     : chr [1:352] "A.D." "A.D." "A.D." "A.D." ...
##  $ VISIT    : chr [1:352] "visit1." "Visit2." "visit3." "visit4." ...
##  $ MOT_MLU  : num [1:352] 3.62 3.86 4.32 4.42 5.21 ...
##  $ MOT_LUstd: num [1:352] 2.16 2.42 2.52 2.45 2.81 ...
##  $ MOT_LU_q1: num [1:352] 2 2 2 2 3 2 2 2 3 3 ...
##  $ MOT_LU_q2: num [1:352] 4 4 4 4 5 5 3 4 5 5 ...
##  $ MOT_LU_q3: num [1:352] 5 6 6 6 7.25 7 5 6 7 8 ...
##  $ CHI_MLU  : num [1:352] 1.25 1.01 1.56 2.25 3.24 ...
##  $ CHI_LUstd: num [1:352] 0.474 0.116 0.747 1.578 2.356 ...
##  $ CHI_LU_q1: num [1:352] 1 1 1 1 1 1 1 1 1 1 ...
##  $ CHI_LU_q2: num [1:352] 1 1 1 2 3 2 2 2 2 4 ...
##  $ CHI_LU_q3: num [1:352] 1 1 2 3 5 4 2 4 4 5 ...
##  - attr(*, "spec")=
##   .. cols(
##   ..   SUBJ = col_character(),
##   ..   VISIT = col_character(),
##   ..   MOT_MLU = col_double(),
##   ..   MOT_LUstd = col_double(),
##   ..   MOT_LU_q1 = col_double(),
##   ..   MOT_LU_q2 = col_double(),
##   ..   MOT_LU_q3 = col_double(),
##   ..   CHI_MLU = col_double(),
##   ..   CHI_LUstd = col_double(),
##   ..   CHI_LU_q1 = col_double(),
##   ..   CHI_LU_q2 = col_double(),
##   ..   CHI_LU_q3 = col_double()
##   .. )
```

```r
str(token_train) #SUBJ, VISIT. 
```

```
## tibble [352 × 8] (S3: spec_tbl_df/tbl_df/tbl/data.frame)
##  $ SUBJ        : chr [1:352] "A.D." "A.D." "A.D." "A.D." ...
##  $ VISIT       : chr [1:352] "visit1." "Visit2." "visit3." "visit4." ...
##  $ types_MOT   : num [1:352] 378 403 455 533 601 595 334 464 482 449 ...
##  $ types_CHI   : num [1:352] 14 18 97 133 182 210 51 149 164 206 ...
##  $ types_shared: num [1:352] 9 15 82 113 156 181 19 130 146 165 ...
##  $ tokens_MOT  : num [1:352] 1835 2160 2149 2260 2553 ...
##  $ tokens_CHI  : num [1:352] 139 148 255 321 472 686 260 530 542 754 ...
##  $ X           : logi [1:352] NA NA NA NA NA NA ...
##  - attr(*, "spec")=
##   .. cols(
##   ..   SUBJ = col_character(),
##   ..   VISIT = col_character(),
##   ..   types_MOT = col_double(),
##   ..   types_CHI = col_double(),
##   ..   types_shared = col_double(),
##   ..   tokens_MOT = col_double(),
##   ..   tokens_CHI = col_double(),
##   ..   X = col_logical()
##   .. )
```

```r
demo_train <- demo_train %>%
  rename(SUBJ = Child.ID, 
         VISIT = Visit)

str(demo_train)
```

```
## tibble [372 × 36] (S3: spec_tbl_df/tbl_df/tbl/data.frame)
##  $ SUBJ                      : chr [1:372] "A.A." "A.D." "A.D." "A.D." ...
##  $ VISIT                     : num [1:372] 1 1 2 3 4 5 6 1 2 3 ...
##  $ Ethnicity                 : chr [1:372] "White" "White" "White" "White" ...
##  $ Diagnosis                 : chr [1:372] "B" "B" "B" "B" ...
##  $ ASD_check                 : num [1:372] 0 0 0 0 0 0 0 0 0 0 ...
##  $ ASD2                      : num [1:372] 0 0 0 0 0 0 0 0 0 0 ...
##  $ Gender                    : num [1:372] 1 1 1 1 1 1 1 2 2 2 ...
##  $ Birthdate                 : chr [1:372] "18/02/09" "20/12/04" "20/12/04" "20/12/04" ...
##  $ Age                       : num [1:372] 18.1 19.8 23.9 27.7 32.9 ...
##  $ Total..Understands...Says.: num [1:372] 0 16 NA NA NA NA NA 28 NA NA ...
##  $ Total..Understands.       : num [1:372] 183 245 NA NA NA NA NA 266 NA NA ...
##  $ Total.of.Both             : num [1:372] 183 261 133 563 76 95 94 294 303 570 ...
##  $ Age2                      : num [1:372] 18.1 19.8 23.9 27.7 32.5 ...
##  $ ADOS                      : num [1:372] 15 0 NA NA NA 0 NA 1 NA NA ...
##  $ CARS                      : num [1:372] 29 16 NA NA NA 15 NA 15.5 NA NA ...
##  $ CDI1                      : num [1:372] 183 261 NA NA NA NA NA 294 NA NA ...
##  $ VinelandStandardScore     : num [1:372] 90 100 100 103 107 110 106 100 103 103 ...
##  $ VinelandReceptive         : num [1:372] 20 26 27 26 29 29 31 24 25 25 ...
##  $ VinelandExpressive        : num [1:372] 19 19 31 57 73 80 87 30 60 70 ...
##  $ VinelandWritten           : num [1:372] NA NA NA NA NA NA 6 NA NA NA ...
##  $ DailyLivingSkills         : num [1:372] 121 119 113 109 102 107 109 91 95 93 ...
##  $ Socialization             : num [1:372] 104 108 110 109 102 107 100 88 89 91 ...
##  $ MotorSkills               : num [1:372] 111 111 113 114 105 114 107 84 88 88 ...
##  $ MullenRaw                 : num [1:372] NA 28 NA NA 33 NA 42 29 NA NA ...
##  $ MullenTScore              : num [1:372] NA 66 NA NA 53 NA 57 59 NA NA ...
##  $ MullenAge                 : num [1:372] NA 26 NA NA 33 NA 46 27 NA NA ...
##  $ FineMotorRaw              : num [1:372] NA 22 NA NA NA NA 39 21 NA NA ...
##  $ FineMotorTScore           : num [1:372] NA 52 NA NA NA NA 59 36 NA NA ...
##  $ FIneMotorAge              : num [1:372] NA 21 NA NA NA NA 45 20 NA NA ...
##  $ ReceptiveLanguageRaw      : num [1:372] NA 28 NA NA NA NA 40 27 NA NA ...
##  $ ReceptiveLanguageTScore   : num [1:372] NA 72 NA NA NA NA 61 59 NA NA ...
##  $ ReceptiveLanguageAge      : num [1:372] NA 30 NA NA NA NA 49 28 NA NA ...
##  $ ExpressiveLangRaw         : num [1:372] NA 14 NA NA NA NA 44 18 NA NA ...
##  $ ExpressiveLangTScore      : num [1:372] NA 33 NA NA NA NA 68 38 NA NA ...
##  $ ExpressiveLangAge         : num [1:372] NA 14 NA NA NA NA 55 18 NA NA ...
##  $ EarlyLearningComposite    : num [1:372] NA 112 NA NA NA NA 122 96 NA NA ...
##  - attr(*, "spec")=
##   .. cols(
##   ..   Child.ID = col_character(),
##   ..   Visit = col_double(),
##   ..   Ethnicity = col_character(),
##   ..   Diagnosis = col_character(),
##   ..   ASD_check = col_double(),
##   ..   ASD2 = col_double(),
##   ..   Gender = col_double(),
##   ..   Birthdate = col_character(),
##   ..   Age = col_double(),
##   ..   Total..Understands...Says. = col_double(),
##   ..   Total..Understands. = col_double(),
##   ..   Total.of.Both = col_double(),
##   ..   Age2 = col_double(),
##   ..   ADOS = col_double(),
##   ..   CARS = col_double(),
##   ..   CDI1 = col_double(),
##   ..   VinelandStandardScore = col_double(),
##   ..   VinelandReceptive = col_double(),
##   ..   VinelandExpressive = col_double(),
##   ..   VinelandWritten = col_double(),
##   ..   DailyLivingSkills = col_double(),
##   ..   Socialization = col_double(),
##   ..   MotorSkills = col_double(),
##   ..   MullenRaw = col_double(),
##   ..   MullenTScore = col_double(),
##   ..   MullenAge = col_double(),
##   ..   FineMotorRaw = col_double(),
##   ..   FineMotorTScore = col_double(),
##   ..   FIneMotorAge = col_double(),
##   ..   ReceptiveLanguageRaw = col_double(),
##   ..   ReceptiveLanguageTScore = col_double(),
##   ..   ReceptiveLanguageAge = col_double(),
##   ..   ExpressiveLangRaw = col_double(),
##   ..   ExpressiveLangTScore = col_double(),
##   ..   ExpressiveLangAge = col_double(),
##   ..   EarlyLearningComposite = col_double()
##   .. )
```

2b. Find a way to homogeneize the way "visit" is reported (visit1 vs. 1).

Tip: The stringr package is what you need. str_extract () will allow you to extract only the digit (number) from a string, by using the regular expression \\d.



```r
#how are they represented. 
compare_df_cols(demo_train, LU_train, token_train)
```

```
##                   column_name demo_train  LU_train token_train
## 1                        ADOS    numeric      <NA>        <NA>
## 2                         Age    numeric      <NA>        <NA>
## 3                        Age2    numeric      <NA>        <NA>
## 4                   ASD_check    numeric      <NA>        <NA>
## 5                        ASD2    numeric      <NA>        <NA>
## 6                   Birthdate  character      <NA>        <NA>
## 7                        CARS    numeric      <NA>        <NA>
## 8                        CDI1    numeric      <NA>        <NA>
## 9                   CHI_LU_q1       <NA>   numeric        <NA>
## 10                  CHI_LU_q2       <NA>   numeric        <NA>
## 11                  CHI_LU_q3       <NA>   numeric        <NA>
## 12                  CHI_LUstd       <NA>   numeric        <NA>
## 13                    CHI_MLU       <NA>   numeric        <NA>
## 14          DailyLivingSkills    numeric      <NA>        <NA>
## 15                  Diagnosis  character      <NA>        <NA>
## 16     EarlyLearningComposite    numeric      <NA>        <NA>
## 17                  Ethnicity  character      <NA>        <NA>
## 18          ExpressiveLangAge    numeric      <NA>        <NA>
## 19          ExpressiveLangRaw    numeric      <NA>        <NA>
## 20       ExpressiveLangTScore    numeric      <NA>        <NA>
## 21               FIneMotorAge    numeric      <NA>        <NA>
## 22               FineMotorRaw    numeric      <NA>        <NA>
## 23            FineMotorTScore    numeric      <NA>        <NA>
## 24                     Gender    numeric      <NA>        <NA>
## 25                  MOT_LU_q1       <NA>   numeric        <NA>
## 26                  MOT_LU_q2       <NA>   numeric        <NA>
## 27                  MOT_LU_q3       <NA>   numeric        <NA>
## 28                  MOT_LUstd       <NA>   numeric        <NA>
## 29                    MOT_MLU       <NA>   numeric        <NA>
## 30                MotorSkills    numeric      <NA>        <NA>
## 31                  MullenAge    numeric      <NA>        <NA>
## 32                  MullenRaw    numeric      <NA>        <NA>
## 33               MullenTScore    numeric      <NA>        <NA>
## 34       ReceptiveLanguageAge    numeric      <NA>        <NA>
## 35       ReceptiveLanguageRaw    numeric      <NA>        <NA>
## 36    ReceptiveLanguageTScore    numeric      <NA>        <NA>
## 37              Socialization    numeric      <NA>        <NA>
## 38                       SUBJ  character character   character
## 39                 tokens_CHI       <NA>      <NA>     numeric
## 40                 tokens_MOT       <NA>      <NA>     numeric
## 41        Total..Understands.    numeric      <NA>        <NA>
## 42 Total..Understands...Says.    numeric      <NA>        <NA>
## 43              Total.of.Both    numeric      <NA>        <NA>
## 44                  types_CHI       <NA>      <NA>     numeric
## 45                  types_MOT       <NA>      <NA>     numeric
## 46               types_shared       <NA>      <NA>     numeric
## 47         VinelandExpressive    numeric      <NA>        <NA>
## 48          VinelandReceptive    numeric      <NA>        <NA>
## 49      VinelandStandardScore    numeric      <NA>        <NA>
## 50            VinelandWritten    numeric      <NA>        <NA>
## 51                      VISIT    numeric character   character
## 52                          X       <NA>      <NA>     logical
```

```r
#SOLUTION 1: 
LU_train <- LU_train %>%
  mutate_at("VISIT", str_extract, "\\d") %>% #(.tbl, .funs, ...) 
  mutate(VISIT = as.integer(VISIT)) #or as_factor()

token_train <- token_train %>%
  mutate_at("VISIT", str_extract, "\\d") %>%
  mutate(VISIT = as.integer(VISIT))

demo_train <- demo_train %>%
  mutate(VISIT = as.integer(VISIT))


#solution 2 (function):
#define function. 
extractor <- function(df){
  
df <- df %>%
  mutate_at("VISIT", str_extract, "\\d") %>%
  mutate(VISIT = as.integer(VISIT))
  
}

#use function. 
demo_train <- extractor(demo_train)
LU_train <- extractor(LU_train)
token_train <- extractor(token_train)

#check same type. 
compare_df_cols(demo_train, LU_train, token_train)
```

```
##                   column_name demo_train  LU_train token_train
## 1                        ADOS    numeric      <NA>        <NA>
## 2                         Age    numeric      <NA>        <NA>
## 3                        Age2    numeric      <NA>        <NA>
## 4                   ASD_check    numeric      <NA>        <NA>
## 5                        ASD2    numeric      <NA>        <NA>
## 6                   Birthdate  character      <NA>        <NA>
## 7                        CARS    numeric      <NA>        <NA>
## 8                        CDI1    numeric      <NA>        <NA>
## 9                   CHI_LU_q1       <NA>   numeric        <NA>
## 10                  CHI_LU_q2       <NA>   numeric        <NA>
## 11                  CHI_LU_q3       <NA>   numeric        <NA>
## 12                  CHI_LUstd       <NA>   numeric        <NA>
## 13                    CHI_MLU       <NA>   numeric        <NA>
## 14          DailyLivingSkills    numeric      <NA>        <NA>
## 15                  Diagnosis  character      <NA>        <NA>
## 16     EarlyLearningComposite    numeric      <NA>        <NA>
## 17                  Ethnicity  character      <NA>        <NA>
## 18          ExpressiveLangAge    numeric      <NA>        <NA>
## 19          ExpressiveLangRaw    numeric      <NA>        <NA>
## 20       ExpressiveLangTScore    numeric      <NA>        <NA>
## 21               FIneMotorAge    numeric      <NA>        <NA>
## 22               FineMotorRaw    numeric      <NA>        <NA>
## 23            FineMotorTScore    numeric      <NA>        <NA>
## 24                     Gender    numeric      <NA>        <NA>
## 25                  MOT_LU_q1       <NA>   numeric        <NA>
## 26                  MOT_LU_q2       <NA>   numeric        <NA>
## 27                  MOT_LU_q3       <NA>   numeric        <NA>
## 28                  MOT_LUstd       <NA>   numeric        <NA>
## 29                    MOT_MLU       <NA>   numeric        <NA>
## 30                MotorSkills    numeric      <NA>        <NA>
## 31                  MullenAge    numeric      <NA>        <NA>
## 32                  MullenRaw    numeric      <NA>        <NA>
## 33               MullenTScore    numeric      <NA>        <NA>
## 34       ReceptiveLanguageAge    numeric      <NA>        <NA>
## 35       ReceptiveLanguageRaw    numeric      <NA>        <NA>
## 36    ReceptiveLanguageTScore    numeric      <NA>        <NA>
## 37              Socialization    numeric      <NA>        <NA>
## 38                       SUBJ  character character   character
## 39                 tokens_CHI       <NA>      <NA>     numeric
## 40                 tokens_MOT       <NA>      <NA>     numeric
## 41        Total..Understands.    numeric      <NA>        <NA>
## 42 Total..Understands...Says.    numeric      <NA>        <NA>
## 43              Total.of.Both    numeric      <NA>        <NA>
## 44                  types_CHI       <NA>      <NA>     numeric
## 45                  types_MOT       <NA>      <NA>     numeric
## 46               types_shared       <NA>      <NA>     numeric
## 47         VinelandExpressive    numeric      <NA>        <NA>
## 48          VinelandReceptive    numeric      <NA>        <NA>
## 49      VinelandStandardScore    numeric      <NA>        <NA>
## 50            VinelandWritten    numeric      <NA>        <NA>
## 51                      VISIT    integer   integer     integer
## 52                          X       <NA>      <NA>     logical
```

2c. We also need to make a small adjustment to the content of the Child.ID column in the demographic data. Within this column, names that are not abbreviations do not end with "." (i.e. Adam), which is the case in the other two data sets (i.e. Adam.). If The content of the two variables isn't identical the rows will not be merged.
A neat way to solve the problem is simply to remove all "." in all datasets.

Tip: stringr is helpful again. Look up str_replace_all
Tip: You can either have one line of code for each child name that is to be changed (easier, more typing) or specify the pattern that you want to match (more complicated: look up "regular expressions", but less typing)



```r
#Solution 1 (less typing): 
#working on the "SUBJ"-column. 
str(demo_train$SUBJ) #has dots.
```

```
##  chr [1:372] "A.A." "A.D." "A.D." "A.D." "A.D." "A.D." "A.D." "A.H." "A.H." ...
```

```r
str(LU_train$SUBJ) #has dots. 
```

```
##  chr [1:352] "A.D." "A.D." "A.D." "A.D." "A.D." "A.D." "A.H." "A.H." "A.H." ...
```

```r
str(token_train$SUBJ) #has dots. 
```

```
##  chr [1:352] "A.D." "A.D." "A.D." "A.D." "A.D." "A.D." "A.H." "A.H." "A.H." ...
```

```r
demo_train <- demo_train %>%
   mutate_at("SUBJ", str_replace_all, "\\.", "")

LU_train <- LU_train %>%
  mutate_at("SUBJ", str_replace_all, "\\.", "")

token_train <- token_train %>%
  mutate_at("SUBJ", str_replace_all, "[.]", "") #the same. 

str(demo_train$SUBJ)
```

```
##  chr [1:372] "AA" "AD" "AD" "AD" "AD" "AD" "AD" "AH" "AH" "AH" "AH" "AH" ...
```

```r
str(LU_train$SUBJ)
```

```
##  chr [1:352] "AD" "AD" "AD" "AD" "AD" "AD" "AH" "AH" "AH" "AH" "AH" "AH" ...
```

```r
str(token_train$SUBJ)
```

```
##  chr [1:352] "AD" "AD" "AD" "AD" "AD" "AD" "AH" "AH" "AH" "AH" "AH" "AH" ...
```

```r
#Solution 2 (function): 
stringer <- function(df){
  
df <- df %>%
  mutate_at("SUBJ", str_replace_all, "\\.", "") # "\\." because "\" escapes. 
  
}

#using the function: 
demo_train <- stringer(demo_train)
LU_train <- stringer(LU_train)
token_train <- stringer(token_train)

#Solution 3 (one_liner), but I like this less: 
demo_train$SUBJ <-  str_replace_all(demo_train$SUBJ,"\\.", "")
```

2d. Now that the nitty gritty details of the different data sets are fixed, we want to make a subset of each data set only containig the variables that we wish to use in the final data set.
For this we use the tidyverse package dplyr, which contains the function select(). 

The variables we need are: 
* Child.ID, 
* Visit, 
* Diagnosis, 
* Ethnicity, 
* Gender, 
* Age, 
* ADOS,  
* MullenRaw, 
* ExpressiveLangRaw, 
* Socialization
* MOT_MLU, 
* CHI_MLU, 
* types_MOT, 
* types_CHI, 
* tokens_MOT, 
* tokens_CHI.

Most variables should make sense, here the less intuitive ones. 
* ADOS (Autism Diagnostic Observation Schedule) indicates the severity of the autistic symptoms (the higher the score, the worse the symptoms). Ref: https://link.springer.com/article/10.1023/A:1005592401947
* MLU stands for mean length of utterance (usually a proxy for syntactic complexity)
* types stands for unique words (e.g. even if "doggie" is used 100 times it only counts for 1)
* tokens stands for overall amount of words (if "doggie" is used 100 times it counts for 100) 
* MullenRaw indicates non verbal IQ, as measured by Mullen Scales of Early Learning (MSEL https://link.springer.com/referenceworkentry/10.1007%2F978-1-4419-1698-3_596)
* ExpressiveLangRaw indicates verbal IQ, as measured by MSEL
* Socialization indicates social interaction skills and social responsiveness, as measured by Vineland (https://cloudfront.ualberta.ca/-/media/ualberta/faculties-and-programs/centres-institutes/community-university-partnership/resources/tools---assessment/vinelandjune-2012.pdf)

Feel free to rename the variables into something you can remember (i.e. nonVerbalIQ, verbalIQ)


```r
"
* Child.ID, 
* Visit, 
* Diagnosis, 
* Ethnicity, 
* Gender, 
* Age, 
* ADOS,  
* MullenRaw, 
* ExpressiveLangRaw, 
* Socialization
* MOT_MLU, 
* CHI_MLU, 
* types_MOT, 
* types_CHI, 
* tokens_MOT, 
* tokens_CHI.
"
```

```
## [1] "\n* Child.ID, \n* Visit, \n* Diagnosis, \n* Ethnicity, \n* Gender, \n* Age, \n* ADOS,  \n* MullenRaw, \n* ExpressiveLangRaw, \n* Socialization\n* MOT_MLU, \n* CHI_MLU, \n* types_MOT, \n* types_CHI, \n* tokens_MOT, \n* tokens_CHI.\n"
```

```r
# function: 
selector <- function(df){
  
  df <- df %>%
    select(SUBJ, VISIT, Diagnosis, Ethnicity,
           Gender, Age, ADOS, MullenRaw, ExpressiveLangRaw,
           Socialization, MOT_MLU, CHI_MLU, types_MOT,
           types_CHI, tokens_MOT, tokens_CHI)

}

# fails with those that do not exist: 
#selector(demo_train)
#selector(LU_train)
#selector(token_train)

# function (second try): 
selector2 <- function(df){
  
  df <- df %>%
    select(one_of(c("SUBJ", "VISIT", "Diagnosis", "Ethnicity",
           "Gender", "Age", "ADOS", "MullenRaw", "ExpressiveLangRaw",
           "Socialization", "MOT_MLU", "CHI_MLU", "types_MOT",
           "types_CHI", "tokens_MOT", "tokens_CHI"))) #one_of!! 

}

#this approach works. 
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

```r
#another approach is simply to select(c(....)) at manually select 
#all the appropriate columns (that are in the data-sets). 
```

2e. Finally we are ready to merge all the data sets into just one. 

Some things to pay attention to:
* make sure to check that the merge has included all relevant data (e.g. by comparing the number of rows)
* make sure to understand whether (and if so why) there are NAs in the dataset (e.g. some measures were not taken at all visits, some recordings were lost or permission to use was withdrawn)


```r
#Solution 1: (pretty dumb, does the job (giving us 372 obs). 
merged_data <- merge(demo_train, LU_train, all = T) #all rows returned. 
merged_data <- merge(merged_data, token_train, all = T) 

#Solution 2: (pipe?)
```


2f. Only using clinical measures from Visit 1
In order for our models to be useful, we want to minimize the need to actually test children as they develop. In other words, we would like to be able to understand and predict the children's linguistic development after only having tested them once. Therefore we need to make sure that our ADOS, MullenRaw, ExpressiveLangRaw and Socialization variables are reporting (for all visits) only the scores from visit 1.

A possible way to do so:
* create a new dataset with only visit 1, child id and the 4 relevant clinical variables to be merged with the old dataset
* rename the clinical variables (e.g. ADOS to ADOS1) and remove the visit (so that the new clinical variables are reported for all 6 visits)
* merge the new dataset with the old




```r
#new_data; VISIT == 1, ID & 4 clinical variables. 
new_data <- merged_data %>%
  filter(VISIT == 1) %>%
  select(SUBJ,
         ADOS1 = ADOS, #new_name = old_name. 
         verbalIQ1 = ExpressiveLangRaw,
         nonVerbalIQ1 = MullenRaw,
         Socialization1 = Socialization)
  
data <- merge(new_data, merged_data, all = T)
```

2g. Final touches

Now we want to
* anonymize our participants (they are real children!). 
* make sure the variables have sensible values. E.g. right now gender is marked 1 and 2, but in two weeks you will not be able to remember, which gender were connected to which number, so change the values from 1 and 2 to F and M in the gender variable. For the same reason, you should also change the values of Diagnosis from A and B to ASD (autism spectrum disorder) and TD (typically developing). Tip: Try taking a look at ifelse(), or google "how to rename levels in R".
* Save the data set using into a csv file. Hint: look into write.csv()


```r
#anonymization. 
str(data$SUBJ)
```

```
##  chr [1:372] "AA" "AD" "AD" "AD" "AD" "AD" "AD" "Adam" "Adam" "Adam" "Adam" ...
```

```r
str(data$VISIT)
```

```
##  int [1:372] 1 1 2 3 4 5 6 1 2 3 ...
```

```r
data <- data %>%
  mutate(SUBJ = as.numeric(as_factor(SUBJ))) %>%
  mutate(VISIT = as.numeric(VISIT))

#relabelling (gender, diagnosis): use ifelse() 
#Approach 1 (tidyverse, not ifelse). 
data <- data %>%
  mutate(Gender = recode(Gender,
    "1" = "F", #not clear whether this or opposite. 
    "2" = "M" #the text and Ricco's code disagrees. 
  ),
  Diagnosis = recode(Diagnosis,
    "A" = "ASD",
    "B" = "TD"
  ))

#Approach 2 (ifelse). Pretty sure this works (& it is better).
data_test <- data %>%
  mutate(Gender = if_else(Gender == "F", "1", "2"))

#remove everything but "data" from working environment. 
rm(list=setdiff(ls(), "data")) 

#write.csv(), or perhaps write_csv()? 
write_csv(data, "data_cleaned_draft.csv") #check that this matches. 

#read it to check. 
test <- read_csv("data_cleaned_draft.csv")
```

```
## Parsed with column specification:
## cols(
##   .default = col_double(),
##   Diagnosis = col_character(),
##   Ethnicity = col_character(),
##   Gender = col_character()
## )
```

```
## See spec(...) for full column specifications.
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
## 1     2     1 1.252252        TD
## 2     6     1 1.394737       ASD
## 3     7     1 1.000000       ASD
## 4     8     1 1.264151       ASD
## 5    10     1 1.037879        TD
## 6    11     1 1.037500        TD
## 7    13     1 1.216867        TD
## 8    14     1 1.087719        TD
## 9    16     1 1.039604        TD
## 10   17     1 1.164706        TD
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
## 1    16     6 3.811404        TD
## 2     4     6 3.710345        TD
## 3    15     6 3.701195        TD
## 4    54     6 3.595000        TD
## 5    14     6 3.504950        TD
## 6    65     6 3.441558       ASD
## 7     3     6 3.413502       ASD
## 8    29     6 3.364341       ASD
## 9     5     6 3.278195       ASD
## 10   58     6 3.243902        TD
```

```r
data %>% filter(VISIT == 1) %>%
  arrange(CHI_MLU) %>%
  select(SUBJ, VISIT, CHI_MLU, Diagnosis) %>%
  slice_head(n = 10)
```

```
##    SUBJ VISIT   CHI_MLU Diagnosis
## 1    62     1 0.0000000       ASD
## 2    22     1 0.1857143       ASD
## 3    40     1 0.4805825       ASD
## 4    48     1 0.5584416        TD
## 5    33     1 0.9000000       ASD
## 6     7     1 1.0000000       ASD
## 7    23     1 1.0000000       ASD
## 8    32     1 1.0086207       ASD
## 9    43     1 1.0175439       ASD
## 10   44     1 1.0262009        TD
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
##    Diagnosis  CHI_MLU tokens_CHI
## 1        ASD 4.302326        674
## 2        ASD 4.131868        698
## 3        ASD 4.043478        539
## 4        ASD 3.919689       1293
## 5        ASD 3.523810        714
## 6        ASD 3.518072        490
## 7        ASD 3.453039        562
## 8        ASD 3.441558        713
## 9        ASD 3.413502        698
## 10       ASD 3.400000        483
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
