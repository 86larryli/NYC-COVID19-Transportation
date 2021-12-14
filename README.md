# ETL / Cleaning:

## Data Source 1 (Rachel Xing @RuixuanXing): New York State Statewide COVID-19 Testing

### Input Location:

- Put `input/covid/New_York_State_Statewide_COVID-19_Testing.csv` in HDFS

### Commands:

1. In `etl_code/covid/clean_1`, run:

    - `hadoop jar clean.jar Clean input/New_York_State_Statewide_COVID-19_Testing.csv {YOUR_COVID_OUTPUT_PATH_1}`
    
    - Rename the previous output: `hdfs dfs -mv {YOUR_COVID_OUTPUT_PATH_1}/part-r-00000 {YOUR_COVID_OUTPUT_PATH_1}/cleaned.txt`
    
    - The cleaned output is `{YOUR_COVID_OUTPUT_PATH_1}/cleaned.txt` in HDFS.


2. In `etl_code/covid/clean_2`, run:

    - `hadoop jar clean.jar Clean input/New_York_State_Statewide_COVID-19_Testing.csv {YOUR_COVID_OUTPUT_PATH_2}`

    - Rename Output: `hdfs dfs -mv {YOUR_COVID_OUTPUT_PATH_2}/part-r-00000 {YOUR_COVID_OUTPUT_PATH_2}/whole.csv`

    - The cleaned output is `{YOUR_COVID_OUTPUT_PATH_2}/whole.csv` in HDFS.


## Data Source 2 (Edward Zhu @EddieSource): New York City TLC Trip Record Data

### Input Location: 

- Put `input/fhv` in HDFS

### Commands:

1. In `etl_code/fhv`, run:

    - `hadoop jar clean.jar Clean input/fhv {YOUR_FHV_OUTPUT_PATH}`

    - The cleaned output path is `{YOUR_FHV_OUTPUT_PATH}/part-r-00000` in HDFS.


## Data Source 3 (Larry Li @86larryli): MTA Turnstile Data

### Input Location: 

- `input/turnstile_data` in HDFS

### Commands:

1. In `etl_code/mta`, run:

    - `hadoop jar clean.jar Clean input/turnstile_data {YOUR_MTA_OUTPUT_PATH}`

    - The cleaned output path is `{YOUR_MTA_OUTPUT_PATH}/part-r-00000` in HDFS.

-----

# Profiling:

## Data Source 1 (Rachel Xing @RuixuanXing): New York State Statewide COVID-19 Testing

### Input Location:

- Before Cleaning: `input/New_York_State_Statewide_COVID-19_Testing.csv` in HDFS

- After Cleaning 1: `{YOUR_COVID_OUTPUT_PATH_1}/cleaned.txt` in HDFS

- After Cleaning 2: `{YOUR_COVID_OUTPUT_PATH_2}/whole.csv` in HDFS

### Commands:

1. Before Cleaning: In `profiling_code/covid`, run:

    - `hadoop jar countRecs.jar CountRecs input/New_York_State_Statewide_COVID-19_Testing.csv {YOUR_COVID_PROF_OUTPUT_PATH_1}`

    - The output is `{YOUR_COVID_PROF_OUTPUT_PATH_1}/part-r-00000` in HDFS.

2. After Cleaning 1: In `profiling_code/covid`, run:

    - `hadoop jar countRecs.jar CountRecs {YOUR_COVID_OUTPUT_PATH_1}/cleaned.txt {YOUR_COVID_PROF_OUTPUT_PATH_2}`

    - The output is `{YOUR_COVID_PROF_OUTPUT_PATH_2}/part-r-00000` in HDFS.

2. After Cleaning 2: In `profiling_code/covid`, run:

    - `hadoop jar countRecs.jar CountRecs {YOUR_COVID_OUTPUT_PATH_2}/whole.csv {YOUR_COVID_PROF_OUTPUT_PATH_3}`

    - The output is `{YOUR_COVID_PROF_OUTPUT_PATH_3}/part-r-00000` in HDFS.

## Data Source 2 (Edward Zhu @EddieSource): New York City TLC Trip Record Data

### Input Location: 

- `input/fhv` in HDFS

### Commands:

1. In `profiling_code/fhv`, run:

    - `hadoop jar countRecs.jar CountRecs {YOUR_FHV_OUTPUT_PATH} {YOUR_FHV_PROF_OUTPUT_PATH}`

    - The output is `{YOUR_FHV_PROF_OUTPUT_PATH}/part-r-00000` in HDFS.

2. In `profiling_code/fhv`, run:

    - Open `pyspark` shell: `pyspark --deploy-mode client`
    
    - Run the script in `pyspark` shell: `profiling_code/fhv/pyspark_script.py`

## Data Source 3 (Larry Li @86larryli): MTA Turnstile Data

### Input Location:

- `{YOUR_MTA_OUTPUT_PATH}/part-r-00000` in HDFS

### Commands:

1. In `profiling_code/mta`, run:
    
    - Remove `_SUCCESS` File: `hdfs dfs -rm -r -f {YOUR_MTA_OUTPUT_PATH}/_SUCCESS`
    
    - `hadoop jar countRecs.jar CountRecs {YOUR_MTA_OUTPUT_PATH} {YOUR_MTA_PROF_OUTPUT_PATH}`

    - The output is `{YOUR_MTA_PROF_OUTPUT_PATH}/part-r-00000` in HDFS.

2. In `profiling_code/mta`, run:

    - Open `pyspark` shell: `pyspark --deploy-mode client`
    
    - Run the script in `pyspark` shell: `profiling_code/mta/pyspark_script.txt`

-----

# Ingestion:

## Data Source 1 (Rachel Xing @RuixuanXing): New York State Statewide COVID-19 Testing

### Commands:

1. Remove `_SUCCESS` File: `hdfs dfs -rm -r -f {YOUR_COVID_OUTPUT_PATH_1}/_SUCCESS`

2. Open `Hive` Shell: `beeline`

3. Run the script in `Hive` shell: `data_ingest/covid_create_table.sql`

## Data Source 2 (Edward Zhu @EddieSource): New York City TLC Trip Record Data

### Commands:

1. Open `Hive` Shell: `beeline`

2. Run the script in `Hive` shell: `data_ingest/fhv_create_table.sql`

## Data Source 3 (Larry Li @86larryli): MTA Turnstile Data

### Commands:

1. Open `Hive` Shell: `beeline`

2. Run the script in `Hive` shell: `data_ingest/mta_create_table_1.sql`

3. Export a Query into `.csv` File Using Hive: `hive -e 'USE mta; SELECT station, log_date, SUM(entries_count) AS entries_count FROM turnstile_group_by_scp GROUP BY station, log_date ORDER BY station, log_date;' | sed 's/[\t]/,/g;  s/NULL//g' > turnstile_group_by_station.csv`

4. Upload the File into HDFS: `hdfs dfs -put turnstile_group_by_station.csv output/mta/by_station/`

5. Run the script in `Hive` shell: `data_ingest/mta_create_table_2.sql`

6. Export a Query into `.csv` File Using Hive: `hive -e 'USE mta; SELECT x.station, x.log_date, x.entries_count - LAG(x.entries_count) OVER(PARTITION BY station ORDER BY x.log_date) AS entries_count FROM turnstile_group_by_station x WHERE entries_count IS NOT NULL;' | sed 's/[\t]/,/g; s/NULL//g' > turnstile_group_by_station_daily_new.csv`

7. Upload the File into HDFS: `hdfs dfs -put turnstile_group_by_station_daily_new.csv output/mta/by_station_daily_new/`

8. Run the script in `Hive` shell: `data_ingest/mta_create_table_3.sql`

9. Export a Query into `.csv` File Using Hive: `hive -e 'USE mta; SELECT * FROM turnstile_group_by_station_daily_new WHERE entries_count IS NOT NULL AND entries_count > 0 AND entries_count < 28018;' | sed 's/[\t]/,/g; s/NULL//g' > turnstile_group_by_station_remove_outliers.csv`

10. Upload the File into HDFS: `hdfs dfs -put turnstile_group_by_station_remove_outliers.csv output/mta/by_station_remove_outliers/`

11. Run the script in `Hive` shell: `data_ingest/mta_create_table_4.sql`

12. Export a Query into `.csv` File Using Hive: `hive -e 'USE mta; SELECT log_date, SUM(entries_count) AS entries_count FROM turnstile_group_by_station_remove_outliers GROUP BY log_date ORDER BY log_date;' | sed 's/[\t]/,/g; s/NULL//g' > mta.csv`

13. Upload the File into HDFS: `hdfs dfs -put mta.csv output/mta/mta/`

14. Run the script in `Hive` shell: `data_ingest/mta_create_table_5.sql`

-----

# Analysis

## Data Source 1 (Rachel Xing @RuixuanXing): New York State Statewide COVID-19 Testing

### Commands:

1. Load Python Module: `module load python/gcc/3.7.9`

2. Open `pyspark` Shell: `pyspark --deploy-mode client`

3. Run the Commands in `pyspark` Shell: `ana_code/covid_analysis.py`

    - **Screenshots of Running the Above Script is in `screenshots\covid_analysis[1-4].png`**

## Data Source 2 (Edward Zhu @EddieSource): New York City TLC Trip Record Data

### Commands:

1. Open `Hive` Shell: `beeline`

2. Run the Script in Hive Shell: `ana_code/fhv_analysis_hive.sql`

    - **Screenshots of Running the Above Script is in `screenshots\trips_analysis[1-4].png`**

3. Exist `Hive` Shell.

4. Load Python Module: `module load python/gcc/3.7.9`

5. Open `pyspark` Shell: `pyspark --deploy-mode client`

6. Run the Commands in `pyspark` Shell: `ana_code/fhv_analysis_pyspark.py`

    - **Screenshots of Running the Above Commands is in `screenshots\trips_analysis[5-6].png`**
   
## Data Source 3 (Larry Li @86larryli): MTA Turnstile Data

1. Open `Hive` Shell: `beeline`

2. Run the Script in Hive Shell: `ana_code/mta_analysis.sql`

    - **Screenshots of Running the Above Script is in `screenshots\mta_analysis[1-4].png`**

## Merged Analysis

1. *In order to merge the 3 tables:*

    - *The ingestion procedures are repeated using the same commands and input as stated above.*

    - *After the ingestion steps, {YOUR_DATABASE} should have 3 tables: `covid`, `date_trips`, `mta`.*

2. Run the Script in Hive Shell: `ana_code/merge_create_join_table.sql`

    - **Screenshots of Running the Above Script is in `screenshots\merge_create_table1-[1-3]`**


3. Export the Joined Table into `.csv` File Using Hive: `hive -e 'USE mta; SELECT * FROM project WHERE 1=1;' | sed 's/[\t]/,/g; s/NULL//g' > project.csv`

4. Upload the File into HDFS: `hdfs dfs -put project.csv output/merged/`

5. Load Python Module: `module load python/gcc/3.7.9`

6. Open `pyspark` Shell: `pyspark --deploy-mode client`

7. Run the Commands in `pyspark` Shell: `ana_code/merge_analysis.py`

    - **Screenshots of Running the Above Commands is in `screenshots\merge_analysis[1-5]`**
