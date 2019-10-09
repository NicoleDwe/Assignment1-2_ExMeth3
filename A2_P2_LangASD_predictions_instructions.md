Welcome to the second exciting part of the Language Development in ASD exercise
-------------------------------------------------------------------------------

In this exercise we will delve more in depth with different practices of model comparison and model selection, by first evaluating your models from last time against some new data. Does the model generalize well? Then we will learn to do better by cross-validating models and systematically compare them.

The questions to be answered (in a separate document) are: 1- Discuss the differences in performance of your model in training and testing data 2- Which individual differences should be included in a model that maximizes your ability to explain/predict new data? 3- Predict a new kid's performance (Bernie) and discuss it against expected performance of the two groups

Learning objectives
-------------------

-   Critically appraise the predictive framework (contrasted to the explanatory framework)
-   Learn the basics of machine learning workflows: training/testing, cross-validation, feature selections

Let's go
--------

N.B. There are several datasets for this exercise, so pay attention to which one you are using!

1.  The (training) dataset from last time (the awesome one you produced :-) ).
2.  The (test) datasets on which you can test the models from last time:

-   Demographic and clinical data: <https://www.dropbox.com/s/ra99bdvm6fzay3g/demo_test.csv?dl=1>
-   Utterance Length data: <https://www.dropbox.com/s/uxtqqzl18nwxowq/LU_test.csv?dl=1>
-   Word data: <https://www.dropbox.com/s/1ces4hv8kh0stov/token_test.csv?dl=1>

### Exercise 1) Testing model performance

How did your models from last time perform? In this exercise you have to compare the results on the training data () and on the test data. Report both of them. Compare them. Discuss why they are different.

-   recreate the models you chose last time (just write the model code again and apply it to your training data (from the first assignment))
-   calculate performance of the model on the training data: root mean square error is a good measure. (Tip: google the function rmse())
-   create the test dataset (apply the code from assignment 1 to clean up the 3 test datasets)
-   test the performance of the models on the test data (Tips: google the functions "predict()")
-   optional: predictions are never certain, can you identify the uncertainty of the predictions? (e.g. google predictinterval())

\[HERE GOES YOUR ANSWER\]

### Exercise 2) Model Selection via Cross-validation (N.B: ChildMLU!)

One way to reduce bad surprises when testing a model on new data is to train the model via cross-validation.

In this exercise you have to use cross-validation to calculate the predictive error of your models and use this predictive error to select the best possible model.

-   Use cross-validation to compare your model from last week with the basic model (Child MLU as a function of Time and Diagnosis, and don't forget the random effects!)
-   (Tips): google the function "createFolds"; loop through each fold, train both models on the other folds and test them on the fold)

-   Now try to find the best possible predictive model of ChildMLU, that is, the one that produces the best cross-validated results.

-   Bonus Question 1: What is the effect of changing the number of folds? Can you plot RMSE as a function of number of folds?
-   Bonus Question 2: compare the cross-validated predictive error against the actual predictive error on the test data

``` r
#CV: split data into parts (often 5), take only one at a time as test data, train the model on the rest and then test it on the test set, repeat with every slice, gives 5 rmse, of which we can take the mean 
#can use it to choose among the models we are testing, look at sd and see how reliable it is (if slices are very different or not, can also mean that data is very different)

#R2 doesn't tell you that it's overfitting 
#Anova log likelihood, only telling you is the increase big enough but not are we being careful 
#AIC and BIC, only approximations of what we are doing 

#### PREP ####
#Kenneth function for convergenece issues
lmerControl(optCtrl=list(xtol_abs=1e-8, ftol_abs=1e-8))
```

    ## $optimizer
    ## [1] "nloptwrap"
    ## 
    ## $restart_edge
    ## [1] TRUE
    ## 
    ## $boundary.tol
    ## [1] 1e-05
    ## 
    ## $calc.derivs
    ## [1] TRUE
    ## 
    ## $use.last.params
    ## [1] FALSE
    ## 
    ## $checkControl
    ## $checkControl$check.nobs.vs.rankZ
    ## [1] "ignore"
    ## 
    ## $checkControl$check.nobs.vs.nlev
    ## [1] "stop"
    ## 
    ## $checkControl$check.nlev.gtreq.5
    ## [1] "ignore"
    ## 
    ## $checkControl$check.nlev.gtr.1
    ## [1] "stop"
    ## 
    ## $checkControl$check.nobs.vs.nRE
    ## [1] "stop"
    ## 
    ## $checkControl$check.rankX
    ## [1] "message+drop.cols"
    ## 
    ## $checkControl$check.scaleX
    ## [1] "warning"
    ## 
    ## $checkControl$check.formula.LHS
    ## [1] "stop"
    ## 
    ## 
    ## $checkConv
    ## $checkConv$check.conv.grad
    ## $checkConv$check.conv.grad$action
    ## [1] "warning"
    ## 
    ## $checkConv$check.conv.grad$tol
    ## [1] 0.002
    ## 
    ## $checkConv$check.conv.grad$relTol
    ## NULL
    ## 
    ## 
    ## $checkConv$check.conv.singular
    ## $checkConv$check.conv.singular$action
    ## [1] "message"
    ## 
    ## $checkConv$check.conv.singular$tol
    ## [1] 1e-04
    ## 
    ## 
    ## $checkConv$check.conv.hess
    ## $checkConv$check.conv.hess$action
    ## [1] "warning"
    ## 
    ## $checkConv$check.conv.hess$tol
    ## [1] 1e-06
    ## 
    ## 
    ## 
    ## $optCtrl
    ## $optCtrl$xtol_abs
    ## [1] 1e-08
    ## 
    ## $optCtrl$ftol_abs
    ## [1] 1e-08
    ## 
    ## 
    ## attr(,"class")
    ## [1] "lmerControl" "merControl"

``` r
#### CV BASIC MODEL ####
#Create the basic model of ChildMLU as a function of Time and Diagnosis (don't forget the random effects!).
basic_model <- lmer(CHI_MLU ~ Visit + Diagnosis + (1|Child.ID) + (0+Visit|Child.ID), data = train_data)

#Make a cross-validated version of the model. (Tips: google the function "createFolds";  loop through each fold, train a model on the other folds and test it on the fold)
#fold data 
fold_data <- groupdata2::fold(train_data, k = 5,
             cat_col = 'Diagnosis',
             id_col = 'Child.ID') %>% 
             arrange(.folds) %>%
             mutate(CHI_MLU = scale(CHI_MLU))

#show first 15 rows of data
fold_data %>% head(15) %>% kable()
```

|  Child.ID|  Visit| Ethnicity        | Diagnosis | Gender |    Age|  ADOS|  MullenRaw|  ExpressiveLangRaw|  Socialization|  MOT\_MLU|    CHI\_MLU|  types\_MOT|  types\_CHI|  tokens\_MOT|  tokens\_CHI|  Ados1|  verbalIQ1|  nonVerbalIQ1|  Socialization1| .folds |
|---------:|------:|:-----------------|:----------|:-------|------:|-----:|----------:|------------------:|--------------:|---------:|-----------:|-----------:|-----------:|------------:|------------:|------:|----------:|-------------:|---------------:|:-------|
|         3|      1| White            | ASD       | M      |  28.80|    13|         34|                 27|             85|  4.098446|   0.0548776|         317|         146|         1428|          461|     13|         27|            34|              85| 1      |
|         3|      2| White            | ASD       | M      |  33.17|    NA|         NA|                 NA|            105|  4.964664|   1.2002172|         307|         171|         1270|          562|     13|         27|            34|              85| 1      |
|         3|      3| White            | ASD       | M      |  37.07|    NA|         NA|                 NA|             77|  4.147059|   0.8752469|         351|         262|         1445|          983|     13|         27|            34|              85| 1      |
|         3|      4| White            | ASD       | M      |  41.07|    NA|         49|                 NA|             75|  5.309804|   2.0272356|         335|         200|         1286|          674|     13|         27|            34|              85| 1      |
|         3|      6| White            | ASD       | M      |  49.70|    NA|         48|                 48|             81|  4.588477|   1.1617173|         304|         245|          999|          698|     13|         27|            34|              85| 1      |
|        23|      1| African American | ASD       | M      |  27.53|    21|         22|                  8|             72|  4.390879|  -1.1885024|         485|           8|         2826|          122|     21|          8|            22|              72| 1      |
|        23|      2| African American | ASD       | M      |  31.80|    NA|         NA|                 NA|             74|  3.381510|  -1.1495512|         426|           9|         2524|           78|     21|          8|            22|              72| 1      |
|        23|      3| African American | ASD       | M      |  36.17|    NA|         NA|                 NA|             77|  3.646778|  -0.2147226|         434|          13|         2764|           96|     21|          8|            22|              72| 1      |
|        23|      4| African American | ASD       | M      |  40.27|    NA|         27|                 NA|             68|  3.600624|  -1.8967059|         427|          10|         2283|           93|     21|          8|            22|              72| 1      |
|        23|      5| African American | ASD       | M      |  44.70|    19|         NA|                 NA|             72|  4.019231|  -1.9675262|         373|           2|         1963|            5|     21|          8|            22|              72| 1      |
|        23|      6| African American | ASD       | M      |  48.97|    NA|         26|                 16|             65|  3.943636|  -1.6753923|         388|           2|         2077|            2|     21|          8|            22|              72| 1      |
|        29|      1| White            | ASD       | M      |  30.40|    11|         32|                 33|            100|  4.690751|   1.1485692|         278|         119|         1450|          483|     11|         33|            32|             100| 1      |
|        29|      3| White            | ASD       | M      |  38.60|    NA|         NA|                 NA|             83|  4.316279|   1.6546320|         333|         307|         1668|         1293|     11|         33|            32|             100| 1      |
|        29|      4| White            | ASD       | M      |  42.63|    NA|         41|                 NA|             86|  4.857143|   1.2691324|         398|         188|         2518|          714|     11|         33|            32|             100| 1      |
|        29|      5| White            | ASD       | M      |  46.93|     4|         NA|                 NA|             97|  4.345515|   1.0433909|         437|         261|         2410|         1154|     11|         33|            32|             100| 1      |

``` r
#cross-validate single model
CV1 <- cross_validate(fold_data, "CHI_MLU ~ Diagnosis + Visit + (1|Child.ID) + (0+Visit|Child.ID)",
                      fold_cols = '.folds',
                      family = 'gaussian',
                        control = lmerControl(
                          optimizer = "nloptwrap",
                          calc.dervis = FALSE, 
                          optCtrl = list(
                            xtol_abs = 1e-10,
                            xtol_abs = 1e-10,
                            maxeval = 1000000),
                      rm_nc = F,
                      REML = FALSE))
```

    ## Registered S3 method overwritten by 'MuMIn':
    ##   method         from
    ##   predict.merMod lme4

``` r
CV1 %>% select_metrics() %>% kable()
```

|       RMSE|        MAE|       r2m|        r2c|       AIC|      AICc|      BIC| Dependent | Fixed           | Random                          |
|----------:|----------:|---------:|----------:|---------:|---------:|--------:|:----------|:----------------|:--------------------------------|
|  0.8873951|  0.6894149|  0.191653|  0.8068263|  531.5109|  531.8169|  553.353| CHI\_MLU  | Diagnosis+Visit | (1|Child.ID)+(0+Visit|Child.ID) |

``` r
tableCV1 <- CV1[1:7]
tableCV1 <- round(tableCV1, digits = 3)
write.csv(tableCV1, "tableCV1.csv", row.names = F) #write to csv
  

#### CV MULTIPLE MODELS ####
#had to change random effect becasue it would not converge
mixed_models <- c("CHI_MLU ~ Diagnosis + (1 | Child.ID)",
                  "CHI_MLU ~ Visit + (1| Child.ID)",
                  "CHI_MLU ~ Visit + Diagnosis + (1 | Child.ID)", 
                  "CHI_MLU ~ Visit*Diagnosis + (1 | Child.ID)",
                  "CHI_MLU ~ Visit*Diagnosis + verbalIQ1 + (1 | Child.ID)",
                  "CHI_MLU ~ Visit*Diagnosis + verbalIQ1*MOT_MLU + (1 | Child.ID)",
                  "CHI_MLU ~ Visit*Diagnosis + verbalIQ1 + (1 | Child.ID)",
                  "CHI_MLU ~ Visit*Diagnosis + Ados1 + (1 | Child.ID)",
                  "CHI_MLU ~ Visit*Diagnosis + MOT_MLU + (1 | Child.ID)",
                  "CHI_MLU ~ Visit*Diagnosis + nonVerbalIQ1 +  (1 | Child.ID)",
                  "CHI_MLU ~ Visit*Diagnosis + nonVerbalIQ1 + verbalIQ1 + (1 | Child.ID)",
                  "CHI_MLU ~ Visit*Diagnosis + nonVerbalIQ1 + Ados1 + (1 | Child.ID)")
                  #"CHI_MLU ~ Diagnosis * Visit * verbalIQ1 + MOT_MLU * verbalIQ1 + (1 | Child.ID)", 
                  #"CHI_MLU ~ Diagnosis * Visit + nonVerbalIQ1 + verbalIQ1  + MOT_MLU + (1|Child.ID) ",
                  #"CHI_MLU ~ Diagnosis * Visit * Ados1 + MOT_MLU * Visit  + (1 | Child.ID)", 
                  #"CHI_MLU ~ Diagnosis * Visit * Ados1 * verbalIQ1  + (1 | Child.ID)",
                  #"CHI_MLU ~ Diagnosis * Visit  + MOT_MLU * verbalIQ1 + (1 | Child.ID)", 
                  #"CHI_MLU ~ Diagnosis * Visit * verbalIQ1 + MOT_MLU * Visit  + (1 | Child.ID)",
                  #"CHI_MLU ~ Diagnosis * Visit * Socialization1 + MOT_MLU * verbalIQ1  + (1 | Child.ID)") 



CV_mixed <- cross_validate(fold_data, mixed_models,
                      fold_cols = '.folds',
                      family = 'gaussian',
                        control = lmerControl(
                          optimizer = "nloptwrap",
                          calc.dervis = FALSE, 
                          optCtrl = list(
                            xtol_abs = 1e-10,
                            xtol_abs = 1e-10,
                            maxeval = 1000000),
                      rm_nc = F,
                      REML = FALSE))

CV_mixed %>% select_metrics() %>% kable()
```

|       RMSE|        MAE|        r2m|        r2c|       AIC|      AICc|       BIC| Dependent | Fixed                                   | Random       |
|----------:|----------:|----------:|----------:|---------:|---------:|---------:|:----------|:----------------------------------------|:-------------|
|  0.9296078|  0.7688195|  0.1358667|  0.4570165|  702.3296|  702.4743|  716.8910| CHI\_MLU  | Diagnosis                               | (1|Child.ID) |
|  0.8950692|  0.6960438|  0.1939836|  0.6875323|  586.9969|  587.1415|  601.5582| CHI\_MLU  | Visit                                   | (1|Child.ID) |
|  0.8193048|  0.6417097|  0.3305971|  0.6869941|  574.5983|  574.8160|  592.8000| CHI\_MLU  | Visit+Diagnosis                         | (1|Child.ID) |
|  0.7853815|  0.6170700|  0.3867774|  0.7536208|  520.7682|  521.0742|  542.6103| CHI\_MLU  | Visit\*Diagnosis                        | (1|Child.ID) |
|  0.6211464|  0.4731147|  0.6336463|  0.7530394|  477.8996|  478.3091|  503.3821| CHI\_MLU  | Visit\*Diagnosis+verbalIQ1              | (1|Child.ID) |
|  0.6026058|  0.4504894|  0.6805595|  0.7950305|  439.9168|  440.5797|  472.6799| CHI\_MLU  | Visit*Diagnosis+verbalIQ1*MOT\_MLU      | (1|Child.ID) |
|  0.6211464|  0.4731147|  0.6336463|  0.7530394|  477.8996|  478.3091|  503.3821| CHI\_MLU  | Visit\*Diagnosis+verbalIQ1              | (1|Child.ID) |
|  0.7235399|  0.5604602|  0.5145107|  0.7553572|  505.0623|  505.4718|  530.5447| CHI\_MLU  | Visit\*Diagnosis+Ados1                  | (1|Child.ID) |
|  0.7272820|  0.5700245|  0.4695141|  0.7804251|  481.0808|  481.4902|  506.5632| CHI\_MLU  | Visit\*Diagnosis+MOT\_MLU               | (1|Child.ID) |
|  0.7039449|  0.5414403|  0.5121039|  0.7541829|  505.1446|  505.5541|  530.6270| CHI\_MLU  | Visit\*Diagnosis+nonVerbalIQ1           | (1|Child.ID) |
|  0.6287375|  0.4805170|  0.6355580|  0.7531556|  479.3703|  479.8987|  508.4931| CHI\_MLU  | Visit\*Diagnosis+nonVerbalIQ1+verbalIQ1 | (1|Child.ID) |
|  0.6888562|  0.5244937|  0.5569976|  0.7554867|  499.2303|  499.7587|  528.3531| CHI\_MLU  | Visit\*Diagnosis+nonVerbalIQ1+Ados1     | (1|Child.ID) |

``` r
tableCV_mixed <- CV_mixed[,1:7]
tableCV_mixed <- round(tableCV_mixed, digits = 3)
write.csv(tableCV_mixed, "tableCVmix.csv", row.names = T)

#test 3 best models on test data? 

#- Report the results and comment on them. 
#- Now try to find the best possible predictive model of ChildMLU, that is, the one that produces the best cross-validated results.

# Bonus Question 1: What is the effect of changing the number of folds? Can you plot RMSE as a function of number of folds?
# Bonus Question 2: compare the cross-validated predictive error against the actual predictive error on the test data
```

\[HERE GOES YOUR ANSWER\]

### Exercise 3) Assessing the single child

Let's get to business. This new kiddo - Bernie - has entered your clinic. This child has to be assessed according to his group's average and his expected development.

Bernie is one of the six kids in the test dataset, so make sure to extract that child alone for the following analysis.

You want to evaluate:

-   how does the child fare in ChildMLU compared to the average TD child at each visit? Define the distance in terms of absolute difference between this Child and the average TD.

-   how does the child fare compared to the model predictions at Visit 6? Is the child below or above expectations? (tip: use the predict() function on Bernie's data only and compare the prediction with the actual performance of the child)

``` r
#get only bernies data
Bernie <- filter(test_data, Child.ID == "2")

TD_skills <- subset(train_data, Diagnosis == "TD") %>% group_by(Visit) %>% summarise(AverageTD = mean(CHI_MLU))
Bernie_skills <- subset(Bernie) %>% group_by(Visit) %>% summarize (Bernie = CHI_MLU)

#combine the 2 dataframes from above and add a column with the difference in total numbers
superior_Bernie <- merge(Bernie_skills, TD_skills)
superior_Bernie$Difference <- superior_Bernie$Bernie - superior_Bernie$AverageTD

#create model
THEmodel <- lmer(CHI_MLU ~ Diagnosis*Visit + verbalIQ1*MOT_MLU + (1|Child.ID), data = train_data)

#predict the values for each visit for Bernie
predictbernie <- predict(THEmodel, Bernie, allow.new.levels = TRUE)
#add predictions to dataframe
predict <- as.data.frame(predictbernie) 
predict$Visit <- c(1:6)
superior_Bernie$Predicted <- predictbernie
superior_Bernie <- round(superior_Bernie, digits = 2)
write.csv(superior_Bernie, "Bernie.csv") #get csv of all values

#ggplot: bernie, other kids, predictions
#plot: bernie vs the other TD kids 
td_kids_all <- filter(train_data, Diagnosis=="TD")
td_kids_all$Visit <- as.factor(td_kids_all$Visit)

ggplot()+
  geom_point(data=td_kids_all, aes(x=Visit, y=CHI_MLU, group=Visit), color = "black")+
  geom_point(data=Bernie, aes(x=Visit, y=CHI_MLU), colour = "blue")+
  geom_point(data=predict, aes(x = Visit, y=predictbernie), colour = "orange")+
  labs(x="Number of visit", y="Child MLU", title="Child MLU: Bernie vs the other TD children and Bernie's predicitions")+
  theme(legend.position = "right")
```

![](A2_P2_LangASD_predictions_instructions_files/figure-markdown_github/unnamed-chunk-3-1.png)

\[HERE GOES YOUR ANSWER\]

### OPTIONAL: Exercise 4) Model Selection via Information Criteria

Another way to reduce the bad surprises when testing a model on new data is to pay close attention to the relative information criteria between the models you are comparing. Let's learn how to do that!

Re-create a selection of possible models explaining ChildMLU (the ones you tested for exercise 2, but now trained on the full dataset and not cross-validated).

Then try to find the best possible predictive model of ChildMLU, that is, the one that produces the lowest information criterion.

-   Bonus question for the optional exercise: are information criteria correlated with cross-validated RMSE? That is, if you take AIC for Model 1, Model 2 and Model 3, do they co-vary with their cross-validated RMSE?

### OPTIONAL: Exercise 5): Using Lasso for model selection

Welcome to the last secret exercise. If you have already solved the previous exercises, and still there's not enough for you, you can expand your expertise by learning about penalizations. Check out this tutorial: <http://machinelearningmastery.com/penalized-regression-in-r/> and make sure to google what penalization is, with a focus on L1 and L2-norms. Then try them on your data!
