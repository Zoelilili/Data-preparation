# Data-preparation
STATA

/******micro variable*********
global dd 06_18
global input "input file location"
global output "output file location"
global process "process file location"

clear all
set more off
cd "$process"

import delimited using "$input\xfile_$dd.csv"
save name_$dd.dta, replace

clear
import delimited using "$input\yfile_$dd.csv"
keep if version==4
replace choice_id=auto_choice_id if choice_id==.
rename status model_status
merge m:1 choice_id version using name_$dd.dta, nogen
sort cp_id survey_id question_id
drop if cp_id=.

/**********format time************
generate appdatenum=clock( approved_dtm,"DM20Y")
gen app_year=year(dofc(appdatenum))
gen app_month=month(dofc(appdatenum))
generate appdatenum2=date(approved_dtm,"DM20Y")
gen app_year2=year(appdatenum2)
gen app_month2=month(appdatenum2)
gen app_year3= app_year if !missing(app_year) 
replace app_year3=app_year2 if !missing(app_year2) 
gen app_month3= app_month if !missing(app_month) 
replace app_month3=app_month2 if !missing(app_month2) 
drop appdatenum appdatenum2 app_year app_year2 app_month app_month2 
rename app_year3 app_year
rename app_month3 app_month

generate creatdatenum=clock(created_dtm,"DM20Y")
gen creat_year=year(dofc(creatdatenum))
gen creat_month=month(dofc(creatdatenum))
generate creatdatenum2=date(created_dtm,"DM20Y")
gen creat_year2=year(creatdatenum2)
gen creat_month2=month(creatdatenum2)
gen creat_year3= creat_year if !missing(creat_year) 
replace creat_year3=creat_year2 if !missing(creat_year2) 
gen creat_month3= creat_month if !missing(creat_month) 
replace creat_month3=creat_month2 if !missing(creat_month2) 
drop creatdatenum creatdatenum2 creat_year creat_year2 creat_month creat_month2 
rename creat_year3 created_year
rename creat_month3 created_month

generate findatenum=clock( date_of_financials,"DM20Y")
gen fin_year=year(dofc(findatenum))
gen fin_month=month(dofc(findatenum))
generate findatenum2=date(date_of_financials,"DM20Y")
gen fin_year2=year(findatenum2)
gen fin_month2=month(findatenum2)
gen fin_year3= fin_year if !missing(fin_year) 
replace fin_year3=fin_year2 if !missing(fin_year2) 
gen fin_month3= fin_month if !missing(fin_month) 
replace fin_month3=fin_month2 if !missing(fin_month2) 
drop findatenum findatenum2 fin_year fin_year2 fin_month fin_month2 
rename fin_year3 finan_year
rename fin_month3 finan_month

/**********loop***************
foreach k of numlist 8515/8523{
preserve
drop if question_id!='k'
gen ws_downgrade'k'=dynamic_choice_id
gen ws_text'k'=question_text
gen ws_applied'k'=selected
gen ws_comment'k'=comments
save name_Q'k'_$dd.dta, replace
restore
}

/********merge the loop**********
foreach k of numlist 8515/8523 {
merge survey_id using name_Q'k'_$dd.dta, sort
drop _merge
}
