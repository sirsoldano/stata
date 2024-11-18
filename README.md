## 被験者解析
~~~
set more off
drop _all
local situation animation
local diagnosis diag
local month age
local iq kabc_mps
local srs srs_total_t
use "ふぁいるぱす.dta"
// import excel "C:/brainstorm_db/AniRest/data/_afterMat/MatToExcel/`situation'/20.xlsx", sheet("Sheet1") firstrow
label define `diagnosis' 0 "TD" 1 "ASD" , replace
label define sex 0 "female" 1 "male" , replace

generate str band = ""
generate str command = ""
generate str respvar = ""
generate str expvar = ""
generate str cond = ""
generate int multi = .
generate int weight = .
generate int robust = .
set pformat %5.4f, perm
generate t = .
generate p = .
generate nInt = .
generate nCon = .

// chi2 with sex and condition
insobs 1, before(1)
replace command = "tabulate `diagnosis' sex if(`situation'==1) ,chi" in 1
tabulate `diagnosis' sex if(`situation'==1) ,chi
replace respvar = "sex" in 1
replace expvar = "`diagnosis'" in 1
replace t = r(chi2) in 1
replace p = r(p) in 1

// ttest with month and condition
insobs 1, before(1)
replace command = "ttest `month' if(`situation'==1), by(`diagnosis')" in 1
ttest `month' if(`situation'==1), by(`diagnosis')
replace respvar = "`month'" in 1
replace expvar = "`diagnosis'" in 1
replace t = r(t) in 1
replace p = r(p) in 1

// ttest with psyExam and condition
foreach e of varlist `iq' `srs'{
	display "`e'"
	insobs 1, before(1)
	replace command = "ttest `e' if(`situation'), by(`diagnosis')" in 1
	ttest `e' if(`situation'), by(`diagnosis')
	replace respvar = "`e'" in 1
	replace expvar = "`diagnosis'" in 1
	replace t = r(t) in 1
	replace p = r(p) in 1
}
drop id_num-gamma_sw
export excel using "ふぁいるぱす.xls", firstrow(variables) replace
drop _all
~~~

## 交互作用込解析
~~~
set more off
drop _all
use "ふぁいるぱす.dta"
generate str band = ""
generate str index = ""
generate str expvar = ""
generate coef = .
generate se = .
generate t = .
generate p = .
foreach band in  "delta" "theta" "alpha" "beta" "gamma"{
	foreach v in "sw" "cc" "pl"{
		insobs 1, before(1)
		replace band = "`band'" in 1
		replace index = "`v'" in 1
		replace expvar = "Diagnosis#Condition" in 1
		meglm `band'_`v' diag##animation age sex ||id_num:,vce(robust)
		matrix res = r(table)
		replace coef = res[1,8] in 1
		replace se = res[2,8] in 1
		replace t = res[3,8] in 1
		replace p = res[4,8] in 1
	}
}
drop id_num-gamma_sw
export excel using "ふぁいるぱす.xls", firstrow(variables) replace
drop _all
~~~
