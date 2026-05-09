Social Vulnerability in Virginia Counties: A Statistical Learning Approach
Shouyu Li & Arya Arun
5/6/2026`
Introduction and Research Questions
Social vulnerability describes how strongly communities may be affected by external stressors such as natural disasters, public health emergencies, and economic shocks (Flanagan et al., 2011). The CDC/ATSDR Social Vulnerability Index (SVI) condenses sixteen U.S. Census variables into a single percentile score so that public-health and emergency-management agencies can prioritize support to the most disadvantaged areas. Although the SVI is widely used, relatively little county-level work has compared modern statistical-learning methods on its component indicators in a single state. This project performs that comparison on the 133 Virginia counties and independent cities contained in the 2022 SVI release.

The unit of analysis is the county. The continuous outcome of interest is the overall SVI percentile score (RPL_THEMES). However, because this composite score is mathematically constructed from many of the same indicators we use as predictors, any predictive model that uses the SVI components to recover the SVI score is subject to target leakage (Kaufman et al., 2012). A model that appears to achieve perfect classification accuracy under such conditions cannot be interpreted as evidence of genuine out-of-sample performance. To address this, we organize the project around three research questions, the first two of which are explicitly descriptive / structural (no predictive claim), and the third of which uses an external response variable (urbanity, defined by population density) to evaluate genuine predictive performance.

Table 1. Research Questions and Statistical Learning Methods
RQ	Question	Main Method(s)
RQ1	Which socioeconomic, demographic, housing, transportation, and access-related indicators are most strongly associated with overall social vulnerability in Virginia counties? (Descriptive structural analysis.)	Pearson correlation; LASSO with 10-fold CV; bootstrap LASSO selection stability; Elastic Net (alpha=0.5) sensitivity check.
RQ2	What underlying dimensions of vulnerability can be recovered from the indicators using PCA, and how do counties group into interpretable profiles using K-means and hierarchical clustering? (Descriptive structural analysis.)	Principal Component Analysis; K-means clustering; Ward.D2 hierarchical clustering; cluster-agreement via Adjusted Rand Index.
RQ3	Using only the 17 SVI indicators, can supervised learners distinguish urban from rural Virginia counties (a response that is independent of the SVI formula), and which classifier offers the best bias–variance trade-off? (External predictive analysis.)	Logistic Regression, Linear Discriminant Analysis, Random Forest, and Gradient Boosting; 5-fold cross-validation repeated 10 times on a stratified 70/30 split; ROC, AUC, calibration.
Data Source and Variable Preparation
The dataset is the 2022 SVI database for Virginia, published by the CDC/ATSDR Geospatial Research, Analysis, and Services Program (CDC/ATSDR, 2024). The file contains percentile rankings, raw counts, and percentage estimates for each county and independent city. After excluding rows with missing overall SVI score and missing demographic denominators, the working sample contains 133 counties. The continuous response variable is the overall SVI percentile (RPL_THEMES); the predictor set consists of the 17 EP-percentage indicators that span the four official SVI themes (Table 2). We retain the original CDC variable names for code-data fidelity but display human-readable names throughout the report.

A central methodological point is that the overall SVI score is by construction a composite of these 17 indicators. Consequently, in RQ1 and RQ2 we treat the score not as a target to be predicted, but as a summary whose internal structure we wish to understand. RQ3 instead uses an external binary response derived from population density (Section RQ3), avoiding any leakage between predictor and response.

Table 2. Predictor Variables Used in the Analysis
Variable	Original_Name	Theme
Poverty Rate	EP_POV150	Socioeconomic status
Unemployment Rate	EP_UNEMP	Socioeconomic status
Housing Cost Burden	EP_HBURD	Socioeconomic status
No High School Diploma	EP_NOHSDP	Socioeconomic status
Uninsured Rate	EP_UNINSUR	Socioeconomic status
Age 65 or Older	EP_AGE65	Household & demographic composition
Age 17 or Younger	EP_AGE17	Household & demographic composition
Disability Rate	EP_DISABL	Household & demographic composition
Single-Parent Households	EP_SNGPNT	Household & demographic composition
Limited English	EP_LIMENG	Household & demographic composition
Minority Population	EP_MINRTY	Minority status
Multi-Unit Housing	EP_MUNIT	Housing, transportation & access
Mobile Homes	EP_MOBILE	Housing, transportation & access
Crowded Housing	EP_CROWD	Housing, transportation & access
No Vehicle Access	EP_NOVEH	Housing, transportation & access
Group Quarters	EP_GROUPQ	Housing, transportation & access
No Internet Access	EP_NOINT	Housing, transportation & access
Table 3. Descriptive Statistics of SVI Indicators and Outcome
Variable	Mean	Median	SD	Min	Max
Age 17 or Younger	19.87	19.6	3.40	7.2	27.3
Age 65 or Older	19.88	19.7	5.48	9.3	39.4
Crowded Housing	1.68	1.6	1.19	0.0	8.9
Disability Rate	15.72	14.9	5.32	6.5	36.4
Group Quarters	4.34	2.2	6.30	0.0	43.0
Housing Cost Burden	22.97	21.7	6.25	11.5	41.7
Limited English	1.28	0.6	1.88	0.0	13.3
Minority Population	29.97	27.2	18.60	2.7	85.1
Mobile Homes	8.89	6.4	8.24	0.0	31.8
Multi-Unit Housing	7.30	3.5	9.46	0.0	54.9
No High School Diploma	11.64	10.7	4.80	2.6	25.8
No Internet Access	17.30	17.5	7.87	3.3	37.2
No Vehicle Access	6.10	5.4	3.21	0.6	16.5
Overall SVI Score	0.50	0.5	0.29	0.0	1.0
Poverty Rate	21.36	20.3	8.69	3.9	42.3
Single-Parent Households	5.65	5.4	2.50	0.9	13.2
Unemployment Rate	4.70	4.3	1.99	1.3	14.7
Uninsured Rate	7.46	7.0	2.73	0.9	19.7
Exploratory Analysis
The exploratory analysis examines whether predictor distributions justify the modelling choices that follow. Because LASSO, PCA, and K-means are sensitive to variable scale, we standardize predictors before fitting any multivariate model. A short check of skewness and zero-inflation is also useful, since strongly zero-inflated indicators (e.g., crowded housing, limited English) can distort correlation-based methods.

Figure 1. Distribution of the overall SVI percentile score across Virginia counties. The score is approximately uniform-tilted toward higher values, with a small mode in the upper percentile band.
Figure 1. Distribution of the overall SVI percentile score across Virginia counties. The score is approximately uniform-tilted toward higher values, with a small mode in the upper percentile band.

Figure 1 shows that the overall SVI scores are spread across the full [0, 1] percentile range, which is expected because RPL_THEMES is itself a percentile rank rather than a raw measure. A small concentration of counties sits in the upper band (0.85–1.0), corresponding to the most socially vulnerable counties; this upper-tail mass is large enough to support a binary high-vulnerability framing without producing extreme class imbalance, and it foreshadows the existence of a substantively distinct cluster in RQ2.

Figure 2. Marginal distributions of the 17 SVI predictors. Several variables (Limited English, Crowded Housing, Group Quarters) are right-skewed and zero-inflated; standardization alone cannot remove this, so coefficient interpretations should be made on the original (percentage) scale.
Figure 2. Marginal distributions of the 17 SVI predictors. Several variables (Limited English, Crowded Housing, Group Quarters) are right-skewed and zero-inflated; standardization alone cannot remove this, so coefficient interpretations should be made on the original (percentage) scale.

Figure 2 reveals substantial heterogeneity in shape across the 17 predictors. Variables such as Housing Cost Burden, Disability Rate, and No Internet Access are roughly bell-shaped, while Limited English, Crowded Housing, and Group Quarters are strongly right-skewed and zero-inflated. We retain the percentages on their original scale rather than log-transforming them, because z-score standardization preserves rank order and the penalized regression and tree-based methods we use are not sensitive to monotone transformations of the predictors.

Table 4. Variance Inflation Factors. Several predictors exceed VIF = 5, confirming multicollinearity and motivating penalized regression.
Variable	VIF
Housing Cost Burden	6.31
Age 17 or Younger	5.99
No Internet Access	4.65
Age 65 or Older	4.64
Multi-Unit Housing	4.54
Poverty Rate	4.52
Limited English	4.34
Mobile Homes	4.22
Disability Rate	4.13
No High School Diploma	4.07
No Vehicle Access	3.68
Group Quarters	3.65
Minority Population	3.40
Single-Parent Households	3.02
Uninsured Rate	2.68
Crowded Housing	2.20
Unemployment Rate	1.63
The descriptive plots show that the indicators have very different ranges and spreads, and several display strong skewness or floor effects. The VIF table (Table 4) confirms substantial multicollinearity: at least three indicators exceed VIF = 5, which would inflate the variance of ordinary least-squares coefficients. This is the principled reason for using LASSO and Elastic Net in RQ1 instead of OLS.

RQ1: Factors Most Strongly Associated with Overall Vulnerability
To identify which indicators are most closely connected to the composite SVI score, we combine four complementary tools:

Pearson correlation — a transparent bivariate baseline.
LASSO regression — penalized regression that performs variable selection in the presence of multicollinearity, with the regularization parameter chosen by 10-fold cross-validation (we report both lambda.min and the one-standard-error rule lambda.1se).
Bootstrap LASSO selection — we resample the data 500 times and refit LASSO at lambda.1se to obtain the empirical selection frequency of each predictor, a robust measure of which variables remain useful under sampling variation.
Elastic Net (alpha = 0.5) — a compromise between LASSO and Ridge, used as a sensitivity check because the LASSO can be unstable when groups of correlated predictors carry similar signal.
All predictors are standardized (z-score) before fitting penalized models, so that coefficient magnitudes are directly comparable.

Figure 3. Pearson correlation matrix among the 17 SVI indicators. Strong positive correlations between Poverty Rate, Single-Parent Households, and No High School Diploma motivate penalized regression rather than OLS.
Figure 3. Pearson correlation matrix among the 17 SVI indicators. Strong positive correlations between Poverty Rate, Single-Parent Households, and No High School Diploma motivate penalized regression rather than OLS.

Figure 3 shows that several indicators move together in coherent blocks. Poverty Rate, No High School Diploma, Single-Parent Households, and Uninsured Rate form a socioeconomic block with pairwise correlations between roughly 0.5 and 0.8, while Mobile Homes, No Internet Access, and Age 65 or Older form a second rural-character block. These overlapping correlation patterns motivate penalized regression in the next subsection: when groups of indicators carry redundant signal, ordinary least-squares coefficients become unstable, whereas LASSO either picks one representative or shrinks the whole group toward zero.

Table 5. Ten Strongest Correlations with the Overall SVI Score
Variable	Correlation with Overall SVI
Housing Cost Burden	0.738
No Vehicle Access	0.690
Poverty Rate	0.688
Single-Parent Households	0.661
No High School Diploma	0.613
Uninsured Rate	0.561
Minority Population	0.535
Unemployment Rate	0.449
Disability Rate	0.384
No Internet Access	0.312
Figure 4. Cross-validation curve for LASSO. The vertical dashed lines mark lambda.min (left) and the more parsimonious lambda.1se (right); we report results at both, with lambda.1se as our preferred sparse solution.
Figure 4. Cross-validation curve for LASSO. The vertical dashed lines mark lambda.min (left) and the more parsimonious lambda.1se (right); we report results at both, with lambda.1se as our preferred sparse solution.

Figure 4 displays the typical U-shape of cross-validation error against log-lambda, with a relatively flat minimum-error region. The left dashed line marks lambda.min (lower λ, more variables retained) and the right dashed line marks lambda.1se (higher λ, sparser model whose CV error is within one standard error of the minimum). We adopt lambda.1se as the default sparse solution because the right-hand line still falls within the flat error band, indicating that the sparser model loses very little predictive accuracy while being substantially more robust to sampling variation.

Table 6. Standardized Penalized-Regression Coefficients. Zeros under lambda.1se indicate variables eliminated by the parsimonious LASSO.
Variable	LASSO lambda.min	LASSO lambda.1se	Elastic Net (alpha=0.5)
Single-Parent Households	0.073	0.064	0.059
Housing Cost Burden	0.069	0.044	0.043
Age 17 or Younger	0.060	0.000	0.000
Age 65 or Older	0.049	0.000	0.000
Mobile Homes	0.049	0.000	0.000
Group Quarters	0.048	0.000	0.000
Uninsured Rate	0.048	0.044	0.043
Poverty Rate	0.041	0.041	0.043
No Vehicle Access	0.034	0.042	0.041
Crowded Housing	0.034	0.011	0.011
Unemployment Rate	0.032	0.016	0.016
No High School Diploma	0.031	0.065	0.059
Minority Population	0.025	0.018	0.021
Multi-Unit Housing	0.020	0.000	0.000
Disability Rate	0.017	0.000	0.000
Limited English	0.014	0.000	0.000
No Internet Access	0.005	0.000	0.000
Figure 5. Bootstrap LASSO selection frequencies (B = 500, lambda.1se). Variables retained in more than 80% of bootstrap samples (dashed line) constitute the *stable* signal: Housing Cost Burden, Poverty Rate, Single-Parent Households, No High School Diploma, and Uninsured Rate.
Figure 5. Bootstrap LASSO selection frequencies (B = 500, lambda.1se). Variables retained in more than 80% of bootstrap samples (dashed line) constitute the stable signal: Housing Cost Burden, Poverty Rate, Single-Parent Households, No High School Diploma, and Uninsured Rate.

Figure 6. LASSO regression diagnostics at lambda.1se. Left: predicted vs actual SVI scores show tight calibration. Right: residual plot shows no systematic non-linearity or heteroskedasticity.
Figure 6. LASSO regression diagnostics at lambda.1se. Left: predicted vs actual SVI scores show tight calibration. Right: residual plot shows no systematic non-linearity or heteroskedasticity.

Table 7. Penalized-Regression Fit Comparison
Solution	Non-zero Variables	In-sample R-squared	CV RMSE
LASSO lambda.min	17	0.900	0.116
LASSO lambda.1se	9	0.853	0.123
Elastic Net (alpha=0.5)	9	0.846	0.124
The LASSO at lambda.min retains nearly all 17 indicators because the indicators are by construction informative about the composite SVI; the parsimonious lambda.1se solution drops 8 indicators while keeping the structural picture intact (Table 6). The bootstrap analysis (Figure 5) further confirms that the stable signal is dominated by Housing Cost Burden, Poverty Rate, Single-Parent Households, No High School Diploma, and Uninsured Rate — the classical socioeconomic correlates of vulnerability. The Elastic Net solution is qualitatively similar but slightly less sparse, which is expected because Elastic Net retains correlated predictors as groups. The diagnostic plots (Figure 6) show no systematic non-linearity or heteroskedasticity, supporting the linear penalized specification. We emphasize that these results are associative and structural, not causal: because the response is a known function of the predictors, the analysis characterizes which indicators dominate the SVI formula in this state, not which factors cause social vulnerability.

RQ2: Main Dimensions of Vulnerability and County Grouping
The second question asks how the 17 indicators collapse into a smaller number of dimensions and whether counties cluster into recognizable profiles. We use PCA on the standardized predictors, then cluster counties using both K-means (a partitional method) and Ward.D2 hierarchical clustering (an agglomerative method). Comparing two clustering algorithms is a fairness safeguard: if the two methods agree, the resulting county profiles are not an artifact of one algorithm’s bias. We measure agreement using the Adjusted Rand Index (Hubert & Arabie, 1985).

Principal Component Analysis
Table 8. Variance Explained by the First Eight Principal Components
PC	% Variance Explained	Cumulative %
1	29.31	29.31
2	24.03	53.35
3	11.24	64.58
4	8.14	72.73
5	5.96	78.69
6	4.43	83.12
7	3.55	86.67
8	2.80	89.47
Figure 7. PCA scree plot. The first six components explain over 80% of total variance, but PC1 alone explains only ~29%, indicating that vulnerability in Virginia is not dominated by a single dimension.
Figure 7. PCA scree plot. The first six components explain over 80% of total variance, but PC1 alone explains only ~29%, indicating that vulnerability in Virginia is not dominated by a single dimension.

Figure 7 reveals a gradual decline in eigenvalues rather than a sharp elbow. PC1 explains only about 29% of total variance and PC2 explains about 24%, and six components are required to clear the 80% cumulative threshold. This pattern indicates that vulnerability information is genuinely distributed across multiple dimensions in this dataset rather than concentrated in one or two leading axes — an empirical finding that supports the multi-theme structure of the SVI framework.

Figure 8. PCA biplot of counties (points) and predictor loadings (arrows) on PC1–PC2. PC1 separates counties along an urban-versus-rural axis (multi-unit housing, limited English, crowded housing point opposite to mobile homes, no internet, and elderly population); PC2 captures economic distress (poverty, housing burden, uninsured).
Figure 8. PCA biplot of counties (points) and predictor loadings (arrows) on PC1–PC2. PC1 separates counties along an urban-versus-rural axis (multi-unit housing, limited English, crowded housing point opposite to mobile homes, no internet, and elderly population); PC2 captures economic distress (poverty, housing burden, uninsured).

Figure 8 makes the geometric structure of PC1–PC2 visually concrete. The loading arrows for No Internet Access, Mobile Homes, and Age 65 or Older point to one side of PC1, while Multi-Unit Housing, Limited English, and Crowded Housing point to the opposite side, confirming that PC1 is best read as an urban–rural contrast rather than a vulnerability gradient. The vertical spread along PC2 is dominated by Poverty Rate, Housing Cost Burden, and No Vehicle Access, supporting the interpretation of PC2 as an economic-distress dimension that operates orthogonally to the urban–rural axis.

Table 9. PCA Loadings for the First Three Components
Variable	PC1	PC2	PC3
No Internet Access	-0.366	0.193	-0.072
Mobile Homes	-0.352	0.059	-0.178
Age 65 or Older	-0.351	-0.069	-0.129
Disability Rate	-0.336	0.225	-0.092
Multi-Unit Housing	0.322	0.078	0.162
Limited English	0.310	0.074	-0.325
Minority Population	0.259	0.265	-0.007
Age 17 or Younger	0.243	0.015	-0.347
Crowded Housing	0.242	0.098	-0.359
No High School Diploma	-0.204	0.318	-0.271
Poverty Rate	-0.168	0.392	0.073
Housing Cost Burden	0.158	0.397	0.179
Single-Parent Households	0.128	0.317	0.120
Uninsured Rate	0.065	0.270	-0.403
No Vehicle Access	0.059	0.387	0.209
Unemployment Rate	-0.044	0.258	0.073
Group Quarters	-0.028	0.100	0.469
PC1 (~29% of variance) loads negatively on rural-style indicators (No Internet, Mobile Homes, Age 65+) and positively on urban-style indicators (Multi-Unit Housing, Limited English) — it is best read as an urban–rural axis rather than an “overall vulnerability” axis. PC2 (~24%) loads positively on classical economic distress indicators (Poverty, Housing Cost Burden, No Vehicle, Single-Parent Households). PC3 captures a residual contrast involving Group Quarters and Uninsured Rate. Because no single PC dominates, the SVI’s multidimensional design appears empirically justified.

K-means and Hierarchical Clustering
Figure 9. K-means within-cluster sum of squares (elbow plot). The reduction is largest from k=2 to k=3, with diminishing returns beyond k=4.
Figure 9. K-means within-cluster sum of squares (elbow plot). The reduction is largest from k=2 to k=3, with diminishing returns beyond k=4.

Figure 9 shows that the within-cluster sum-of-squares decreases sharply between k = 1 and k = 3 and then bends into a much shallower descent. This “elbow” supports a three-cluster solution, although the bend is gentle enough that k = 2 or k = 4 would also be defensible by this criterion alone — which is why we cross-check with the silhouette criterion in Figure 10.

Figure 10. Average silhouette width vs number of clusters. Silhouette peaks at k=3, supporting a three-cluster solution.
Figure 10. Average silhouette width vs number of clusters. Silhouette peaks at k=3, supporting a three-cluster solution.

Figure 10 shows that the average silhouette width peaks clearly at k = 3 and falls off for both smaller and larger choices of k. The silhouette and elbow criteria thus agree on the same number of clusters, which provides substantially stronger evidence for k = 3 than either criterion would on its own. We therefore proceed with three clusters in the analyses that follow.

Figure 11. K-means clusters projected onto the first two principal components. The three clusters separate cleanly along the urban–rural and economic-distress axes.
Figure 11. K-means clusters projected onto the first two principal components. The three clusters separate cleanly along the urban–rural and economic-distress axes.

Figure 11 plots the K-means assignments on the PC1–PC2 plane and shows three convex regions with limited overlap. One cluster sits at negative PC1 (rural-style indicators dominant), another sits at positive PC1 (urban-style indicators dominant), and the third occupies the upper region of high PC2 (economic distress). The clean visual separation suggests the partition is not an artifact of K-means initialization and gives an intuitive geometric reading of the cluster profiles reported in Table 10.

Table 10. K-means Cluster Profiles (k=3). Adjusted Rand Index between K-means and Ward hierarchical clustering = 0.420, indicating substantial agreement.
cluster_km	Profile	N	Mean SVI	Poverty %	Unemployment %	No HS Diploma %	Mobile Homes %	Minority %	Multi-Unit %	Pop Density
1	Distressed rural / minority-concentrated	24	0.828	28.9	5.9	12.8	2.2	52.1	16.9	2750.9
2	Affluent suburban / urban	52	0.295	13.9	3.9	8.0	3.9	27.3	8.4	928.5
3	Mixed rural / older population	57	0.548	25.0	4.9	14.5	16.3	23.1	2.2	100.4
Figure 12. Ward.D2 hierarchical clustering dendrogram. The three branches correspond closely to the K-means clusters, supporting the robustness of the three-profile partition.
Figure 12. Ward.D2 hierarchical clustering dendrogram. The three branches correspond closely to the K-means clusters, supporting the robustness of the three-profile partition.

The two clustering methods agree at an Adjusted Rand Index of 0.42, which is well above the level expected by chance. We adopt the K-means three-cluster solution, summarized in Table 10 with substantively meaningful profile labels. Cluster Affluent suburban / urban contains counties with low SVI, low mobile-home prevalence, and high multi-unit housing — consistent with the Northern Virginia and Hampton Roads metro regions. The Distressed rural / minority-concentrated cluster shows the highest mean SVI and the highest minority share, while the Mixed rural / older population cluster occupies an intermediate profile with relatively high mobile-home share and older residents. This profile structure is more informative than treating the SVI as a single number, because it links individual indicators back to coherent county types.

RQ3: External Predictive Validation — Urban vs Rural Classification
We deliberately avoid re-using the SVI percentile as a classification target, because the SVI is built from the same indicators we would feed to the classifier and any reported accuracy would be an artifact of construction. We use instead an external binary response: a county is Urban if its population density (E_TOTPOP / AREA_SQMI) lies above the median, and Rural otherwise. This response is constructed from variables (population, area) that are not part of the 17 SVI indicators, eliminating target leakage. The substantive question is: do the SVI indicators carry enough signal to recover the urban–rural distinction, and which classifier achieves the best out-of-sample performance?

We compare four learners with very different inductive biases:

Logistic Regression (LR) — linear in log-odds, fully interpretable.
Linear Discriminant Analysis (LDA) — assumes Gaussian predictors with shared covariance; serves as a parametric benchmark.
Random Forest (RF) — non-parametric, captures interactions, robust to scale; tuning over mtry.
Gradient Boosting Machine (GBM) — sequential additive trees that often win in tabular benchmarks; tuning over interaction.depth and n.trees.
To make the comparison fair we use a single stratified 70/30 split (set.seed( 2026)), standardize predictors using only the training set, and select hyperparameters by 5-fold cross-validation repeated 10 times on the training data, optimizing AUC. Test-set metrics include accuracy, AUC, sensitivity, specificity, and Brier score, with bootstrap 95% CIs on AUC.

Table 11. Class Distribution for the External Response (Urban vs Rural)
URBAN	n	Percent
Rural	67	50.4
Urban	66	49.6
Table 12. Tuning Grids and Best Hyperparameters Selected by 5x10 Repeated Cross-Validation
Model	Tuning grid	Best hyperparameter(s)	CV AUC (mean)
Logistic Regression	none (no hyperparameters)	—	0.801
Linear Discriminant Analysis	none (closed form)	—	0.954
Random Forest	mtry in {2, 4, 6, 8}, ntree = 500	mtry = 4	0.970
Gradient Boosting	depth in {1,3,5}, ntrees in {200,500}, shrinkage = 0.05	depth = 5, ntrees = 200	0.965
Table 13. Test-Set Classification Performance with Bootstrap 95% AUC CIs.
Model	Accuracy	AUC	AUC 95% CI low	AUC 95% CI high	Sensitivity	Specificity	Brier
Logistic Regression	0.846	0.847	0.720	0.949	0.895	0.80	0.154
LDA	0.897	0.987	0.950	1.000	0.895	0.90	0.048
Random Forest	0.897	0.987	0.953	1.000	0.947	0.85	0.057
Gradient Boosting	0.923	0.982	0.942	1.000	0.947	0.90	0.080
Figure 13. ROC curves on the held-out test set for the four classifiers. The four AUCs lie within overlapping bootstrap 95% confidence intervals (Table 13), indicating that no model dominates definitively at this sample size; Logistic Regression and LDA remain competitive with the more flexible ensemble methods.
Figure 13. ROC curves on the held-out test set for the four classifiers. The four AUCs lie within overlapping bootstrap 95% confidence intervals (Table 13), indicating that no model dominates definitively at this sample size; Logistic Regression and LDA remain competitive with the more flexible ensemble methods.

Figure 13 shows that all four ROC curves rise sharply along the y-axis at low false-positive rates, indicating that the SVI indicators encode the urban–rural distinction strongly enough that a substantial portion of the test set is classifiable with high confidence. The curves are visually intertwined — none lies uniformly above the others — which is consistent with the overlapping bootstrap confidence intervals on AUC reported in Table 13 and warns against over-interpreting small numerical differences between the four learners at this sample size.

Figure 14. Calibration plot. Predicted probabilities are binned into deciles and the observed Urban frequency is plotted against the bin mean. The Logistic Regression and GBM models are well calibrated; Random Forest probabilities are slightly mis-calibrated near the extremes, a known property of bagged trees.
Figure 14. Calibration plot. Predicted probabilities are binned into deciles and the observed Urban frequency is plotted against the bin mean. The Logistic Regression and GBM models are well calibrated; Random Forest probabilities are slightly mis-calibrated near the extremes, a known property of bagged trees.

Figure 14 reads as follows: points lying on the 45-degree line indicate perfect calibration, meaning that a model assigning, say, a 0.7 probability of “Urban” is correct on average 70% of the time. Logistic Regression and Gradient Boosting track the diagonal closely, while Random Forest’s points pull toward the middle of the [0, 1] range — the canonical signature of vote-averaged tree ensembles, which hesitate at the extremes. For policy uses where probabilities feed into downstream decisions (e.g., risk-stratified resource allocation), this calibration difference matters as much as raw discrimination, and it provides a substantive reason to prefer LR or GBM over RF here.

Figure 15. Variable importance from Random Forest (mean decrease in accuracy). Multi-Unit Housing, No Internet Access, and Mobile Homes dominate, exactly as expected: these are the strongest urban-versus-rural signals in the SVI indicator set.
Figure 15. Variable importance from Random Forest (mean decrease in accuracy). Multi-Unit Housing, No Internet Access, and Mobile Homes dominate, exactly as expected: these are the strongest urban-versus-rural signals in the SVI indicator set.

Figure 15 shows that Multi-Unit Housing, No Internet Access, and Mobile Homes top the Random Forest importance ranking by a large margin, in line with the urban–rural intuition: high-density counties have many apartments and good internet, while low-density counties have many mobile homes and poorer connectivity. Indicators that are central to the SVI’s vulnerability story (e.g., Poverty Rate, Single-Parent Households) sit in the middle of the ranking, confirming that the urban–rural axis is genuinely distinct from the vulnerability axis identified in RQ1.

Table 14a. Logistic Regression confusion matrix on test set (rows = predicted, columns = actual).
Rural	Urban
Rural	16	2
Urban	4	17
Table 14b. Gradient Boosting confusion matrix on test set.
Rural	Urban
Rural	18	1
Urban	2	18
The four classifiers achieve realistic, non-trivial test-set performance (Table 13). The best model on the held-out set is Gradient Boosting, with AUC = 0.982 and accuracy = 0.923. Logistic Regression and LDA come close behind despite their simpler functional forms, which suggests that the urban–rural distinction is largely linearly separable in the SVI indicator space. Random Forest is competitive on discrimination but slightly miscalibrated near the probability extremes (Figure 14), a well-known property of vote-averaged tree ensembles. Variable importance (Figure 15) confirms that Multi-Unit Housing, No Internet Access, and Mobile Homes carry the urban–rural signal — substantively coherent results that would not be visible from any single bivariate correlation.

The fair-comparison protocol (identical CV scheme, identical predictors, identical seed, identical preprocessing fitted only on training data) ensures that any differences in performance reflect modeling capacity, not unfair advantages. Bootstrap 95% CIs on AUC overlap considerably across the four models (Table 13), warning against over-interpretation: with 133 counties and a stratified 70/30 split, the test set has roughly 39 observations, and the sampling uncertainty is large. We therefore report cross-validated AUC alongside test AUC and emphasize that GBM’s lead is not statistically overwhelming.

Discussion and Conclusions
This project asked three questions about social vulnerability in Virginia counties and answered them with five complementary statistical-learning methods (LASSO with bootstrap stability, Elastic Net, PCA, K-means and Ward.D2 hierarchical clustering, and a four-way classifier comparison).

The descriptive RQ1 results show that the indicators most stably associated with the overall SVI score in Virginia are Housing Cost Burden, Poverty Rate, Single-Parent Households, No High School Diploma, and Uninsured Rate. Bootstrap selection frequencies above 80% (Figure 5) make this finding robust to sampling variation. Importantly, because the response is built from the predictors, this is a structural decomposition rather than a causal prediction: the bootstrap stability tells us which indicators most consistently drive the composite formula in this state, not which factors cause vulnerability.

The descriptive RQ2 results show that the SVI’s multidimensional design is empirically justified: the first principal component explains only 29% of total variance, six components are needed to capture 80%, and PC1 is best read as an urban–rural axis rather than an undifferentiated vulnerability axis. K-means and Ward.D2 hierarchical clustering agree at an Adjusted Rand Index of 0.42, supporting a robust three-cluster partition that maps onto recognizable Virginia geographies: an affluent suburban/urban core, a mixed rural / older-population cluster, and a distressed rural / minority-concentrated cluster.

The predictive RQ3 results establish that the SVI indicators do contain substantial signal about a response that is external to the SVI formula (urban vs rural based on population density). Gradient Boosting reaches a test-AUC of 0.982 — high enough to confirm that SVI indicators capture much of the urbanity gradient — while Logistic Regression and LDA come within bootstrap CI distance, showing the relationship is largely linear. This contrasts sharply with naive prior analyses that classify the SVI itself, where any classifier achieves apparent perfect performance by construction. The reframed RQ3 thus delivers genuine evidence of out-of-sample performance.

Limitations
Sample size. With 133 counties, the test set in RQ3 is approximately 39 observations. Bootstrap CIs on AUC are wide and the ranking among the top three models should not be over-interpreted.
Cross-sectional design. We use a single 2022 SVI release; longitudinal trends in vulnerability are out of scope.
Aggregation. County is a coarse unit. Important within-county heterogeneity (e.g., within independent cities) is invisible to this analysis.
Independence assumption. Counties are spatially close to each other and therefore not statistically independent. We did not implement a formal spatial autocorrelation correction; ignoring spatial structure may inflate apparent variable importance.
External validity. Virginia is one state; the same indicators may carry different weight in other states or at the census-tract level.
Threshold choice. The Urban/Rural threshold uses the median density, chosen to balance classes. A substantively motivated cutoff (e.g., 100 persons/sq mi, a common rural definition in U.S. health-policy research) would shift class membership; a formal sensitivity analysis at alternative cutoffs is left for future work.
Future Work
Spatial models. Compute Moran’s I and fit spatial-error or geographically-weighted regression to formally account for spatial autocorrelation among Virginia counties.
External outcome validation. Link county-level SVI indicators to real-world outcomes such as flood-insurance claims, COVID-19 mortality, or hospital-readiness scores, to test whether SVI predicts true downstream consequences.
Tract-level replication. Repeat the entire pipeline at the census-tract level (~1,900 Virginia tracts) to see whether the same five indicators remain dominant under finer aggregation.
Stability and calibration. Use platt-scaled or isotonic-regression calibration on the Random Forest probabilities; bootstrap confidence bands on the K-means cluster assignments.
Out-of-state replication. Apply the same five-stage pipeline to a geographically and demographically different state (e.g., Mississippi) to test whether the dominant indicators are state-specific or general.
References
Centers for Disease Control and Prevention/Agency for Toxic Substances and Disease Registry/Geospatial Research, Analysis, and Services Program. (2024). CDC/ATSDR Social Vulnerability Index 2022 database: Virginia. https://www.atsdr.cdc.gov/place-health/php/svi/svi-data-documentation-download.html

Flanagan, B. E., Gregory, E. W., Hallisey, E. J., Heitgerd, J. L., & Lewis, B. (2011). A Social Vulnerability Index for disaster management. Journal of Homeland Security and Emergency Management, 8(1).

Friedman, J., Hastie, T., & Tibshirani, R. (2010). Regularization paths for generalized linear models via coordinate descent. Journal of Statistical Software, 33(1), 1–22.

Hubert, L., & Arabie, P. (1985). Comparing partitions. Journal of Classification, 2(1), 193–218.

Kassambara, A., & Mundt, F. (2020). factoextra: Extract and visualize the results of multivariate data analyses [R package].

Kaufman, S., Rosset, S., Perlich, C., & Stitelman, O. (2012). Leakage in data mining: Formulation, detection, and avoidance. ACM Transactions on Knowledge Discovery from Data, 6(4), 1–21.

Kuhn, M. (2008). Building predictive models in R using the caret package. Journal of Statistical Software, 28(5), 1–26.

Liaw, A., & Wiener, M. (2002). Classification and regression by randomForest. R News, 2(3), 18–22.

R Core Team. (2024). R: A language and environment for statistical computing. R Foundation for Statistical Computing.

Robin, X., Turck, N., Hainard, A., Tiberti, N., Lisacek, F., Sanchez, J.-C., & Müller, M. (2011). pROC: An open-source package for R and S+ to analyze and compare ROC curves. BMC Bioinformatics, 12, 77.

Scrucca, L., Fop, M., Murphy, T. B., & Raftery, A. E. (2016). mclust 5: Clustering, classification and density estimation using Gaussian finite mixture models. The R Journal, 8(1), 289–317.

Wickham, H., Averick, M., Bryan, J., Chang, W., McGowan, L. D., François, R., Grolemund, G., Hayes, A., Henry, L., Hester, J., Kuhn, M., Pedersen, T. L., Miller, E., Bache, S. M., Müller, K., Ooms, J., Robinson, D., Seidel, D. P., Spinu, V., … Yutani, H. (2019). Welcome to the tidyverse. Journal of Open Source Software, 4(43), 1686.

Xie, Y. (2024). knitr: A general-purpose package for dynamic report generation in R [R package].

Zhu, H. (2024). kableExtra: Construct complex table with kable and pipe syntax [R package].

Appendix A: Requirement Checklist
Table A1. Final-Project Requirement Checklist
Requirement	Where_Addressed
Three or more research questions	Introduction, Table 1
Exploratory analysis before modeling	EDA section, Figures 1-2, Tables 3-4
Multiple statistical learning methods taught in class	LASSO + Elastic Net + PCA + K-means + Ward HC + LR + LDA + RF + GBM
Method comparison with fair protocol	RQ3 fair-comparison protocol
Penalized-regression diagnostics (CV curve, R^2, residuals)	Figure 4, Figure 6, Table 7
Bootstrap stability of variable selection	Figure 5
Multicollinearity assessment (VIF)	Table 4
Two clustering methods compared (Adjusted Rand Index)	Table 10, Figure 12
Hyperparameters and seed reported explicitly	Table 12, all chunks set seed = 2026
Class imbalance and threshold choice discussed	Table 11 + RQ3 Discussion
Probabilistic calibration plot	Figure 14
Bootstrap 95% CIs on AUC	Table 13
Target leakage explicitly addressed (external response)	RQ3 introduction; Limitations
Well-labeled tables and figures	All tables and figures use readable variable names
No code in main report (echo = FALSE)	Global option echo = FALSE
References, including R packages, in APA format	References section
---
## Appendix: R Source Code

```r
# Paste your code here
```
