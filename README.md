# Crime Analytics & Data Visualization | Power BI • SQL • Advanced DAX

# Dashboard 1: Crime Analysis Overview Dashboard**
<img width="847" height="505" alt="dashboard 1" src="https://github.com/user-attachments/assets/769d84da-a41b-463f-a12b-370abd177187" />

I developed an interactive Crime Analysis Overview dashboard using Power BI, focused on identifying crime trends, high-risk periods, and temporal patterns across multiple dimensions.

This dashboard provides:

•	Total crime volume tracking with year-over-year comparison to identify growth and decline patterns

•	Monthly and weekday heatmap analysis to highlight crime intensity across time, enabling rapid identification of peak risk periods

•	Time-group based crime distribution to classify incidents into operationally meaningful time windows

•	Month-over-Month trend indicators using advanced DAX for percentage change and conditional color intelligence

•	Geographical crime distribution through an interactive map to visualize regional crime concentration

•	Dynamic slicers and context-aware filtering for crime type and country selection

**Key technical highlights:**

•	Advanced DAX measures for MoM change detection, Min/Max highlighting, and context preservation using ALLSELECTED

•	Conditional formatting to surface risk signals visually (high vs low crime periods)

•	Scalable data modeling and time intelligence design, suitable for Fabric Lakehouse ingestion

This dashboard is designed as a high-level decision support view, allowing stakeholders to quickly understand where, when, and how crime activity fluctuates.

# Dashboard 2: Crime Time Group Drill-Through & Behavioral Analysis
<img width="839" height="504" alt="dashboard 2" src="https://github.com/user-attachments/assets/fc2a67b1-f811-48e3-bcce-23eeeadc91ab" />

I built a deep-dive drill-through dashboard to support granular crime analysis by weekday and time group, enabling focused investigation into high-risk hours and offense behavior patterns.

This dashboard enables:

•	Drill-through analysis from the overview dashboard into specific weekday and time-group contexts

•	Crime category distribution analysis to understand dominant offense types during high-risk time windows

•	Identification of dangerous vs low-crime time periods, supported by visual indicators and trend markers

•	Fine-grained time-series analysis showing crime fluctuation within narrow time intervals

•	Contextual insights for operational planning, such as patrol timing and resource allocation

**Technical and analytical strengths demonstrated:**

•	Advanced DAX logic to detect high-risk and low-risk time points dynamically

•	Use of trend analytics to differentiate normal variation from risk spikes

•	Strong application of Power BI interaction patterns (drill-through, tooltips, cross-filtering)

•	Clean semantic modeling to ensure consistent results across drill paths

This dashboard acts as an analytical investigation layer, complementing the overview dashboard by enabling root-cause analysis and time-based risk evaluation.

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
# DAX AND MEASURES 
```
(Value) Highlight MinMax = 
VAR _Table =
    CALCULATETABLE(
        ADDCOLUMNS(
            SUMMARIZE(
                'Crimes data',
                'Crimes data'[Crime Time]
            ),
            "@CrimeValue", [Total Crimes]
        ),
        ALLSELECTED ()
    )

VAR _MinCrime =
    MINX ( _Table, [@CrimeValue] )

VAR _MaxCrime =
    MAXX ( _Table, [@CrimeValue] )

VAR _CurrentCrimes =
    [Total Crimes]

RETURN
SWITCH(
    TRUE(),
    _CurrentCrimes = _MinCrime, _CurrentCrimes,
    _CurrentCrimes = _MaxCrime, _CurrentCrimes,
    BLANK()
)& " "
```
```
Blank measure(month) = MAXX(ALL('Date Table'[month]), [Total Crimes]/20) 
```
```
CF (Year) = 
VAR _PrevYear= CALCULATE(
    [Total Crimes], 
    sameperiodlastyear('Date Table'[Date])
    )
VAR _YoYchange= 
    IF(_PrevYear<>BLANK(),[Total Crimes]-_PrevYear,BLANK())
RETURN
SWITCH(
    TRUE(),
    _YoYchange=0, "Grey",
    _YoYchange>=1, "Green",
    _YoYchange<=1, "Red",
    BLANK()
)
```
```
Blank measure(Year) = 200
```
```
CF(Month) = 
VAR _pct_MoMChange =
    IF(
    DIVIDE([Total Crimes]-[Crime Prev Month], [Crime Prev Month])<>BLANK(),
    DIVIDE([Total Crimes]-[Crime Prev Month], [Crime Prev Month]), 
    BLANK()
    )
VAR _Negativepct = -0.01

RETURN
SWITCH(
    TRUE(),
    _pct_MoMChange = 0, "Grey",
    _pct_MoMChange >= _Negativepct, "Green",
    _pct_MoMChange < _Negativepct, "Red",
    BLANK()
)
```
```
Crime Prev Month = CALCULATE(
    [Total Crimes], 
   DATEADD('Date Table'[Date], -1, MONTH)
)
```
```
Crimes Resolved = 
DIVIDE(CALCULATE([Total Crimes],'Crimes data'[Resolved]=1), [Total Crimes])
```
```
Crimes Unresolved = 
DIVIDE(CALCULATE([Total Crimes],'Crimes data'[Resolved]=0), [Total Crimes])
```
```
Drill Through = 
 VAR _weekday= SELECTEDVALUE('Date Table'[weekday])
 VAR _TimeGroup= SELECTEDVALUE('Crimes data'[Time Group])

 RETURN
 "The Entire chart in this page is associated with " & " " &
   SWITCH(
    TRUE(),
    _TimeGroup<>BLANK(),_TimeGroup,
    _weekday<>BLANK(), upper(_weekday),
    BLANK()
   )
```
```
Highlight MinMax = 
VAR _Table =
    CALCULATETABLE(
        VALUES ( 'Crimes data'[Crime Time] ),
        ALLSELECTED ( 'Crimes data'[Crime Time] )
    )

VAR _MinCrime =
    MINX ( _Table, [Total Crimes] )

VAR _MaxCrime =
    MAXX ( _Table, [Total Crimes] )

VAR _CurrentCrimes =
    [Total Crimes]

RETURN
SWITCH(
    TRUE(),
    _CurrentCrimes = _MinCrime, 0,
    _CurrentCrimes = _MaxCrime, 1,
    BLANK()
)
```
```
Label (Year) = 
VAR _PrevYear= CALCULATE(
    [Total Crimes], 
    sameperiodlastyear('Date Table'[Date])
    )
VAR _YoYchange= 
    IF(_PrevYear<>BLANK(),[Total Crimes]-_PrevYear,BLANK())
RETURN
SWITCH(
    TRUE(),
    _YoYchange=0, _YoYchange & "-",
    _YoYchange>=1, "▲" & _YoYchange,
    _YoYchange<=1, "▼" & _YoYchange,
    BLANK()
)
```
```
Label(Month) = 
VAR _pct_MoMChange=
IF(
    DIVIDE([Total Crimes]-[Crime Prev Month], [Crime Prev Month])<>BLANK(),
DIVIDE([Total Crimes]-[Crime Prev Month], [Crime Prev Month]), 
BLANK()
)
VAR _Negativepct= -0.01

RETURN
SWITCH(
    TRUE(),
    _pct_MoMChange=0, _pct_MoMChange & "-",
    _pct_MoMChange>=_Negativepct,FORMAT(_pct_MoMChange,"0.0%") & "▲",
    _pct_MoMChange<=_Negativepct,FORMAT(_pct_MoMChange,"0.0%") & "▼",
    BLANK()
    )
```
```
Total Crimes = COUNTROWS('Crimes data')
```
```
Time Group = 
SWITCH(
    TRUE(),
    MOD(HOUR('Crimes data'[Crime Time]),24)>=0 &&  MOD(HOUR('Crimes data'[Crime Time]),24)<=3, "12:00 AM-2:59 AM",
    MOD(HOUR('Crimes data'[Crime Time]),24)>=3 &&  MOD(HOUR('Crimes data'[Crime Time]),24)<=6, "3:00 AM-5:59 AM",
    MOD(HOUR('Crimes data'[Crime Time]),24)>=6 &&  MOD(HOUR('Crimes data'[Crime Time]),24)<=9, "6:00 AM-8:59 AM",
    MOD(HOUR('Crimes data'[Crime Time]),24)>=9 &&  MOD(HOUR('Crimes data'[Crime Time]),24)<=12, "9:00 AM-11:59 PM",
    MOD(HOUR('Crimes data'[Crime Time]),24)>=12 &&  MOD(HOUR('Crimes data'[Crime Time]),24)<=15, "12:00 PM-14:59 PM",
    MOD(HOUR('Crimes data'[Crime Time]),24)>=15 &&  MOD(HOUR('Crimes data'[Crime Time]),24)<=18, "15:00 PM-17:59 PM",
    MOD(HOUR('Crimes data'[Crime Time]),24)>=18 &&  MOD(HOUR('Crimes data'[Crime Time]),24)<=21, "18:00 PM-20:59 PM",
    MOD(HOUR('Crimes data'[Crime Time]),24)>=21 &&  MOD(HOUR('Crimes data'[Crime Time]),24)<=24, "21:00 PM-23:59 PM",
    BLANK()
)
```
```
Date Table = 
VAR _mindate= YEAR(MIN('Crimes data'[Crime Date]))
VAR _maxdate= YEAR(MAX('Crimes data'[Crime Date]))
RETURN
ADDCOLUMNS(
    FILTER(
        CALENDARAUTO(), 
    YEAR([Date])>=_mindate &&
    YEAR([Date])<=_maxdate
    ),
    "year", YEAR([Date]),
    "month", FORMAT([Date],"mmm"),
    "monthNum", MONTH([Date]),
    "weekday", FORMAT([Date],"ddd"),
    "weeknum",WEEKDAY([Date])
)
```


