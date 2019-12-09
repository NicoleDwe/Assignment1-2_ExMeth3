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

``` r
#load packages
pacman::p_load(readr,dplyr,stringr,lmerTest,Metrics,caret,cvms, groupdata2, knitr, gridExtra)

## Clean up function, included to inspire you
CleanUpData <- function(Demo,LU,Word){
  
  Speech <- merge(LU, Word) %>% 
    rename(
      Child.ID = SUBJ, 
      Visit=VISIT) %>%
    mutate(
      Visit = as.numeric(str_extract(Visit, "\\d")),
      Child.ID = gsub("\\.","", Child.ID)
      ) %>%
    dplyr::select(
      Child.ID, Visit, MOT_MLU, CHI_MLU, types_MOT, types_CHI, tokens_MOT, tokens_CHI
    )
  
  Demo <- Demo %>%
    dplyr::select(
      Child.ID, Visit, Ethnicity, Diagnosis, Gender, Age, ADOS, MullenRaw, ExpressiveLangRaw, Socialization
    ) %>%
    mutate(
      Child.ID = gsub("\\.","", Child.ID)
    )
    
  Data=merge(Demo,Speech,all=T)
  
  Data1= Data %>% 
     subset(Visit=="1") %>% 
     dplyr::select(Child.ID, ADOS, ExpressiveLangRaw, MullenRaw, Socialization) %>%
     rename(Ados1 = ADOS, 
            verbalIQ1 = ExpressiveLangRaw, 
            nonVerbalIQ1 = MullenRaw,
            Socialization1 = Socialization) 
  
  Data=merge(Data, Data1, all=T) %>%
    mutate(
      Child.ID = as.numeric(as.factor(as.character(Child.ID))),
      Visit = as.numeric(as.character(Visit)),
      Gender = recode(Gender, 
         "1" = "M",
         "2" = "F"),
      Diagnosis = recode(Diagnosis,
         "A"  = "ASD",
         "B"  = "TD")
    )

  return(Data)
}

#### LOAD DATA ####
#train data
lu_train <- read.csv("LU_train.csv")
demo_train <- read.csv("demo_train.csv")
token_train <- read.csv("token_train.csv")
train_data1 <- CleanUpData(Demo = demo_train, LU = lu_train, Word = token_train)
train_data <- subset(train_data1, !is.na(CHI_MLU))

#test data
lu_test <- read.csv("LU_test.csv")
demo_test <- read.csv("demo_test.csv")
token_test <- read.csv("token_test.csv")
test_data1 <- CleanUpData(Demo = demo_test, LU = lu_test, Word = token_test)
test_data <- subset(test_data1, !is.na(CHI_MLU))

#### TEST PERFORMANCE OF MODEL ####
#use model from last time and apply to training data
train_model <- lmer(CHI_MLU ~ Visit*Diagnosis + verbalIQ1*MOT_MLU + (1|Child.ID) + (0+Visit|Child.ID), data = train_data)

#calculate perfomance of model on training data (root mean square error)
train_predict <- predict(train_model, train_data) #predict data from trained model and data
rmse(train_data$CHI_MLU, train_predict) #0.342, calculate differences and take the mean = average error
```

    ## [1] 0.3420218

``` r
#calculate performance of model on test data, 
test_predict <- predict(train_model, test_data, allow.new.levels = TRUE)
rmse(test_data$CHI_MLU, test_predict) #0.620, the others get 0.54, so we went with that one
```

    ## [1] 0.6205632

``` r
#rmse: if we only know the mena, how big of an error do we make? (sd), if rmse is smaller, that's better at least
#- optional: predictions are never certain, can you identify the uncertainty of the predictions? (e.g. google predictinterval())
```

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
#lmerControl(optCtrl=list(xtol_abs=1e-8, ftol_abs=1e-8))

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
|        21|      1| White            | ASD       | M      |  35.80|    11|         28|                 20|             88|  2.539823|  -0.6274463|         283|          89|         1019|          227|     11|         20|            28|              88| 1      |
|        21|      2| White            | ASD       | M      |  39.93|    NA|         NA|                 NA|            102|  3.170000|  -0.4998440|         315|          86|         1316|          335|     11|         20|            28|              88| 1      |
|        21|      3| White            | ASD       | M      |  43.90|    NA|         NA|                 NA|             79|  3.615176|  -0.0799004|         298|          88|         1159|          330|     11|         20|            28|              88| 1      |
|        21|      4| White            | ASD       | M      |  47.83|    NA|         35|                 NA|             74|  2.762463|  -0.3681886|         248|          87|          855|          455|     11|         20|            28|              88| 1      |
|        21|      5| White            | ASD       | M      |  51.97|     9|         NA|                 NA|             72|  4.157191|  -0.4022428|         328|          45|         1142|          197|     11|         20|            28|              88| 1      |
|        21|      6| White            | ASD       | M      |  56.73|    NA|         30|                 32|             72|  3.156695|   0.1606802|         256|          55|          995|          274|     11|         20|            28|              88| 1      |
|        29|      1| White            | ASD       | M      |  30.40|    11|         32|                 33|            100|  4.690751|   1.4198582|         278|         119|         1450|          483|     11|         33|            32|             100| 1      |
|        29|      3| White            | ASD       | M      |  38.60|    NA|         NA|                 NA|             83|  4.316279|   1.9412932|         333|         307|         1668|         1293|     11|         33|            32|             100| 1      |
|        29|      4| White            | ASD       | M      |  42.63|    NA|         41|                 NA|             86|  4.857143|   1.5440836|         398|         188|         2518|          714|     11|         33|            32|             100| 1      |
|        29|      5| White            | ASD       | M      |  46.93|     4|         NA|                 NA|             97|  4.345515|   1.3114850|         437|         261|         2410|         1154|     11|         33|            32|             100| 1      |
|        29|      6| White            | ASD       | M      |  51.00|    NA|         46|                 46|             90|  4.111413|   1.3840795|         452|         273|         3076|         1249|     11|         33|            32|             100| 1      |
|        36|      1| African American | ASD       | F      |  25.33|    14|         25|                 11|             76|  2.287293|  -0.7373649|         206|          13|          788|           35|     14|         11|            25|              76| 1      |
|        36|      2| African American | ASD       | F      |  29.70|    NA|         NA|                 NA|             74|  2.879925|  -0.9882048|         285|           2|         1393|           10|     14|         11|            25|              76| 1      |
|        36|      3| African American | ASD       | F      |  34.10|    NA|         NA|                 NA|             74|  3.404959|  -0.9045915|         272|           3|         1504|           26|     14|         11|            25|              76| 1      |
|        36|      4| African American | ASD       | F      |  36.93|    NA|         36|                 NA|             86|  3.110933|  -0.7774150|         281|          18|         1726|          142|     14|         11|            25|              76| 1      |

``` r
#cross-validate single model
CV1 <- cross_validate(fold_data, "CHI_MLU ~ Diagnosis + Visit + (1|Child.ID) + (0+Visit|Child.ID)",
                      fold_cols = '.folds',
                      family = 'gaussian',
                        rm_nc = F,
                        REML = FALSE)
```

    ## Registered S3 method overwritten by 'MuMIn':
    ##   method         from
    ##   predict.merMod lme4

``` r
#control = lmerControl(optimizer = "nloptwrap",calc.dervis = FALSE, optCtrl = list(xtol_abs = 1e-10,xtol_abs = 1e-10,maxeval = 1000000),
                      
CV1 %>% select_metrics() %>% kable()
```

|       RMSE|        MAE|        r2m|        r2c|       AIC|      AICc|       BIC| Dependent | Fixed           | Random                          |
|----------:|----------:|----------:|----------:|---------:|---------:|---------:|:----------|:----------------|:--------------------------------|
|  0.8945133|  0.7006942|  0.1821403|  0.8133701|  522.2534|  522.5594|  544.0958| CHI\_MLU  | Diagnosis+Visit | (1|Child.ID)+(0+Visit|Child.ID) |

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
                      rm_nc = F,
                      REML = FALSE) 

CV_mixed %>% select_metrics() %>% kable()
```

|       RMSE|        MAE|        r2m|        r2c|       AIC|      AICc|       BIC| Dependent | Fixed                                   | Random       |
|----------:|----------:|----------:|----------:|---------:|---------:|---------:|:----------|:----------------------------------------|:-------------|
|  0.9293728|  0.7701577|  0.1300318|  0.4866323|  692.8961|  693.0407|  707.4576| CHI\_MLU  | Diagnosis                               | (1|Child.ID) |
|  0.9004808|  0.7076423|  0.1820124|  0.7043010|  576.8448|  576.9894|  591.4063| CHI\_MLU  | Visit                                   | (1|Child.ID) |
|  0.8271279|  0.6473646|  0.3124565|  0.7036148|  566.0123|  566.2301|  584.2143| CHI\_MLU  | Visit+Diagnosis                         | (1|Child.ID) |
|  0.7940748|  0.6262550|  0.3671054|  0.7682930|  510.5840|  510.8900|  532.4264| CHI\_MLU  | Visit\*Diagnosis                        | (1|Child.ID) |
|  0.6428924|  0.4928835|  0.6109134|  0.7674824|  473.3376|  473.7471|  498.8204| CHI\_MLU  | Visit\*Diagnosis+verbalIQ1              | (1|Child.ID) |
|  0.6088961|  0.4627065|  0.6651125|  0.8030425|  434.5089|  435.1718|  467.2724| CHI\_MLU  | Visit*Diagnosis+verbalIQ1*MOT\_MLU      | (1|Child.ID) |
|  0.6428924|  0.4928835|  0.6109134|  0.7674824|  473.3376|  473.7471|  498.8204| CHI\_MLU  | Visit\*Diagnosis+verbalIQ1              | (1|Child.ID) |
|  0.7229781|  0.5679806|  0.4863880|  0.7698174|  497.6202|  498.0296|  523.1029| CHI\_MLU  | Visit\*Diagnosis+Ados1                  | (1|Child.ID) |
|  0.7318631|  0.5726894|  0.4487894|  0.7872055|  473.4737|  473.8831|  498.9564| CHI\_MLU  | Visit\*Diagnosis+MOT\_MLU               | (1|Child.ID) |
|  0.7101439|  0.5556875|  0.5065738|  0.7688111|  494.1251|  494.5345|  519.6078| CHI\_MLU  | Visit\*Diagnosis+nonVerbalIQ1           | (1|Child.ID) |
|  0.6443610|  0.4952499|  0.6146813|  0.7677227|  474.4486|  474.9769|  503.5717| CHI\_MLU  | Visit\*Diagnosis+nonVerbalIQ1+verbalIQ1 | (1|Child.ID) |
|  0.6854133|  0.5339729|  0.5418509|  0.7698802|  490.3338|  490.8622|  519.4570| CHI\_MLU  | Visit\*Diagnosis+nonVerbalIQ1+Ados1     | (1|Child.ID) |

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

TD_skills <- subset(train_data, Diagnosis == "TD") %>% group_by(Visit) %>% summarise(Mean_TD = mean(CHI_MLU), SD_TD = sd(CHI_MLU))
Bernie_skills <- subset(Bernie) %>% group_by(Visit) %>% summarize (Bernie = CHI_MLU)

#combine the 2 dataframes from above and add a column with the difference in total numbers
superior_Bernie <- merge(Bernie_skills, TD_skills)
superior_Bernie$Difference <- superior_Bernie$Bernie - superior_Bernie$Mean_TD

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
