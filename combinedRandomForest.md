Referencing the relevant packages dplyr and ggvis was used for data
wrangling / data exploration caret/randomForest was used for building
the prediction model doParallel was used to speed up the model building
process setting the seed

    library(caret)
    library(dplyr)
    library(ggvis)
    library(randomForest)
    library(doParallel)
    set.seed(123)

the training and validation datasets were saved to the R working
directory in advance

    training_all <- read.csv("training.csv")
    validation <- read.csv("validation.csv")

removing the first seven columns of both the training and the validation
dataset as they mostly contain redundant timestamp data points

    training_all <- training_all %>% select(-c(1:7))
    validation <- validation %>% select(-c(1:7)) %>% select(-problem_id) %>% mutate(classe = as.factor("A"))

using a quick for loop so that we can faster identify columns containing
NA values

    for(i in 1:(ncol(training_all)-1)) {training_all[,i] <- as.numeric(as.character(training_all[,i]))}
    for(i in 1:(ncol(validation)-1)) {validation[,i] <- as.numeric(as.character(validation[,i]))}

marking NA-free columns

    column_names <- colnames(training_all[colSums(is.na(training_all)) == 0])

adjusting the data set to contain only the NA-free columns

    training_all <- training_all[column_names]
    validation <- validation[column_names]

splitting the training\_all dataset into training/testing sets for cross
validation using the caret package

    inTrain <- createDataPartition(training_all$classe, p = 0.7, list = FALSE)

    training <- training_all[inTrain,]
    testing <- training_all[-inTrain,]

due to the great number of variables and as a rule of thumb, staring
with random forest \#pred &lt;- train(form = class~., method = "rf",
data = training) using parallelization to speed up the script

    registerDoParallel()
    x <- training[-ncol(training)]
    y <- training$classe

    mod <- foreach(ntree=rep(150, 6), .combine=randomForest::combine, .packages='randomForest') %dopar% {
      randomForest(x, y, ntree=ntree) 
    }

model is showing high accuracy for both the training and testing dataset
will stick to the randomForest solution

    pred1 <- predict(mod, newdata = training)
    confusionMatrix(pred1,training$classe)

    pred2 <- predict(mod, newdata=testing)
    confusionMatrix(pred2,testing$classe)

predicting classe values for the 20 cases in the validation data set and
pipelining it to View() for easier submission

    pred3 <- predict(mod, newdata = validation)
    pred3 %>% View()
