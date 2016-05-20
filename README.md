---
title: 'Project: Estimating Risk of Heart Failure'
author: "Suraj Joshi"
date: "May 19, 2016"
output: html_document
---

###Title: Estimating Risk of Heart Failure - Data Simulation

###Reference: 
Kannel WB, D’Agostino RB, Silbershatz H, Belanger AJ, Wilson PWF, Levy D. Profile for estimating risk of heart failure. Archives of Internal Medicine. 1999;159(11):1197–1204. doi:10.1001/archinte.159.11.1197. http://archinte.jamanetwork.com/article.aspx?articleid=485049. Accessed May 19, 2016.


###Background:
Heart Failure is the disease of elderly. It is present in 1% of the population between 50-59 yrs old and 10%  between 80-89 yrs old.
 
Heart failure is a progressive, often terminal stage of cardiac disease that, when symptomatic, curtails survival like many types of cancer. In Framingham Study subjects with overt congestive heart failure, the median survival rate was only 1.7 years for men and 3.2 years for women.

Heart fails dues to multiple causes. Most common are Coronary artery disease and Myocardial infraction. Others causes can be Hypertension, Myocarditis, Aoric stenosis, Mitral stenosis, tricuspid stenosis, regurgitation of mitral or aortic valve, Constrictive pericarditis, Restrictive Cardiomyopathy, Left ventricular hypertrophy, fibrosis, cardiac tamponade, arrythmias, congenital heart diseases etc. Some non cardiac diseases also contribute to the worsening of heart diseases like diabetes mellitus and pulmonary hypertension.

The study considered some conditions which can be easily assessed in the outpatient setting and come up with a risk score for having heart disease in 4 yrs time.  Some of the factors considered were: 
-Age
-Left Ventricular Hypertrophy (LVH) using EKG
-Cardiomegaly (ratio >.5)
-Heart Rate (HR)
-Systolic BP (SBP)
-Diabetes Mellitus (DM)
-Valvular Heart Disease (VHD) presence of murmur > 3/6 and
-Congenital Heart Disease (CHD)

The study on which the referenced paper is based on is the Framingham Study which is a prospective, epidemiological, population-based study designed to investigate the prevalence, incidence, and determinants of cardiovascular disease.


###Objective:
-To come up with the heart failure risk score and predict the occurance of heart failue in a patient using histry and examination in the clinic and simple investigative tools like EKG and Chest X-ray?

-Enable the general practitioners and internists to estimate the risk of developing heart failure in predisposed persons with coronary disease, hypertension, or valvular heart disease—the most common causes of the condition.


###Methodology:

It is a known fact that heart failure increase by age.  Similarly it's contributing factors like hypertension, diabetes mellitus, valvular heart disease, left ventricular hypertrophy etc also increase by age. Here, most features are linearly and positively correlated with age. And each feature also is positively correlated with risk score.

Attempts were made to make the data mimic population prevalence of the conditions and definitely more effort will improve the outcome.
A sample of 8000 observations were generated mimicking the study. The study provided mean values but not the standard deviations.
-Age was generated using random normal distribution with mean age of 63 and standard deviation of 10 yrs.
-LVH, Cardiomegaly, Diabetes Mellitus, VHD and CHD  were generated using random uniform distribution value and logistic equation.
-Heart rate and systolic blood pressure were measured using random normal distribution using mean of 75.8 and 149.5 respectively from the study.
-Few outliers were added to age, heart rate and systolic blood pressure.

The study developed a point system to calculate the risk but to make the calculation simpler I decided to use the weightage multipliers system to mimic the data.


```{r}
N<- 500
generateDataset <- function (N) {
  set.seed(1000)
  Age <- round(rnorm(N, 63, 10))
  par(mfrow=c(2,2))
  hist(Age, 10)
  
  LVH <- runif(N)< .25*logistic((Age-60)/10) # # Left Ventricular Hypertrophy on EKG # boolean 0 = none 1 = present
  (sum(LVH==TRUE)/N)*100 #  ~ 15 % in population to find the prevalence in the given age group

  Cardiomegaly <- runif(N)< .08*logistic((Age-60)/10) # Xray # boolean 0 or 1
  (sum(Cardiomegaly==TRUE)/N)*100

  HR <- round(rnorm(N, 75.8,10)+Age*.05) # heart rate # high if > 120 bpm numerical
  hist(HR)
  
  SBP<- round(rnorm(N,149.5, 10)-70+(Age+Age*.1+5)) # systolic blood pressure >140 mmHg numerical
  hist(SBP)

  DM <- runif(N)< .17*logistic((Age-60)/10) # diabetes mellitus or in insulin or oral hypoglycemics boolean 0 or 
  (sum(DM==TRUE)/N)*100
  
  VHD <- runif(N)< .2*logistic((Age-45)/10)  # valvular heart disease # boolean  0 or 1
  # Epidemiology of acquired valvular heart disease. http://www.ncbi.nlm.nih.gov/pubmed/24986049
  (sum(VHD==TRUE)/N)*100
  
  CHD <- runif(N)< .06*logistic((Age-60)/10) # congenital heart disease # boolean 0 or 1
  (sum(CHD==TRUE)/N)*100
  
  RiskScore <- 10*(Age*.025+LVH*.01+Cardiomegaly*.0001+HR*.0002+SBP*.005+.07*DM+VHD*.01+CHD*.03)
  hist(RiskScore)
  
  # Outliers
  a = sample(1:500, 10)
  Age[a] <- ifelse(Age[a] > 75, Age[a]+10, Age[a])
  HR[1] <- max(HR) + 20
  SBP[1] <- max(SBP) + 20
  
  data.frame(Age,LVH, Cardiomegaly, HR, SBP, DM, VHD, CHD, RiskScore)
}
```


###Analysis:
Data Exploration:
To confirm we have the assumed data distribution, mean and reasonable standard deviation, we polt a histogram of the quantitative measures.

Summary of the dataset
```{r}
summary(dataset)
```

![Distribution] (https://github.com/spa380/DataSimulation616/blob/master/data_dist.png)

The summary of the dataset and the plots confirm the distributions of the data we aimed for.

Next, we look at the correlation among the variables and we find that age is positively correlated with all variables but more profoundly with systolic blood pressure and diabetes mellitus and as expected to the risk score.
```{r}
cor(dataset)

```


We would also want to verify the linearity between the age and few features related to age eg. SBP and diabetes.

```{r}
par(mfrow=c(1,2))
fit.AgeSBP <- lm(SBP~Age, data = dataset)
summary(fit.RiskScore)
plot(fit.RiskScore,1)
plot(fit.RiskScore,2)
```
We see in the figure below that the data is normally distributed  with some outliers.

![Distribution] (https://github.com/spa380/DataSimulation616/blob/master/Age_SBP.png)

We can see the positive co-linearity between Age and SBP in the figure below :
```{r}
ggplot(dataset, aes(x=Age, y = SBP))+
  geom_point()+
  geom_vline(xintercept = mean(dataset$Age))+
  geom_hline(yintercept = mean(dataset$SBP))
```
![Distribution] (https://github.com/spa380/DataSimulation616/blob/master/AgeSBP_Linearity.png)

We can also see from the figure below that about 60% of the diabetics are above the mean age of sample of 63yrs.
```{r}
ggplot(dataset, aes(x=Age, y = DM))+
  geom_point()+
  geom_vline(xintercept = mean(dataset$Age))+
  geom_hline(yintercept = mean(dataset$DM))

```
![Distribution] (https://github.com/spa380/DataSimulation616/blob/master/DiabetesAge.png)








You can also embed plots, for example:

```{r, echo=FALSE}
plot(cars)
```

Note that the `echo = FALSE` parameter was added to the code chunk to prevent printing of the R code that generated the plot.
