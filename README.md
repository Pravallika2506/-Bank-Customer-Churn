FILENAME REFFILE '/home/u63095890/marketing/Bank Customer Churn Prediction.csv';

PROC IMPORT DATAFILE=REFFILE
	DBMS=CSV
	OUT=MARKET.project;
	GETNAMES=YES;
RUN;

PROC CONTENTS DATA=MARKET.project; RUN;


%web_open_table(MARKET.project);

----------------------------------
/*Sorting the data by time variable*/
proc sort data=market.project;
  by tenure;
run;
-----------------------------------
/*tranformation of dataset*/
data market.survival_data;
  set market.project;
  time = tenure;
  status = churn;
  /* Convert categorical variables to indicator variables */
  if country = 'France' then country_France = 1; else country_France = 0;
  if country = 'Spain' then country_Spain = 1; else country_Spain = 0;
  if country = 'Germany' then country_Germany = 1; else country_Germany = 0;
  if gender = 'Male' then gender_Male = 1; else gender_Male = 0;
  /* Add other categorical variables as needed */
run;
-----------------------------------------------
/*cox hazard model*/
proc phreg data= market.survival_data;
  class country gender;
  model time*status(0) = credit_score age balance products_number credit_card active_member estimated_salary
                        country gender / ties=efron;
run;
