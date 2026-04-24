# PeakMerging4D_MassProfilterOutputs
Many peak shapes in mass spectrometry may be detected as multiple peaks, where they should be treated as one peak. To merge these together and improve interpretation of results, this Python-based algorithm has been developed to identify peaks within user-defined m/z, RT, and CCS tolerances and sum their intensities to treat them as one peak. 

This script is designed to work with **unannotated Mass Profiler CSV outputs**. 

# How to use the Peak Merging script
### Requirements
#### 1) Python Environment
This script was developed using Python v3.9.12. The pandas, numpy, and datetime modules are required. 

#### 2) Data structure
The input data should be the exported CSV feature table from Mass Profiler, with the following structure:

<img width="1295" height="179" alt="image" src="https://github.com/user-attachments/assets/ce94c200-e25d-4255-9907-9d28aa90260c" />

Importantly, the input should contain the `RT`, `SD`, `DT`, `SD`, `CCS`, `SD`, `m/z`, `SD`, `Abundance`, `RSD`, `Z`, `Ions`, `Freq.`, `Q Score`, `Sat.`, and `Mark` columns. The script searches for all of these columns specifically and treats all other columns as samples. 

#### 3) Prepare and run the script
Have the working directory of which ever environment you are working in set to the directory containing your desired input data. This is also where the merged-peaks output will be saved. Edit `mz_tol`, `RT_tol`, and `CCS_tol` as numeric inputs and `file_name` as a string containing the name of your input file. Do not include the ".csv" in file_name.  

```
###############################################################################
#TODO: User-defined inputs (please edit according to your peak merging needs)
#Group the m/z, RT and CCS values by user-defined tolerances 
mz_tol = 3 #1 = 1ppm
RT_tol = 0.3 #1 = 1 minute, 0.5 = 30 seconds 
CCS_tol = 2 #1 = 1%

file_name = "20241112_NEG_HRdm_FeatureTable_AllFeatures" #Do not include the ".csv" extension
###############################################################################
```
In the terminal, short summary outputs to indicate the effect each grouping step of the algorithm is having on the data are printed as well as the progress of the script. 

# How the Peak Merging Algorithm works
### Simplified example of initial feature table
Example User-defined thresholds:
- m/z (ppm) = 3
- Retention Time (min) = 0.3
- CCS (%) = 2

Whilst the script handles all the different parameters that are present in Mass Profiler outputs, the only ones used in the algorithm are the m/z, Retention Time (RT), Collision Cross Section (CCS), and intensity values across the samples. 

| FeatureID | m/z     | RT    | CCS    | Samp1   | Samp2   | Samp3   |
|-----------|---------|-------|--------|---------|---------|---------|
| 1         | 60.988  | 0.201 | 304.05 | 49843.4 | 0.001   | 13215.2 |
| 2         | 102.957 | 0.499 | 125.69 | 0.001   | 87321.5 | 0.001   |
| 3         | 102.957 | 0.501 | 235.68 | 4945.14 | 0.001   | 923.112 |
| 4         | 102.957 | 0.506 | 238.65 | 0.001   | 58942.7 | 0.001   |
| 5         | 102.957 | 1.186 | 458.65 | 794355  | 49561.7 | 0.001   |
| 6         | 102.957 | 1.190 | 456.34 | 0.001   | 455132  | 46235.7 |
| 7         | 142.929 | 0.700 | 369.58 | 0.001   | 94621.4 | 0.001   |
| 8         | 142.929 | 0.740 | 273.89 | 8912.45 | 0.001   | 7954.14 |
| 9         | 142.929 | 0.212 | 274.95 | 0.001   | 795645  | 0.001   |
| 10        | 142.929 | 0.213 | 274.97 | 454324  | 0.001   | 0.001   |

### Group features by m/z, RT, and CCS
The data is first split into m/z groups:

| FeatureID | m/z    | RT    | CCS    | Samp1   | Samp2 | Samp3   |
|-----------|--------|-------|--------|---------|-------|---------|
| 1         | 60.988 | 0.201 | 304.05 | 49843.4 | 0.001 | 13215.2 |

| FeatureID | m/z     | RT    | CCS    | Samp1   | Samp2   | Samp3   |
|-----------|---------|-------|--------|---------|---------|---------|
| 2         | 102.957 | 0.499 | 125.69 | 0.001   | 87321.5 | 0.001   |
| 3         | 102.957 | 0.501 | 235.68 | 4945.14 | 0.001   | 923.112 |
| 4         | 102.957 | 0.506 | 238.65 | 0.001   | 58942.7 | 0.001   |
| 5         | 102.957 | 1.186 | 458.65 | 794355  | 49561.7 | 0.001   |
| 6         | 102.957 | 1.190 | 456.34 | 0.001   | 455132  | 46235.7 |

| FeatureID | m/z     | RT    | CCS    | Samp1   | Samp2   | Samp3   |
|-----------|---------|-------|--------|---------|---------|---------|
| 7         | 142.929 | 0.700 | 369.58 | 0.001   | 94621.4 | 0.001   |
| 8         | 142.929 | 0.740 | 273.89 | 8912.45 | 0.001   | 7954.14 |
| 9         | 142.929 | 0.212 | 274.95 | 0.001   | 795645  | 0.001   |
| 10        | 142.929 | 0.213 | 274.97 | 454324  | 0.001   | 0.001   |

---
The script then loops through each m/z group to further group these into appropriate RT groups. 

| FeatureID | m/z    | RT    | CCS    | Samp1   | Samp2 | Samp3   |
|-----------|--------|-------|--------|---------|-------|---------|
| 1         | 60.988 | 0.201 | 304.05 | 49843.4 | 0.001 | 13215.2 |

| FeatureID | m/z     | RT    | CCS    | Samp1   | Samp2   | Samp3   |
|-----------|---------|-------|--------|---------|---------|---------|
| 2         | 102.957 | 0.499 | 125.69 | 0.001   | 87321.5 | 0.001   |
| 3         | 102.957 | 0.501 | 235.68 | 4945.14 | 0.001   | 923.112 |
| 4         | 102.957 | 0.506 | 238.65 | 0.001   | 58942.7 | 0.001   |

| FeatureID | m/z     | RT    | CCS    | Samp1  | Samp2   | Samp3   |
|-----------|---------|-------|--------|--------|---------|---------|
| 5         | 102.957 | 1.186 | 458.65 | 794355 | 49561.7 | 0.001   |
| 6         | 102.957 | 1.190 | 456.34 | 0.001  | 455132  | 46235.7 |

| FeatureID | m/z     | RT    | CCS    | Samp1   | Samp2   | Samp3   |
|-----------|---------|-------|--------|---------|---------|---------|
| 7         | 142.929 | 0.700 | 369.58 | 0.001   | 94621.4 | 0.001   |
| 8         | 142.929 | 0.740 | 273.89 | 8912.45 | 0.001   | 7954.14 |

| FeatureID | m/z     | RT    | CCS    | Samp1  | Samp2  | Samp3 |
|-----------|---------|-------|--------|--------|--------|-------|
| 9         | 142.929 | 0.212 | 274.95 | 0.001  | 795645 | 0.001 |
| 10        | 142.929 | 0.213 | 274.97 | 454324 | 0.001  | 0.001 |

---
Within these m/z-RT groups, these are split further according to their CCS values

| FeatureID | m/z    | RT    | CCS    | Samp1   | Samp2 | Samp3   |
|-----------|--------|-------|--------|---------|-------|---------|
| 1         | 60.988 | 0.201 | 304.05 | 49843.4 | 0.001 | 13215.2 |

| FeatureID | m/z     | RT    | CCS    | Samp1 | Samp2   | Samp3 |
|-----------|---------|-------|--------|-------|---------|-------|
| 2         | 102.957 | 0.499 | 125.69 | 0.001 | 87321.5 | 0.001 |

| FeatureID | m/z     | RT    | CCS    | Samp1   | Samp2   | Samp3   |
|-----------|---------|-------|--------|---------|---------|---------|
| 3         | 102.957 | 0.501 | 235.68 | 4945.14 | 0.001   | 923.112 |
| 4         | 102.957 | 0.506 | 238.65 | 0.001   | 58942.7 | 0.001   |

| FeatureID | m/z     | RT    | CCS    | Samp1  | Samp2   | Samp3   |
|-----------|---------|-------|--------|--------|---------|---------|
| 5         | 102.957 | 1.186 | 458.65 | 794355 | 49561.7 | 0.001   |
| 6         | 102.957 | 1.190 | 456.34 | 0.001  | 455132  | 46235.7 |

| FeatureID | m/z     | RT    | CCS    | Samp1 | Samp2   | Samp3 |
|-----------|---------|-------|--------|-------|---------|-------|
| 7         | 142.929 | 0.700 | 369.58 | 0.001 | 94621.4 | 0.001 |

| FeatureID | m/z     | RT    | CCS    | Samp1   | Samp2 | Samp3   |
|-----------|---------|-------|--------|---------|-------|---------|
| 8         | 142.929 | 0.740 | 273.89 | 8912.45 | 0.001 | 7954.14 |

| FeatureID | m/z     | RT    | CCS    | Samp1  | Samp2  | Samp3 |
|-----------|---------|-------|--------|--------|--------|-------|
| 9         | 142.929 | 0.212 | 274.95 | 0.001  | 795645 | 0.001 |
| 10        | 142.929 | 0.213 | 274.97 | 454324 | 0.001  | 0.001 |

### Merge the feature information together for each m/z-RT-CCS group
To merge the information together, the mean average of each m/z, RT, and CCS value is used in their respective columns. As **Mass Profiler was set to use maximum ion volume** to calculate each feature intensity, the intensities across each sample are brought together by **summing** the intensity values, except 0.001 is "ignored" (indicates 0 value). Each of the features' IDs that were merged together are listed for each merged feature to allow validation against the input data. 

| FeatureID | m/z     | RT     | CCS     | Samp1   | Samp2    | Samp3   |
|-----------|---------|--------|---------|---------|----------|---------|
| [1]       | 60.988  | 0.201  | 304.05  | 49843.4 | 0.001    | 13215.2 |
| [2]       | 102.957 | 0.499  | 125.69  | 0.001   | 87321.5  | 0.001   |
| [3, 4]    | 102.957 | 0.5035 | 237.165 | 4945.14 | 58942.7  | 923.112 |
| [5, 6]    | 102.957 | 1.188  | 457.495 | 794355  | 504693.7 | 46235.7 |
| [7]       | 142.929 | 0.7    | 369.58  | 0.001   | 94621.4  | 0.001   |
| [8]       | 142.929 | 0.740  | 273.89  | 8912.45 | 0.001    | 7954.14 |
| [9, 10]   | 142.929 | 0.2125 | 274.96  | 454324  | 795645   | 0.001   |


