# Adrenal masses classification using radiomics

## Setup
1. installing anaconda environment with python installed: conda create -p venv python==3.10.13 -y
2. install the dependencies: pip install -r requirements.txt

## Data Cleaning
Here, the steps were to remove the columns with too many Nan values and columns with multiple values separated by |


### Modelling only the basal data
To start I decided to focus only on radiomics data without using complementary informations like fx and secretion, and HUpreCONTRASTO';
And here i focus on the classification of:
1. if morf_codificata is in [0,1,5] -> i assign bening tumor (0)
2. if morf_codificata is in [2,3,4] -> i assign malignant tumor (1)

I dropped fx and attributes with |  and i used all the columns except the following until the first basal column

```python
Basal_Dropped_columns =[ 'età','Data di nascita GIUSTE','fx','luogoTC_codificato','secrezione','mm asse1','mm asse2','mm asse3','HUpreCONTRASTO','HUfaseARTERIOSA','HUfaseVENOSA','HUfaseTARDIVA',
                  'washoutAssoluto','washoutRelativo','ABS_WO_calculated','REL_WO_calculated','SUVFDGsurrene','COGNOME','Unnamed: 20',
                  'BAS_MORPHOLOGICAL_Hoc(IBSI:No)','BAS_MORPHOLOGICAL_NormalizedHocRadiusRoi(IBSI:No)','BAS_MORPHOLOGICAL_NormalizedHocRadiusSphere(IBSI:No)',
                  'BAS_MORPHOLOGICAL_NormalizedCentreOfMassShiftMaxRadiusRoi(IBSI:No)','BAS_MORPHOLOGICAL_NormalizedCentreOfMassShiftRadiusSphere(IBSI:No)','BAS_MORPHOLOGICAL_SphereDiameter(IBSI:No)',
                  'BAS_INTENSITY-BASED_AreaUnderCurveCsh(HU)IBSI:No','BAS_LOCAL_INTENSITY_BASED_IntensityPeakDiscretizedVolumeSought(0.5mL)(mL)IBSI:No',
                  'BAS_LOCAL_INTENSITY_BASED_GlobalIntensityPeak(0.5mL)(HU)IBSI:No',        
                  'BAS_LOCAL_INTENSITY_BASED_IntensityPeakDiscretizedVolumeSought(1mL)(mL)IBSI:No',         
                  'BAS_LOCAL_INTENSITY_BASED_GlobalIntensityPeak(1mL)(HU)IBSI:0F91',                           
                  'BAS_LOCAL_INTENSITY_BASED_LocalIntensityPeak(HU)IBSI:VJGA',
                  'BAS_LOCAL_INTENSITY_HISTOGRAM_IntensityPeakDiscretizedVolumeSought(0.5mL)(mL)IBSI:No',
                  'BAS_LOCAL_INTENSITY_HISTOGRAM_GlobalIntensityPeak(0.5mL)(HU)IBSI:No',
                  'BAS_LOCAL_INTENSITY_HISTOGRAM_IntensityPeakDiscretizedVolumeSought(1mL)(mL)IBSI:No',
                  'BAS_LOCAL_INTENSITY_HISTOGRAM_GlobalIntensityPeak(1mL)(HU)IBSI:No',
                  'BAS_LOCAL_INTENSITY_HISTOGRAM_LocalIntensityPeak(HU)IBSI:No'
                  ]
 

df_dropped = df.drop(Basal_Dropped_columns, axis=1)
index = df_dropped.columns.get_loc('BAS_GLSZM_ZoneSizeEntropy(IBSI:GU8N)')
Basal_df = df_dropped.iloc[:, :(index + 1)]
```

## Cleaned dataset:
- 0: 427
- 1: 166
- 2: 31
- 3: 45
- 4: 48
- 5: 18

the entire cleaned dataframe has the following shape: (735, 126)

## Splitting in train and test and label transformation
![splitting](https://github.com/SimBoex/Bio_Thesis/blob/3c1846e8322cb8c1d593f8a4e418bc6a3a1e310c/Basal_BinaryClassification/Hist_binaryClassification_TrainAndTest.png)


## Boruta and RusBoost

- Feature Selection: after Boruta, **38** features were selected

### Visualization of the train dataset
t-SNE is a dimensionality reduction technique ideal for visualizing high-dimensional datasets. It excels in revealing the non-linear structure of the data by preserving local neighborhoods, making it a powerful tool for exploring the inherent clustering and patterns that might not be apparent with linear methods.

#### T-sne plot
![tsne](https://github.com/SimBoex/Bio_Thesis/blob/d610f2e0d52b8e48cc42214e1c59e3160b7e4a38/Basal_BinaryClassification/TSNE_interactive_plots/Tsne_plot_Boruta.png)

 
#### PCA plot
![pca](https://github.com/SimBoex/Bio_Thesis/blob/e3cb96512e62d03ff693aec9f3b47c5577d00223/Basal_BinaryClassification/pca_plots/pca_plot_Rus.png)

#### Best Hyperparmeters 5-folds Cross Validation
```python
best_params = {'feature_selection__estimator': RandomForestClassifier(random_state=42), 'feature_selection__estimator__max_depth': 5, 'feature_selection__estimator__n_estimators': 50, 'rusboostclassifier__learning_rate': 0.05, 'rusboostclassifier__n_estimators': 300}
```

### Confusion Matrix

![img](https://github.com/SimBoex/Bio_Thesis/blob/68c9d591202d552f91160a00ef095f2c5a97781b/Basal_BinaryClassification/ConfusionMatrix_RusBoost.png)


The performances are the following:
1. Precision: 0.698 
2. Recall: 0.811
3. F1-score: 0.75 
4. Specificity: 0.929
5. Accuracy 0.9095
6. AUC_ROC 0.969


![img2](https://github.com/SimBoex/Bio_Thesis/blob/b6a6270e0c574144e6e227582903010dd5f53ec6/RusBoostCurves.png)

## Standardization, SMOTE, Multi-level feture selection
- Feature selection:  **23** features were selected

### Visualization of the train dataset




#### T-sne plot
![Tsne](https://github.com/SimBoex/Bio_Thesis/blob/4b50c561a945882dd813a134358fce30b5eda14a/Basal_BinaryClassification/TSNE_interactive_plots/Tsne_plot_Multi.png)

#### PCA plot
![PCA](https://github.com/SimBoex/Bio_Thesis/blob/b4d9bfd02d5425a4cfaeabebf448a610dea0fea4/Basal_BinaryClassification/pca_plots/pca_plot_Multi.png)

#### Best Hyperparmeters 5-folds Cross Validation
```python
best_params ={'LassoCoeffSelector__alpha': 0.004, 'logistic_regression__C': 0.1, 'logistic_regression__class_weight': 'balanced', 'logistic_regression__max_iter': 300, 'logistic_regression__penalty': 'l2', 'logistic_regression__solver': 'newton-cg', 'smote__k_neighbors': 5, 'smote__sampling_strategy': 'auto'}
```


### Confusion Matrix

![img](https://github.com/SimBoex/Bio_Thesis/blob/6e1a09611792fce31172da57f4cf53b7e95162ec/Basal_BinaryClassification/ConfusionMatrix_LogisticRegression.png))

1. Precision: 0.689
2. Recall: 0.838
3. F1-score: 0.756
4. Specificity: 0.924
5. Accuracy 0.9095 
6. AUC_ROC 0.9696

![img2](https://github.com/SimBoex/Bio_Thesis/blob/a965c26bed280bf10d07bcb0808b6d8938490778/LogisticRegressionCurves.png)


## Overview:
- For both the approaches, the performances are high in term of AUC_roc, specificity, and accuracy. The F1-score shows a good balance between recall and precision.
I tried other methods but the performances were lower or very similar.
Seems like there is no  a significant difference in term of performances between these two, maybe this is something that changes for a multi-class classification.

- the attribute fx that was not selected for this analysis, has a really impactful impact for the classification, however if there is the  dropping the NaN values, there is the removal of an entire label category.
    - could be interesting to set to a value  all the nan and look at the performances.
    - still, there is the need to know if using that attribute i'm introducing data leakage in the model.

