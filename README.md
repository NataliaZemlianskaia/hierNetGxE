# gesso

The package is developed to fit a regularized regression model that we call **gesso** for the joint selection of gene-environment (GxE) interactions. The model focuses on a single environmental exposure and induces a "main-effect-before-interaction" hierarchical structure. We developed and implemented an efficient fitting algorithm and screening rules that can discard large numbers of irrelevant predictors with high accuracy.

<img src="./man/figures/gesso_model.png" width="652" height="150.4">

**gesso** model induces hierarchical selection of the (GxE) interaction terms via overlapped group lasso structure. The model has two tuning parameters λ1 and λ2 allowing flexible, data dependent control over group sparsity and additional interactions sparsity.

## Introduction
`gesso` package can be used

 * for the selection of the gene-environment interaction terms in a joint "main-effect-before-interaction" hierarchical manner
 * for building a joint *well-formlated* prediction model containing interaction terms with a specific exposure of interest, where the final prediction model only includes interaction terms for which their respective main effects are also included in the model
 
The package supports sparse matrices `dgCMatrix` and (filebacked) bigmatrix format from the `bigmemory` package for large or out of RAM datasets. 

For more information and examples please see package vignette.

## Installation
```R
## install.packages("devtools")

library(devtools)
devtools::install_github("NataliaZemlianskaia/gesso")
```
## Example
```R
library(gesso)

## generate the data: 1,000 main effects and 1,000 interaction effects 
## with 15 non-zero main effects and 10 non-zero interaction effects, sample size equal to 200
data = data.gen(sample_size=200, p=1000, 
                n_g_non_zero=15, n_gxe_non_zero=10, 
                family="gaussian", mode="strong_hierarchical")

## tune the model over a 2D grid of hyperparameters   
tune_model = gesso.cv(data$G_train, data$E_train, data$Y_train, 
                           grid_size=20, tolerance=1e-4,
                           parallel=TRUE, nfold=3,
                           normalize=TRUE,
                           seed=1)

## obtain interaction and main effect coefficietns corresponding to the best model  
coefficients = gesso.coef(fit=tune_model$fit, lambda=tune_model$lambda_min)
gxe_coefficients = coefficients$beta_gxe                      
g_coefficients = coefficients$beta_g    

## check if all non-zero features were recovered by the model
cbind(data$Beta_GxE[data$Beta_GxE != 0], gxe_coefficients[data$Beta_GxE != 0])

## calculate principal selection metrics
selection = selection.metrics(true_b_g=data$Beta_G, true_b_gxe=data$Beta_GxE, 
                              estimated_b_g=g_coefficients, estimated_b_gxe=gxe_coefficients)
cbind(selection)

##                selection
## b_g_non_zero    109      
## b_gxe_non_zero  43       
## mse_beta        0.3624545
## mse_beta_GxE    0.884622 
## sensitivity_g   1        
## specificity_g   0.9045685
## sensitivity_gxe 0.9      
## specificity_gxe 0.9656566
## precision_g     0.1376147
## precision_gxe   0.2093023
## auc_gxe         0.9232314
## auc_g           0.9999993
```
