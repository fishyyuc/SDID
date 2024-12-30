# SDID
This repository is for storing codes of SDID, as well as other codes relevant to analysis.
import excel "C:\Users\cindy\OneDrive\桌面\學術-LAPTOP-T8S7SF39\06 DID\rawdata\Filtered_Dataset.xlsx", sheet("Sheet1") firstrow clear


rename GOS_AllIndustries GOS
rename Income_TotalWages Income
rename GDP rGDP
gen GDP= rGDP/Population*1000
drop in 951/952
generate treatment = 0
replace treatment = 1 if State == 4


*使用 Variance Inflation Factor (VIF) 測試共線性：

reg GDP ETS Labor Carbon_Electricity  GOS Income
vif

reg Carbon_General ETS Labor GOS Income GDP
vif

*如果某變數的 VIF > 10，則可以考慮移除或合併該變數
*回歸結果顯示高多重共線性：
//Income_TotalWages GOS_AllIndustries 的 VIF 高，則是高度相關。
//Labor 的 VIF 為 19.38，雖然相對較低，但仍顯示一定程度的多重共線性。
//ETS Carbon_Electricity的 VIF 相對低，表示處理變數與其他控制變數的相。
//R-squared = 0.996，模型幾乎完全解釋了 GDP 的變異性，但這可能是高多重共線性和冗餘變數導致的過度擬合

*，通過主成分分析 (PCA) 提取出一個綜合指標，降低多重共線性影響
pca GOS Income 
predict PCA_GOS_Income

pca Labor GDP Income GOS
predict PCA_Labor_GDP_Income_GOS


ttest GDP ,  by(treatment)
*GDP
*model 1
sdid GDP State Year ETS, vce(placebo) reps(50) method(sdid) graph covariates(PCA_GOS_Income) g1on returnweights generate(omega1_)
*model 2
sdid GDP State Year ETS, vce(placebo) reps(50) method(sdid) graph covariates(GOS Income ) 
*model 3
sdid GDP State Year ETS, vce(placebo) reps(50) method(sdid) graph g1on returnweights generate(omega3_)
*Carbon
*model 1

sdid Carbon_General State Year ETS, vce(placebo) reps(50) method(sdid) graph covariates( PCA_Labor_GDP_Income_GOS) g1on returnweights generate(omega2_)

sdid Carbon_Industry State Year ETS, vce(placebo) reps(50) method(sdid) graph covariates(PCA_Labor_GDP_Income_GOS)g1on returnweights generate(omega3_)

sdid Carbon_Transport State Year ETS, vce(placebo) reps(50) method(sdid) graph covariates( PCA_Labor_GDP_Income_GOS)g1on returnweights generate(omega4_)

sdid Carbon_Electricity State Year ETS, vce(placebo) reps(50) method(sdid) graph covariates(PCA_Labor_GDP_Income_GOS)g1on returnweights generate(omega5_)


*model 2

sdid Carbon_General State Year ETS, vce(placebo) reps(50) method(sdid) graph covariates( Labor GDP Income GOS)

sdid Carbon_Industry State Year ETS, vce(placebo) reps(50) method(sdid) graph covariates( Labor GDP Income GOS)

sdid Carbon_Transport State Year ETS, vce(placebo) reps(50) method(sdid) graph covariates( Labor GDP Income GOS)

sdid Carbon_Electricity State Year ETS, vce(placebo) reps(50) method(sdid) graph covariates( Labor GDP Income GOS)
*model 3
sdid Carbon_General State Year ETS, vce(placebo) reps(50) method(sdid) graph 

sdid Carbon_Industry State Year ETS, vce(placebo) reps(50) method(sdid) graph 

sdid Carbon_Transport State Year ETS, vce(placebo) reps(50) method(sdid) graph 

sdid Carbon_Electricity State Year ETS, vce(placebo) reps(50) method(sdid) graph 

*placebo
generate placebo05 =0
replace placebo05=1 if State==4 & Year>=2005
generate placebo06 =0
replace placebo06=1 if State==4 & Year>=2006
generate placebo07 =0
replace placebo07=1 if State==4 & Year>=2007
generate placebo08 =0
replace placebo08=1 if State==4 & Year>=2008
generate placebo09 =0
replace placebo09 =1 if State==4 & Year>=2009
generate placebo10 =0
replace placebo10=1 if State==4 & Year>=2010
generate placebo11 =0
replace placebo11=1 if State==4 & Year>=2011
generate placebo12 =0
replace placebo12=1 if State==4 & Year>=2012
sdid Carbon_Transport State Year placebo05, vce(placebo) reps(10) method(sdid) graph
sdid Carbon_Transport State Year placebo06, vce(placebo) reps(10) method(sdid) graph
sdid Carbon_Transport State Year placebo07, vce(placebo) reps(10) method(sdid) graph
sdid Carbon_Transport State Year placebo08, vce(placebo) reps(10) method(sdid) graph
sdid Carbon_Transport State Year placebo09, vce(placebo) reps(10) method(sdid) graph
sdid Carbon_Transport State Year placebo10, vce(placebo) reps(10) method(sdid) graph
sdid Carbon_Transport State Year placebo11, vce(placebo) reps(10) method(sdid) graph
sdid Carbon_Transport State Year placebo12, vce(placebo) reps(10) method(sdid) graph

log close

*若用一般did
*平衡趨勢檢定
gen event = Year - 2012
replace event = -4 if event <= -4
replace event = 8 if event >=8

tab event, gen(eventt)
forvalues i = 1/13 {
    replace eventt`i' = 0 if eventt`i' == .
}

gen scaled_GDP = GDP / 10000 // 將原始係數除以 10000

xtset State Year
xtreg scaled_GDP eventt* Labor Carbon_Electricity PCA_GOS_Income, fe robust

graph set window fontface "Arial" // 設置圖形字體為 Arial
graph set window fontfacesans "Arial" // 設置圖形字體樣式為 Arial

coefplot, keep(eventt2 eventt3 eventt4 eventt5 eventt6 eventt7 eventt8 eventt9 ///
    eventt10 eventt11 eventt12 eventt13) ///
    coeflabels(eventt2 = "-3" eventt3 = "-2" eventt4 = "-1" eventt5 = "0" ///
    eventt6 = "1" eventt7 = "2" eventt8 = "3" eventt9 = "4" ///
    eventt10 = "5" eventt11 = "6" eventt12 = "7" eventt13 = "8")  ///
    vertical yline(0, lcolor(navy) lwidth(medium)) ///
    ytitle("Coefficient") xtitle("Time (Relative to Policy Implementation Year)") ///
    scheme(s2color) addplot(line @b @at) ///
    ciopts(recast(rcap) lcolor(navy) lpattern(dash)) ///
	 ylabel(-1 (1) 4)
gen treatment = 0
replace treatment = 1 if State == 4
gen group = cond(treatment == 1, "Treated", "Control")  // 為分組標記
egen avg_GDP = mean(GDP), by(treatment Year)
twoway (line avg_GDP Year if treatment == 1, lcolor(blue) lwidth(medium) lpattern(solid)) ///
       (line avg_GDP Year if treatment == 0, lcolor(red) lwidth(medium) lpattern(dash)), ///
       legend(order(1 "Treated" 2 "Control") ///
              col(1) position(1)) ///
       xline(2012, lcolor(black) lwidth(medium)) /// 添加干預年份的垂直線
       ytitle("GDP ") xtitle("Year") ///
       title(" GDP Trends: Treated vs Control (with Intervention)") ///
       scheme(s2color)


