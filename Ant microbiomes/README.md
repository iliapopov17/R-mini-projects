# Download the data

**_Input_**
```
git clone https://github.com/iliapopov17/R-mini-projects.git && cd Ant microbiomes
```

- `Ant microbes.Rmd` - contains detailed pipeline of this project (the same as you're reading in README)
- `Ant microbes.html` - contains the same pipeline in `html` format
- `data` - folder with the data used

Then open RStudio (or VS Code if you're mad...)

**_Input_**

```r libraries, message=FALSE, warning=FALSE
library(ggplot2)
library(dplyr)
library(stats)
library(psych)
library(readr)
library(tidyr)
library(ggpubr)
library(purrr)
```

**_Input_**

```r
main_dir <- dirname(rstudioapi::getSourceEditorContext()$path) 
setwd(main_dir)
```

# **Part 1 - pH of the ant goitre**

Questions to be answered:<br>

1. Does the acidity of the goitre depend on feeding?<br>
2. At what point in time is the pH lowest?<br>
3. Does the acidity differ between ants from different colonies?<br>

**_Input_**

```r
ant_data_pH <- read.csv("data/Supplementary_Data_Fig.1a.txt", sep="\t")
```

**_Input_**

```r
summary(ant_data_pH)
```

**_Output_**

```
   species             colony              worker     
 Length:231         Length:231         Min.   : 1.00  
 Class :character   Class :character   1st Qu.:15.00  
 Mode  :character   Mode  :character   Median :29.00  
                                       Mean   :29.47  
                                       3rd Qu.:44.00  
                                       Max.   :60.00  
   workerall         time                 ph       
 Min.   :  1.0   Length:231         Min.   :1.500  
 1st Qu.: 58.5   Class :character   1st Qu.:2.500  
 Median :116.0   Mode  :character   Median :3.500  
 Mean   :116.0                      Mean   :3.355  
 3rd Qu.:173.5                      3rd Qu.:4.000  
 Max.   :231.0                      Max.   :5.000  
```

**_Input_**

```r
describe(ant_data_pH)
```

**_Output_**

<div style='justify-content: center'>
<img src="https://github.com/iliapopov17/R-mini-projects/blob/main/Ant%20microbiomes/imgs/describe1.png" align='center', width="100%">
</div>

**_Input_**

```r
str(ant_data_pH)
```

**_Output_**

```
'data.frame':	231 obs. of  6 variables:
 $ species  : chr  "c.floridanus" "c.floridanus" "c.floridanus" "c.floridanus" ...
 $ colony   : chr  "C62" "C62" "C62" "C62" ...
 $ worker   : int  1 2 3 4 5 6 7 8 9 10 ...
 $ workerall: int  1 2 3 4 5 6 7 8 9 10 ...
 $ time     : chr  "0+4h" "0+4h" "0+4h" "0+4h" ...
 $ ph       : num  4 4.5 4 4 3.5 4 3 4.5 4.5 4.5 ...
```

The dataset contains 231 entries for the _Camponotus floridanus_ ants, detailing their `colony`, `worker` ID, `time` of pH measurement, and the `pH` values themselves. Here are a few observations and next steps:<br>

**Species Column**: All entries belong to the species _c.floridanus_, so we don't need to account for different species in our analysis.<br>
**Time and pH**: The dataset records the pH of the goitre at different time points after feeding (0+4h, 0+24h, 0+48h, 48+4h). We'll compare these to see if feeding time influences pH.<br>
**pH Values**: pH ranges from 1.5 to 5.0. The mean pH is approximately 3.35, suggesting a generally acidic environment, which is expected given that formic acid is a primary component of the gland secretion mentioned.<br>

## **Visualisation**

Let's start with visualizing the pH distribution overall and at each specified time to see if there's an apparent trend or shift related to feeding times. We'll use histograms and box plots for these visualizations.

**_Input_**

```r
ggplot(ant_data_pH, aes(x = ph, fill = time)) +
  geom_histogram(aes(y = ..density..), binwidth = 0.1, alpha = 0.6, position = "identity") +
  geom_density(alpha = 0.5) +
  facet_wrap(~time) +
  labs(title = "Density and Distribution of pH over Time")
```

**_Output_**

<div style='justify-content: center'>
<img src="https://github.com/iliapopov17/R-mini-projects/blob/main/Ant%20microbiomes/imgs/density%20and%20distribution%20of%20ph%20over%20time.png" align='center', width="100%">
</div>

**_Input_**

```r
ggplot(ant_data_pH, aes(x = time, y = ph)) +
  geom_boxplot() +
  labs(title = "Boxplot of pH Values by Time")
```

**_Output_**

<div style='justify-content: center'>
<img src="https://github.com/iliapopov17/R-mini-projects/blob/main/Ant%20microbiomes/imgs/boxplot%20of%20ph%20values%20by%20time.png" align='center', width="100%">
</div>

### **Visualisation results**
**Density Plot**: Shows variations in the pH distribution at each time point. We can observe shifts in pH concentration and spread, indicating possible changes due to feeding.<br>
**Boxplot**: Highlights median values and spread of pH at each time point. Notably, the interquartile range and median values vary, suggesting significant changes in pH levels related to the time post-feeding.<br>

## **Statistical Analysis**

**ANOVA for Time Dependency**: We'll conduct an ANOVA to test if there are statistically significant differences in pH levels at different times.<br>
**ANOVA for Colony Differences**: Another ANOVA will help determine if pH varies significantly across different ant colonies.<br>

**_Input_**

```r
anova_time <- aov(ph ~ time, data = ant_data_pH)
summary(anova_time)

anova_colony <- aov(ph ~ colony, data = ant_data_pH)
summary(anova_colony)
```

**_Output_**

```
             Df Sum Sq Mean Sq F value Pr(>F)    
time          3 166.34   55.45   216.8 <2e-16 ***
Residuals   227  58.05    0.26                   
---
Signif. codes:  
0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
             Df Sum Sq Mean Sq F value Pr(>F)
colony        5   0.74  0.1486    0.15   0.98
Residuals   225 223.65  0.9940               
```

### **Statistical Analysis Results**

**ANOVA for Time Dependency**:<br>
The ANOVA results show a highly significant _F_ statistic ( _F_ =216.81 ), with a _p_-value close to zero ( _p_ < 0.001 ). This strongly suggests that the pH levels vary significantly across different times, confirming the impact of feeding time on the goitre's acidity.<br>
**ANOVA for Colony Differences**:<br>
The ANOVA comparing different colonies shows a non-significant result ( _p_ = 0.980 ). This implies that the differences in pH levels between colonies are not statistically significant, suggesting that colony does not influence the acidity of the goitre.<br>

# **Part 2 - Immobilisation of ants after feeding**

Questions to be answered:<br>

1. Does the acidity of the goitre depend on the immobilisation of the ant?<br>
2. Does goitre acidity differ between immobilised ants from different colonies?<br>

**_Input_**

```r
ant_data_immobilisation <- read.csv("data/Supplementary_Data_Fig.1b.txt", sep="\t")
```

**_Input_**

```r
summary(ant_data_immobilisation)
```

**_Output_**

```
   species              worker       treatment        
 Length:45          Min.   : 1.00   Length:45         
 Class :character   1st Qu.: 6.00   Class :character  
 Mode  :character   Median :12.00   Mode  :character  
                    Mean   :11.76                     
                    3rd Qu.:17.00                     
                    Max.   :23.00                     
       ph         colony         
 Min.   :2.0   Length:45         
 1st Qu.:2.5   Class :character  
 Median :3.0   Mode  :character  
 Mean   :3.2                     
 3rd Qu.:4.0                     
 Max.   :5.0                     
```

**_Input_**

```r
describe(ant_data_immobilisation)
```

**_Output_**

<div style='justify-content: center'>
<img src="https://github.com/iliapopov17/R-mini-projects/blob/main/Ant%20microbiomes/imgs/describe2.png" align='center', width="100%">
</div>

**_Input_**

```r
str(ant_data_immobilisation)
```

**_Output_**

```
'data.frame':	45 obs. of  5 variables:
 $ species  : chr  "c.floridanus" "c.floridanus" "c.floridanus" "c.floridanus" ...
 $ worker   : int  1 2 3 4 5 6 7 8 9 10 ...
 $ treatment: chr  "FA+" "FA+" "FA+" "FA+" ...
 $ ph       : num  2.5 3 3 3 3 2.5 2.5 4 2 2 ...
 $ colony   : chr  "C219" "C219" "C219" "C219" ...
```

The dataset includes measurements of pH in the goitre of _Camponotus floridanus_ ants across different treatments and colonies. Here are some key observations:<br>

**Treatment**: The treatment column, labeled as `treatment`, likely indicates whether ants were immobilized (FA+ suggests presence of formic acid immobilization treatment, if other categories exist we'll check them).<br>
**Columns**: Includes `species`, `worker` ID, `treatment`, `pH`, and `colony.`<br>

Let's first verify the categories under `treatment` to understand the groups we are comparing, then visualize and analyze the data accordingly.<br>

**_Input_**

```r
unique(ant_data_immobilisation['treatment'])
```

**_Output_**

```
   treatment
1  FA+
24 FA-
```

The `treatment` column has two categories:<br>

**FA+**: Ants that were immobilized.<br>
**FA-**: Ants that were not immobilized.<br>

## **Visualisation**

Let's start by visualizing the pH distribution for immobilized vs. non-immobilized ants and then proceed with the statistical analysis.<br>

**_Input_**

```r
ggplot(ant_data_immobilisation, aes(x = ph, fill = treatment)) +
  geom_histogram(aes(y = ..density..), binwidth = 0.1, alpha = 0.6, position = "identity") +
  geom_density(alpha = 0.5) +
  facet_wrap(~treatment) +
  labs(title = "Density and Distribution of pH by Treatment")
```

**_Output_**

<div style='justify-content: center'>
<img src="https://github.com/iliapopov17/R-mini-projects/blob/main/Ant%20microbiomes/imgs/density%20and%20distribution%20of%20ph%20by%20treatment.png" align='center', width="100%">
</div>

**_Input_**

```r
ggplot(ant_data_immobilisation, aes(x = treatment, y = ph)) +
  geom_boxplot() +
  labs(title = "Boxplot of pH Values by Treatment")
```

**_Output_**

<div style='justify-content: center'>
<img src="https://github.com/iliapopov17/R-mini-projects/blob/main/Ant%20microbiomes/imgs/boxplot%20of%20ph%20values%20by%20treatment.png" align='center', width="100%">
</div>

### **Visualization Results**
**Density Plot**: Shows distinct distributions of pH values for immobilized (FA+) and non-immobilized (FA-) ants, suggesting differences in acidity levels depending on the treatment.<br>
**Boxplot**: Highlights the central tendency and variability of pH values for each treatment group. The plot indicates that the pH may be slightly higher on average in non-immobilized ants.<br>

## **Statistical Analysis**
To substantiate the observations from the visualizations, we'll perform:

**T-test** for independent samples to test if the mean pH values significantly differ between the FA+ and FA- groups.<br>
**ANOVA** to assess pH differences between colonies within immobilized ants (FA+).<br>

**_Input_**

```r
fa_plus <- filter(ant_data_immobilisation, treatment == "FA+")
fa_minus <- filter(ant_data_immobilisation, treatment == "FA-")
t_test_results <- t.test(fa_plus$ph, fa_minus$ph, var.equal = FALSE)
```

**_Input_**

```r
anova_colonies_fa_plus <- aov(ph ~ colony, data = fa_plus)
```

**_Input_**

```r
print(t_test_results)
print(summary(anova_colonies_fa_plus))
```

**_Output_**

```

	Welch Two Sample t-test

data:  fa_plus$ph and fa_minus$ph
t = -8.603, df = 42.759, p-value = 7.07e-11
alternative hypothesis: true difference in means is not equal to 0
95 percent confidence interval:
 -1.712624 -1.062079
sample estimates:
mean of x mean of y 
 2.521739  3.909091 

            Df Sum Sq Mean Sq F value Pr(>F)
colony       3  1.164  0.3880   1.453  0.259
Residuals   19  5.075  0.2671               
```

### **Statistical Analysis Results**

**T-test between FA+ (immobilized) and FA- (non-immobilized)**:<br>
The t-test result shows a highly significant difference in pH values between immobilized and non-immobilized ants ( _p_ ≈ 7.07×10<sup>-11</sup>), with a large negative t-statistic suggesting lower pH in immobilized ants.<br>
**ANOVA for pH differences among colonies (FA+ only)**:<br>
The ANOVA results indicate no significant differences in pH levels among different colonies for immobilized ants ( _p_ = 0.259), suggesting that colony variation does not significantly impact pH within the immobilized group.<br>

# **Part 3 - Do they pass?**

Question to be answered:<br>
1. Do _Asaia_ and _Serratia_ bacteria pass from the goitre to the midgut?

**_Input_**

```r
ant_data_pass <- read.csv("data/Supplementary_Data_Fig.5ab.txt", sep = "\t")
```

**_Input_**

```r
summary(ant_data_pass)
```

**_Output_**

```
   bacteria          gutregion              time      
 Length:653         Length:653         Min.   : 0.00  
 Class :character   Class :character   1st Qu.: 0.50  
 Mode  :character   Mode  :character   Median : 4.00  
                                       Mean   :10.99  
                                       3rd Qu.:24.00  
                                       Max.   :48.00  
      cfu           colony         
 Min.   :    0   Length:653        
 1st Qu.:    0   Class :character  
 Median :    0   Mode  :character  
 Mean   : 1416                     
 3rd Qu.:  145                     
 Max.   :50000                     
```

**_Input_**

```r
describe(ant_data_pass)
```

**_Output_**

<div style='justify-content: center'>
<img src="https://github.com/iliapopov17/R-mini-projects/blob/main/Ant%20microbiomes/imgs/describe3.png" align='center', width="100%">
</div>

**_Input_**

```r
str(ant_data_pass)
```

**_Output_**

```
'data.frame':	653 obs. of  5 variables:
 $ bacteria : chr  "asaia" "asaia" "asaia" "asaia" ...
 $ gutregion: chr  "crop" "crop" "crop" "crop" ...
 $ time     : num  0 0 0 0 0 0 0 0 0 0 ...
 $ cfu      : int  10000 20000 20000 40000 3700 9930 30000 40000 10000 10000 ...
 $ colony   : chr  "C313" "C313" "C313" "C313" ...
```

The dataset consists of 653 entries with the following structure:<br>

**bacteria**: Name of the bacteria (e.g., Asaia, Serratia).<br>
**gutregion**: Part of the gut where the sample was taken (e.g., crop, midgut).<br>
**time**: Timepoint of the sample collection in hours.<br>
**cfu**: Colony-forming units counted.<br>
**colony**: Colony identifier.<br>

## **Visualisation**

Let's start with visualizing the CFU distribution in two different gut regions.

**_Input_**

```r
ggplot(ant_data_pass, aes(x = cfu)) +
  geom_histogram(bins = 30, fill = "skyblue", color = "black") +
  facet_grid(bacteria ~ gutregion, scales = "free_y") +
  theme_minimal() +
  labs(title = "Distribution of CFU Counts",
       x = "CFU",
       y = "Frequency") +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))
```

**_Output_**

<div style='justify-content: center'>
<img src="https://github.com/iliapopov17/R-mini-projects/blob/main/Ant%20microbiomes/imgs/distribution%20of%20CFU%20counts.png" align='center', width="100%">
</div>

### **Histogram Analysis**
Here are the histograms showing the distribution of colony-forming units (CFU) for both Asaia and Serratia bacteria in different gut regions:<br>

**Asaia in Crop**: The distribution shows a broad range of values with some peaks, indicating variability in the colony counts at different sampling times.<br>
**Asaia in Midgut**: The counts are significantly lower, mostly near zero, suggesting fewer bacteria pass into or survive in the midgut.<br>
**Serratia in Crop**: Similar to Asaia, Serratia shows a varied distribution in the crop but generally at lower counts compared to Asaia.<br>
**Serratia in Midgut**: The counts are very low, similar to Asaia, indicating minimal presence in the midgut.<br>

**_Input_**

```r
ggplot(ant_data_pass, aes(x = interaction(time, gutregion), y = cfu, fill = gutregion)) +
  geom_boxplot() +
  facet_wrap(~bacteria, scales = "free_x") +
  scale_y_log10() +
  labs(title = "CFU Distribution by Bacteria Type Across Time and Gut Regions",
       x = "Time and Gut Region",
       y = "Colony Forming Units (CFU, log scale)",
       fill = "Gut Region") +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 45, hjust = 1),
        strip.background = element_blank(),
        strip.text.x = element_text(size = 12, face = "bold"))
```

**_Output_**

<div style='justify-content: center'>
<img src="https://github.com/iliapopov17/R-mini-projects/blob/main/Ant%20microbiomes/imgs/CFU%20distribution%20by%20bacteria%20type%20actoss%20time%20and%20gut%20regions.png" align='center', width="100%">
</div>

### **Boxplot Analysis**
The boxplots display the distribution of colony-forming units (CFU) for both Asaia and Serratia bacteria in different gut regions across various times, with the y-axis on a logarithmic scale for better visualization of the spread of data:<br>

**Asaia in Crop over Time**: There is a noticeable decrease in the median and range of CFU over time, suggesting a decline in bacterial population in the crop as time progresses.<br>
**Asaia in Midgut over Time**: The CFU counts remain consistently low across all time points, with some slight increases but still considerably lower than in the crop.<br>
**Serratia in Crop over Time**: Similar to Asaia, there's a clear decrease in CFU over time, indicating a reduction in bacterial presence in the crop.<br>
**Serratia in Midgut over Time**: The counts are generally low, comparable to those of Asaia, and remain low over time, though there are some outliers or variations at specific times.<br>

## **Statistical Analysis**

Statistics will be done using Mann-Whitney U Test, because:<br>
**Non-Parametric Test**: Mann-Whitney U test does not require the data to follow a normal distribution. This is particularly useful since the distribution of CFU counts, as seen in the histograms, does not clearly follow a normal distribution.<br>
**Comparative Analysis**: Our goal is to compare bacterial counts between two independent samples (crop vs. midgut) at various time points. The Mann-Whitney U test is designed to assess whether there is a statistically significant difference in the medians between two independent samples, which is exactly our situation.<br>
**Appropriate for Ordinal and Continuous Data**: The Mann-Whitney U test is suitable for continuous data (like CFU counts) and ordinal data. It compares the ranks of the data in the two groups, rather than their actual values, providing a robust measure of differences that is less sensitive to outliers.<br>

**_Input_**

```r
#Asaia
# I know this solution is not the best one.
# This code can (must) be vectorized, yet all my efforts resulted in wrong results.
# However, this simple code provides correct results. So what's the difference so far!
crop_data_1 <- filter(ant_data_pass, bacteria == "asaia", gutregion == "crop", time == 0)$cfu
midgut_data_1 <- filter(ant_data_pass, bacteria == "asaia", gutregion == "midgut", time == 0)$cfu
crop_data_2 <- filter(ant_data_pass, bacteria == "asaia", gutregion == "crop", time == 0.5)$cfu
midgut_data_2 <- filter(ant_data_pass, bacteria == "asaia", gutregion == "midgut", time == 0.5)$cfu
crop_data_3 <- filter(ant_data_pass, bacteria == "asaia", gutregion == "crop", time == 4.0)$cfu
midgut_data_3 <- filter(ant_data_pass, bacteria == "asaia", gutregion == "midgut", time == 4.0)$cfu
crop_data_4 <- filter(ant_data_pass, bacteria == "asaia", gutregion == "crop", time == 24.0)$cfu
midgut_data_4 <- filter(ant_data_pass, bacteria == "asaia", gutregion == "midgut", time == 24.0)$cfu
crop_data_5 <- filter(ant_data_pass, bacteria == "asaia", gutregion == "crop", time == 48.0)$cfu
midgut_data_5 <- filter(ant_data_pass, bacteria == "asaia", gutregion == "midgut", time == 48.0)$cfu
```

**_Input_**

```r
wilcox.test(crop_data_1, midgut_data_1, exact = FALSE)
wilcox.test(crop_data_2, midgut_data_2, exact = FALSE)
wilcox.test(crop_data_3, midgut_data_3, exact = FALSE)
wilcox.test(crop_data_4, midgut_data_4, exact = FALSE)
wilcox.test(crop_data_5, midgut_data_5, exact = FALSE)
```

**_Output_**

```

	Wilcoxon rank sum test with continuity
	correction

data:  crop_data_1 and midgut_data_1
W = 225, p-value = 6.624e-07
alternative hypothesis: true location shift is not equal to 0


	Wilcoxon rank sum test with continuity
	correction

data:  crop_data_2 and midgut_data_2
W = 225, p-value = 2.082e-06
alternative hypothesis: true location shift is not equal to 0


	Wilcoxon rank sum test with continuity
	correction

data:  crop_data_3 and midgut_data_3
W = 225, p-value = 3.058e-06
alternative hypothesis: true location shift is not equal to 0


	Wilcoxon rank sum test with continuity
	correction

data:  crop_data_4 and midgut_data_4
W = 184, p-value = 0.003223
alternative hypothesis: true location shift is not equal to 0


	Wilcoxon rank sum test with continuity
	correction

data:  crop_data_5 and midgut_data_5
W = 34.5, p-value = 0.001056
alternative hypothesis: true location shift is not equal to 0
```

**_Input_**

```r
#Serratia
# I know this solution is not the best one.
# This code can (must) be vectorized, yet all my efforts resulted in wrong results.
# However, this simple code provides correct results. So what's the difference so far!
crop_data_6 <- filter(ant_data_pass, bacteria == "serratia", gutregion == "crop", time == 0)$cfu
midgut_data_6 <- filter(ant_data_pass, bacteria == "serratia", gutregion == "midgut", time == 0)$cfu
crop_data_7 <- filter(ant_data_pass, bacteria == "serratia", gutregion == "crop", time == 0.5)$cfu
midgut_data_7 <- filter(ant_data_pass, bacteria == "serratia", gutregion == "midgut", time == 0.5)$cfu
crop_data_8 <- filter(ant_data_pass, bacteria == "serratia", gutregion == "crop", time == 4.0)$cfu
midgut_data_8 <- filter(ant_data_pass, bacteria == "serratia", gutregion == "midgut", time == 4.0)$cfu
crop_data_9 <- filter(ant_data_pass, bacteria == "serratia", gutregion == "crop", time == 24.0)$cfu
midgut_data_9 <- filter(ant_data_pass, bacteria == "serratia", gutregion == "midgut", time == 24.0)$cfu
crop_data_10 <- filter(ant_data_pass, bacteria == "serratia", gutregion == "crop", time == 48.0)$cfu
midgut_data_10 <- filter(ant_data_pass, bacteria == "serratia", gutregion == "midgut", time == 48.0)$cfu
```

**_Input_**

```r
wilcox.test(crop_data_6, midgut_data_6, exact = FALSE)
wilcox.test(crop_data_7, midgut_data_7, exact = FALSE)
wilcox.test(crop_data_8, midgut_data_8, exact = FALSE)
wilcox.test(crop_data_9, midgut_data_9, exact = FALSE)
wilcox.test(crop_data_10, midgut_data_10, exact = FALSE)
```

**_Output_**

```

	Wilcoxon rank sum test with continuity
	correction

data:  crop_data_6 and midgut_data_6
W = 3396, p-value < 2.2e-16
alternative hypothesis: true location shift is not equal to 0


	Wilcoxon rank sum test with continuity
	correction

data:  crop_data_7 and midgut_data_7
W = 3279, p-value = 5.6e-16
alternative hypothesis: true location shift is not equal to 0


	Wilcoxon rank sum test with continuity
	correction

data:  crop_data_8 and midgut_data_8
W = 1920, p-value = 0.04371
alternative hypothesis: true location shift is not equal to 0


	Wilcoxon rank sum test with continuity
	correction

data:  crop_data_9 and midgut_data_9
W = 1413.5, p-value = 0.695
alternative hypothesis: true location shift is not equal to 0


	Wilcoxon rank sum test with continuity
	correction

data:  crop_data_10 and midgut_data_10
W = 199.5, p-value = 0.1629
alternative hypothesis: true location shift is not equal to 0
```

### **Statistical Analysis Results**

**Asaia Results**:<br>
0 hours: U Statistic = 225, P-value ≈ 0.00000066 (Significant)<br>
0.5 hours: U Statistic = 225, P-value ≈ 0.00000208 (Significant)<br>
4 hours: U Statistic = 225, P-value ≈ 0.00000306 (Significant)<br>
24 hours: U Statistic = 184, P-value ≈ 0.00322 (Significant)<br>
48 hours: U Statistic = 34.5, P-value ≈ 0.00106 (Significant)<br>
**Serratia Results**:<br>
0 hours: U Statistic = 3396, P-value ≈ 0.0000000000000000347 (Significant)<br>
0.5 hours: U Statistic = 3279, P-value ≈ 0.00000000000000056 (Significant)<br>
4 hours: U Statistic = 1920, P-value ≈ 0.0437 (Significant)<br>
24 hours: U Statistic = 1413.5, P-value ≈ 0.695 (Not significant)<br>
48 hours: U Statistic = 199.5, P-value ≈ 0.1629 (Not significant)<br>

For _Asaia_, there is a statistically significant difference between the crop and midgut CFU counts at all measured time points. This suggests that there is a consistent difference in the bacterial population between these two gut regions across time, supporting the hypothesis that Asaia does not maintain equal populations between the crop and the midgut.<br>
For _Serratia_, significant differences are observed in the early time points (0, 0.5, and 4 hours), indicating initial differences in bacterial populations between the crop and the midgut. However, at 24 and 48 hours, the p-values indicate no significant differences, which might suggest that over time, any initial differences in population may equalize or become less distinct.

# **References**

The following packages must be installed to work with the dataset:

```r
knitr::write_bib(c("ggplot2", "dplyr", "stats", "psych",
                   "readr", "tidyr", "ggpubr",
                   "purrr"), file = "packages.bib")
```
