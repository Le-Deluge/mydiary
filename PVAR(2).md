// Stata do‑file: Panel VAR 및 패널 그레인저 인과검정

clear all
set more off

// 0. 작업 폴더 설정 (필요시 변경)
cd "C:\Users\ksmw9\Desktop\상상과 창조\DB 논문 공모전\master_csv"

// 1. CSV 파일 불러오기
import delimited "panel_data_clean1.csv", clear
list in 1/5

// 패널 VAR 분석 ===================================================================================


// 3. 변수명 재설정
// MD제공용_가구고유번호 -> id, 조사연도 -> time, 원리금연체여부 -> Y, new_interaction -> X
rename md제공용_가구고유번호 id
rename 조사연도 time
rename 원리금연체여부 Y
rename new_interaction X

// 4. 조사연도 2017 ~ 2023로 제한
keep if time >= 2017 & time <= 2023

// 5. 필요한 열만 선택: id, time, Y, X
keep id time Y X

// 6. time은 연도 정보이므로 1월 1일 날짜로 변환
gen date = mdy(1, 1, time)
format date %td

// 7. 균형 패널 데이터 생성: 각 id별로 관측횟수가 5회 이상인 자료 선택
bysort id: gen count = _N
keep if count >= 5
drop count
sort id date

// 8. 패널 데이터 구조 설정
xtset id date

// 9. pvar 명령 설치 
ssc install st0455, replace

// 10. X에 대해 ln(X+1) 변환 수행
gen ints = ln(X*10000)

// 11. xtset 재확인 및 lag 변수 생성 
xtset id time
bysort id: gen lagY = L.Y
list id time Y lagY in 1/10  // lagY가 잘 생성되었는지 확인

// 12. Panel VAR 모형 추정 (VAR(2))
//     Y, lnX, one 변수를 사용하여 2시차 VAR 모형 추정
pvar Y ints, lags(2)

// 13. Panel VAR 그레인저 인과검정 수행 ============================================================
pvargranger

// ------------------------------------------------------------
// End of do‑file
