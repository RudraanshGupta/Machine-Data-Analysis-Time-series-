``   # Cyclone Sensor Data Analysis ``

``  This project performs a deep-dive analysis into three years of sensor data from a cyclone machine. The goal is to move beyond raw data to uncover operational patterns, detect anomalies, and provide actionable insights for improving machine performance and maintenance.  The entire analysis is contained within the `task1_analysis.py` script, which runs end-to-end.  ## How to Run the Analysis  ### Prerequisites  - Python 3.8+  - The following Python libraries: `pandas`, `numpy`, `matplotlib`, `seaborn`, `scikit-learn`, `statsmodels`. You can install them via pip:    ```bash    pip install pandas numpy matplotlib seaborn scikit-learn statsmodels   ``

### Setup

1.  Place the task1\_analysis.py script in a project folder.
    
2.  Place the dataset data.csv in the same folder.
    

### Execution

Navigate to the project folder in your terminal and run the script:

Bash

`   python task1_analysis.py   `

### Outputs

The script will automatically create two folders:

*   outputs/: Contains all generated CSV files (shutdown periods, cluster summaries, anomalies, and forecasts).
    
*   plots/: Contains all generated visualizations as PNG images.
    

Analytical Approach
-------------------

The analysis follows a structured, multi-stage process to build a comprehensive understanding of the machine's behavior.

**1\. Data Cleaning and EDA**First, the raw data.csv was loaded. A significant effort was made to clean the data, handle missing values, and, most importantly, enforce a strict 5-minute time series frequency. This involved reindexing and interpolating the data to fill any gaps, ensuring the integrity of time-based calculations.

**2\. Identifying Machine Downtime**To isolate operational data, I developed a rule to programmatically identify shutdown periods. A "shutdown" was defined as a period where multiple key temperature sensors dropped below their known operational thresholds. This allowed us to calculate key metrics like overall machine availability and average downtime duration.

**3\. Clustering for Machine States**With the active (non-shutdown) data, I used KMeans clustering to segment the machine's behavior into distinct operational states. After testing several cluster counts, four states provided the most interpretable results. I dynamically named these states based on their sensor characteristics:

*   **Startup/Low Load:** Low temperatures, low activity.
    
*   **Normal Operation:** Stable, moderate temperatures and pressures.
    
*   **High Load:** Elevated temperatures and drafts, indicating high throughput.
    
*   **Degraded:** High temperatures but with unstable or unusual draft pressures, suggesting inefficiency or a potential issue.
    

**4\. Contextual Anomaly Detection**Instead of a one-size-fits-all approach, I built a more sophisticated anomaly detection system. An Isolation Forest model was trained _for each individual machine state_. This allows the system to find subtle anomalies relative to the machine's current context (e.g., a pressure reading that is normal during 'High Load' but anomalous during 'Startup'). The system also provides a simple root-cause hypothesis for the most severe anomalies.

**5\. Forecasting Inlet Temperature**Finally, I explored the predictability of the machine's thermal behavior by forecasting the inlet temperature one hour ahead. I compared a Random Forest model (using lag features) against a simple persistence baseline. The Random Forest model proved to be significantly more accurate, showing that the machine's behavior is moderately predictable.

Key Findings
------------

*   **High but Interrupted Availability:** The machine has a high overall uptime of **92.1%**. However, this availability is punctuated by frequent shutdowns (81 over 3 years), which have an average duration of **5.3 hours**.
    
*   **Dominant Operating State:** The machine spends the majority of its active time in the **'Normal Operation'** state (49.8%). However, a significant amount of time is also spent in 'High Load' (26.6%) and 'Startup/Low Load' (20.9%) states.
    
*   **Anomalies are Not Random:** Anomalies are heavily concentrated in the **'Degraded'** state. This confirms that the clustering was successful in isolating periods of inefficient or problematic operation, making it a prime target for monitoring.
    
*   **Predictable Behavior:** The Random Forest model was able to forecast the inlet temperature with an RMSE of only **2.29°C**, a **71.7%** improvement over the baseline. This indicates that the machine's thermal process is stable and predictable enough for an early warning system.
    

Actionable Recommendations
--------------------------

Based on the findings, here are several actionable recommendations for the operations and maintenance teams:

#### For Monitoring & Operator Alerts:

1.  **Implement State-Based Alerts:** Trigger a high-priority alert for operators if the machine remains in the **'Degraded'** state for more than 30 minutes. This state has the highest concentration of anomalies and likely indicates an unresolved issue.
    
2.  **Monitor Correlation Drift:** The strong correlation between inlet/outlet temperatures and inlet/outlet drafts is a sign of a healthy system. Create an alert that triggers if these correlations deviate more than 10-15% from their baseline, as this points to a system imbalance (e.g., a blockage or fan issue).
    
3.  **Deploy the Forecast Model:** Use the forecasting model to create an early warning system. An alert should be raised if the predicted temperature for the next hour deviates significantly from the planned process temperature, giving operators time to react.
    

#### For Maintenance Teams:

1.  **Align Maintenance with Downtime:** Use the shutdown\_periods.csv output to schedule preventive maintenance during known long shutdown periods, minimizing impact on production.
    
2.  **Investigate Long Shutdowns:** Proactively investigate the root cause of any unplanned shutdown that lasts significantly longer than the average of **5.3 hours**. These events are likely indicative of more serious underlying failures.
    
3.  **Focus on High-Risk States:** Since the 'Degraded' state is a hotspot for anomalies, maintenance teams should focus their diagnostic efforts on understanding the transitions into this state.
    

#### For Future Data Collection:

1.  **Add a Material Flow Rate Sensor:** The single biggest missing piece of data is the material flow rate. Adding this sensor would allow for the direct calculation of operational efficiency for each of the identified machine states.
    
2.  **Log Operator Interventions:** Recording when operators make manual adjustments would create an invaluable labeled dataset. This could be used to train supervised machine learning models to predict failures or recommend specific operator actions.
    

File Structure
--------------

`   project_folder/  
├── task1_analysis.py  
├── data.csv  ├── outputs/  
│   
├── anomalous_periods.csv  
│   
├── clusters_summary.csv  
│   
├── forecasts.csv  
│   
└── shutdown_periods.csv  
└── plots/      
├── 01_correlation_matrix.png      
├── 02_one_week_view.png      
├── 03_one_year_view.png      
├── 04_shutdowns_one_year.png      
├── 05_clustering_optimization.png      
├── 06_clusters_visualization.png      
├── 07_anomaly_1_context.png      
├── 07_anomaly_2_context.png      
├── 07_anomaly_3_context.png      
├── 08_forecasting_results.png      
└── 09_comprehensive_summary.png   `
