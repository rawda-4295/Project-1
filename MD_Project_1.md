-   [Project 1: NYC Community Health Survey 2020: HIV Testing and Condom
    Use](#project-1-nyc-community-health-survey-2020-hiv-testing-and-condom-use)
-   [Executive Summary](#executive-summary)
-   [1. Introduction](#introduction)
-   [2. Data Description](#data-description)
-   [3. Data Preparation](#data-preparation)
    -   [3.1 Import Data](#import-data)
    -   [3.2 Assign and Create Variables](#assign-and-create-variables)
    -   [3.3 Data Cleaning](#data-cleaning)
-   [4. Exploratory Data Analysis (EDA)](#exploratory-data-analysis-eda)
    -   [4.1. Summary Table](#summary-table)
    -   [4.2 Bar Charts](#bar-charts)
    -   [4.3 Pie Charts](#pie-charts)
    -   [4.4 Box Plots](#box-plots)
    -   [4.5 Summary Report](#summary-report)
-   [5. Statistical Modeling](#statistical-modeling)
    -   [5.1 Unadjusted Logistic
        Regression](#unadjusted-logistic-regression)
    -   [5.2 Adjusted Logistic
        Regression](#adjusted-logistic-regression)
    -   [5.3 Adjusted LRM Table](#adjusted-lrm-table)
-   [6. Discussion](#discussion)
-   [7. Conclusion](#conclusion)
-   [8. Refrences and Resources](#refrences-and-resources)

# Project 1: NYC Community Health Survey 2020: HIV Testing and Condom Use

**Rawda Alaswad**  
**Date:** July 15, 2025

# Executive Summary

This report analyzes data from the 2020 New York City Community Health
Survey (CHS) to examine demographic and behavioral factors associated
with HIV testing among adults in NYC. The survey included approximately
10,000 adults, with data collected via a cross-sectional, self-reported
telephone survey in multiple languages.

Key variables analyzed include race/ethnicity, age group, HIV testing in
the past 12 months, condom use, and number of sex partners. Data were
cleaned and recoded for clarity, and survey weights were applied to
account for the complex sampling design.

Descriptive analyses showed that the largest racial/ethnic group was
White/North African/Middle Eastern, non-Hispanic, and most participants
were aged 25–64. Only 28.6% of respondents reported being tested for HIV
in the past year, and 17.3% reported condom use at last sex.

Logistic regression models were used to assess associations between HIV
testing and behavioral/demographic factors. In un-adjusted analysis,
condom use was significantly associated with higher odds of HIV testing.
However, after adjusting for age group, race/ethnicity, and number of
sex partners, this association was no longer statistically significant.
Black, Hispanic, and Other non-Hispanic participants, as well as those
with more sex partners, were more likely to have been tested for HIV,
while older age groups were less likely.

These findings highlight disparities in HIV testing and suggest the need
for targeted public health interventions to increase testing rates among
specific demographic groups in NYC.

# 1. Introduction

This report utilizes data from the 2020 New York City Community Health
Survey (CHS) to explore patterns of HIV testing among adults in NYC. The
analysis focuses on the relationships between HIV testing and key
variables such as race/ethnicity, age group, condom use, and number of
sex partners. By identifying groups with lower testing rates and
examining associated behaviors, this study aims to provide actionable
insights for improving HIV prevention and care in NYC.

# 2. Data Description

**Source:** The data used in this project is the New York City Community
Health Survey (CHS) of 2020, conducted annually by the NYC Department of
Health and Mental Hygiene (DOHMH).
<https://www.nyc.gov/site/doh/data/data-sets/community-health-survey-public-use-data.page>

**Sample Size:** The survey includes approximately 10,000 adults aged 18
and older, residing in New York City and its five boroughs.

**Collection Method:** Data were collected via a cross-sectional,
self-reported telephone survey. Interviews were conducted in English,
Spanish, Russian, and Chinese (Mandarin and Cantonese).

**Variables Key:**

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Original Variable Name</th>
<th>Recoded Name/ Label</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>newrace</td>
<td>Race/Ethnicity</td>
<td>Self-identified race/ethnicity of participant</td>
</tr>
<tr class="even">
<td>agegroup</td>
<td>Age Group</td>
<td>Age group in years</td>
</tr>
<tr class="odd">
<td>hiv12months20</td>
<td>Tested for HIV (Past 12 Months)</td>
<td>Whether participant was tested for HIV in past 12 months</td>
</tr>
<tr class="even">
<td>condom20</td>
<td>Condom Use (Past 12 Months)</td>
<td>Whether participant used a condom the last time they had sex</td>
</tr>
<tr class="odd">
<td>sexpartner</td>
<td>Number of Sex Partners (Past 12 Months)</td>
<td>Number of sex partners in the past 12 months</td>
</tr>
</tbody>
</table>

**Statistical Methods:** Survey weights and design variables were used
to account for the complex sampling design. Descriptive statistics,
summary tables, and visualizations were produced. Logistic regression
models (both unadjusted and adjusted for confounders) were fitted using
the ‘survey’ package in R to assess associations between HIV testing and
behavioral/ demographic factors.

**Libraries Used:**

    library(haven)
    library(dplyr)
    library(survey)
    library(ggplot2)
    library(forcats)
    library(broom)
    library(knitr)
    library(viridis)

# 3. Data Preparation

## 3.1 Import Data

-   The data was downlaoded on to local harddrive from the NYC CHS
    website (2020 SAS Data File named chs2020\_public.sas7bdat). The
    data set was assigned the named chs20 for this project.

<!-- -->

    library(haven)
    chs20 <- read_sas("C:/Users/rawda/Desktop/chs2020_public.sas7bdat", 
        NULL)

    View(chs20)

## 3.2 Assign and Create Variables

-   Assigning numeric values to the categories of each factor to reocde
    them for the epi table. The recoded data set is called
    chs20\_recode.

<!-- -->

    library(dplyr)

    chs20_recode <- chs20 %>%
      mutate(
        newrace = recode_factor(newrace,
          `1` = "White/N Afri/MidEastern, non-Hispanic",
          `2` = "Black, non-Hispanic",
          `3` = "Hispanic",
          `4` = "Asian/PI, non-Hispanic",
          `5` = "Other, non-Hispanic"
        ),
        agegroup = recode_factor(agegroup,
          `1` = "18-24",
          `2` = "25-44",
          `3` = "45-64",
          `4` = "65+"
        ),
        hiv12months20 = recode_factor(hiv12months20,
          `2` = "No",
          `1` = "Yes"
        ),
        condom20 = recode_factor(condom20,
          `2` = "No",
          `1` = "Yes"
        ),
        sexpartner = factor(sexpartner, ordered = TRUE),
        strata = as.character(strata)
      ) %>%
      rename(
        "Race/Ethnicity" = newrace,
        "Age Group" = agegroup,
        "Tested for HIV (Past 12 Months)" = hiv12months20,
        "Condom Use (Past 12 Months)" = condom20,
        "Number of Sex Partners (Past 12 Months)" = sexpartner
      )

## 3.3 Data Cleaning

-   The recoded data was checked to ensure that the recoding process was
    completed correctly and that all variables and cases are present.

<!-- -->

    vars_of_interest <- c("Race/Ethnicity", "Age Group","Tested for HIV (Past 12 Months)","Condom Use (Past 12 Months)", "Number of Sex Partners (Past 12 Months)")

    colSums(is.na(chs20_recode[vars_of_interest]))

    ##                          Race/Ethnicity                               Age Group 
    ##                                       0                                      14 
    ##         Tested for HIV (Past 12 Months)             Condom Use (Past 12 Months) 
    ##                                     152                                    3688 
    ## Number of Sex Partners (Past 12 Months) 
    ##                                     861

    for (var in vars_of_interest) {
      print(var)
      print(table(chs20_recode[[var]], useNA = "ifany"))
    }

    ## [1] "Race/Ethnicity"
    ## 
    ## White/N Afri/MidEastern, non-Hispanic                   Black, non-Hispanic 
    ##                                  2859                                  1837 
    ##                              Hispanic                Asian/PI, non-Hispanic 
    ##                                  2457                                  1340 
    ##                   Other, non-Hispanic 
    ##                                   288 
    ## [1] "Age Group"
    ## 
    ## 18-24 25-44 45-64   65+  <NA> 
    ##   706  3189  2951  1921    14 
    ## [1] "Tested for HIV (Past 12 Months)"
    ## 
    ##   No  Yes <NA> 
    ## 6122 2507  152 
    ## [1] "Condom Use (Past 12 Months)"
    ## 
    ##   No  Yes <NA> 
    ## 3570 1523 3688 
    ## [1] "Number of Sex Partners (Past 12 Months)"
    ## 
    ##    1    2    3    4 <NA> 
    ## 2686 4263  374  597  861

# 4. Exploratory Data Analysis (EDA)

## 4.1. Summary Table

    # Creating Survey
    library(survey)
    chs.dsgn <-
      svydesign(
        ids = ~ 1,
        strata = ~ strata,
        weights =  ~ wt21_dual, # match to current year dataset
        data = chs20_recode, # match to current year dataset 
        nest = TRUE,
        na.rm = TRUE
      )

    knitr::kable(
      summary(chs20_recode[, c("Race/Ethnicity", "Age Group", "Tested for HIV (Past 12 Months)", "Condom Use (Past 12 Months)", "Number of Sex Partners (Past 12 Months)" )]),
      caption = "Epidemiological Summary Table"
    )

<table>
<caption>Epidemiological Summary Table</caption>
<colgroup>
<col style="width: 1%" />
<col style="width: 28%" />
<col style="width: 6%" />
<col style="width: 20%" />
<col style="width: 17%" />
<col style="width: 25%" />
</colgroup>
<thead>
<tr class="header">
<th style="text-align: left;"></th>
<th style="text-align: left;">Race/Ethnicity</th>
<th style="text-align: left;">Age Group</th>
<th style="text-align: left;">Tested for HIV (Past 12 Months)</th>
<th style="text-align: left;">Condom Use (Past 12 Months)</th>
<th style="text-align: left;">Number of Sex Partners (Past 12
Months)</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;"></td>
<td style="text-align: left;">White/N Afri/MidEastern,
non-Hispanic:2859</td>
<td style="text-align: left;">18-24: 706</td>
<td style="text-align: left;">No :6122</td>
<td style="text-align: left;">No :3570</td>
<td style="text-align: left;">1 :2686</td>
</tr>
<tr class="even">
<td style="text-align: left;"></td>
<td style="text-align: left;">Black, non-Hispanic :1837</td>
<td style="text-align: left;">25-44:3189</td>
<td style="text-align: left;">Yes :2507</td>
<td style="text-align: left;">Yes :1523</td>
<td style="text-align: left;">2 :4263</td>
</tr>
<tr class="odd">
<td style="text-align: left;"></td>
<td style="text-align: left;">Hispanic :2457</td>
<td style="text-align: left;">45-64:2951</td>
<td style="text-align: left;">NA’s: 152</td>
<td style="text-align: left;">NA’s:3688</td>
<td style="text-align: left;">3 : 374</td>
</tr>
<tr class="even">
<td style="text-align: left;"></td>
<td style="text-align: left;">Asian/PI, non-Hispanic :1340</td>
<td style="text-align: left;">65+ :1921</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">4 : 597</td>
</tr>
<tr class="odd">
<td style="text-align: left;"></td>
<td style="text-align: left;">Other, non-Hispanic : 288</td>
<td style="text-align: left;">NA’s : 14</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">NA’s: 861</td>
</tr>
</tbody>
</table>

Epidemiological Summary Table

    colnames(chs20_recode)

    ##   [1] "cid"                                    
    ##   [2] "strata"                                 
    ##   [3] "survey"                                 
    ##   [4] "wt21_dual"                              
    ##   [5] "wt21_dual_q1"                           
    ##   [6] "strata_q1"                              
    ##   [7] "qxvers"                                 
    ##   [8] "mood1"                                  
    ##   [9] "mood2"                                  
    ##  [10] "mood3"                                  
    ##  [11] "mood4"                                  
    ##  [12] "mood5"                                  
    ##  [13] "mood6"                                  
    ##  [14] "mood9"                                  
    ##  [15] "mood8"                                  
    ##  [16] "mood11"                                 
    ##  [17] "nutrition1"                             
    ##  [18] "Race/Ethnicity"                         
    ##  [19] "newrace6"                               
    ##  [20] "Age Group"                              
    ##  [21] "agegroup5"                              
    ##  [22] "agegroup6"                              
    ##  [23] "age21up"                                
    ##  [24] "age25up"                                
    ##  [25] "age40new"                               
    ##  [26] "age45up"                                
    ##  [27] "age50up"                                
    ##  [28] "age18_64"                               
    ##  [29] "birthsex"                               
    ##  [30] "imputed_neighpovgroup4_1519"            
    ##  [31] "imputed_povertygroup"                   
    ##  [32] "imputed_povgroup3"                      
    ##  [33] "imputed_pov200"                         
    ##  [34] "generalhealth"                          
    ##  [35] "insuredgateway20"                       
    ##  [36] "insured"                                
    ##  [37] "insure5"                                
    ##  [38] "pcp20"                                  
    ##  [39] "medplace"                               
    ##  [40] "didntgetcare20"                         
    ##  [41] "regularrx"                              
    ##  [42] "skiprxcost"                             
    ##  [43] "toldhighbp20"                           
    ##  [44] "toldprescription20"                     
    ##  [45] "takingmeds20"                           
    ##  [46] "checkedbp20_q1"                         
    ##  [47] "diabetes20"                             
    ##  [48] "ageatdiabetes"                          
    ##  [49] "diabcntrlmeds"                          
    ##  [50] "toohighblsugar"                         
    ##  [51] "everasthma"                             
    ##  [52] "currentasthma20"                        
    ##  [53] "stillasthmaall"                         
    ##  [54] "firsttoldasthma"                        
    ##  [55] "k6"                                     
    ##  [56] "nspd"                                   
    ##  [57] "mhtreat20_all"                          
    ##  [58] "delaypayrent"                           
    ##  [59] "workingac_q1"                           
    ##  [60] "rodentsstreet"                          
    ##  [61] "helpneighbors20_q1"                     
    ##  [62] "discussissues"                          
    ##  [63] "helpcommproj"                           
    ##  [64] "didntcleandog"                          
    ##  [65] "trustkeys"                              
    ##  [66] "proudneigh"                             
    ##  [67] "smoker"                                 
    ##  [68] "everyday"                               
    ##  [69] "numberperdaya"                          
    ##  [70] "cpd20a"                                 
    ##  [71] "heavysmoker20a"                         
    ##  [72] "everydaycpda"                           
    ##  [73] "smokecat"                               
    ##  [74] "mentholcigs20"                          
    ##  [75] "sourcelastcig"                          
    ##  [76] "cost20cigarettes"                       
    ##  [77] "cigpurchase20"                          
    ##  [78] "cigarillo20_q1"                         
    ##  [79] "smokeecig12m20_q1"                      
    ##  [80] "smokeecig30days20_q1"                   
    ##  [81] "likedecigsflavs_q1"                     
    ##  [82] "smokehookah12m_q1"                      
    ##  [83] "smellcigsmoke20_q1"                     
    ##  [84] "newrace6_b"                             
    ##  [85] "usborn"                                 
    ##  [86] "maritalstatus20"                        
    ##  [87] "sexualid20"                             
    ##  [88] "education"                              
    ##  [89] "employment20"                           
    ##  [90] "emp3"                                   
    ##  [91] "bmi"                                    
    ##  [92] "weightall"                              
    ##  [93] "weight20in4"                            
    ##  [94] "weight20in5"                            
    ##  [95] "fluvaccineshot"                         
    ##  [96] "whereflu20"                             
    ##  [97] "fruitveg20"                             
    ##  [98] "avgsodaperday20"                        
    ##  [99] "twoplussoda"                            
    ## [100] "nsugardrinkperday20"                    
    ## [101] "avgsugarperday20"                       
    ## [102] "nsodasugarperday20"                     
    ## [103] "avgsodasugarperday20"                   
    ## [104] "ssb"                                    
    ## [105] "exercise20"                             
    ## [106] "cyclingfreq"                            
    ## [107] "cycling20"                              
    ## [108] "swim"                                   
    ## [109] "difficultdailyact"                      
    ## [110] "assistdevice"                           
    ## [111] "evercolon20"                            
    ## [112] "colonoscopy10yr20"                      
    ## [113] "evercolon20_45"                         
    ## [114] "colonoscopy10yr_45"                     
    ## [115] "Tested for HIV (Past 12 Months)"        
    ## [116] "everhivtest20"                          
    ## [117] "Condom Use (Past 12 Months)"            
    ## [118] "analsex"                                
    ## [119] "analstdtest"                            
    ## [120] "analsexcondomuse20"                     
    ## [121] "sexbehav_active20"                      
    ## [122] "wsw"                                    
    ## [123] "wswexclusive"                           
    ## [124] "sexuallyactive20"                       
    ## [125] "Number of Sex Partners (Past 12 Months)"
    ## [126] "everheardofprep"                        
    ## [127] "everusedprep20"                         
    ## [128] "msm"                                    
    ## [129] "msmexclusive"                           
    ## [130] "bthcontrollastsex20_q1"                 
    ## [131] "condomusetrend"                         
    ## [132] "drinker"                                
    ## [133] "daysalc30"                              
    ## [134] "averagedrink20"                         
    ## [135] "heavydrink20"                           
    ## [136] "bingenew"                               
    ## [137] "ipvphy"                                 
    ## [138] "insultipv"                              
    ## [139] "wt_compare"                             
    ## [140] "insure20r"                              
    ## [141] "hhsize"                                 
    ## [142] "child"

## 4.2 Bar Charts

    library(ggplot2)

    bar_vars <- c("Race/Ethnicity", "Age Group","Tested for HIV (Past 12 Months)", "Condom Use (Past 12 Months)",
      "Number of Sex Partners (Past 12 Months)")

    for (var in bar_vars) {
      p <- ggplot(chs20_recode, aes(x = .data[[var]])) +
        geom_bar(fill = "light blue") +
        geom_text(stat = "count", aes(label = ..count..), vjust = -0.5, size = 3) +
        labs(
          title = paste("Distribution of", var),
          x = var,
          y = "Count"
        ) +
        theme_minimal()
      
      print(p)
      safe_var <- gsub("[ /]", "_", var)
      ggsave(filename = paste0("C:/Users/rawda/Desktop/Notion/", safe_var, "_bar_chart.png"), plot = p, width = 6, height = 4)
    }

![](MD_Project_1_files/figure-markdown_strict/unnamed-chunk-8-1.png)![](MD_Project_1_files/figure-markdown_strict/unnamed-chunk-8-2.png)![](MD_Project_1_files/figure-markdown_strict/unnamed-chunk-8-3.png)![](MD_Project_1_files/figure-markdown_strict/unnamed-chunk-8-4.png)![](MD_Project_1_files/figure-markdown_strict/unnamed-chunk-8-5.png)

![](Race_Ethnicity_bar_chart.png) ![](Age_Group_bar_chart.png)
![](Tested_HIV_12mo_bar_chart.png) ![](Condom_Use_12mo_bar_chart.png)
![](Num_Sex_Partners_12mo_bar_chart.png)

## 4.3 Pie Charts

    library(ggplot2)
    library(dplyr)

    cat_vars <- c(
      "Race/Ethnicity",
      "Age Group",
      "Tested for HIV (Past 12 Months)",
      "Condom Use (Past 12 Months)",
      "Number of Sex Partners (Past 12 Months)"
    )

    for (var in cat_vars) {
      pie_data <- chs20_recode %>%
        count(.data[[var]]) %>%
        mutate(perc = n / sum(n) * 100,
               label = paste0(.data[[var]], " (", round(perc, 1), "%)"))
      
      p <- ggplot(pie_data, aes(x = "", y = n, fill = .data[[var]])) +
        geom_col(width = 1) +
        coord_polar(theta = "y") +
        labs(
          title = paste("Distribution of", var),
          fill = var
        ) +
        theme_void() +
        geom_text(aes(label = label), position = position_stack(vjust = 0.5), size = 3)
      
      print(p)
      safe_var <- gsub("[ /]", "_", var)
      ggsave(filename = paste0("C:/Users/rawda/Desktop/Notion/", safe_var, "_pie_chart.png"), plot = p, width = 6, height = 6)
    }

![](MD_Project_1_files/figure-markdown_strict/unnamed-chunk-9-1.png)![](MD_Project_1_files/figure-markdown_strict/unnamed-chunk-9-2.png)![](MD_Project_1_files/figure-markdown_strict/unnamed-chunk-9-3.png)![](MD_Project_1_files/figure-markdown_strict/unnamed-chunk-9-4.png)![](MD_Project_1_files/figure-markdown_strict/unnamed-chunk-9-5.png)

![](Race_Ethnicity_pie_chart.png) ![](Age_Group_pie_chart.png)
![](Tested_HIV_12mo_pie_chart.png) ![](Condom_Use_12mo_pie_chart.png)
![](Num_Sex_Partners_12mo_pie_chart.png)

## 4.4 Box Plots

    library(ggplot2)
    library(forcats)
    library(dplyr)
    library(viridis)

    box_vars <- c(
      "Condom Use (Past 12 Months)",
      "Race/Ethnicity",
      "Number of Sex Partners (Past 12 Months)"
    )

    chs20_recode$`Age Group` <- forcats::fct_explicit_na(chs20_recode$`Age Group`, na_level = "Missing")
    chs20_recode$`Age Group` <- factor(
      chs20_recode$`Age Group`,
      levels = c("18-24", "25-44", "45-64", "65+", "Missing")
    )

    for (fill_var in box_vars) {
      plot_data <- chs20_recode %>%
        group_by(`Tested for HIV (Past 12 Months)`, .data[[fill_var]], `Age Group`) %>%
        summarise(n = n(), .groups = "drop") %>%
        group_by(`Tested for HIV (Past 12 Months)`, `Age Group`) %>%
        mutate(perc = n / sum(n) * 100)
      
      p <- ggplot(plot_data, aes(x = `Tested for HIV (Past 12 Months)`, y = n, fill = .data[[fill_var]])) +
        geom_bar(stat = "identity", position = "stack", color = "black") +
        facet_grid(. ~ `Age Group`, labeller = label_value) +
        labs(
          title = paste("HIV Testing by", fill_var),
          subtitle = "Stratified by Age Group",
          x = "Tested for HIV (Past 12 Months)",
          y = "Number of Participants",
          fill = fill_var
        ) +
        scale_fill_viridis_d(option = "D") +
        theme_bw()
      
      print(p)
      safe_var <- gsub("[ /()]", "_", fill_var)
      ggsave(filename = paste0("C:/Users/rawda/Desktop/Notion/HIV_Testing_by_", safe_var, "_bar_chart.png"), plot = p, width = 7, height = 4)
    }

![](MD_Project_1_files/figure-markdown_strict/unnamed-chunk-10-1.png)![](MD_Project_1_files/figure-markdown_strict/unnamed-chunk-10-2.png)![](MD_Project_1_files/figure-markdown_strict/unnamed-chunk-10-3.png)

![](HIV_Testing_by_Condom_Use_12mo_bar_chart.png)
![](HIV_Testing_by_Race_Ethnicity_bar_chart.png)
![](HIV_Testing_by_Num_Sex_Partners_12mo_bar_chart.png)

## 4.5 Summary Report

**Race/Ethnicity:** The largest group was White/North African/Middle
Eastern, non-Hispanic (32.6%), followed by Hispanic (28.0%), Black,
non-Hispanic (20.9%), Asian/Pacific Islander, non-Hispanic (15.3%), and
Other, non-Hispanic (3.3%).

**Age Group:** Most participants were aged 25–44 (36.3%) or 45–64
(33.6%), with smaller proportions aged 65+ (21.9%) and 18–24 (8.0%).

**HIV Testing (Past 12 Months):** 28.6% reported being tested for HIV in
the past year, while 69.7% had not been tested.

**Condom Use (Past 12 Months):** 17.3% reported using a condom the last
time they had sex, 40.7% did not, and 42.0% had missing data for this
question.

**Number of Sex Partners (Past 12 Months):** Nearly half (48.5%)
reported having 2 sex partners, 30.6% had 1 partner, and smaller
proportions reported 3 (4.3%) or 4 (6.8%) partners. About 9.8% had
missing data for this variable.

# 5. Statistical Modeling

## 5.1 Unadjusted Logistic Regression

This step was done to highlight the effect of confounded on p-value
significance. As you can see below there is an extremely signifncant
statistical association between condom HIV testing and condom use.
Condom use in the past 12 months was significantly associated with
higher odds of HIV testing in the past year (p &lt; 0.001).

    fit_unadj <- svyglm(`Tested for HIV (Past 12 Months)` ~ `Condom Use (Past 12 Months)`, 
                        design = chs.dsgn, 
                        family = quasibinomial())
    summary(fit_unadj)

    ## 
    ## Call:
    ## svyglm(formula = `Tested for HIV (Past 12 Months)` ~ `Condom Use (Past 12 Months)`, 
    ##     design = chs.dsgn, family = quasibinomial())
    ## 
    ## Survey design:
    ## svydesign(ids = ~1, strata = ~strata, weights = ~wt21_dual, data = chs20_recode, 
    ##     nest = TRUE, na.rm = TRUE)
    ## 
    ## Coefficients:
    ##                                  Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)                      -0.75078    0.05229 -14.359  < 2e-16 ***
    ## `Condom Use (Past 12 Months)`Yes  0.49119    0.09332   5.263 1.47e-07 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for quasibinomial family taken to be 1.072588)
    ## 
    ## Number of Fisher Scoring iterations: 4

## 5.2 Adjusted Logistic Regression

After adjusting for age group, race/ethnicity, and number of sex
partners, condom use was no longer significantly associated with HIV
testing in the past year (p = 0.34).

    fit_adj <- svyglm(
      `Tested for HIV (Past 12 Months)` ~ 
      `Condom Use (Past 12 Months)` + `Age Group` + `Race/Ethnicity` + `Number of Sex Partners (Past 12 Months)`,
      design = chs.dsgn,
      family = quasibinomial()
    )
    summary(fit_adj)

    ## 
    ## Call:
    ## svyglm(formula = `Tested for HIV (Past 12 Months)` ~ `Condom Use (Past 12 Months)` + 
    ##     `Age Group` + `Race/Ethnicity` + `Number of Sex Partners (Past 12 Months)`, 
    ##     design = chs.dsgn, family = quasibinomial())
    ## 
    ## Survey design:
    ## svydesign(ids = ~1, strata = ~strata, weights = ~wt21_dual, data = chs20_recode, 
    ##     nest = TRUE, na.rm = TRUE)
    ## 
    ## Coefficients:
    ##                                              Estimate Std. Error t value
    ## (Intercept)                                 -0.937320   0.178702  -5.245
    ## `Condom Use (Past 12 Months)`Yes             0.099512   0.105003   0.948
    ## `Age Group`25-44                             0.200233   0.166301   1.204
    ## `Age Group`45-64                            -0.397506   0.177955  -2.234
    ## `Age Group`65+                              -1.187429   0.257802  -4.606
    ## `Race/Ethnicity`Black, non-Hispanic          1.437544   0.130397  11.024
    ## `Race/Ethnicity`Hispanic                     1.327000   0.118557  11.193
    ## `Race/Ethnicity`Asian/PI, non-Hispanic       0.229180   0.175734   1.304
    ## `Race/Ethnicity`Other, non-Hispanic          0.729676   0.288983   2.525
    ## `Number of Sex Partners (Past 12 Months)`.L  0.862675   0.112311   7.681
    ## `Number of Sex Partners (Past 12 Months)`.Q -0.008584   0.149061  -0.058
    ##                                             Pr(>|t|)    
    ## (Intercept)                                 1.63e-07 ***
    ## `Condom Use (Past 12 Months)`Yes              0.3433    
    ## `Age Group`25-44                              0.2286    
    ## `Age Group`45-64                              0.0255 *  
    ## `Age Group`65+                              4.21e-06 ***
    ## `Race/Ethnicity`Black, non-Hispanic          < 2e-16 ***
    ## `Race/Ethnicity`Hispanic                     < 2e-16 ***
    ## `Race/Ethnicity`Asian/PI, non-Hispanic        0.1923    
    ## `Race/Ethnicity`Other, non-Hispanic           0.0116 *  
    ## `Number of Sex Partners (Past 12 Months)`.L 1.89e-14 ***
    ## `Number of Sex Partners (Past 12 Months)`.Q   0.9541    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for quasibinomial family taken to be 1.051299)
    ## 
    ## Number of Fisher Scoring iterations: 4

## 5.3 Adjusted LRM Table

    library(broom)
    library(dplyr)
    library(knitr)

    var_labels <- c(
      "Condom Use (Past 12 Months)",
      "Age Group",
      "Race/Ethnicity",
      "Number of Sex Partners (Past 12 Months)"
    )

    fit_adj_tidy <- tidy(fit_adj, exponentiate = TRUE, conf.int = TRUE)

    # Place the improved pretty_term function here
    pretty_term <- function(term) {
      if (term == "(Intercept)") return("Intercept")
      if (term == "Condom Use (Past 12 Months)Yes") return("Condom Use (Past 12 Months): Yes")
      if (grepl("^Age Group", term)) return(paste0("Age Group: ", sub("Age Group", "", term)))
      if (grepl("^Race/Ethnicity", term)) return(paste0("Race/Ethnicity: ", sub("Race/Ethnicity", "", term)))
      if (grepl("Number of Sex Partners (Past 12 Months).L", term)) return("Number of Sex Partners (Past 12 Months): Linear")
      if (grepl("Number of Sex Partners (Past 12 Months).Q", term)) return("Number of Sex Partners (Past 12 Months): Quadratic")
      return(term)
    }

    fit_adj_tidy <- fit_adj_tidy %>%
      mutate(
        term = sapply(term, pretty_term),
        across(where(is.numeric), ~round(., 3)),
        p.value = ifelse(
          p.value < 0.001,
          "0.000",
          sprintf("%.3f", p.value)
        ),
        p.value = ifelse(as.numeric(p.value) < 0.05, paste0(p.value, " *"), p.value)
      )

    fit_adj_tidy %>%
      select(term, estimate, std.error, statistic, p.value, conf.low, conf.high) %>%
      knitr::kable(
        caption = "Adjusted Logistic Regression Results: Odds Ratios and 95% Confidence Intervals",
        col.names = c("Variable", "Odds Ratio", "Std. Error", "z value", "p-value", "CI Lower", "CI Upper"),
        align = "c"
      )

<table>
<caption>Adjusted Logistic Regression Results: Odds Ratios and 95%
Confidence Intervals</caption>
<colgroup>
<col style="width: 42%" />
<col style="width: 11%" />
<col style="width: 11%" />
<col style="width: 8%" />
<col style="width: 8%" />
<col style="width: 9%" />
<col style="width: 9%" />
</colgroup>
<thead>
<tr class="header">
<th style="text-align: center;">Variable</th>
<th style="text-align: center;">Odds Ratio</th>
<th style="text-align: center;">Std. Error</th>
<th style="text-align: center;">z value</th>
<th style="text-align: center;">p-value</th>
<th style="text-align: center;">CI Lower</th>
<th style="text-align: center;">CI Upper</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: center;">Intercept</td>
<td style="text-align: center;">0.392</td>
<td style="text-align: center;">0.179</td>
<td style="text-align: center;">-5.245</td>
<td style="text-align: center;">0.000 *</td>
<td style="text-align: center;">0.276</td>
<td style="text-align: center;">0.556</td>
</tr>
<tr class="even">
<td
style="text-align: center;"><code>Condom Use (Past 12 Months)</code>Yes</td>
<td style="text-align: center;">1.105</td>
<td style="text-align: center;">0.105</td>
<td style="text-align: center;">0.948</td>
<td style="text-align: center;">0.343</td>
<td style="text-align: center;">0.899</td>
<td style="text-align: center;">1.357</td>
</tr>
<tr class="odd">
<td style="text-align: center;"><code>Age Group</code>25-44</td>
<td style="text-align: center;">1.222</td>
<td style="text-align: center;">0.166</td>
<td style="text-align: center;">1.204</td>
<td style="text-align: center;">0.229</td>
<td style="text-align: center;">0.882</td>
<td style="text-align: center;">1.693</td>
</tr>
<tr class="even">
<td style="text-align: center;"><code>Age Group</code>45-64</td>
<td style="text-align: center;">0.672</td>
<td style="text-align: center;">0.178</td>
<td style="text-align: center;">-2.234</td>
<td style="text-align: center;">0.026 *</td>
<td style="text-align: center;">0.474</td>
<td style="text-align: center;">0.953</td>
</tr>
<tr class="odd">
<td style="text-align: center;"><code>Age Group</code>65+</td>
<td style="text-align: center;">0.305</td>
<td style="text-align: center;">0.258</td>
<td style="text-align: center;">-4.606</td>
<td style="text-align: center;">0.000 *</td>
<td style="text-align: center;">0.184</td>
<td style="text-align: center;">0.506</td>
</tr>
<tr class="even">
<td style="text-align: center;"><code>Race/Ethnicity</code>Black,
non-Hispanic</td>
<td style="text-align: center;">4.210</td>
<td style="text-align: center;">0.130</td>
<td style="text-align: center;">11.024</td>
<td style="text-align: center;">0.000 *</td>
<td style="text-align: center;">3.261</td>
<td style="text-align: center;">5.437</td>
</tr>
<tr class="odd">
<td style="text-align: center;"><code>Race/Ethnicity</code>Hispanic</td>
<td style="text-align: center;">3.770</td>
<td style="text-align: center;">0.119</td>
<td style="text-align: center;">11.193</td>
<td style="text-align: center;">0.000 *</td>
<td style="text-align: center;">2.988</td>
<td style="text-align: center;">4.756</td>
</tr>
<tr class="even">
<td style="text-align: center;"><code>Race/Ethnicity</code>Asian/PI,
non-Hispanic</td>
<td style="text-align: center;">1.258</td>
<td style="text-align: center;">0.176</td>
<td style="text-align: center;">1.304</td>
<td style="text-align: center;">0.192</td>
<td style="text-align: center;">0.891</td>
<td style="text-align: center;">1.775</td>
</tr>
<tr class="odd">
<td style="text-align: center;"><code>Race/Ethnicity</code>Other,
non-Hispanic</td>
<td style="text-align: center;">2.074</td>
<td style="text-align: center;">0.289</td>
<td style="text-align: center;">2.525</td>
<td style="text-align: center;">0.012 *</td>
<td style="text-align: center;">1.177</td>
<td style="text-align: center;">3.655</td>
</tr>
<tr class="even">
<td
style="text-align: center;"><code>Number of Sex Partners (Past 12 Months)</code>.L</td>
<td style="text-align: center;">2.369</td>
<td style="text-align: center;">0.112</td>
<td style="text-align: center;">7.681</td>
<td style="text-align: center;">0.000 *</td>
<td style="text-align: center;">1.901</td>
<td style="text-align: center;">2.953</td>
</tr>
<tr class="odd">
<td
style="text-align: center;"><code>Number of Sex Partners (Past 12 Months)</code>.Q</td>
<td style="text-align: center;">0.991</td>
<td style="text-align: center;">0.149</td>
<td style="text-align: center;">-0.058</td>
<td style="text-align: center;">0.954</td>
<td style="text-align: center;">0.740</td>
<td style="text-align: center;">1.328</td>
</tr>
</tbody>
</table>

Adjusted Logistic Regression Results: Odds Ratios and 95% Confidence
Intervals

# 6. Discussion

**LRM:** Without accounting for hypothesized confounders (age, race, \#
of partners) the association of HIV testing and condom usage is
significant. In relation to adjusting for the hypothesized confounders
we can see that participants who used condoms the last time the had sex
compared to those who didn’t had a 0.09951 increase in the log odds of
having HIV testing with in the last 12 months. However, the positive
association is not significant as the p-value is 0.3433.

# 7. Conclusion

Participants are more likely to have HIV testing done if they identify
as: Black- non-Hispanic, Hispanic, or Other-non-Hispanic, compared to
the other ethno-racial groups. Survey participants were less likely to
HIV testing done if they were aged: 45-64, 65+ as opposed to the other
age groups. Furthermore, there is a positive linear relationship between
the number of sex partners and HIV testing meaning the more sex partners
someone has the more likely a participant is to have HIV testing done.

# 8. Refrences and Resources

**Disclaimer:** This project was completed as a part of my
Bio-statistics II course with Dr. Levi Waldron. This was submitted under
Assignment 1 for the class however, this is an updated and completely
reformed version of the assignment with the visual models and commentary
being an addition. Materials and set up for this project are entirely
organized by Dr. Waldron who is a Professor at City University of New
York School of Public Health and Health Policy, and the Department
Chairperson of Epidemiology and Bio-statistics.

**Libraries Used**

-   Wickham, H. & Miller, E. (2024). haven: Import and Export ‘SPSS’,
    ‘Stata’ and ‘SAS’ Files. R package version 2.5.4.
    <https://CRAN.R-project.org/package=haven>

-   Wickham, H., François, R., Henry, L., & Müller, K. (2023). dplyr: A
    Grammar of Data Manipulation. R package version 1.1.4.
    <https://CRAN.R-project.org/package=dplyr>

-   Lumley, T. (2024). survey: analysis of complex survey samples. R
    package version 4.3-2. <https://CRAN.R-project.org/package=survey>

-   Harrell, F.E. Jr. (2024). Hmisc: Harrell Miscellaneous. R package
    version 5.1-1. <https://CRAN.R-project.org/package=Hmisc>

-   Rich, B. (2023). table1: Tables of Descriptive Statistics in HTML. R
    package version 1.4.4. <https://CRAN.R-project.org/package=table1>

-   Wickham, H. (2016). ggplot2: Elegant Graphics for Data Analysis.
    Springer-Verlag New York. <https://ggplot2.tidyverse.org/>

-   Wickham, H. (2024). forcats: Tools for Working with Categorical
    Variables (Factors). R package version 1.0.1.
    <https://CRAN.R-project.org/package=forcats>

-   Robinson, D., Hayes, A., & Couch, S. (2024). broom: Convert
    Statistical Objects into Tidy Tibbles. R package version 1.0.5.
    <https://CRAN.R-project.org/package=broom>

-   Xie, Y. (2024). knitr: A General-Purpose Package for Dynamic Report
    Generation in R. R package version 1.46. <https://yihui.org/knitr/>

-   Zhu, H. (2024). kableExtra: Construct Complex Table with ‘kable’ and
    Pipe Syntax. R package version 1.4.0.
    <https://CRAN.R-project.org/package=kableExtra>

-   Garnier, S. (2024). viridis: Colorblind-Friendly Color Maps for R. R
    package version 0.6.5. <https://CRAN.R-project.org/package=viridis>
