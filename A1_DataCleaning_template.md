Assignment 1, Language development in Autism Spectrum Disorder (ASD) - Brushing up your code skills
===================================================================================================

In this first part of the assignment we will brush up your programming skills, and make you familiar with the data sets you will be analysing for the next parts of the assignment.

In this warm-up assignment you will: 1) Create a Github (or gitlab) account, link it to your RStudio, and create a new repository/project 2) Use small nifty lines of code to transform several data sets into just one. The final data set will contain only the variables that are needed for the analysis in the next parts of the assignment 3) Warm up your tidyverse skills (especially the sub-packages stringr and dplyr), which you will find handy for later assignments.

N.B: Usually you'll also have to doc/pdf with a text. Not for Assignment 1.

Learning objectives:
--------------------

-   Become comfortable with tidyverse (and R in general)
-   Test out the git integration with RStudio
-   Build expertise in data wrangling (which will be used in future assignments)

0. First an introduction on the data
------------------------------------

Language development in Autism Spectrum Disorder (ASD)
======================================================

Reference to the study: <https://www.ncbi.nlm.nih.gov/pubmed/30396129>

Background: Autism Spectrum Disorder (ASD) is often related to language impairment, and language impairment strongly affects the patients ability to function socially (maintaining a social network, thriving at work, etc.). It is therefore crucial to understand how language abilities develop in children with ASD, and which factors affect them (to figure out e.g. how a child will develop in the future and whether there is a need for language therapy). However, language impairment is always quantified by relying on the parent, teacher or clinician subjective judgment of the child, and measured very sparcely (e.g. at 3 years of age and again at 6).

In this study we videotaped circa 30 kids with ASD and circa 30 comparison kids (matched by linguistic performance at visit 1) for ca. 30 minutes of naturalistic interactions with a parent. We repeated the data collection 6 times per kid, with 4 months between each visit. We transcribed the data and counted: i) the amount of words that each kid uses in each video. Same for the parent. ii) the amount of unique words that each kid uses in each video. Same for the parent. iii) the amount of morphemes per utterance (Mean Length of Utterance) displayed by each child in each video. Same for the parent.

Different researchers involved in the project provide you with different datasets: 1) demographic and clinical data about the children (recorded by a clinical psychologist) 2) length of utterance data (calculated by a linguist) 3) amount of unique and total words used (calculated by a fumbling jack-of-all-trade, let's call him RF)

Your job in this assignment is to double check the data and make sure that it is ready for the analysis proper (Assignment 2), in which we will try to understand how the children's language develops as they grow as a function of cognitive and social factors and which are the "cues" suggesting a likely future language impairment.

1. Let's get started on GitHub
------------------------------

In the assignments you will be asked to upload your code on Github and the GitHub repositories will be part of the portfolio, therefore all students must make an account and link it to their RStudio (you'll thank us later for this!).

Follow the link to one of the tutorials indicated in the syllabus: \* Recommended: <https://happygitwithr.com/> \* Alternative (if the previous doesn't work): <https://support.rstudio.com/hc/en-us/articles/200532077-Version-Control-with-Git-and-SVN> \* Alternative (if the previous doesn't work): <https://docs.google.com/document/d/1WvApy4ayQcZaLRpD6bvAqhWncUaPmmRimT016-PrLBk/mobilebasic>

N.B. Create a GitHub repository for the Assignment 1 and link it to a project on your RStudio.

2. Now let's take dirty dirty data sets and make them into a tidy one
---------------------------------------------------------------------

If you're not in a project in Rstudio, make sure to set your working directory here. If you created an RStudio project, then your working directory (the directory with your data and code for these assignments) is the project directory.

``` r
pacman::p_load(tidyverse,janitor, dplyr)
```

Load the three data sets, after downloading them from dropbox and saving them in your working directory: \* Demographic data for the participants: <https://www.dropbox.com/s/w15pou9wstgc8fe/demo_train.csv?dl=0> \* Length of utterance data: <https://www.dropbox.com/s/usyauqm37a76of6/LU_train.csv?dl=0> \* Word data: <https://www.dropbox.com/s/8ng1civpl2aux58/token_train.csv?dl=0>

``` r
#load data
demo <- read.csv("demo_train.csv")
lu <- read.csv("LU_train.csv")
token <- read.csv("token_train.csv")
```

Explore the 3 datasets (e.g. visualize them, summarize them, etc.). You will see that the data is messy, since the psychologist collected the demographic data, the linguist analyzed the length of utterance in May 2014 and the fumbling jack-of-all-trades analyzed the words several months later. In particular: - the same variables might have different names (e.g. participant and visit identifiers) - the same variables might report the values in different ways (e.g. participant and visit IDs) Welcome to real world of messy data :-)

Before being able to combine the data sets we need to make sure the relevant variables have the same names and the same kind of values.

So:

2a. Identify which variable names do not match (that is are spelled differently) and find a way to transform variable names. Pay particular attention to the variables indicating participant and visit.

Tip: look through the chapter on data transformation in R for data science (<http://r4ds.had.co.nz>). Alternatively you can look into the package dplyr (part of tidyverse), or google "how to rename variables in R". Or check the janitor R package. There are always multiple ways of solving any problem and no absolute best method.

``` r
names(demo)[1] <- "SUBJ"
names(demo)[2] <- "VISIT"
```

2b. Find a way to homogeneize the way "visit" is reported (visit1 vs. 1).

Tip: The stringr package is what you need. str\_extract () will allow you to extract only the digit (number) from a string, by using the regular expression \\d.

``` r
lu$VISIT <- str_extract(lu$VISIT, "\\d")
token$VISIT <- str_extract(token$VISIT, "\\d")
```

2c. We also need to make a small adjustment to the content of the Child.ID coloumn in the demographic data. Within this column, names that are not abbreviations do not end with "." (i.e. Adam), which is the case in the other two data sets (i.e. Adam.). If The content of the two variables isn't identical the rows will not be merged. A neat way to solve the problem is simply to remove all "." in all datasets.

Tip: stringr is helpful again. Look up str\_replace\_all Tip: You can either have one line of code for each child name that is to be changed (easier, more typing) or specify the pattern that you want to match (more complicated: look up "regular expressions", but less typing)

``` r
lu$SUBJ <- str_replace_all(lu$SUBJ, "[.]", "")
token$SUBJ <- str_replace_all(token$SUBJ, "[.]", "")
demo$SUBJ <- str_replace_all(demo$SUBJ, "[.]", "")
```

2d. Now that the nitty gritty details of the different data sets are fixed, we want to make a subset of each data set only containig the variables that we wish to use in the final data set. For this we use the tidyverse package dplyr, which contains the function select().

The variables we need are: \* Child.ID, \* Visit, \* Diagnosis, \* Ethnicity, \* Gender, \* Age, \* ADOS,
\* MullenRaw, \* ExpressiveLangRaw, \* Socialization \* MOT\_MLU, \* CHI\_MLU, \* types\_MOT, \* types\_CHI, \* tokens\_MOT, \* tokens\_CHI.

Most variables should make sense, here the less intuitive ones. \* ADOS (Autism Diagnostic Observation Schedule) indicates the severity of the autistic symptoms (the higher the score, the worse the symptoms). Ref: <https://link.springer.com/article/10.1023/A:1005592401947> \* MLU stands for mean length of utterance (usually a proxy for syntactic complexity) \* types stands for unique words (e.g. even if "doggie" is used 100 times it only counts for 1) \* tokens stands for overall amount of words (if "doggie" is used 100 times it counts for 100) \* MullenRaw indicates non verbal IQ, as measured by Mullen Scales of Early Learning (MSEL <https://link.springer.com/referenceworkentry/10.1007%2F978-1-4419-1698-3_596>) \* ExpressiveLangRaw indicates verbal IQ, as measured by MSEL \* Socialization indicates social interaction skills and social responsiveness, as measured by Vineland (<https://cloudfront.ualberta.ca/-/media/ualberta/faculties-and-programs/centres-institutes/community-university-partnership/resources/tools---assessment/vinelandjune-2012.pdf>)

Feel free to rename the variables into something you can remember (i.e. nonVerbalIQ, verbalIQ)

``` r
sub_lu <- select(lu, SUBJ, VISIT, MOT_MLU, CHI_MLU)
sub_token <- select(token, SUBJ, VISIT, types_MOT, types_CHI, tokens_MOT, tokens_CHI)
sub_demo <- select(demo, SUBJ, VISIT, Diagnosis, Ethnicity, Gender, Age, ADOS, MullenRaw, ExpressiveLangRaw, Socialization)
names(sub_demo)[8] <- "nonVerbalIQ"
names(sub_demo)[9] <- "verbalIQ"
```

2e. Finally we are ready to merge all the data sets into just one.

Some things to pay attention to: \* make sure to check that the merge has included all relevant data (e.g. by comparing the number of rows) \* make sure to understand whether (and if so why) there are NAs in the dataset (e.g. some measures were not taken at all visits, some recordings were lost or permission to use was withdrawn)

``` r
df1 <- merge(sub_lu, sub_token)
df <- merge(df1, sub_demo)
```

2f. Only using clinical measures from Visit 1 In order for our models to be useful, we want to miimize the need to actually test children as they develop. In other words, we would like to be able to understand and predict the children's linguistic development after only having tested them once. Therefore we need to make sure that our ADOS, MullenRaw, ExpressiveLangRaw and Socialization variables are reporting (for all visits) only the scores from visit 1.

A possible way to do so: \* create a new dataset with only visit 1, child id and the 4 relevant clinical variables to be merged with the old dataset \* rename the clinical variables (e.g. ADOS to ADOS1) and remove the visit (so that the new clinical variables are reported for all 6 visits) \* merge the new dataset with the old

``` r
clinical1 <- df  %>% 
  filter(VISIT == "1") %>% 
  select(SUBJ, ADOS, nonVerbalIQ, verbalIQ, Socialization)

names(clinical1) <- c("SUBJ", "ADOS1", "nonVerbalIQ1", "verbalIQ1", "Socialisation1")

df_merged <- merge(df, clinical1)
```

2g. Final touches

Now we want to \* anonymize our participants (they are real children!). \* make sure the variables have sensible values. E.g. right now gender is marked 1 and 2, but in two weeks you will not be able to remember, which gender were connected to which number, so change the values from 1 and 2 to F and M in the gender variable. For the same reason, you should also change the values of Diagnosis from A and B to ASD (autism spectrum disorder) and TD (typically developing). Tip: Try taking a look at ifelse(), or google "how to rename levels in R". \* Save the data set using into a csv file. Hint: look into write.csv()

``` r
#anonymisation
df_merged$SUBJ <- as.factor(df_merged$SUBJ)
levels(df_merged$SUBJ) <- c(1:61)

#change gender
df_merged$Gender <- as.factor(df$Gender)
levels(df_merged$Gender)[levels(df_merged$Gender)=="1"] <- "M"
levels(df_merged$Gender)[levels(df_merged$Gender)=="2"] <- "F"

#change Diagnosis
df_merged$Diagnosis <- as.factor(df_merged$Diagnosis)
levels(df_merged$Diagnosis)[levels(df_merged$Diagnosis)=="A"] <- "ASD"
levels(df_merged$Diagnosis)[levels(df_merged$Diagnosis)=="B"] <- "TD"

#write csv 
?write.csv()
write.csv(df_merged, file = "assignment1df.csv")
```

1.  BONUS QUESTIONS The aim of this last section is to make sure you are fully fluent in the tidyverse. Here's the link to a very helpful book, which explains each function: <http://r4ds.had.co.nz/index.html>

2.  USING FILTER List all kids who:

<!-- -->

1.  have a mean length of utterance (across all visits) of more than 2.7 morphemes.
2.  have a mean length of utterance of less than 1.5 morphemes at the first visit
3.  have not completed all trials. Tip: Use pipes to solve this

``` r
#1:
filter1 <- df_merged %>% group_by(SUBJ) %>% 
  summarize(mean(CHI_MLU)) %>% filter(`mean(CHI_MLU)` > 2.7)

#2:
filter2 <- filter(df_merged, VISIT == 1 & CHI_MLU < 1.5)

#3: what do you mean by trials? 
```

USING ARRANGE

1.  Sort kids to find the kid who produced the most words on the 6th visit
2.  Sort kids to find the kid who produced the least amount of words on the 1st visit.

``` r
#1:
arrange(df_merged, desc(VISIT), desc(tokens_CHI))
```

    ##     SUBJ VISIT  MOT_MLU   CHI_MLU types_MOT types_CHI tokens_MOT
    ## 1     55     6 3.957230 2.9092742       367       260       1731
    ## 2     26     6 4.111413 3.3643411       452       273       3076
    ## 3     18     6 4.013353 2.9095355       516       235       2576
    ## 4     25     6 4.472993 3.0614525       555       237       2895
    ## 5     43     6 4.250853 2.6794872       374       219       2227
    ## 6     37     6 4.240798 3.0774194       429       217       2510
    ## 7     13     6 4.847418 3.7011952       388       210       1959
    ## 8     22     6 4.211321 3.0911950       478       221       2044
    ## 9     14     6 4.664286 3.8114035       359       178       1802
    ## 10     4     6 3.532374 3.2781955       410       166       2171
    ## 11    42     6 3.391525 3.0727969       357       219       1764
    ## 12    60     6 4.080446 3.4415584       505       226       3072
    ## 13    16     6 4.445872 2.9482072       491       250       2264
    ## 14    41     6 4.061475 2.8526316       397       213       1741
    ## 15     2     6 4.588477 3.4135021       304       245        999
    ## 16     1     6 4.664013 2.8651685       595       210       2586
    ## 17    11     6 4.287582 2.7571429       260       168       1138
    ## 18    59     6 4.113158 2.8480000       303       158       1460
    ## 19    49     6 5.247093 3.5950000       383       217       1701
    ## 20    12     6 4.468493 3.5049505       339       201       1498
    ## 21    58     6 3.320000 2.1553398       335       197       2221
    ## 22    35     6 4.239198 2.3815789       411       156       2438
    ## 23    34     6 3.370937 2.2745098       342       157       1540
    ## 24    20     6 5.379798 2.9027778       433       155       2389
    ## 25    40     6 3.926928 2.1586207       417       170       2508
    ## 26    36     6 4.388235 2.5614754       324       165       1978
    ## 27     5     6 4.587179 2.7665198       462       179       3182
    ## 28    27     6 4.254505 2.4800000       385       154       1788
    ## 29    28     6 4.239437 2.9607843       391       164       1889
    ## 30    54     6 4.186161 2.8921569       437       183       2347
    ## 31     3     6 5.229885 3.7103448       486       173       2564
    ## 32    52     6 5.587332 2.4292929       548       163       2887
    ## 33    31     6 3.603473 2.0717489       475       185       2363
    ## 34    15     6 4.347709 2.8690476       327       156       1395
    ## 35    51     6 5.153639 2.7612903       383       140       1866
    ## 36    53     6 3.706790 3.2439024       249       102       1024
    ## 37    48     6 3.370093 1.4734513       319        98       1565
    ## 38     6     6 2.483146 1.4967320       158        66        536
    ## 39    19     6 3.156695 2.1450382       256        55        995
    ## 40    23     6 3.534173 1.3052632       372        47       1748
    ## 41    61     6 3.514403 1.4012346       311        58       1481
    ## 42    29     6 4.349353 1.2883436       454        52       2391
    ## 43     8     6 4.366972 2.2258065       322        64       1352
    ## 44    30     6 4.341549 1.0843373       425       101       2219
    ## 45    10     6 4.235585 2.7051282       400        73       2271
    ## 46    24     6 3.860795 1.1680000       309        62       1349
    ## 47    46     6 3.460548 1.1610169       351        10       1918
    ## 48    39     6 4.100707 1.6447368       330        55       1987
    ## 49    32     6 4.003257 2.6315789       585        12       2202
    ## 50    33     6 3.965392 0.7536232       326        14       1852
    ## 51    57     6 3.241422 0.0156250       444         2       2450
    ## 52    47     6 4.676101 2.7600000       322        34       1371
    ## 53    17     6 3.650235 1.0000000       281         3       1454
    ## 54    56     6 3.474725 1.0588235       284         6       1390
    ## 55    50     6 3.886842 1.3333333       370         4       1396
    ## 56    21     6 3.943636 0.5000000       388         2       2077
    ## 57    26     5 4.345515 3.2919897       437       261       2410
    ## 58    25     5 4.267409 2.7413793       578       298       2940
    ## 59    43     5 4.232258 2.8665049       396       262       2326
    ## 60    13     5 4.487842 3.2301136       511       247       2668
    ## 61    12     5 4.910494 4.3647541       290       222       1467
    ## 62    42     5 3.921444 3.7749077       358       213       1600
    ## 63    37     5 4.658802 2.7468750       407       228       2314
    ## 64    53     5 4.857143 3.8225806       278       217       1067
    ## 65    38     5 4.917927 3.5185185       447       210       1999
    ## 66    20     5 4.837884 2.2506739       506       219       2504
    ## 67    18     5 3.877102 3.0902778       575       219       2881
    ## 68    14     5 4.446281 3.3750000       390       238       1864
    ## 69    35     5 4.069659 2.5541796       436       161       2279
    ## 70    54     5 3.750000 3.6072874       394       244       1977
    ## 71    52     5 4.983389 2.8321678       563       219       2772
    ## 72    31     5 3.906856 2.1420613       491       211       2729
    ## 73    22     5 4.746606 3.7000000       436       178       1965
    ## 74    41     5 3.969147 2.2892308       415       187       1936
    ## 75    59     5 4.424837 2.6038462       341       132       1916
    ## 76    28     5 3.708934 3.5852535       272       177       1098
    ## 77     3     5 4.566038 3.2985782       534       207       2672
    ## 78    49     5 5.743772 3.5371429       425       209       1525
    ## 79    16     5 5.433579 3.1095238       517       219       2762
    ## 80     4     5 4.602851 4.0434783       397       146       2082
    ## 81    34     5 3.419664 2.5758929       298       120       1223
    ## 82    58     5 3.290683 1.7591463       360       158       2276
    ## 83    11     5 4.543210 2.5482456       276       148       1300
    ## 84    61     5 2.965928 1.6688963       320       122       1527
    ## 85    45     5 4.329810 2.2558140       418       142       1791
    ## 86     1     5 5.209615 3.2380952       601       182       2553
    ## 87    36     5 4.137405 2.2787611       366       145       1979
    ## 88    44     5 4.113846 3.2000000       307       131       1207
    ## 89    10     5 3.867601 2.2835821       359        97       2208
    ## 90     5     5 4.292135 2.4232804       418       144       2749
    ## 91    60     5 4.223572 2.9655172       521       145       3090
    ## 92     6     5 1.948454 1.6363636        92        80        209
    ## 93     9     5 4.384946 2.8358209       428       121       1879
    ## 94    39     5 2.902089 1.6511628       321        90       1951
    ## 95     8     5 4.564103 1.9408602       387        40       2345
    ## 96    27     5 3.479393 2.1791908       327       114       1385
    ## 97    47     5 3.673418 2.7388060       315       131       1224
    ## 98    48     5 3.162554 1.4102564       369        79       1871
    ## 99    51     5 5.185941 2.9215686       388       108       1986
    ## 100   19     5 4.157191 1.5840000       328        45       1142
    ## 101   23     5 2.822259 1.4133333       306        50       1518
    ## 102    7     5 3.992095 1.5299145       234        73        891
    ## 103   30     5 3.859624 0.4130435       465        42       2360
    ## 104   29     5 4.872014 0.2553191       415         9       2560
    ## 105   15     5 4.165414 2.4691358       356        69       1426
    ## 106   33     5 4.185417 0.6756757       383         9       1917
    ## 107   24     5 4.421222 1.0000000       303         4       1278
    ## 108   32     5 3.414894 1.0000000       355         3       1388
    ## 109   46     5 3.637965 1.0000000       344         1       1609
    ## 110   57     5 3.693846 0.2000000       435         3       2070
    ## 111   17     5 3.996071 1.0000000       475         6       2088
    ## 112   21     5 4.019231 0.2000000       373         2       1963
    ## 113   50     5 3.864407 0.0000000       364         0       1447
    ## 114   25     4 5.362500 3.7310924       553       291       2978
    ## 115   18     4 4.131579 3.2128205       565       207       2965
    ## 116   55     4 4.177340 2.8620690       318       205       1524
    ## 117   53     4 4.458937 3.6340694       232       217        798
    ## 118   47     4 4.190698 3.1622419       343       211       1559
    ## 119   43     4 3.905213 2.6837838       390       243       2158
    ## 120   42     4 4.611247 3.8300395       339       193       1726
    ## 121   16     4 4.251605 2.3225806       508       231       3077
    ## 122    3     4 5.301053 3.9292035       449       206       2397
    ## 123   31     4 3.667638 2.7272727       480       186       2233
    ## 124   20     4 4.560201 2.2485380       422       153       2485
    ## 125   26     4 4.857143 3.5238095       398       188       2518
    ## 126   54     4 3.819961 3.4065041       332       153       1699
    ## 127   60     4 4.085409 3.3598326       539       214       3163
    ## 128   49     4 4.880240 3.4809524       340       229       1452
    ## 129    2     4 5.309804 4.3023256       335       200       1286
    ## 130   52     4 5.338462 3.0775510       554       177       2585
    ## 131   59     4 4.697624 2.7148289       330       157       1974
    ## 132    5     4 4.744000 2.9072581       384       187       2685
    ## 133   22     4 3.991667 2.3740741       480       169       2256
    ## 134   38     4 4.169162 2.6475410       513       195       2482
    ## 135   58     4 3.471510 2.0127796       359       144       2109
    ## 136   14     4 3.801292 2.3178295       369       158       1984
    ## 137   13     4 4.083700 2.7982833       463       203       2361
    ## 138   37     4 5.050000 2.5159817       410       148       2252
    ## 139   28     4 3.683406 2.3500000       309       139       1470
    ## 140   15     4 4.584071 2.5613208       357       161       1517
    ## 141    7     4 2.736842 1.9115646       198       126        814
    ## 142    8     4 4.658333 3.0265957       375       134       2069
    ## 143   40     4 3.812930 2.9900000       430       180       2377
    ## 144    4     4 4.301624 3.2571429       356       163       1711
    ## 145   36     4 3.705441 2.2594142       295       136       1715
    ## 146   19     4 2.762463 1.6179402       248        87        855
    ## 147   34     4 3.997126 2.1962617       321       107       1224
    ## 148   41     4 4.118790 2.2325581       366       151       1689
    ## 149   45     4 4.019928 1.7600000       424       108       1909
    ## 150   10     4 3.790896 1.6479401       411        96       2345
    ## 151    9     4 4.262195 2.7756410       400       121       1934
    ## 152   27     4 3.973469 1.7405858       364       114       1754
    ## 153   44     4 4.256790 2.5370370       302       122       1556
    ## 154   11     4 4.732733 2.1348315       272        88       1387
    ## 155   51     4 4.137014 3.0289855       363       112       1829
    ## 156   61     4 3.312388 1.3204225       348        61       1644
    ## 157    1     4 4.415330 2.2515723       533       133       2260
    ## 158   12     4 3.988304 3.0000000       197       122        632
    ## 159    6     4 2.035714 1.7142857       125        50        360
    ## 160   23     4 3.012594 1.0974026       270        21       1054
    ## 161   24     4 3.814815 1.2032520       353        29       1482
    ## 162   33     4 3.110932 1.2100840       281        18       1726
    ## 163   39     4 3.714556 1.7000000       302        36       1675
    ## 164   21     4 3.600624 0.2727273       427        10       2283
    ## 165   30     4 3.961832 1.1250000       409        26       1863
    ## 166   35     4 4.001613 1.9743590       361        29       2145
    ## 167   50     4 4.009934 1.0000000       358         1       1145
    ## 168   46     4 2.377171 1.1200000       203        11        861
    ## 169   48     4 3.339114 1.0000000       280         1       1482
    ## 170   29     4 3.885113 0.8947368       361        13       2214
    ## 171   32     4 3.743333 1.0000000       352         6       2054
    ## 172   57     4 3.536415 0.0000000       400         1       2196
    ## 173   26     3 4.316279 3.9196891       333       307       1668
    ## 174   31     3 3.552459 2.9875260       396       233       1872
    ## 175   43     3 3.573574 2.3053892       331       221       2141
    ## 176    2     3 4.147059 3.1193182       351       262       1445
    ## 177   18     3 4.116057 3.3400000       487       201       2479
    ## 178   52     3 5.288991 3.3037543       474       200       2142
    ## 179   25     3 4.127941 2.8042169       455       217       2589
    ## 180   16     3 4.164789 2.8076923       408       200       2675
    ## 181   38     3 4.577491 2.8690476       420       175       2146
    ## 182   12     3 4.974684 3.1857708       318       169       1463
    ## 183   35     3 3.675134 2.5710145       366       155       2328
    ## 184   60     3 4.907591 4.1318681       459       196       2841
    ## 185   37     3 3.642412 1.8812665       330       175       1570
    ## 186   55     3 4.231362 2.4932432       292       168       1470
    ## 187   49     3 4.737127 3.1124498       323       151       1609
    ## 188   53     3 4.078292 3.1313559       219       152       1020
    ## 189   13     3 3.871968 2.0798817       435       139       2571
    ## 190   42     3 4.057047 2.4911032       367       155       2124
    ## 191    3     3 5.134892 2.6916300       482       164       2630
    ## 192   44     3 4.106212 2.2313433       300       102       1740
    ## 193   20     3 4.063518 1.6109325       361       132       2092
    ## 194    4     3 3.818681 3.5180723       388       165       1788
    ## 195   47     3 3.960265 2.2893617       364       149       1568
    ## 196    5     3 4.198973 2.5324074       412       159       2939
    ## 197    9     3 4.592593 1.8320312       369       129       1991
    ## 198   59     3 4.210407 2.1157895       257       101       1610
    ## 199   40     3 3.723664 2.0704225       346       134       2148
    ## 200   28     3 3.210084 1.8369099       233       131       1006
    ## 201   58     3 3.774030 1.8000000       346       153       1891
    ## 202   14     3 3.848175 1.1855072       378        92       2328
    ## 203   34     3 3.866834 1.8056872       318       100       1350
    ## 204   45     3 3.395899 1.1722689       370        95       1912
    ## 205   41     3 4.050505 1.9086294       304       127       1426
    ## 206   19     3 3.615176 1.9052632       298        88       1159
    ## 207   22     3 3.094880 1.4549550       432       112       1811
    ## 208   27     3 3.846626 1.4976077       318        89       1706
    ## 209   54     3 4.298667 2.3309353       299        85       1418
    ## 210    1     3 4.321881 1.5568862       455        97       2149
    ## 211   61     3 4.520325 1.5828571       313        70       1618
    ## 212   15     3 4.182979 1.4408602       409        92       1743
    ## 213   39     3 3.607664 1.8880000       327        51       1709
    ## 214   36     3 4.595568 1.7500000       323        76       1504
    ## 215    7     3 2.633929 1.3974359       196        50        768
    ## 216   11     3 3.940711 1.5234375       227        56        878
    ## 217   10     3 4.298942 1.5520000       409        55       2936
    ## 218   30     3 4.015699 1.0066667       430        21       2278
    ## 219   23     3 3.215859 1.0863309       259        29       1332
    ## 220   51     3 4.022624 1.5529412       333        56       1562
    ## 221   21     3 3.646778 2.0000000       434        13       2764
    ## 222   50     3 3.979950 1.1666667       368        27       1423
    ## 223    6     3 1.856115 1.1698113        74        12        250
    ## 224   17     3 2.919169 1.0000000       252         2       1145
    ## 225   57     3 4.145588 0.1621622       542         3       2717
    ## 226   24     3 3.910448 1.0000000       344         1       1393
    ## 227   29     3 4.382550 0.6842105       345        13       2417
    ## 228   33     3 3.404959 1.0833333       272         3       1504
    ## 229   56     3 2.963303 1.7500000       252         9       1393
    ## 230   46     3 2.821782 1.0000000       245         2       1018
    ## 231   32     3 3.990826 1.0000000       323         1       1590
    ## 232   35     2 3.636519 1.9342916       300       128       1784
    ## 233   18     2 4.357911 2.7220339       461       160       2687
    ## 234   16     2 4.750455 2.7441077       391       196       2303
    ## 235   60     2 4.604341 2.7840000       413       149       2534
    ## 236   25     2 4.600000 2.4847328       419       145       2562
    ## 237   47     2 4.183445 2.2481481       295       128       1653
    ## 238    2     2 4.964664 3.4530387       307       171       1270
    ## 239    4     2 4.130337 2.5630252       343       170       1657
    ## 240    8     2 3.967213 1.6019108       305       122       2099
    ## 241   31     2 4.222034 2.2481481       405       141       2219
    ## 242   43     2 3.505582 1.9832215       309       170       2031
    ## 243    3     2 4.572086 2.6418605       464       149       2694
    ## 244    5     2 4.886646 2.5743590       441       152       2955
    ## 245   53     2 4.557471 3.2179487       197       126        686
    ## 246   38     2 4.073282 2.1727273       435       142       2290
    ## 247   37     2 4.054054 1.3432836       416       100       2340
    ## 248   42     2 4.100167 2.0046729       368       124       2201
    ## 249   22     2 3.978599 1.7804878       392       121       1843
    ## 250   59     2 4.001724 1.7709251       334       120       1959
    ## 251   13     2 3.817109 1.4761905       411       119       2260
    ## 252   34     2 2.667722 1.9481132       220       100        748
    ## 253   20     2 2.841004 1.0800000       288        73       1822
    ## 254   49     2 4.745238 2.4967742       408       126       1867
    ## 255   61     2 3.302663 1.0990991       250        48       1205
    ## 256   19     2 3.170000 1.4867257       315        86       1316
    ## 257   52     2 4.567329 1.7359551       415       122       1845
    ## 258   14     2 3.239832 1.0744681       346        65       1982
    ## 259   54     2 3.609881 2.0466667       352        88       1905
    ## 260   41     2 3.751332 1.1974249       404        97       1847
    ## 261   58     2 3.372320 1.4432990       260        78       1511
    ## 262   28     2 3.367292 1.6557377       255        71       1139
    ## 263   55     2 3.789634 1.7902098       219        70       1058
    ## 264   27     2 3.869654 1.4228571       287        72       1663
    ## 265   12     2 3.444664 1.3647799       297        72       1551
    ## 266   29     2 4.003072 1.0114286       331        18       2448
    ## 267    7     2 2.417344 1.3211679       186        42        755
    ## 268   45     2 3.171717 0.9051724       288        41       1375
    ## 269   15     2 4.224839 1.0460526       363        25       1819
    ## 270   23     2 2.903723 1.4273504       301        61       1990
    ## 271    6     2 1.995885 0.7606838       140        29        464
    ## 272    1     2 3.857367 1.0136054       403        18       2160
    ## 273   40     2 2.900838 1.2195122       295        52       1882
    ## 274   50     2 3.563356 1.3628319       381        20       1851
    ## 275    9     2 4.079060 1.6162791       369        45       1702
    ## 276   39     2 2.760976 1.2571429       277        55       1481
    ## 277   44     2 3.445545 1.3043478       271        52       1515
    ## 278   11     2 2.821918 1.0630631       185        50        703
    ## 279   30     2 3.843411 1.0265487       403        15       2211
    ## 280   36     2 3.942164 1.3103448       321        44       1843
    ## 281   51     2 4.073350 1.1976744       309        20       1483
    ## 282   17     2 2.947230 1.0125000       243         7        999
    ## 283   21     2 3.381510 1.0400000       426         9       2524
    ## 284   10     2 4.378840 1.1200000       358        18       2263
    ## 285   57     2 3.688478 1.0000000       389         1       2274
    ## 286   46     2 3.830795 1.2258065       392         6       1973
    ## 287   24     2 3.815141 1.0000000       434         6       1995
    ## 288   56     2 3.430642 1.0000000       285         1       1417
    ## 289   33     2 2.879925 1.0000000       285         2       1393
    ## 290   32     2 4.142562 1.0000000       454         2       1856
    ## 291   48     2 3.555804 1.0000000       272         2       1344
    ## 292   26     1 4.690751 3.4000000       278       119       1450
    ## 293   53     1 4.159159 1.9397163       236       120       1170
    ## 294   60     1 3.604140 2.8763441       400       149       2587
    ## 295    2     1 4.098446 2.2768595       317       146       1428
    ## 296   43     1 3.341418 1.9144981       271       101       1591
    ## 297   25     1 4.643200 1.5827338       373        71       2740
    ## 298   18     1 3.587855 1.7948718       467       108       2555
    ## 299   38     1 3.420975 1.3322785       342        96       2035
    ## 300   37     1 4.135036 0.4805825       381        39       1988
    ## 301   31     1 3.494327 1.5333333       322        91       1870
    ## 302   52     1 5.344227 1.4086957       441        92       2267
    ## 303   49     1 4.030075 1.4258373       255        73       1452
    ## 304    4     1 3.459370 2.0972222       379       102       2009
    ## 305    3     1 3.757269 1.8776978       334        51       2674
    ## 306   35     1 3.561364 1.2641509       291        24       1344
    ## 307   16     1 4.910190 1.5988372       375        90       2585
    ## 308   13     1 3.960254 1.5740741       329        69       2139
    ## 309   59     1 3.432387 1.1830065       345        62       1805
    ## 310   41     1 3.093146 1.0262009       333        32       1547
    ## 311   58     1 2.747100 1.1809045       338        98       2084
    ## 312   19     1 2.539823 1.3595506       283        89       1019
    ## 313   20     1 2.524740 0.1857143       321        16       1787
    ## 314   47     1 3.943005 1.0761421       260        13       1347
    ## 315    5     1 3.986357 1.3947368       324        57       2859
    ## 316   34     1 2.743455 1.3766234       214        67        893
    ## 317   42     1 4.033333 1.3034483       373        47       2334
    ## 318   61     1 3.030405 1.0375000       303        15       1579
    ## 319   55     1 3.509138 1.1846154       178        11       1144
    ## 320    8     1 3.265082 1.5600000       288        59       1564
    ## 321    1     1 3.621993 1.2522523       378        14       1835
    ## 322    9     1 3.544419 1.0378788       363        36       1408
    ## 323    7     1 2.244755 1.2641509       152        29        578
    ## 324   21     1 4.390879 1.0000000       485         8       2826
    ## 325   22     1 3.630476 1.2371134       343        36       1698
    ## 326   29     1 3.304189 1.0086207       295         6       1643
    ## 327   28     1 2.788793 1.2758621       178        34        584
    ## 328   39     1 3.298748 1.2043011       274        36       1537
    ## 329   54     1 3.487871 1.3139535       214        32       1118
    ## 330   14     1 3.420315 1.0396040       287         7       1625
    ## 331   23     1 3.024548 1.4324324       328        41       2138
    ## 332   11     1 3.380463 1.2168675       215        24       1136
    ## 333   15     1 3.967078 1.1647059       277        27       1643
    ## 334   27     1 3.616867 1.3661972       317        37       1361
    ## 335   45     1 2.776181 0.5584416       258        20       1188
    ## 336   10     1 4.204846 1.0375000       289        15       1808
    ## 337   44     1 3.088757 1.2592593       275         9       1417
    ## 338   46     1 4.883966 1.1666667       387        10       2144
    ## 339   12     1 4.195335 1.0877193       235        17       1262
    ## 340   40     1 2.997050 1.0175439       252         9       1827
    ## 341   56     1 2.548969 1.0444444       195         9        955
    ## 342   50     1 3.704110 1.1000000       265         8       1215
    ## 343   48     1 3.765528 1.2500000       303        17       2147
    ## 344   51     1 3.435743 1.1818182       331         7       1503
    ## 345   17     1 3.182390 1.0277778       281         9       1418
    ## 346   33     1 2.287293 1.2500000       206        13        788
    ## 347    6     1 2.618729 1.0000000       212         4        761
    ## 348   24     1 2.917355 1.0833333       193         6        654
    ## 349   30     1 3.607088 0.9000000       366        11       2054
    ## 350   36     1 3.921109 1.2307692       281         8       1631
    ## 351   32     1 3.686347 1.5000000       228         3        927
    ## 352   57     1 3.833770 0.0000000       386         0       2613
    ##     tokens_CHI Diagnosis        Ethnicity Gender   Age ADOS nonVerbalIQ
    ## 1         1294        TD            White      M 41.93   NA          45
    ## 2         1249       ASD            White      M 51.00   NA          46
    ## 3         1079        TD            White      M 44.07   NA          50
    ## 4         1010        TD            White      M 42.47   NA          48
    ## 5          921       ASD            White      M 57.37   NA          49
    ## 6          897       ASD            White      M 58.77   NA          50
    ## 7          864        TD            White      M 39.40   NA          47
    ## 8          847        TD            White      F 39.23   NA          43
    ## 9          793        TD            White      M 39.43   NA          44
    ## 10         738       ASD     White/Latino      M 51.37   NA          44
    ## 11         719        TD            White      M 40.13   NA          46
    ## 12         713       ASD            White      M 54.73   NA          50
    ## 13         710        TD            White      M 42.93   NA          49
    ## 14         702        TD            White      M 39.93   NA          45
    ## 15         698       ASD            White      M 49.70   NA          48
    ## 16         686        TD            White      M 40.13   NA          42
    ## 17         666        TD            White      M 40.27   NA          42
    ## 18         659        TD            White      M 41.00   NA          NA
    ## 19         646        TD            White      M 43.40   NA          45
    ## 20         640        TD            White      M 41.50   NA          44
    ## 21         618       ASD            White      M 46.07   NA          45
    ## 22         611        TD            White      M 39.07   NA          45
    ## 23         605       ASD            White      M 53.77   NA          44
    ## 24         590       ASD            White      M 37.30   NA          43
    ## 25         571       ASD         Lebanese      M 46.40   NA          41
    ## 26         568        TD            White      M 38.53   NA          46
    ## 27         538       ASD            White      M 54.13   NA          40
    ## 28         530        TD            White      M 41.17   NA          43
    ## 29         521        TD            White      M 39.43   NA          39
    ## 30         517        TD            White      M 39.30   NA          41
    ## 31         460        TD            White      F 45.07   NA          45
    ## 32         436        TD            White      M 43.03   NA          47
    ## 33         418        TD            White      M 43.80   NA          45
    ## 34         410        TD            White      M 40.30   NA          40
    ## 35         395        TD            Asian      F 42.10   NA          46
    ## 36         358        TD            White      M 44.43   NA          45
    ## 37         306       ASD            White      M    NA   NA          32
    ## 38         300       ASD      Bangladeshi      F 46.53   NA          39
    ## 39         274       ASD            White      M 56.73   NA          30
    ## 40         236       ASD     White/Latino      M 47.50   NA          30
    ## 41         210       ASD            White      M 62.33   NA          41
    ## 42         204       ASD            White      M 55.17   NA          39
    ## 43         197        TD            White      M 40.17   NA          43
    ## 44         195       ASD            White      M 56.43   NA          49
    ## 45         189        TD            White      M 40.43   NA          33
    ## 46         143       ASD            White      M 62.40   NA          30
    ## 47         137       ASD            White      M 57.43   NA          32
    ## 48         110       ASD      White/Asian      M 53.63   NA          30
    ## 49         100       ASD            White      M 54.63   NA          28
    ## 50          79       ASD African American      F 46.17   NA          33
    ## 51          64       ASD            White      F 61.70   NA          32
    ## 52          61        TD            White      F 40.37   NA          43
    ## 53          37       ASD            White      M 54.43   NA          27
    ## 54          36       ASD            White      M    NA   NA          34
    ## 55           8       ASD            White      M 62.40   NA          24
    ## 56           2       ASD African American      M 48.97   NA          26
    ## 57        1154       ASD            White      M 46.93    4          NA
    ## 58        1145        TD            White      M 37.93    0          NA
    ## 59        1054       ASD            White      M 53.40   11          NA
    ## 60         932        TD            White      M 35.37    0          NA
    ## 61         916        TD            White      M 35.87    0          NA
    ## 62         875        TD            White      M 36.40    0          NA
    ## 63         815       ASD            White      M 55.17    4          NA
    ## 64         814        TD            White      M 40.07    0          NA
    ## 65         800        TD            White      F 36.13    0          NA
    ## 66         792       ASD            White      M 34.13    8          NA
    ## 67         769        TD            White      M 38.70    0          NA
    ## 68         755        TD            White      M 35.10    0          NA
    ## 69         736        TD            White      M 34.93    2          NA
    ## 70         725        TD            White      M 35.17    0          NA
    ## 71         723        TD            White      M 38.60    0          NA
    ## 72         690        TD            White      M 39.53    0          NA
    ## 73         686        TD            White      F 35.13    0          NA
    ## 74         651        TD            White      M 35.67    0          NA
    ## 75         636        TD            White      M 37.34    5          NA
    ## 76         622        TD            White      M 36.43    0          NA
    ## 77         588        TD            White      F 39.47    0          NA
    ## 78         586        TD            White      M 39.47    0          NA
    ## 79         583        TD            White      M 38.17    0          NA
    ## 80         539       ASD     White/Latino      M 47.40    9          NA
    ## 81         537       ASD            White      M 50.30   16          NA
    ## 82         531       ASD            White      M 42.30   16          NA
    ## 83         513        TD            White      M 35.63    0          NA
    ## 84         511       ASD            White      M 58.57   15          NA
    ## 85         502        TD            White      M 36.70   10          NA
    ## 86         472        TD            White      M 35.90    0          NA
    ## 87         459        TD            White      M 34.40    2          NA
    ## 88         433        TD            White      M 35.83    1          NA
    ## 89         431        TD            White      M 36.53    5          NA
    ## 90         404       ASD            White      M 49.30    9          NA
    ## 91         357       ASD            White      M 51.13   15          NA
    ## 92         349       ASD      Bangladeshi      F 42.43    8          NA
    ## 93         346        TD            White      F 35.00    1          NA
    ## 94         340       ASD      White/Asian      M 48.63   11          NA
    ## 95         323        TD            White      M 36.07    5          NA
    ## 96         318        TD            White      M 37.73    0          NA
    ## 97         304        TD            White      F 39.10   NA          NA
    ## 98         255       ASD            White      M 52.77   17          NA
    ## 99         238        TD            Asian      F 37.67    3          NA
    ## 100        197       ASD            White      M 51.97    9          NA
    ## 101        197       ASD     White/Latino      M 44.93   14          NA
    ## 102        194       ASD            White      F 57.73   17          NA
    ## 103        193       ASD            White      M 52.90   15          NA
    ## 104        179       ASD            White      M 50.97   20          NA
    ## 105        160        TD            White      M 35.90    3          NA
    ## 106         42       ASD African American      F 42.97   25          NA
    ## 107         34       ASD            White      M 57.20   17          NA
    ## 108         33       ASD            White      M 50.73   20          NA
    ## 109         32       ASD            White      M 54.33   17          NA
    ## 110         30       ASD            White      F 58.27   13          NA
    ## 111          9       ASD            White      M 50.93   15          NA
    ## 112          5       ASD African American      M 44.70   19          NA
    ## 113          0       ASD            White      M 57.20   17          NA
    ## 114       1225        TD            White      M 34.00   NA          47
    ## 115       1092        TD            White      M 34.43   NA          50
    ## 116       1051        TD            White      M 32.03   NA          40
    ## 117        978        TD            White      M 36.40   NA          45
    ## 118        973        TD            White      F 32.13   NA          42
    ## 119        822       ASD            White      M 49.30   NA          50
    ## 120        820        TD            White      M 32.07   NA          39
    ## 121        793        TD            White      M 34.27   NA          45
    ## 122        754        TD            White      F 35.53   NA          39
    ## 123        750        TD            White      M 35.03   NA          43
    ## 124        717       ASD            White      M 30.80   NA          43
    ## 125        714       ASD            White      M 42.63   NA          41
    ## 126        694        TD            White      M 30.83   NA          39
    ## 127        693       ASD            White      M 47.00   NA          47
    ## 128        684        TD            White      M 35.63   NA          45
    ## 129        674       ASD            White      M 41.07   NA          49
    ## 130        642        TD            White      M 35.03   NA          45
    ## 131        637        TD            White      M 33.60   NA          34
    ## 132        604       ASD            White      M 45.53   NA          38
    ## 133        592        TD            White      F 30.93   NA          40
    ## 134        576        TD            White      F 32.17   NA          44
    ## 135        574       ASD            White      M 38.20   NA          40
    ## 136        557        TD            White      M 31.47   NA          29
    ## 137        538        TD            White      M 31.03   NA          40
    ## 138        510       ASD            White      M 51.67   NA          49
    ## 139        502        TD            White      M 31.00   NA          28
    ## 140        495        TD            White      M 31.27   NA          33
    ## 141        494       ASD            White      F 52.27   NA          29
    ## 142        493        TD            White      M 32.07   NA          42
    ## 143        493       ASD         Lebanese      M    NA   NA          NA
    ## 144        479       ASD     White/Latino      M 43.13   NA          41
    ## 145        477        TD            White      M 30.40   NA          35
    ## 146        455       ASD            White      M 47.83   NA          35
    ## 147        438       ASD            White      M 46.13   NA          31
    ## 148        434        TD            White      M 32.07   NA          41
    ## 149        433        TD            White      M 32.77   NA          26
    ## 150        414        TD            White      M 32.43   NA          31
    ## 151        390        TD            White      F 31.07   NA          45
    ## 152        384        TD            White      M 33.30   NA          42
    ## 153        378        TD            White      M 32.07   NA          40
    ## 154        354        TD            White      M 31.20   NA          35
    ## 155        348        TD            Asian      F 34.43   NA          30
    ## 156        334       ASD            White      M 54.13   NA          28
    ## 157        321        TD            White      M 32.90   NA          33
    ## 158        243        TD            White      M 32.07   NA          42
    ## 159        205       ASD      Bangladeshi      F 38.47   NA          33
    ## 160        162       ASD     White/Latino      M 42.23   NA          29
    ## 161        143       ASD            White      M 52.13   NA          29
    ## 162        142       ASD African American      F 36.93   NA          36
    ## 163        135       ASD      White/Asian      M 45.33   NA          45
    ## 164         93       ASD African American      M 40.27   NA          27
    ## 165         79       ASD            White      M 48.10   NA          47
    ## 166         74        TD            White      M 30.80   NA          36
    ## 167         70       ASD            White      M 52.13   NA          22
    ## 168         55       ASD            White      M 50.23   NA          29
    ## 169         45       ASD            White      M 43.63   NA          33
    ## 170         39       ASD            White      M 46.63   NA          27
    ## 171         14       ASD            White      M 47.10   NA          27
    ## 172          1       ASD            White      F 53.90   NA          30
    ## 173       1293       ASD            White      M 38.60   NA          NA
    ## 174       1246        TD            White      M 32.07   NA          NA
    ## 175       1023       ASD            White      M 46.23   NA          NA
    ## 176        983       ASD            White      M 37.07   NA          NA
    ## 177        940        TD            White      M 30.03   NA          NA
    ## 178        897        TD            White      M 30.77   NA          NA
    ## 179        826        TD            White      M 30.13   NA          NA
    ## 180        825        TD            White      M 30.63   NA          NA
    ## 181        825        TD            White      F 28.07   NA          NA
    ## 182        733        TD            White      M 28.27   NA          NA
    ## 183        702        TD            White      M 27.13   NA          NA
    ## 184        698       ASD            White      M 42.47   NA          NA
    ## 185        694       ASD            White      M 47.73   NA          NA
    ## 186        672        TD            White      M 28.03   NA          NA
    ## 187        671        TD            White      M 31.63   NA          NA
    ## 188        660        TD            White      M 32.50   NA          NA
    ## 189        630        TD            White      M 27.07   NA          NA
    ## 190        604        TD            White      M 28.90   NA          NA
    ## 191        542        TD            White      F 31.83   NA          NA
    ## 192        528        TD            White      M 27.67   NA          NA
    ## 193        503       ASD            White      M 26.77   NA          NA
    ## 194        490       ASD     White/Latino      M 38.90   NA          NA
    ## 195        486        TD            White      F 29.53   NA          NA
    ## 196        468       ASD            White      M 41.57   NA          NA
    ## 197        453        TD            White      F 26.63   NA          NA
    ## 198        424        TD            White      M 29.67   NA          NA
    ## 199        404       ASD         Lebanese      M 34.00   NA          NA
    ## 200        403        TD            White      M 27.03   NA          NA
    ## 201        393       ASD            White      M 34.27   NA          NA
    ## 202        368        TD            White      M 26.80   NA          NA
    ## 203        365       ASD            White      M 43.00   NA          NA
    ## 204        354        TD            White      M 28.63   NA          NA
    ## 205        352        TD            White      M 28.47   NA          NA
    ## 206        330       ASD            White      M 43.90   NA          NA
    ## 207        305        TD            White      F 26.67   NA          NA
    ## 208        299        TD            White      M 29.07   NA          NA
    ## 209        261        TD            White      M 27.07   NA          NA
    ## 210        255        TD            White      M 27.70   NA          NA
    ## 211        254       ASD            White      M 50.63   NA          NA
    ## 212        252        TD            White      M 28.10   NA          NA
    ## 213        232       ASD      White/Asian      M 41.73   NA          NA
    ## 214        218        TD            White      M 26.87   NA          NA
    ## 215        210       ASD            White      F 49.20   NA          NA
    ## 216        193        TD            White      M 27.27   NA          NA
    ## 217        188        TD            White      M 28.53   NA          NA
    ## 218        151       ASD            White      M 44.67   NA          NA
    ## 219        148       ASD     White/Latino      M 35.93   NA          NA
    ## 220        122        TD            Asian      F 29.67   NA          NA
    ## 221         96       ASD African American      M 36.17   NA          NA
    ## 222         77       ASD            White      M 46.37   NA          NA
    ## 223         62       ASD      Bangledeshi      F 34.57   NA          NA
    ## 224         58       ASD            White      M    NA   NA          NA
    ## 225         41       ASD            White      F 49.97   NA          NA
    ## 226         40       ASD            White      M 46.37   NA          NA
    ## 227         39       ASD            White      M 43.10   NA          NA
    ## 228         26       ASD African American      F 34.10   NA          NA
    ## 229         14       ASD            White      M 44.47   NA          NA
    ## 230          3       ASD            White      M 45.57   NA          NA
    ## 231          1       ASD            White      M 42.13   NA          NA
    ## 232        879        TD            White      M 23.43   NA          NA
    ## 233        738        TD            White      M 26.13   NA          NA
    ## 234        733        TD            White      M 26.27   NA          NA
    ## 235        670       ASD            White      M 38.63   NA          NA
    ## 236        623        TD            White      M 25.47   NA          NA
    ## 237        571        TD            White      F 24.40   NA          NA
    ## 238        562       ASD            White      M 33.17   NA          NA
    ## 239        555       ASD     White/Latino      M 35.30   NA          NA
    ## 240        554        TD            White      M 24.07   NA          NA
    ## 241        542        TD            White      M 27.70   NA          NA
    ## 242        536       ASD            White      M 42.77   NA          NA
    ## 243        530        TD            White      F 27.83   NA          NA
    ## 244        487       ASD            White      M 37.57   NA          NA
    ## 245        449        TD            White      M 28.60   NA          NA
    ## 246        439        TD            White      F 24.17   NA          NA
    ## 247        433       ASD            White      M 43.68   NA          NA
    ## 248        423        TD            White      M    NA   NA          NA
    ## 249        414        TD            White      F 22.90   NA          NA
    ## 250        401        TD            White      M 25.87   NA          NA
    ## 251        393        TD            White      M 22.97   NA          NA
    ## 252        368       ASD            White      M 38.13   NA          NA
    ## 253        362       ASD            White      M 22.50   NA          NA
    ## 254        356        TD            White      M 27.47   NA          NA
    ## 255        356       ASD            White      M 46.77   NA          NA
    ## 256        335       ASD            White      M 39.93   NA          NA
    ## 257        298        TD            White      M 26.63   NA          NA
    ## 258        297        TD            White      M 22.87   NA          NA
    ## 259        287        TD            White      M 23.17   NA          NA
    ## 260        264        TD            White      M 23.07   NA          NA
    ## 261        260       ASD            White      M 30.10   NA          NA
    ## 262        258        TD            White      M 23.57   NA          NA
    ## 263        244        TD            White      M 23.93   NA          NA
    ## 264        236        TD            White      M 25.23   NA          NA
    ## 265        203        TD            White      M 23.83   NA          NA
    ## 266        177       ASD            White      M 38.13   NA          NA
    ## 267        169       ASD            White      F 49.20   NA          NA
    ## 268        165        TD            White      M 24.68   NA          NA
    ## 269        159        TD            White      M 24.80   NA          NA
    ## 270        157       ASD     White/Latino      M 32.10   NA          NA
    ## 271        149       ASD      Bangladeshi      F 30.27   NA          NA
    ## 272        148        TD            White      M 23.93   NA          NA
    ## 273        148       ASD         Lebanese      M 32.20   NA          NA
    ## 274        147       ASD            White      M 42.20   NA          NA
    ## 275        132        TD            White      F 23.73   NA          NA
    ## 276        126       ASD      White/Asian      M 37.37   NA          NA
    ## 277        120        TD            White      M 23.63   NA          NA
    ## 278        117        TD            White      M    NA   NA          NA
    ## 279        116       ASD            White      M 40.50   NA          NA
    ## 280        107        TD            White      M 22.87   NA          NA
    ## 281         97        TD            Asian      F 25.40   NA          NA
    ## 282         82       ASD            White      M 38.73   NA          NA
    ## 283         78       ASD African American      M 31.80   NA          NA
    ## 284         55        TD            White      M 23.83   NA          NA
    ## 285         44       ASD            White      F 41.07   NA          NA
    ## 286         38       ASD            White      M 41.50   NA          NA
    ## 287         32       ASD            White      M 42.20   NA          NA
    ## 288         19       ASD            White      M 39.40   NA          NA
    ## 289         10       ASD African American      F 29.70   NA          NA
    ## 290          7       ASD            White      M 38.33   NA          NA
    ## 291          4       ASD            White      M 37.20   NA          NA
    ## 292        483       ASD            White      M 30.40   11          32
    ## 293        473        TD            White      M 23.90    0          30
    ## 294        469       ASD            White      M 34.00   13          30
    ## 295        461       ASD            White      M 28.80   13          34
    ## 296        450       ASD            White      M 37.03   14          42
    ## 297        433        TD            White      M 21.77    0          29
    ## 298        406        TD            White      M 20.77    0          29
    ## 299        398        TD            White      F 19.87    1          29
    ## 300        337       ASD            White      M 39.50    7          33
    ## 301        319        TD            White      M 23.13    0          27
    ## 302        319        TD            White      M 22.57    0          29
    ## 303        286        TD            White      M 23.07    0          27
    ## 304        269       ASD     White/Latino      M 31.03    8          31
    ## 305        260        TD            White      F 23.50    1          29
    ## 306        260        TD            White      M 19.30    1          23
    ## 307        254        TD            White      M 21.67    0          29
    ## 308        249        TD            White      M 19.00    0          25
    ## 309        244        TD            White      M 20.80    4          29
    ## 310        235        TD            White      M 19.23    0          25
    ## 311        233       ASD            White      M 26.00   15          30
    ## 312        227       ASD            White      M 35.80   11          28
    ## 313        214       ASD            White      M 18.77    9          26
    ## 314        212        TD            White      F 20.03    0          27
    ## 315        197       ASD            White      M 34.03    9          34
    ## 316        180       ASD            White      M 33.77   10          27
    ## 317        176        TD            White      M 19.37    0          24
    ## 318        166       ASD            White      M 42.00   15          27
    ## 319        154        TD            White      M 19.97    0          26
    ## 320        143        TD            White      M 20.10    5          32
    ## 321        139        TD            White      M 19.80    0          28
    ## 322        137        TD            White      F 18.30    0          24
    ## 323        130       ASD            White      F 41.00   18          24
    ## 324        122       ASD African American      M 27.53   21          22
    ## 325        118        TD            White      F 18.93    0          21
    ## 326        117       ASD            White      M 34.87   17          26
    ## 327        111        TD            White      M 19.93    0          20
    ## 328        109       ASD      White/Asian      M 33.20   11          26
    ## 329        109        TD            White      M 19.10    1          24
    ## 330        105        TD            White      M 18.97    0          23
    ## 331        103       ASD     White/Latino      M 27.37   14          25
    ## 332        101        TD            White      M 19.23    0          21
    ## 333         99        TD            White      M 19.27    0          24
    ## 334         95        TD            White      M 21.03    0          26
    ## 335         91        TD            White      M 20.03    5          24
    ## 336         83        TD            White      M 19.27    3          27
    ## 337         68        TD            White      M 19.77    3          30
    ## 338         63       ASD            White      M 36.73   20          21
    ## 339         62        TD            White      M 20.07    0          30
    ## 340         58       ASD         Lebanese      M 24.90   13          27
    ## 341         47       ASD            White      M 35.50   14          27
    ## 342         43       ASD            White      M 37.47   19          17
    ## 343         40       ASD            White      M 31.63   17          28
    ## 344         38        TD            Asian      F 20.87    1          22
    ## 345         37       ASD            White      M 34.80   14          25
    ## 346         35       ASD African American      F 25.33   14          25
    ## 347         29       ASD      Bangladeshi      F 26.17   17          20
    ## 348         26       ASD            White      M 37.47   20          13
    ## 349         21       ASD            White      M 36.53   12          31
    ## 350         16        TD            White      M 19.20    3          19
    ## 351          3       ASD            White      M 34.27   21          21
    ## 352          0       ASD            White      F 41.07   15          28
    ##     verbalIQ Socialization ADOS1 nonVerbalIQ1 verbalIQ1 Socialisation1
    ## 1         38            95     0           26        17             96
    ## 2         46            90    11           32        33            100
    ## 3         50           116     0           29        33            106
    ## 4         39           103     0           29        22             90
    ## 5         45            77    14           42        27             65
    ## 6         48           116     7           33        26             70
    ## 7         41            92     0           25        17            102
    ## 8         40           110     0           21        19            106
    ## 9         37            83     0           23        17             92
    ## 10        44            74     8           31        27             82
    ## 11        46           101     0           24        19             96
    ## 12        50            97    13           30        30             87
    ## 13        46            97     0           29        26            102
    ## 14        36           101     0           25        17            102
    ## 15        48            81    13           34        27             85
    ## 16        44           100     0           28        14            108
    ## 17        27           101     0           21        15            106
    ## 18        NA            NA     4           29        22            108
    ## 19        48           101     0           27        27            108
    ## 20        47            97     0           30        16            104
    ## 21        30            79    15           30        24             88
    ## 22        34            97     1           23        21            102
    ## 23        32            92    10           27        22             82
    ## 24        48           103     9           26        14             86
    ## 25        37            94    13           27        13             86
    ## 26        31            97     3           19        13             98
    ## 27        37            75     9           34        27             82
    ## 28        35           106     0           26        18             96
    ## 29        32            95     0           20        16            102
    ## 30        41           101     1           24        22            115
    ## 31        45            86     1           29        18             88
    ## 32        44            97     0           29        22             94
    ## 33        45           108     0           27        22            100
    ## 34        40            94     0           24        15             86
    ## 35        40           108     1           22        14            102
    ## 36        36           101     0           30        30             98
    ## 37        21            63    17           28        10             82
    ## 38        26            74    17           20        17             68
    ## 39        32            72    11           28        20             88
    ## 40        18            65    14           25        19             65
    ## 41        31           103    15           27        16             79
    ## 42        21            72    17           26        14             72
    ## 43        39           108     5           32        31            102
    ## 44        22            85    12           31        13             70
    ## 45        33           103     3           27        18            104
    ## 46        12            68    20           13        11             67
    ## 47        10            59    20           21         9             75
    ## 48        29            63    11           26        19             88
    ## 49        10            66    21           21         9             69
    ## 50        18            83    14           25        11             76
    ## 51         9            68    15           28        10             66
    ## 52        46           101     0           27        20            102
    ## 53        11            65    14           25        11             74
    ## 54        17            NA    14           27        11             77
    ## 55        11            66    19           17        10             64
    ## 56        16            65    21           22         8             72
    ## 57        NA            97    11           32        33            100
    ## 58        NA           101     0           29        22             90
    ## 59        NA            77    14           42        27             65
    ## 60        NA           100     0           25        17            102
    ## 61        NA           101     0           30        16            104
    ## 62        NA           100     0           24        19             96
    ## 63        NA           105     7           33        26             70
    ## 64        NA           103     0           30        30             98
    ## 65        NA           120     1           29        28            104
    ## 66        NA           107     9           26        14             86
    ## 67        NA           118     0           29        33            106
    ## 68        NA            87     0           23        17             92
    ## 69        NA           103     1           23        21            102
    ## 70        NA           105     1           24        22            115
    ## 71        NA            94     0           29        22             94
    ## 72        NA           106     0           27        22            100
    ## 73        NA           114     0           21        19            106
    ## 74        NA           105     0           25        17            102
    ## 75        NA           116     4           29        22            108
    ## 76        NA           100     0           20        16            102
    ## 77        NA            86     1           29        18             88
    ## 78        NA           101     0           27        27            108
    ## 79        NA           103     0           29        26            102
    ## 80        NA            79     8           31        27             82
    ## 81        NA            90    10           27        22             82
    ## 82        NA            83    15           30        24             88
    ## 83        NA           100     0           21        15            106
    ## 84        NA            95    15           27        16             79
    ## 85        NA           122     5           24        20            113
    ## 86        NA           107     0           28        14            108
    ## 87        NA            91     3           19        13             98
    ## 88        NA           107     3           30        20             94
    ## 89        NA           103     3           27        18            104
    ## 90        NA            75     9           34        27             82
    ## 91        NA            92    13           30        30             87
    ## 92        NA            68    17           20        17             68
    ## 93        NA           107     0           24        18            100
    ## 94        NA            63    11           26        19             88
    ## 95        NA           100     5           32        31            102
    ## 96        NA           112     0           26        18             96
    ## 97        NA            95     0           27        20            102
    ## 98        NA            68    17           28        10             82
    ## 99        NA           112     1           22        14            102
    ## 100       NA            72    11           28        20             88
    ## 101       NA            65    14           25        19             65
    ## 102       NA            63    18           24        14             65
    ## 103       NA            77    12           31        13             70
    ## 104       NA            68    17           26        14             72
    ## 105       NA            86     0           24        15             86
    ## 106       NA            85    14           25        11             76
    ## 107       NA            72    20           13        11             67
    ## 108       NA            66    21           21         9             69
    ## 109       NA            63    20           21         9             75
    ## 110       NA            66    15           28        10             66
    ## 111       NA            66    14           25        11             74
    ## 112       NA            72    21           22         8             72
    ## 113       NA            68    19           17        10             64
    ## 114       NA           102     0           29        22             90
    ## 115       NA           105     0           29        33            106
    ## 116       NA            93     0           26        17             96
    ## 117       NA           108     0           30        30             98
    ## 118       NA            93     0           27        20            102
    ## 119       NA            75    14           42        27             65
    ## 120       NA           109     0           24        19             96
    ## 121       NA           100     0           29        26            102
    ## 122       NA            93     1           29        18             88
    ## 123       NA           107     0           27        22            100
    ## 124       NA            98     9           26        14             86
    ## 125       NA            86    11           32        33            100
    ## 126       NA           105     1           24        22            115
    ## 127       NA           100    13           30        30             87
    ## 128       NA           105     0           27        27            108
    ## 129       NA            75    13           34        27             85
    ## 130       NA           100     0           29        22             94
    ## 131       NA           116     4           29        22            108
    ## 132       NA            77     9           34        27             82
    ## 133       NA           109     0           21        19            106
    ## 134       NA           125     1           29        28            104
    ## 135       NA            83    15           30        24             88
    ## 136       NA            96     0           23        17             92
    ## 137       NA           107     0           25        17            102
    ## 138       NA           101     7           33        26             70
    ## 139       NA            96     0           20        16            102
    ## 140       NA            87     0           24        15             86
    ## 141       NA            61    18           24        14             65
    ## 142       NA           109     5           32        31            102
    ## 143       NA            86    13           27        13             86
    ## 144       NA            77     8           31        27             82
    ## 145       NA           100     3           19        13             98
    ## 146       NA            74    11           28        20             88
    ## 147       NA            92    10           27        22             82
    ## 148       NA           100     0           25        17            102
    ## 149       NA           114     5           24        20            113
    ## 150       NA           103     3           27        18            104
    ## 151       NA           103     0           24        18            100
    ## 152       NA           100     0           26        18             96
    ## 153       NA            95     3           30        20             94
    ## 154       NA           100     0           21        15            106
    ## 155       NA           120     1           22        14            102
    ## 156       NA            92    15           27        16             79
    ## 157       NA           102     0           28        14            108
    ## 158       NA           103     0           30        16            104
    ## 159       NA            66    17           20        17             68
    ## 160       NA            63    14           25        19             65
    ## 161       NA            72    20           13        11             67
    ## 162       NA            86    14           25        11             76
    ## 163       NA            66    11           26        19             88
    ## 164       NA            68    21           22         8             72
    ## 165       NA            74    12           31        13             70
    ## 166       NA           105     1           23        21            102
    ## 167       NA            68    19           17        10             64
    ## 168       NA            55    20           21         9             75
    ## 169       NA            68    17           28        10             82
    ## 170       NA            72    17           26        14             72
    ## 171       NA            72    21           21         9             69
    ## 172       NA            68    15           28        10             66
    ## 173       NA            83    11           32        33            100
    ## 174       NA           107     0           27        22            100
    ## 175       NA            70    14           42        27             65
    ## 176       NA            77    13           34        27             85
    ## 177       NA           112     0           29        33            106
    ## 178       NA           103     0           29        22             94
    ## 179       NA           100     0           29        22             90
    ## 180       NA            95     0           29        26            102
    ## 181       NA           125     1           29        28            104
    ## 182       NA           105     0           30        16            104
    ## 183       NA           112     1           23        21            102
    ## 184       NA           100    13           30        30             87
    ## 185       NA           100     7           33        26             70
    ## 186       NA            91     0           26        17             96
    ## 187       NA           105     0           27        27            108
    ## 188       NA           107     0           30        30             98
    ## 189       NA           109     0           25        17            102
    ## 190       NA           109     0           24        19             96
    ## 191       NA            91     1           29        18             88
    ## 192       NA           100     3           30        20             94
    ## 193       NA            95     9           26        14             86
    ## 194       NA            81     8           31        27             82
    ## 195       NA            98     0           27        20            102
    ## 196       NA            77     9           34        27             82
    ## 197       NA           105     0           24        18            100
    ## 198       NA           116     4           29        22            108
    ## 199       NA            87    13           27        13             86
    ## 200       NA           100     0           20        16            102
    ## 201       NA            86    15           30        24             88
    ## 202       NA            91     0           23        17             92
    ## 203       NA            92    10           27        22             82
    ## 204       NA           112     5           24        20            113
    ## 205       NA           103     0           25        17            102
    ## 206       NA            79    11           28        20             88
    ## 207       NA           114     0           21        19            106
    ## 208       NA            98     0           26        18             96
    ## 209       NA           107     1           24        22            115
    ## 210       NA           109     0           28        14            108
    ## 211       NA            94    15           27        16             79
    ## 212       NA            86     0           24        15             86
    ## 213       NA            63    11           26        19             88
    ## 214       NA           103     3           19        13             98
    ## 215       NA            63    18           24        14             65
    ## 216       NA           102     0           21        15            106
    ## 217       NA           111     3           27        18            104
    ## 218       NA            79    12           31        13             70
    ## 219       NA            70    14           25        19             65
    ## 220       NA           105     1           22        14            102
    ## 221       NA            77    21           22         8             72
    ## 222       NA            72    19           17        10             64
    ## 223       NA            70    17           20        17             68
    ## 224       NA            68    14           25        11             74
    ## 225       NA            65    15           28        10             66
    ## 226       NA            74    20           13        11             67
    ## 227       NA            72    17           26        14             72
    ## 228       NA            74    14           25        11             76
    ## 229       NA            70    14           27        11             77
    ## 230       NA            61    20           21         9             75
    ## 231       NA            68    21           21         9             69
    ## 232       NA           105     1           23        21            102
    ## 233       NA           107     0           29        33            106
    ## 234       NA           102     0           29        26            102
    ## 235       NA            46    13           30        30             87
    ## 236       NA           108     0           29        22             90
    ## 237       NA           100     0           27        20            102
    ## 238       NA           105    13           34        27             85
    ## 239       NA           115     8           31        27             82
    ## 240       NA           102     5           32        31            102
    ## 241       NA           103     0           27        22            100
    ## 242       NA            72    14           42        27             65
    ## 243       NA            89     1           29        18             88
    ## 244       NA            94     9           34        27             82
    ## 245       NA            86     0           30        30             98
    ## 246       NA           116     1           29        28            104
    ## 247       NA            83     7           33        26             70
    ## 248       NA           105     0           24        19             96
    ## 249       NA           102     0           21        19            106
    ## 250       NA           118     4           29        22            108
    ## 251       NA           100     0           25        17            102
    ## 252       NA            74    10           27        22             82
    ## 253       NA            84     9           26        14             86
    ## 254       NA            70     0           27        27            108
    ## 255       NA            79    15           27        16             79
    ## 256       NA           102    11           28        20             88
    ## 257       NA            81     0           29        22             94
    ## 258       NA           102     0           23        17             92
    ## 259       NA            59     1           24        22            115
    ## 260       NA            92     0           25        17            102
    ## 261       NA            72    15           30        24             88
    ## 262       NA            76     0           20        16            102
    ## 263       NA            96     0           26        17             96
    ## 264       NA            98     0           26        18             96
    ## 265       NA           115     0           30        16            104
    ## 266       NA            68    17           26        14             72
    ## 267       NA            63    18           24        14             65
    ## 268       NA           118     5           24        20            113
    ## 269       NA            86     0           24        15             86
    ## 270       NA            65    14           25        19             65
    ## 271       NA            74    17           20        17             68
    ## 272       NA           110     0           28        14            108
    ## 273       NA            38    13           27        13             86
    ## 274       NA            77    19           17        10             64
    ## 275       NA           106     0           24        18            100
    ## 276       NA            72    11           26        19             88
    ## 277       NA           106     3           30        20             94
    ## 278       NA           104     0           21        15            106
    ## 279       NA            79    12           31        13             70
    ## 280       NA           102     3           19        13             98
    ## 281       NA           109     1           22        14            102
    ## 282       NA           104    14           25        11             74
    ## 283       NA            74    21           22         8             72
    ## 284       NA           106     3           27        18            104
    ## 285       NA            66    15           28        10             66
    ## 286       NA            74    20           21         9             75
    ## 287       NA            78    20           13        11             67
    ## 288       NA            65    14           27        11             77
    ## 289       NA            74    14           25        11             76
    ## 290       NA            80    21           21         9             69
    ## 291       NA            66    17           28        10             82
    ## 292       33           100    11           32        33            100
    ## 293       30            98     0           30        30             98
    ## 294       30            87    13           30        30             87
    ## 295       27            85    13           34        27             85
    ## 296       27            65    14           42        27             65
    ## 297       22            90     0           29        22             90
    ## 298       33           106     0           29        33            106
    ## 299       28           104     1           29        28            104
    ## 300       26            70     7           33        26             70
    ## 301       22           100     0           27        22            100
    ## 302       22            94     0           29        22             94
    ## 303       27           108     0           27        27            108
    ## 304       27            82     8           31        27             82
    ## 305       18            88     1           29        18             88
    ## 306       21           102     1           23        21            102
    ## 307       26           102     0           29        26            102
    ## 308       17           102     0           25        17            102
    ## 309       22           108     4           29        22            108
    ## 310       17           102     0           25        17            102
    ## 311       24            88    15           30        24             88
    ## 312       20            88    11           28        20             88
    ## 313       14            86     9           26        14             86
    ## 314       20           102     0           27        20            102
    ## 315       27            82     9           34        27             82
    ## 316       22            82    10           27        22             82
    ## 317       19            96     0           24        19             96
    ## 318       16            79    15           27        16             79
    ## 319       17            96     0           26        17             96
    ## 320       31           102     5           32        31            102
    ## 321       14           108     0           28        14            108
    ## 322       18           100     0           24        18            100
    ## 323       14            65    18           24        14             65
    ## 324        8            72    21           22         8             72
    ## 325       19           106     0           21        19            106
    ## 326       14            72    17           26        14             72
    ## 327       16           102     0           20        16            102
    ## 328       19            88    11           26        19             88
    ## 329       22           115     1           24        22            115
    ## 330       17            92     0           23        17             92
    ## 331       19            65    14           25        19             65
    ## 332       15           106     0           21        15            106
    ## 333       15            86     0           24        15             86
    ## 334       18            96     0           26        18             96
    ## 335       20           113     5           24        20            113
    ## 336       18           104     3           27        18            104
    ## 337       20            94     3           30        20             94
    ## 338        9            75    20           21         9             75
    ## 339       16           104     0           30        16            104
    ## 340       13            86    13           27        13             86
    ## 341       11            77    14           27        11             77
    ## 342       10            64    19           17        10             64
    ## 343       10            82    17           28        10             82
    ## 344       14           102     1           22        14            102
    ## 345       11            74    14           25        11             74
    ## 346       11            76    14           25        11             76
    ## 347       17            68    17           20        17             68
    ## 348       11            67    20           13        11             67
    ## 349       13            70    12           31        13             70
    ## 350       13            98     3           19        13             98
    ## 351        9            69    21           21         9             69
    ## 352       10            66    15           28        10             66

``` r
#kid nr. 55

#2: 
arrange(df_merged, VISIT, tokens_CHI)
```

    ##     SUBJ VISIT  MOT_MLU   CHI_MLU types_MOT types_CHI tokens_MOT
    ## 1     57     1 3.833770 0.0000000       386         0       2613
    ## 2     32     1 3.686347 1.5000000       228         3        927
    ## 3     36     1 3.921109 1.2307692       281         8       1631
    ## 4     30     1 3.607088 0.9000000       366        11       2054
    ## 5     24     1 2.917355 1.0833333       193         6        654
    ## 6      6     1 2.618729 1.0000000       212         4        761
    ## 7     33     1 2.287293 1.2500000       206        13        788
    ## 8     17     1 3.182390 1.0277778       281         9       1418
    ## 9     51     1 3.435743 1.1818182       331         7       1503
    ## 10    48     1 3.765528 1.2500000       303        17       2147
    ## 11    50     1 3.704110 1.1000000       265         8       1215
    ## 12    56     1 2.548969 1.0444444       195         9        955
    ## 13    40     1 2.997050 1.0175439       252         9       1827
    ## 14    12     1 4.195335 1.0877193       235        17       1262
    ## 15    46     1 4.883966 1.1666667       387        10       2144
    ## 16    44     1 3.088757 1.2592593       275         9       1417
    ## 17    10     1 4.204846 1.0375000       289        15       1808
    ## 18    45     1 2.776181 0.5584416       258        20       1188
    ## 19    27     1 3.616867 1.3661972       317        37       1361
    ## 20    15     1 3.967078 1.1647059       277        27       1643
    ## 21    11     1 3.380463 1.2168675       215        24       1136
    ## 22    23     1 3.024548 1.4324324       328        41       2138
    ## 23    14     1 3.420315 1.0396040       287         7       1625
    ## 24    39     1 3.298748 1.2043011       274        36       1537
    ## 25    54     1 3.487871 1.3139535       214        32       1118
    ## 26    28     1 2.788793 1.2758621       178        34        584
    ## 27    29     1 3.304189 1.0086207       295         6       1643
    ## 28    22     1 3.630476 1.2371134       343        36       1698
    ## 29    21     1 4.390879 1.0000000       485         8       2826
    ## 30     7     1 2.244755 1.2641509       152        29        578
    ## 31     9     1 3.544419 1.0378788       363        36       1408
    ## 32     1     1 3.621993 1.2522523       378        14       1835
    ## 33     8     1 3.265082 1.5600000       288        59       1564
    ## 34    55     1 3.509138 1.1846154       178        11       1144
    ## 35    61     1 3.030405 1.0375000       303        15       1579
    ## 36    42     1 4.033333 1.3034483       373        47       2334
    ## 37    34     1 2.743455 1.3766234       214        67        893
    ## 38     5     1 3.986357 1.3947368       324        57       2859
    ## 39    47     1 3.943005 1.0761421       260        13       1347
    ## 40    20     1 2.524740 0.1857143       321        16       1787
    ## 41    19     1 2.539823 1.3595506       283        89       1019
    ## 42    58     1 2.747100 1.1809045       338        98       2084
    ## 43    41     1 3.093146 1.0262009       333        32       1547
    ## 44    59     1 3.432387 1.1830065       345        62       1805
    ## 45    13     1 3.960254 1.5740741       329        69       2139
    ## 46    16     1 4.910190 1.5988372       375        90       2585
    ## 47     3     1 3.757269 1.8776978       334        51       2674
    ## 48    35     1 3.561364 1.2641509       291        24       1344
    ## 49     4     1 3.459370 2.0972222       379       102       2009
    ## 50    49     1 4.030075 1.4258373       255        73       1452
    ## 51    31     1 3.494327 1.5333333       322        91       1870
    ## 52    52     1 5.344227 1.4086957       441        92       2267
    ## 53    37     1 4.135036 0.4805825       381        39       1988
    ## 54    38     1 3.420975 1.3322785       342        96       2035
    ## 55    18     1 3.587855 1.7948718       467       108       2555
    ## 56    25     1 4.643200 1.5827338       373        71       2740
    ## 57    43     1 3.341418 1.9144981       271       101       1591
    ## 58     2     1 4.098446 2.2768595       317       146       1428
    ## 59    60     1 3.604140 2.8763441       400       149       2587
    ## 60    53     1 4.159159 1.9397163       236       120       1170
    ## 61    26     1 4.690751 3.4000000       278       119       1450
    ## 62    48     2 3.555804 1.0000000       272         2       1344
    ## 63    32     2 4.142562 1.0000000       454         2       1856
    ## 64    33     2 2.879925 1.0000000       285         2       1393
    ## 65    56     2 3.430642 1.0000000       285         1       1417
    ## 66    24     2 3.815141 1.0000000       434         6       1995
    ## 67    46     2 3.830795 1.2258065       392         6       1973
    ## 68    57     2 3.688478 1.0000000       389         1       2274
    ## 69    10     2 4.378840 1.1200000       358        18       2263
    ## 70    21     2 3.381510 1.0400000       426         9       2524
    ## 71    17     2 2.947230 1.0125000       243         7        999
    ## 72    51     2 4.073350 1.1976744       309        20       1483
    ## 73    36     2 3.942164 1.3103448       321        44       1843
    ## 74    30     2 3.843411 1.0265487       403        15       2211
    ## 75    11     2 2.821918 1.0630631       185        50        703
    ## 76    44     2 3.445545 1.3043478       271        52       1515
    ## 77    39     2 2.760976 1.2571429       277        55       1481
    ## 78     9     2 4.079060 1.6162791       369        45       1702
    ## 79    50     2 3.563356 1.3628319       381        20       1851
    ## 80     1     2 3.857367 1.0136054       403        18       2160
    ## 81    40     2 2.900838 1.2195122       295        52       1882
    ## 82     6     2 1.995885 0.7606838       140        29        464
    ## 83    23     2 2.903723 1.4273504       301        61       1990
    ## 84    15     2 4.224839 1.0460526       363        25       1819
    ## 85    45     2 3.171717 0.9051724       288        41       1375
    ## 86     7     2 2.417344 1.3211679       186        42        755
    ## 87    29     2 4.003072 1.0114286       331        18       2448
    ## 88    12     2 3.444664 1.3647799       297        72       1551
    ## 89    27     2 3.869654 1.4228571       287        72       1663
    ## 90    55     2 3.789634 1.7902098       219        70       1058
    ## 91    28     2 3.367292 1.6557377       255        71       1139
    ## 92    58     2 3.372320 1.4432990       260        78       1511
    ## 93    41     2 3.751332 1.1974249       404        97       1847
    ## 94    54     2 3.609881 2.0466667       352        88       1905
    ## 95    14     2 3.239832 1.0744681       346        65       1982
    ## 96    52     2 4.567329 1.7359551       415       122       1845
    ## 97    19     2 3.170000 1.4867257       315        86       1316
    ## 98    49     2 4.745238 2.4967742       408       126       1867
    ## 99    61     2 3.302663 1.0990991       250        48       1205
    ## 100   20     2 2.841004 1.0800000       288        73       1822
    ## 101   34     2 2.667722 1.9481132       220       100        748
    ## 102   13     2 3.817109 1.4761905       411       119       2260
    ## 103   59     2 4.001724 1.7709251       334       120       1959
    ## 104   22     2 3.978599 1.7804878       392       121       1843
    ## 105   42     2 4.100167 2.0046729       368       124       2201
    ## 106   37     2 4.054054 1.3432836       416       100       2340
    ## 107   38     2 4.073282 2.1727273       435       142       2290
    ## 108   53     2 4.557471 3.2179487       197       126        686
    ## 109    5     2 4.886646 2.5743590       441       152       2955
    ## 110    3     2 4.572086 2.6418605       464       149       2694
    ## 111   43     2 3.505582 1.9832215       309       170       2031
    ## 112   31     2 4.222034 2.2481481       405       141       2219
    ## 113    8     2 3.967213 1.6019108       305       122       2099
    ## 114    4     2 4.130337 2.5630252       343       170       1657
    ## 115    2     2 4.964664 3.4530387       307       171       1270
    ## 116   47     2 4.183445 2.2481481       295       128       1653
    ## 117   25     2 4.600000 2.4847328       419       145       2562
    ## 118   60     2 4.604341 2.7840000       413       149       2534
    ## 119   16     2 4.750455 2.7441077       391       196       2303
    ## 120   18     2 4.357911 2.7220339       461       160       2687
    ## 121   35     2 3.636519 1.9342916       300       128       1784
    ## 122   32     3 3.990826 1.0000000       323         1       1590
    ## 123   46     3 2.821782 1.0000000       245         2       1018
    ## 124   56     3 2.963303 1.7500000       252         9       1393
    ## 125   33     3 3.404959 1.0833333       272         3       1504
    ## 126   29     3 4.382550 0.6842105       345        13       2417
    ## 127   24     3 3.910448 1.0000000       344         1       1393
    ## 128   57     3 4.145588 0.1621622       542         3       2717
    ## 129   17     3 2.919169 1.0000000       252         2       1145
    ## 130    6     3 1.856115 1.1698113        74        12        250
    ## 131   50     3 3.979950 1.1666667       368        27       1423
    ## 132   21     3 3.646778 2.0000000       434        13       2764
    ## 133   51     3 4.022624 1.5529412       333        56       1562
    ## 134   23     3 3.215859 1.0863309       259        29       1332
    ## 135   30     3 4.015699 1.0066667       430        21       2278
    ## 136   10     3 4.298942 1.5520000       409        55       2936
    ## 137   11     3 3.940711 1.5234375       227        56        878
    ## 138    7     3 2.633929 1.3974359       196        50        768
    ## 139   36     3 4.595568 1.7500000       323        76       1504
    ## 140   39     3 3.607664 1.8880000       327        51       1709
    ## 141   15     3 4.182979 1.4408602       409        92       1743
    ## 142   61     3 4.520325 1.5828571       313        70       1618
    ## 143    1     3 4.321881 1.5568862       455        97       2149
    ## 144   54     3 4.298667 2.3309353       299        85       1418
    ## 145   27     3 3.846626 1.4976077       318        89       1706
    ## 146   22     3 3.094880 1.4549550       432       112       1811
    ## 147   19     3 3.615176 1.9052632       298        88       1159
    ## 148   41     3 4.050505 1.9086294       304       127       1426
    ## 149   45     3 3.395899 1.1722689       370        95       1912
    ## 150   34     3 3.866834 1.8056872       318       100       1350
    ## 151   14     3 3.848175 1.1855072       378        92       2328
    ## 152   58     3 3.774030 1.8000000       346       153       1891
    ## 153   28     3 3.210084 1.8369099       233       131       1006
    ## 154   40     3 3.723664 2.0704225       346       134       2148
    ## 155   59     3 4.210407 2.1157895       257       101       1610
    ## 156    9     3 4.592593 1.8320312       369       129       1991
    ## 157    5     3 4.198973 2.5324074       412       159       2939
    ## 158   47     3 3.960265 2.2893617       364       149       1568
    ## 159    4     3 3.818681 3.5180723       388       165       1788
    ## 160   20     3 4.063518 1.6109325       361       132       2092
    ## 161   44     3 4.106212 2.2313433       300       102       1740
    ## 162    3     3 5.134892 2.6916300       482       164       2630
    ## 163   42     3 4.057047 2.4911032       367       155       2124
    ## 164   13     3 3.871968 2.0798817       435       139       2571
    ## 165   53     3 4.078292 3.1313559       219       152       1020
    ## 166   49     3 4.737127 3.1124498       323       151       1609
    ## 167   55     3 4.231362 2.4932432       292       168       1470
    ## 168   37     3 3.642412 1.8812665       330       175       1570
    ## 169   60     3 4.907591 4.1318681       459       196       2841
    ## 170   35     3 3.675134 2.5710145       366       155       2328
    ## 171   12     3 4.974684 3.1857708       318       169       1463
    ## 172   16     3 4.164789 2.8076923       408       200       2675
    ## 173   38     3 4.577491 2.8690476       420       175       2146
    ## 174   25     3 4.127941 2.8042169       455       217       2589
    ## 175   52     3 5.288991 3.3037543       474       200       2142
    ## 176   18     3 4.116057 3.3400000       487       201       2479
    ## 177    2     3 4.147059 3.1193182       351       262       1445
    ## 178   43     3 3.573574 2.3053892       331       221       2141
    ## 179   31     3 3.552459 2.9875260       396       233       1872
    ## 180   26     3 4.316279 3.9196891       333       307       1668
    ## 181   57     4 3.536415 0.0000000       400         1       2196
    ## 182   32     4 3.743333 1.0000000       352         6       2054
    ## 183   29     4 3.885113 0.8947368       361        13       2214
    ## 184   48     4 3.339114 1.0000000       280         1       1482
    ## 185   46     4 2.377171 1.1200000       203        11        861
    ## 186   50     4 4.009934 1.0000000       358         1       1145
    ## 187   35     4 4.001613 1.9743590       361        29       2145
    ## 188   30     4 3.961832 1.1250000       409        26       1863
    ## 189   21     4 3.600624 0.2727273       427        10       2283
    ## 190   39     4 3.714556 1.7000000       302        36       1675
    ## 191   33     4 3.110932 1.2100840       281        18       1726
    ## 192   24     4 3.814815 1.2032520       353        29       1482
    ## 193   23     4 3.012594 1.0974026       270        21       1054
    ## 194    6     4 2.035714 1.7142857       125        50        360
    ## 195   12     4 3.988304 3.0000000       197       122        632
    ## 196    1     4 4.415330 2.2515723       533       133       2260
    ## 197   61     4 3.312388 1.3204225       348        61       1644
    ## 198   51     4 4.137014 3.0289855       363       112       1829
    ## 199   11     4 4.732733 2.1348315       272        88       1387
    ## 200   44     4 4.256790 2.5370370       302       122       1556
    ## 201   27     4 3.973469 1.7405858       364       114       1754
    ## 202    9     4 4.262195 2.7756410       400       121       1934
    ## 203   10     4 3.790896 1.6479401       411        96       2345
    ## 204   45     4 4.019928 1.7600000       424       108       1909
    ## 205   41     4 4.118790 2.2325581       366       151       1689
    ## 206   34     4 3.997126 2.1962617       321       107       1224
    ## 207   19     4 2.762463 1.6179402       248        87        855
    ## 208   36     4 3.705441 2.2594142       295       136       1715
    ## 209    4     4 4.301624 3.2571429       356       163       1711
    ## 210    8     4 4.658333 3.0265957       375       134       2069
    ## 211   40     4 3.812930 2.9900000       430       180       2377
    ## 212    7     4 2.736842 1.9115646       198       126        814
    ## 213   15     4 4.584071 2.5613208       357       161       1517
    ## 214   28     4 3.683406 2.3500000       309       139       1470
    ## 215   37     4 5.050000 2.5159817       410       148       2252
    ## 216   13     4 4.083700 2.7982833       463       203       2361
    ## 217   14     4 3.801292 2.3178295       369       158       1984
    ## 218   58     4 3.471510 2.0127796       359       144       2109
    ## 219   38     4 4.169162 2.6475410       513       195       2482
    ## 220   22     4 3.991667 2.3740741       480       169       2256
    ## 221    5     4 4.744000 2.9072581       384       187       2685
    ## 222   59     4 4.697624 2.7148289       330       157       1974
    ## 223   52     4 5.338462 3.0775510       554       177       2585
    ## 224    2     4 5.309804 4.3023256       335       200       1286
    ## 225   49     4 4.880240 3.4809524       340       229       1452
    ## 226   60     4 4.085409 3.3598326       539       214       3163
    ## 227   54     4 3.819961 3.4065041       332       153       1699
    ## 228   26     4 4.857143 3.5238095       398       188       2518
    ## 229   20     4 4.560201 2.2485380       422       153       2485
    ## 230   31     4 3.667638 2.7272727       480       186       2233
    ## 231    3     4 5.301053 3.9292035       449       206       2397
    ## 232   16     4 4.251605 2.3225806       508       231       3077
    ## 233   42     4 4.611247 3.8300395       339       193       1726
    ## 234   43     4 3.905213 2.6837838       390       243       2158
    ## 235   47     4 4.190698 3.1622419       343       211       1559
    ## 236   53     4 4.458937 3.6340694       232       217        798
    ## 237   55     4 4.177340 2.8620690       318       205       1524
    ## 238   18     4 4.131579 3.2128205       565       207       2965
    ## 239   25     4 5.362500 3.7310924       553       291       2978
    ## 240   50     5 3.864407 0.0000000       364         0       1447
    ## 241   21     5 4.019231 0.2000000       373         2       1963
    ## 242   17     5 3.996071 1.0000000       475         6       2088
    ## 243   57     5 3.693846 0.2000000       435         3       2070
    ## 244   46     5 3.637965 1.0000000       344         1       1609
    ## 245   32     5 3.414894 1.0000000       355         3       1388
    ## 246   24     5 4.421222 1.0000000       303         4       1278
    ## 247   33     5 4.185417 0.6756757       383         9       1917
    ## 248   15     5 4.165414 2.4691358       356        69       1426
    ## 249   29     5 4.872014 0.2553191       415         9       2560
    ## 250   30     5 3.859624 0.4130435       465        42       2360
    ## 251    7     5 3.992095 1.5299145       234        73        891
    ## 252   19     5 4.157191 1.5840000       328        45       1142
    ## 253   23     5 2.822259 1.4133333       306        50       1518
    ## 254   51     5 5.185941 2.9215686       388       108       1986
    ## 255   48     5 3.162554 1.4102564       369        79       1871
    ## 256   47     5 3.673418 2.7388060       315       131       1224
    ## 257   27     5 3.479393 2.1791908       327       114       1385
    ## 258    8     5 4.564103 1.9408602       387        40       2345
    ## 259   39     5 2.902089 1.6511628       321        90       1951
    ## 260    9     5 4.384946 2.8358209       428       121       1879
    ## 261    6     5 1.948454 1.6363636        92        80        209
    ## 262   60     5 4.223572 2.9655172       521       145       3090
    ## 263    5     5 4.292135 2.4232804       418       144       2749
    ## 264   10     5 3.867601 2.2835821       359        97       2208
    ## 265   44     5 4.113846 3.2000000       307       131       1207
    ## 266   36     5 4.137405 2.2787611       366       145       1979
    ## 267    1     5 5.209615 3.2380952       601       182       2553
    ## 268   45     5 4.329810 2.2558140       418       142       1791
    ## 269   61     5 2.965928 1.6688963       320       122       1527
    ## 270   11     5 4.543210 2.5482456       276       148       1300
    ## 271   58     5 3.290683 1.7591463       360       158       2276
    ## 272   34     5 3.419664 2.5758929       298       120       1223
    ## 273    4     5 4.602851 4.0434783       397       146       2082
    ## 274   16     5 5.433579 3.1095238       517       219       2762
    ## 275   49     5 5.743772 3.5371429       425       209       1525
    ## 276    3     5 4.566038 3.2985782       534       207       2672
    ## 277   28     5 3.708934 3.5852535       272       177       1098
    ## 278   59     5 4.424837 2.6038462       341       132       1916
    ## 279   41     5 3.969147 2.2892308       415       187       1936
    ## 280   22     5 4.746606 3.7000000       436       178       1965
    ## 281   31     5 3.906856 2.1420613       491       211       2729
    ## 282   52     5 4.983389 2.8321678       563       219       2772
    ## 283   54     5 3.750000 3.6072874       394       244       1977
    ## 284   35     5 4.069659 2.5541796       436       161       2279
    ## 285   14     5 4.446281 3.3750000       390       238       1864
    ## 286   18     5 3.877102 3.0902778       575       219       2881
    ## 287   20     5 4.837884 2.2506739       506       219       2504
    ## 288   38     5 4.917927 3.5185185       447       210       1999
    ## 289   53     5 4.857143 3.8225806       278       217       1067
    ## 290   37     5 4.658802 2.7468750       407       228       2314
    ## 291   42     5 3.921444 3.7749077       358       213       1600
    ## 292   12     5 4.910494 4.3647541       290       222       1467
    ## 293   13     5 4.487842 3.2301136       511       247       2668
    ## 294   43     5 4.232258 2.8665049       396       262       2326
    ## 295   25     5 4.267409 2.7413793       578       298       2940
    ## 296   26     5 4.345515 3.2919897       437       261       2410
    ## 297   21     6 3.943636 0.5000000       388         2       2077
    ## 298   50     6 3.886842 1.3333333       370         4       1396
    ## 299   56     6 3.474725 1.0588235       284         6       1390
    ## 300   17     6 3.650235 1.0000000       281         3       1454
    ## 301   47     6 4.676101 2.7600000       322        34       1371
    ## 302   57     6 3.241422 0.0156250       444         2       2450
    ## 303   33     6 3.965392 0.7536232       326        14       1852
    ## 304   32     6 4.003257 2.6315789       585        12       2202
    ## 305   39     6 4.100707 1.6447368       330        55       1987
    ## 306   46     6 3.460548 1.1610169       351        10       1918
    ## 307   24     6 3.860795 1.1680000       309        62       1349
    ## 308   10     6 4.235585 2.7051282       400        73       2271
    ## 309   30     6 4.341549 1.0843373       425       101       2219
    ## 310    8     6 4.366972 2.2258065       322        64       1352
    ## 311   29     6 4.349353 1.2883436       454        52       2391
    ## 312   61     6 3.514403 1.4012346       311        58       1481
    ## 313   23     6 3.534173 1.3052632       372        47       1748
    ## 314   19     6 3.156695 2.1450382       256        55        995
    ## 315    6     6 2.483146 1.4967320       158        66        536
    ## 316   48     6 3.370093 1.4734513       319        98       1565
    ## 317   53     6 3.706790 3.2439024       249       102       1024
    ## 318   51     6 5.153639 2.7612903       383       140       1866
    ## 319   15     6 4.347709 2.8690476       327       156       1395
    ## 320   31     6 3.603473 2.0717489       475       185       2363
    ## 321   52     6 5.587332 2.4292929       548       163       2887
    ## 322    3     6 5.229885 3.7103448       486       173       2564
    ## 323   54     6 4.186161 2.8921569       437       183       2347
    ## 324   28     6 4.239437 2.9607843       391       164       1889
    ## 325   27     6 4.254505 2.4800000       385       154       1788
    ## 326    5     6 4.587179 2.7665198       462       179       3182
    ## 327   36     6 4.388235 2.5614754       324       165       1978
    ## 328   40     6 3.926928 2.1586207       417       170       2508
    ## 329   20     6 5.379798 2.9027778       433       155       2389
    ## 330   34     6 3.370937 2.2745098       342       157       1540
    ## 331   35     6 4.239198 2.3815789       411       156       2438
    ## 332   58     6 3.320000 2.1553398       335       197       2221
    ## 333   12     6 4.468493 3.5049505       339       201       1498
    ## 334   49     6 5.247093 3.5950000       383       217       1701
    ## 335   59     6 4.113158 2.8480000       303       158       1460
    ## 336   11     6 4.287582 2.7571429       260       168       1138
    ## 337    1     6 4.664013 2.8651685       595       210       2586
    ## 338    2     6 4.588477 3.4135021       304       245        999
    ## 339   41     6 4.061475 2.8526316       397       213       1741
    ## 340   16     6 4.445872 2.9482072       491       250       2264
    ## 341   60     6 4.080446 3.4415584       505       226       3072
    ## 342   42     6 3.391525 3.0727969       357       219       1764
    ## 343    4     6 3.532374 3.2781955       410       166       2171
    ## 344   14     6 4.664286 3.8114035       359       178       1802
    ## 345   22     6 4.211321 3.0911950       478       221       2044
    ## 346   13     6 4.847418 3.7011952       388       210       1959
    ## 347   37     6 4.240798 3.0774194       429       217       2510
    ## 348   43     6 4.250853 2.6794872       374       219       2227
    ## 349   25     6 4.472993 3.0614525       555       237       2895
    ## 350   18     6 4.013353 2.9095355       516       235       2576
    ## 351   26     6 4.111413 3.3643411       452       273       3076
    ## 352   55     6 3.957230 2.9092742       367       260       1731
    ##     tokens_CHI Diagnosis        Ethnicity Gender   Age ADOS nonVerbalIQ
    ## 1            0       ASD            White      F 41.07   15          28
    ## 2            3       ASD            White      M 34.27   21          21
    ## 3           16        TD            White      M 19.20    3          19
    ## 4           21       ASD            White      M 36.53   12          31
    ## 5           26       ASD            White      M 37.47   20          13
    ## 6           29       ASD      Bangladeshi      F 26.17   17          20
    ## 7           35       ASD African American      F 25.33   14          25
    ## 8           37       ASD            White      M 34.80   14          25
    ## 9           38        TD            Asian      F 20.87    1          22
    ## 10          40       ASD            White      M 31.63   17          28
    ## 11          43       ASD            White      M 37.47   19          17
    ## 12          47       ASD            White      M 35.50   14          27
    ## 13          58       ASD         Lebanese      M 24.90   13          27
    ## 14          62        TD            White      M 20.07    0          30
    ## 15          63       ASD            White      M 36.73   20          21
    ## 16          68        TD            White      M 19.77    3          30
    ## 17          83        TD            White      M 19.27    3          27
    ## 18          91        TD            White      M 20.03    5          24
    ## 19          95        TD            White      M 21.03    0          26
    ## 20          99        TD            White      M 19.27    0          24
    ## 21         101        TD            White      M 19.23    0          21
    ## 22         103       ASD     White/Latino      M 27.37   14          25
    ## 23         105        TD            White      M 18.97    0          23
    ## 24         109       ASD      White/Asian      M 33.20   11          26
    ## 25         109        TD            White      M 19.10    1          24
    ## 26         111        TD            White      M 19.93    0          20
    ## 27         117       ASD            White      M 34.87   17          26
    ## 28         118        TD            White      F 18.93    0          21
    ## 29         122       ASD African American      M 27.53   21          22
    ## 30         130       ASD            White      F 41.00   18          24
    ## 31         137        TD            White      F 18.30    0          24
    ## 32         139        TD            White      M 19.80    0          28
    ## 33         143        TD            White      M 20.10    5          32
    ## 34         154        TD            White      M 19.97    0          26
    ## 35         166       ASD            White      M 42.00   15          27
    ## 36         176        TD            White      M 19.37    0          24
    ## 37         180       ASD            White      M 33.77   10          27
    ## 38         197       ASD            White      M 34.03    9          34
    ## 39         212        TD            White      F 20.03    0          27
    ## 40         214       ASD            White      M 18.77    9          26
    ## 41         227       ASD            White      M 35.80   11          28
    ## 42         233       ASD            White      M 26.00   15          30
    ## 43         235        TD            White      M 19.23    0          25
    ## 44         244        TD            White      M 20.80    4          29
    ## 45         249        TD            White      M 19.00    0          25
    ## 46         254        TD            White      M 21.67    0          29
    ## 47         260        TD            White      F 23.50    1          29
    ## 48         260        TD            White      M 19.30    1          23
    ## 49         269       ASD     White/Latino      M 31.03    8          31
    ## 50         286        TD            White      M 23.07    0          27
    ## 51         319        TD            White      M 23.13    0          27
    ## 52         319        TD            White      M 22.57    0          29
    ## 53         337       ASD            White      M 39.50    7          33
    ## 54         398        TD            White      F 19.87    1          29
    ## 55         406        TD            White      M 20.77    0          29
    ## 56         433        TD            White      M 21.77    0          29
    ## 57         450       ASD            White      M 37.03   14          42
    ## 58         461       ASD            White      M 28.80   13          34
    ## 59         469       ASD            White      M 34.00   13          30
    ## 60         473        TD            White      M 23.90    0          30
    ## 61         483       ASD            White      M 30.40   11          32
    ## 62           4       ASD            White      M 37.20   NA          NA
    ## 63           7       ASD            White      M 38.33   NA          NA
    ## 64          10       ASD African American      F 29.70   NA          NA
    ## 65          19       ASD            White      M 39.40   NA          NA
    ## 66          32       ASD            White      M 42.20   NA          NA
    ## 67          38       ASD            White      M 41.50   NA          NA
    ## 68          44       ASD            White      F 41.07   NA          NA
    ## 69          55        TD            White      M 23.83   NA          NA
    ## 70          78       ASD African American      M 31.80   NA          NA
    ## 71          82       ASD            White      M 38.73   NA          NA
    ## 72          97        TD            Asian      F 25.40   NA          NA
    ## 73         107        TD            White      M 22.87   NA          NA
    ## 74         116       ASD            White      M 40.50   NA          NA
    ## 75         117        TD            White      M    NA   NA          NA
    ## 76         120        TD            White      M 23.63   NA          NA
    ## 77         126       ASD      White/Asian      M 37.37   NA          NA
    ## 78         132        TD            White      F 23.73   NA          NA
    ## 79         147       ASD            White      M 42.20   NA          NA
    ## 80         148        TD            White      M 23.93   NA          NA
    ## 81         148       ASD         Lebanese      M 32.20   NA          NA
    ## 82         149       ASD      Bangladeshi      F 30.27   NA          NA
    ## 83         157       ASD     White/Latino      M 32.10   NA          NA
    ## 84         159        TD            White      M 24.80   NA          NA
    ## 85         165        TD            White      M 24.68   NA          NA
    ## 86         169       ASD            White      F 49.20   NA          NA
    ## 87         177       ASD            White      M 38.13   NA          NA
    ## 88         203        TD            White      M 23.83   NA          NA
    ## 89         236        TD            White      M 25.23   NA          NA
    ## 90         244        TD            White      M 23.93   NA          NA
    ## 91         258        TD            White      M 23.57   NA          NA
    ## 92         260       ASD            White      M 30.10   NA          NA
    ## 93         264        TD            White      M 23.07   NA          NA
    ## 94         287        TD            White      M 23.17   NA          NA
    ## 95         297        TD            White      M 22.87   NA          NA
    ## 96         298        TD            White      M 26.63   NA          NA
    ## 97         335       ASD            White      M 39.93   NA          NA
    ## 98         356        TD            White      M 27.47   NA          NA
    ## 99         356       ASD            White      M 46.77   NA          NA
    ## 100        362       ASD            White      M 22.50   NA          NA
    ## 101        368       ASD            White      M 38.13   NA          NA
    ## 102        393        TD            White      M 22.97   NA          NA
    ## 103        401        TD            White      M 25.87   NA          NA
    ## 104        414        TD            White      F 22.90   NA          NA
    ## 105        423        TD            White      M    NA   NA          NA
    ## 106        433       ASD            White      M 43.68   NA          NA
    ## 107        439        TD            White      F 24.17   NA          NA
    ## 108        449        TD            White      M 28.60   NA          NA
    ## 109        487       ASD            White      M 37.57   NA          NA
    ## 110        530        TD            White      F 27.83   NA          NA
    ## 111        536       ASD            White      M 42.77   NA          NA
    ## 112        542        TD            White      M 27.70   NA          NA
    ## 113        554        TD            White      M 24.07   NA          NA
    ## 114        555       ASD     White/Latino      M 35.30   NA          NA
    ## 115        562       ASD            White      M 33.17   NA          NA
    ## 116        571        TD            White      F 24.40   NA          NA
    ## 117        623        TD            White      M 25.47   NA          NA
    ## 118        670       ASD            White      M 38.63   NA          NA
    ## 119        733        TD            White      M 26.27   NA          NA
    ## 120        738        TD            White      M 26.13   NA          NA
    ## 121        879        TD            White      M 23.43   NA          NA
    ## 122          1       ASD            White      M 42.13   NA          NA
    ## 123          3       ASD            White      M 45.57   NA          NA
    ## 124         14       ASD            White      M 44.47   NA          NA
    ## 125         26       ASD African American      F 34.10   NA          NA
    ## 126         39       ASD            White      M 43.10   NA          NA
    ## 127         40       ASD            White      M 46.37   NA          NA
    ## 128         41       ASD            White      F 49.97   NA          NA
    ## 129         58       ASD            White      M    NA   NA          NA
    ## 130         62       ASD      Bangledeshi      F 34.57   NA          NA
    ## 131         77       ASD            White      M 46.37   NA          NA
    ## 132         96       ASD African American      M 36.17   NA          NA
    ## 133        122        TD            Asian      F 29.67   NA          NA
    ## 134        148       ASD     White/Latino      M 35.93   NA          NA
    ## 135        151       ASD            White      M 44.67   NA          NA
    ## 136        188        TD            White      M 28.53   NA          NA
    ## 137        193        TD            White      M 27.27   NA          NA
    ## 138        210       ASD            White      F 49.20   NA          NA
    ## 139        218        TD            White      M 26.87   NA          NA
    ## 140        232       ASD      White/Asian      M 41.73   NA          NA
    ## 141        252        TD            White      M 28.10   NA          NA
    ## 142        254       ASD            White      M 50.63   NA          NA
    ## 143        255        TD            White      M 27.70   NA          NA
    ## 144        261        TD            White      M 27.07   NA          NA
    ## 145        299        TD            White      M 29.07   NA          NA
    ## 146        305        TD            White      F 26.67   NA          NA
    ## 147        330       ASD            White      M 43.90   NA          NA
    ## 148        352        TD            White      M 28.47   NA          NA
    ## 149        354        TD            White      M 28.63   NA          NA
    ## 150        365       ASD            White      M 43.00   NA          NA
    ## 151        368        TD            White      M 26.80   NA          NA
    ## 152        393       ASD            White      M 34.27   NA          NA
    ## 153        403        TD            White      M 27.03   NA          NA
    ## 154        404       ASD         Lebanese      M 34.00   NA          NA
    ## 155        424        TD            White      M 29.67   NA          NA
    ## 156        453        TD            White      F 26.63   NA          NA
    ## 157        468       ASD            White      M 41.57   NA          NA
    ## 158        486        TD            White      F 29.53   NA          NA
    ## 159        490       ASD     White/Latino      M 38.90   NA          NA
    ## 160        503       ASD            White      M 26.77   NA          NA
    ## 161        528        TD            White      M 27.67   NA          NA
    ## 162        542        TD            White      F 31.83   NA          NA
    ## 163        604        TD            White      M 28.90   NA          NA
    ## 164        630        TD            White      M 27.07   NA          NA
    ## 165        660        TD            White      M 32.50   NA          NA
    ## 166        671        TD            White      M 31.63   NA          NA
    ## 167        672        TD            White      M 28.03   NA          NA
    ## 168        694       ASD            White      M 47.73   NA          NA
    ## 169        698       ASD            White      M 42.47   NA          NA
    ## 170        702        TD            White      M 27.13   NA          NA
    ## 171        733        TD            White      M 28.27   NA          NA
    ## 172        825        TD            White      M 30.63   NA          NA
    ## 173        825        TD            White      F 28.07   NA          NA
    ## 174        826        TD            White      M 30.13   NA          NA
    ## 175        897        TD            White      M 30.77   NA          NA
    ## 176        940        TD            White      M 30.03   NA          NA
    ## 177        983       ASD            White      M 37.07   NA          NA
    ## 178       1023       ASD            White      M 46.23   NA          NA
    ## 179       1246        TD            White      M 32.07   NA          NA
    ## 180       1293       ASD            White      M 38.60   NA          NA
    ## 181          1       ASD            White      F 53.90   NA          30
    ## 182         14       ASD            White      M 47.10   NA          27
    ## 183         39       ASD            White      M 46.63   NA          27
    ## 184         45       ASD            White      M 43.63   NA          33
    ## 185         55       ASD            White      M 50.23   NA          29
    ## 186         70       ASD            White      M 52.13   NA          22
    ## 187         74        TD            White      M 30.80   NA          36
    ## 188         79       ASD            White      M 48.10   NA          47
    ## 189         93       ASD African American      M 40.27   NA          27
    ## 190        135       ASD      White/Asian      M 45.33   NA          45
    ## 191        142       ASD African American      F 36.93   NA          36
    ## 192        143       ASD            White      M 52.13   NA          29
    ## 193        162       ASD     White/Latino      M 42.23   NA          29
    ## 194        205       ASD      Bangladeshi      F 38.47   NA          33
    ## 195        243        TD            White      M 32.07   NA          42
    ## 196        321        TD            White      M 32.90   NA          33
    ## 197        334       ASD            White      M 54.13   NA          28
    ## 198        348        TD            Asian      F 34.43   NA          30
    ## 199        354        TD            White      M 31.20   NA          35
    ## 200        378        TD            White      M 32.07   NA          40
    ## 201        384        TD            White      M 33.30   NA          42
    ## 202        390        TD            White      F 31.07   NA          45
    ## 203        414        TD            White      M 32.43   NA          31
    ## 204        433        TD            White      M 32.77   NA          26
    ## 205        434        TD            White      M 32.07   NA          41
    ## 206        438       ASD            White      M 46.13   NA          31
    ## 207        455       ASD            White      M 47.83   NA          35
    ## 208        477        TD            White      M 30.40   NA          35
    ## 209        479       ASD     White/Latino      M 43.13   NA          41
    ## 210        493        TD            White      M 32.07   NA          42
    ## 211        493       ASD         Lebanese      M    NA   NA          NA
    ## 212        494       ASD            White      F 52.27   NA          29
    ## 213        495        TD            White      M 31.27   NA          33
    ## 214        502        TD            White      M 31.00   NA          28
    ## 215        510       ASD            White      M 51.67   NA          49
    ## 216        538        TD            White      M 31.03   NA          40
    ## 217        557        TD            White      M 31.47   NA          29
    ## 218        574       ASD            White      M 38.20   NA          40
    ## 219        576        TD            White      F 32.17   NA          44
    ## 220        592        TD            White      F 30.93   NA          40
    ## 221        604       ASD            White      M 45.53   NA          38
    ## 222        637        TD            White      M 33.60   NA          34
    ## 223        642        TD            White      M 35.03   NA          45
    ## 224        674       ASD            White      M 41.07   NA          49
    ## 225        684        TD            White      M 35.63   NA          45
    ## 226        693       ASD            White      M 47.00   NA          47
    ## 227        694        TD            White      M 30.83   NA          39
    ## 228        714       ASD            White      M 42.63   NA          41
    ## 229        717       ASD            White      M 30.80   NA          43
    ## 230        750        TD            White      M 35.03   NA          43
    ## 231        754        TD            White      F 35.53   NA          39
    ## 232        793        TD            White      M 34.27   NA          45
    ## 233        820        TD            White      M 32.07   NA          39
    ## 234        822       ASD            White      M 49.30   NA          50
    ## 235        973        TD            White      F 32.13   NA          42
    ## 236        978        TD            White      M 36.40   NA          45
    ## 237       1051        TD            White      M 32.03   NA          40
    ## 238       1092        TD            White      M 34.43   NA          50
    ## 239       1225        TD            White      M 34.00   NA          47
    ## 240          0       ASD            White      M 57.20   17          NA
    ## 241          5       ASD African American      M 44.70   19          NA
    ## 242          9       ASD            White      M 50.93   15          NA
    ## 243         30       ASD            White      F 58.27   13          NA
    ## 244         32       ASD            White      M 54.33   17          NA
    ## 245         33       ASD            White      M 50.73   20          NA
    ## 246         34       ASD            White      M 57.20   17          NA
    ## 247         42       ASD African American      F 42.97   25          NA
    ## 248        160        TD            White      M 35.90    3          NA
    ## 249        179       ASD            White      M 50.97   20          NA
    ## 250        193       ASD            White      M 52.90   15          NA
    ## 251        194       ASD            White      F 57.73   17          NA
    ## 252        197       ASD            White      M 51.97    9          NA
    ## 253        197       ASD     White/Latino      M 44.93   14          NA
    ## 254        238        TD            Asian      F 37.67    3          NA
    ## 255        255       ASD            White      M 52.77   17          NA
    ## 256        304        TD            White      F 39.10   NA          NA
    ## 257        318        TD            White      M 37.73    0          NA
    ## 258        323        TD            White      M 36.07    5          NA
    ## 259        340       ASD      White/Asian      M 48.63   11          NA
    ## 260        346        TD            White      F 35.00    1          NA
    ## 261        349       ASD      Bangladeshi      F 42.43    8          NA
    ## 262        357       ASD            White      M 51.13   15          NA
    ## 263        404       ASD            White      M 49.30    9          NA
    ## 264        431        TD            White      M 36.53    5          NA
    ## 265        433        TD            White      M 35.83    1          NA
    ## 266        459        TD            White      M 34.40    2          NA
    ## 267        472        TD            White      M 35.90    0          NA
    ## 268        502        TD            White      M 36.70   10          NA
    ## 269        511       ASD            White      M 58.57   15          NA
    ## 270        513        TD            White      M 35.63    0          NA
    ## 271        531       ASD            White      M 42.30   16          NA
    ## 272        537       ASD            White      M 50.30   16          NA
    ## 273        539       ASD     White/Latino      M 47.40    9          NA
    ## 274        583        TD            White      M 38.17    0          NA
    ## 275        586        TD            White      M 39.47    0          NA
    ## 276        588        TD            White      F 39.47    0          NA
    ## 277        622        TD            White      M 36.43    0          NA
    ## 278        636        TD            White      M 37.34    5          NA
    ## 279        651        TD            White      M 35.67    0          NA
    ## 280        686        TD            White      F 35.13    0          NA
    ## 281        690        TD            White      M 39.53    0          NA
    ## 282        723        TD            White      M 38.60    0          NA
    ## 283        725        TD            White      M 35.17    0          NA
    ## 284        736        TD            White      M 34.93    2          NA
    ## 285        755        TD            White      M 35.10    0          NA
    ## 286        769        TD            White      M 38.70    0          NA
    ## 287        792       ASD            White      M 34.13    8          NA
    ## 288        800        TD            White      F 36.13    0          NA
    ## 289        814        TD            White      M 40.07    0          NA
    ## 290        815       ASD            White      M 55.17    4          NA
    ## 291        875        TD            White      M 36.40    0          NA
    ## 292        916        TD            White      M 35.87    0          NA
    ## 293        932        TD            White      M 35.37    0          NA
    ## 294       1054       ASD            White      M 53.40   11          NA
    ## 295       1145        TD            White      M 37.93    0          NA
    ## 296       1154       ASD            White      M 46.93    4          NA
    ## 297          2       ASD African American      M 48.97   NA          26
    ## 298          8       ASD            White      M 62.40   NA          24
    ## 299         36       ASD            White      M    NA   NA          34
    ## 300         37       ASD            White      M 54.43   NA          27
    ## 301         61        TD            White      F 40.37   NA          43
    ## 302         64       ASD            White      F 61.70   NA          32
    ## 303         79       ASD African American      F 46.17   NA          33
    ## 304        100       ASD            White      M 54.63   NA          28
    ## 305        110       ASD      White/Asian      M 53.63   NA          30
    ## 306        137       ASD            White      M 57.43   NA          32
    ## 307        143       ASD            White      M 62.40   NA          30
    ## 308        189        TD            White      M 40.43   NA          33
    ## 309        195       ASD            White      M 56.43   NA          49
    ## 310        197        TD            White      M 40.17   NA          43
    ## 311        204       ASD            White      M 55.17   NA          39
    ## 312        210       ASD            White      M 62.33   NA          41
    ## 313        236       ASD     White/Latino      M 47.50   NA          30
    ## 314        274       ASD            White      M 56.73   NA          30
    ## 315        300       ASD      Bangladeshi      F 46.53   NA          39
    ## 316        306       ASD            White      M    NA   NA          32
    ## 317        358        TD            White      M 44.43   NA          45
    ## 318        395        TD            Asian      F 42.10   NA          46
    ## 319        410        TD            White      M 40.30   NA          40
    ## 320        418        TD            White      M 43.80   NA          45
    ## 321        436        TD            White      M 43.03   NA          47
    ## 322        460        TD            White      F 45.07   NA          45
    ## 323        517        TD            White      M 39.30   NA          41
    ## 324        521        TD            White      M 39.43   NA          39
    ## 325        530        TD            White      M 41.17   NA          43
    ## 326        538       ASD            White      M 54.13   NA          40
    ## 327        568        TD            White      M 38.53   NA          46
    ## 328        571       ASD         Lebanese      M 46.40   NA          41
    ## 329        590       ASD            White      M 37.30   NA          43
    ## 330        605       ASD            White      M 53.77   NA          44
    ## 331        611        TD            White      M 39.07   NA          45
    ## 332        618       ASD            White      M 46.07   NA          45
    ## 333        640        TD            White      M 41.50   NA          44
    ## 334        646        TD            White      M 43.40   NA          45
    ## 335        659        TD            White      M 41.00   NA          NA
    ## 336        666        TD            White      M 40.27   NA          42
    ## 337        686        TD            White      M 40.13   NA          42
    ## 338        698       ASD            White      M 49.70   NA          48
    ## 339        702        TD            White      M 39.93   NA          45
    ## 340        710        TD            White      M 42.93   NA          49
    ## 341        713       ASD            White      M 54.73   NA          50
    ## 342        719        TD            White      M 40.13   NA          46
    ## 343        738       ASD     White/Latino      M 51.37   NA          44
    ## 344        793        TD            White      M 39.43   NA          44
    ## 345        847        TD            White      F 39.23   NA          43
    ## 346        864        TD            White      M 39.40   NA          47
    ## 347        897       ASD            White      M 58.77   NA          50
    ## 348        921       ASD            White      M 57.37   NA          49
    ## 349       1010        TD            White      M 42.47   NA          48
    ## 350       1079        TD            White      M 44.07   NA          50
    ## 351       1249       ASD            White      M 51.00   NA          46
    ## 352       1294        TD            White      M 41.93   NA          45
    ##     verbalIQ Socialization ADOS1 nonVerbalIQ1 verbalIQ1 Socialisation1
    ## 1         10            66    15           28        10             66
    ## 2          9            69    21           21         9             69
    ## 3         13            98     3           19        13             98
    ## 4         13            70    12           31        13             70
    ## 5         11            67    20           13        11             67
    ## 6         17            68    17           20        17             68
    ## 7         11            76    14           25        11             76
    ## 8         11            74    14           25        11             74
    ## 9         14           102     1           22        14            102
    ## 10        10            82    17           28        10             82
    ## 11        10            64    19           17        10             64
    ## 12        11            77    14           27        11             77
    ## 13        13            86    13           27        13             86
    ## 14        16           104     0           30        16            104
    ## 15         9            75    20           21         9             75
    ## 16        20            94     3           30        20             94
    ## 17        18           104     3           27        18            104
    ## 18        20           113     5           24        20            113
    ## 19        18            96     0           26        18             96
    ## 20        15            86     0           24        15             86
    ## 21        15           106     0           21        15            106
    ## 22        19            65    14           25        19             65
    ## 23        17            92     0           23        17             92
    ## 24        19            88    11           26        19             88
    ## 25        22           115     1           24        22            115
    ## 26        16           102     0           20        16            102
    ## 27        14            72    17           26        14             72
    ## 28        19           106     0           21        19            106
    ## 29         8            72    21           22         8             72
    ## 30        14            65    18           24        14             65
    ## 31        18           100     0           24        18            100
    ## 32        14           108     0           28        14            108
    ## 33        31           102     5           32        31            102
    ## 34        17            96     0           26        17             96
    ## 35        16            79    15           27        16             79
    ## 36        19            96     0           24        19             96
    ## 37        22            82    10           27        22             82
    ## 38        27            82     9           34        27             82
    ## 39        20           102     0           27        20            102
    ## 40        14            86     9           26        14             86
    ## 41        20            88    11           28        20             88
    ## 42        24            88    15           30        24             88
    ## 43        17           102     0           25        17            102
    ## 44        22           108     4           29        22            108
    ## 45        17           102     0           25        17            102
    ## 46        26           102     0           29        26            102
    ## 47        18            88     1           29        18             88
    ## 48        21           102     1           23        21            102
    ## 49        27            82     8           31        27             82
    ## 50        27           108     0           27        27            108
    ## 51        22           100     0           27        22            100
    ## 52        22            94     0           29        22             94
    ## 53        26            70     7           33        26             70
    ## 54        28           104     1           29        28            104
    ## 55        33           106     0           29        33            106
    ## 56        22            90     0           29        22             90
    ## 57        27            65    14           42        27             65
    ## 58        27            85    13           34        27             85
    ## 59        30            87    13           30        30             87
    ## 60        30            98     0           30        30             98
    ## 61        33           100    11           32        33            100
    ## 62        NA            66    17           28        10             82
    ## 63        NA            80    21           21         9             69
    ## 64        NA            74    14           25        11             76
    ## 65        NA            65    14           27        11             77
    ## 66        NA            78    20           13        11             67
    ## 67        NA            74    20           21         9             75
    ## 68        NA            66    15           28        10             66
    ## 69        NA           106     3           27        18            104
    ## 70        NA            74    21           22         8             72
    ## 71        NA           104    14           25        11             74
    ## 72        NA           109     1           22        14            102
    ## 73        NA           102     3           19        13             98
    ## 74        NA            79    12           31        13             70
    ## 75        NA           104     0           21        15            106
    ## 76        NA           106     3           30        20             94
    ## 77        NA            72    11           26        19             88
    ## 78        NA           106     0           24        18            100
    ## 79        NA            77    19           17        10             64
    ## 80        NA           110     0           28        14            108
    ## 81        NA            38    13           27        13             86
    ## 82        NA            74    17           20        17             68
    ## 83        NA            65    14           25        19             65
    ## 84        NA            86     0           24        15             86
    ## 85        NA           118     5           24        20            113
    ## 86        NA            63    18           24        14             65
    ## 87        NA            68    17           26        14             72
    ## 88        NA           115     0           30        16            104
    ## 89        NA            98     0           26        18             96
    ## 90        NA            96     0           26        17             96
    ## 91        NA            76     0           20        16            102
    ## 92        NA            72    15           30        24             88
    ## 93        NA            92     0           25        17            102
    ## 94        NA            59     1           24        22            115
    ## 95        NA           102     0           23        17             92
    ## 96        NA            81     0           29        22             94
    ## 97        NA           102    11           28        20             88
    ## 98        NA            70     0           27        27            108
    ## 99        NA            79    15           27        16             79
    ## 100       NA            84     9           26        14             86
    ## 101       NA            74    10           27        22             82
    ## 102       NA           100     0           25        17            102
    ## 103       NA           118     4           29        22            108
    ## 104       NA           102     0           21        19            106
    ## 105       NA           105     0           24        19             96
    ## 106       NA            83     7           33        26             70
    ## 107       NA           116     1           29        28            104
    ## 108       NA            86     0           30        30             98
    ## 109       NA            94     9           34        27             82
    ## 110       NA            89     1           29        18             88
    ## 111       NA            72    14           42        27             65
    ## 112       NA           103     0           27        22            100
    ## 113       NA           102     5           32        31            102
    ## 114       NA           115     8           31        27             82
    ## 115       NA           105    13           34        27             85
    ## 116       NA           100     0           27        20            102
    ## 117       NA           108     0           29        22             90
    ## 118       NA            46    13           30        30             87
    ## 119       NA           102     0           29        26            102
    ## 120       NA           107     0           29        33            106
    ## 121       NA           105     1           23        21            102
    ## 122       NA            68    21           21         9             69
    ## 123       NA            61    20           21         9             75
    ## 124       NA            70    14           27        11             77
    ## 125       NA            74    14           25        11             76
    ## 126       NA            72    17           26        14             72
    ## 127       NA            74    20           13        11             67
    ## 128       NA            65    15           28        10             66
    ## 129       NA            68    14           25        11             74
    ## 130       NA            70    17           20        17             68
    ## 131       NA            72    19           17        10             64
    ## 132       NA            77    21           22         8             72
    ## 133       NA           105     1           22        14            102
    ## 134       NA            70    14           25        19             65
    ## 135       NA            79    12           31        13             70
    ## 136       NA           111     3           27        18            104
    ## 137       NA           102     0           21        15            106
    ## 138       NA            63    18           24        14             65
    ## 139       NA           103     3           19        13             98
    ## 140       NA            63    11           26        19             88
    ## 141       NA            86     0           24        15             86
    ## 142       NA            94    15           27        16             79
    ## 143       NA           109     0           28        14            108
    ## 144       NA           107     1           24        22            115
    ## 145       NA            98     0           26        18             96
    ## 146       NA           114     0           21        19            106
    ## 147       NA            79    11           28        20             88
    ## 148       NA           103     0           25        17            102
    ## 149       NA           112     5           24        20            113
    ## 150       NA            92    10           27        22             82
    ## 151       NA            91     0           23        17             92
    ## 152       NA            86    15           30        24             88
    ## 153       NA           100     0           20        16            102
    ## 154       NA            87    13           27        13             86
    ## 155       NA           116     4           29        22            108
    ## 156       NA           105     0           24        18            100
    ## 157       NA            77     9           34        27             82
    ## 158       NA            98     0           27        20            102
    ## 159       NA            81     8           31        27             82
    ## 160       NA            95     9           26        14             86
    ## 161       NA           100     3           30        20             94
    ## 162       NA            91     1           29        18             88
    ## 163       NA           109     0           24        19             96
    ## 164       NA           109     0           25        17            102
    ## 165       NA           107     0           30        30             98
    ## 166       NA           105     0           27        27            108
    ## 167       NA            91     0           26        17             96
    ## 168       NA           100     7           33        26             70
    ## 169       NA           100    13           30        30             87
    ## 170       NA           112     1           23        21            102
    ## 171       NA           105     0           30        16            104
    ## 172       NA            95     0           29        26            102
    ## 173       NA           125     1           29        28            104
    ## 174       NA           100     0           29        22             90
    ## 175       NA           103     0           29        22             94
    ## 176       NA           112     0           29        33            106
    ## 177       NA            77    13           34        27             85
    ## 178       NA            70    14           42        27             65
    ## 179       NA           107     0           27        22            100
    ## 180       NA            83    11           32        33            100
    ## 181       NA            68    15           28        10             66
    ## 182       NA            72    21           21         9             69
    ## 183       NA            72    17           26        14             72
    ## 184       NA            68    17           28        10             82
    ## 185       NA            55    20           21         9             75
    ## 186       NA            68    19           17        10             64
    ## 187       NA           105     1           23        21            102
    ## 188       NA            74    12           31        13             70
    ## 189       NA            68    21           22         8             72
    ## 190       NA            66    11           26        19             88
    ## 191       NA            86    14           25        11             76
    ## 192       NA            72    20           13        11             67
    ## 193       NA            63    14           25        19             65
    ## 194       NA            66    17           20        17             68
    ## 195       NA           103     0           30        16            104
    ## 196       NA           102     0           28        14            108
    ## 197       NA            92    15           27        16             79
    ## 198       NA           120     1           22        14            102
    ## 199       NA           100     0           21        15            106
    ## 200       NA            95     3           30        20             94
    ## 201       NA           100     0           26        18             96
    ## 202       NA           103     0           24        18            100
    ## 203       NA           103     3           27        18            104
    ## 204       NA           114     5           24        20            113
    ## 205       NA           100     0           25        17            102
    ## 206       NA            92    10           27        22             82
    ## 207       NA            74    11           28        20             88
    ## 208       NA           100     3           19        13             98
    ## 209       NA            77     8           31        27             82
    ## 210       NA           109     5           32        31            102
    ## 211       NA            86    13           27        13             86
    ## 212       NA            61    18           24        14             65
    ## 213       NA            87     0           24        15             86
    ## 214       NA            96     0           20        16            102
    ## 215       NA           101     7           33        26             70
    ## 216       NA           107     0           25        17            102
    ## 217       NA            96     0           23        17             92
    ## 218       NA            83    15           30        24             88
    ## 219       NA           125     1           29        28            104
    ## 220       NA           109     0           21        19            106
    ## 221       NA            77     9           34        27             82
    ## 222       NA           116     4           29        22            108
    ## 223       NA           100     0           29        22             94
    ## 224       NA            75    13           34        27             85
    ## 225       NA           105     0           27        27            108
    ## 226       NA           100    13           30        30             87
    ## 227       NA           105     1           24        22            115
    ## 228       NA            86    11           32        33            100
    ## 229       NA            98     9           26        14             86
    ## 230       NA           107     0           27        22            100
    ## 231       NA            93     1           29        18             88
    ## 232       NA           100     0           29        26            102
    ## 233       NA           109     0           24        19             96
    ## 234       NA            75    14           42        27             65
    ## 235       NA            93     0           27        20            102
    ## 236       NA           108     0           30        30             98
    ## 237       NA            93     0           26        17             96
    ## 238       NA           105     0           29        33            106
    ## 239       NA           102     0           29        22             90
    ## 240       NA            68    19           17        10             64
    ## 241       NA            72    21           22         8             72
    ## 242       NA            66    14           25        11             74
    ## 243       NA            66    15           28        10             66
    ## 244       NA            63    20           21         9             75
    ## 245       NA            66    21           21         9             69
    ## 246       NA            72    20           13        11             67
    ## 247       NA            85    14           25        11             76
    ## 248       NA            86     0           24        15             86
    ## 249       NA            68    17           26        14             72
    ## 250       NA            77    12           31        13             70
    ## 251       NA            63    18           24        14             65
    ## 252       NA            72    11           28        20             88
    ## 253       NA            65    14           25        19             65
    ## 254       NA           112     1           22        14            102
    ## 255       NA            68    17           28        10             82
    ## 256       NA            95     0           27        20            102
    ## 257       NA           112     0           26        18             96
    ## 258       NA           100     5           32        31            102
    ## 259       NA            63    11           26        19             88
    ## 260       NA           107     0           24        18            100
    ## 261       NA            68    17           20        17             68
    ## 262       NA            92    13           30        30             87
    ## 263       NA            75     9           34        27             82
    ## 264       NA           103     3           27        18            104
    ## 265       NA           107     3           30        20             94
    ## 266       NA            91     3           19        13             98
    ## 267       NA           107     0           28        14            108
    ## 268       NA           122     5           24        20            113
    ## 269       NA            95    15           27        16             79
    ## 270       NA           100     0           21        15            106
    ## 271       NA            83    15           30        24             88
    ## 272       NA            90    10           27        22             82
    ## 273       NA            79     8           31        27             82
    ## 274       NA           103     0           29        26            102
    ## 275       NA           101     0           27        27            108
    ## 276       NA            86     1           29        18             88
    ## 277       NA           100     0           20        16            102
    ## 278       NA           116     4           29        22            108
    ## 279       NA           105     0           25        17            102
    ## 280       NA           114     0           21        19            106
    ## 281       NA           106     0           27        22            100
    ## 282       NA            94     0           29        22             94
    ## 283       NA           105     1           24        22            115
    ## 284       NA           103     1           23        21            102
    ## 285       NA            87     0           23        17             92
    ## 286       NA           118     0           29        33            106
    ## 287       NA           107     9           26        14             86
    ## 288       NA           120     1           29        28            104
    ## 289       NA           103     0           30        30             98
    ## 290       NA           105     7           33        26             70
    ## 291       NA           100     0           24        19             96
    ## 292       NA           101     0           30        16            104
    ## 293       NA           100     0           25        17            102
    ## 294       NA            77    14           42        27             65
    ## 295       NA           101     0           29        22             90
    ## 296       NA            97    11           32        33            100
    ## 297       16            65    21           22         8             72
    ## 298       11            66    19           17        10             64
    ## 299       17            NA    14           27        11             77
    ## 300       11            65    14           25        11             74
    ## 301       46           101     0           27        20            102
    ## 302        9            68    15           28        10             66
    ## 303       18            83    14           25        11             76
    ## 304       10            66    21           21         9             69
    ## 305       29            63    11           26        19             88
    ## 306       10            59    20           21         9             75
    ## 307       12            68    20           13        11             67
    ## 308       33           103     3           27        18            104
    ## 309       22            85    12           31        13             70
    ## 310       39           108     5           32        31            102
    ## 311       21            72    17           26        14             72
    ## 312       31           103    15           27        16             79
    ## 313       18            65    14           25        19             65
    ## 314       32            72    11           28        20             88
    ## 315       26            74    17           20        17             68
    ## 316       21            63    17           28        10             82
    ## 317       36           101     0           30        30             98
    ## 318       40           108     1           22        14            102
    ## 319       40            94     0           24        15             86
    ## 320       45           108     0           27        22            100
    ## 321       44            97     0           29        22             94
    ## 322       45            86     1           29        18             88
    ## 323       41           101     1           24        22            115
    ## 324       32            95     0           20        16            102
    ## 325       35           106     0           26        18             96
    ## 326       37            75     9           34        27             82
    ## 327       31            97     3           19        13             98
    ## 328       37            94    13           27        13             86
    ## 329       48           103     9           26        14             86
    ## 330       32            92    10           27        22             82
    ## 331       34            97     1           23        21            102
    ## 332       30            79    15           30        24             88
    ## 333       47            97     0           30        16            104
    ## 334       48           101     0           27        27            108
    ## 335       NA            NA     4           29        22            108
    ## 336       27           101     0           21        15            106
    ## 337       44           100     0           28        14            108
    ## 338       48            81    13           34        27             85
    ## 339       36           101     0           25        17            102
    ## 340       46            97     0           29        26            102
    ## 341       50            97    13           30        30             87
    ## 342       46           101     0           24        19             96
    ## 343       44            74     8           31        27             82
    ## 344       37            83     0           23        17             92
    ## 345       40           110     0           21        19            106
    ## 346       41            92     0           25        17            102
    ## 347       48           116     7           33        26             70
    ## 348       45            77    14           42        27             65
    ## 349       39           103     0           29        22             90
    ## 350       50           116     0           29        33            106
    ## 351       46            90    11           32        33            100
    ## 352       38            95     0           26        17             96

``` r
#kid nr. 57
```

USING SELECT

1.  Make a subset of the data including only kids with ASD, mlu and word tokens
2.  What happens if you include the name of a variable multiple times in a select() call?

``` r
#1:
select_sub1 <- df_merged %>% 
  filter(Diagnosis == "ASD") %>% 
  select(SUBJ, VISIT, CHI_MLU, MOT_MLU, tokens_MOT, tokens_CHI)

#2:
select_sub2 <- df_merged %>% 
  filter(Diagnosis == "ASD") %>% 
  select(SUBJ, SUBJ, VISIT, CHI_MLU, MOT_MLU, tokens_MOT, tokens_CHI)
#nothing happened
```

USING MUTATE, SUMMARISE and PIPES 1. Add a column to the data set that represents the mean number of words spoken during all visits. 2. Use the summarise function and pipes to add an column in the data set containing the mean amount of words produced by each trial across all visits. HINT: group by Child.ID 3. The solution to task above enables us to assess the average amount of words produced by each child. Why don't we just use these average values to describe the language production of the children? What is the advantage of keeping all the data?

``` r
#1: 
mutate1 <- df_merged %>% mutate(meanallvisit = mean(tokens_CHI))

#2:
mutate2 <- df_merged %>% group_by(SUBJ) %>% mutate(meanallvisit = mean(tokens_CHI))

#3:
#The development of the children is probably important, i.e. how they change, and not just how many words they can speak on average. That would only be a very simplified measure. 
```
