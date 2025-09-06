# Faers-Research-data


Step 0: Load dataset and create a library
/* Set library */
libname mydata "/home/u64263069/faers";

/* Load your cleaned final dataset */
data final;
    set mydata.final_statin;
run;

Step 1: Sort dataset for transpose
proc sort data=final;
    by primaryid age sex;
run;

Step 2: Transpose Atorvastatin PTs to long format
proc transpose data=final out=ATOR_long prefix=ATORpt;
    by primaryid age sex;
    var ATORpt:; /* All columns starting with ATORpt */
run;


/* Step 0: Sort first */
proc sort data=final;
    by primaryid age sex;
run;

/* Step 1: Transpose Atorvastatin PTs (character variables) */
proc transpose data=final out=ATOR_long(drop=_LABEL_);
    by primaryid age sex;
    var ATORpt:;
run;

data ATOR_long2;
    set ATOR_long;
    /* Transposed character variable is called _NAME_ or _CHARACTER_1, check proc contents */
    pt = _NAME_; /* If character values are in _NAME_ */
    if pt ne '';
    drop _NAME_;
run;

/* Step 2: Transpose Rosuvastatin PTs */
proc transpose data=final out=ROSU_long(drop=_LABEL_);
    by primaryid age sex;
    var ROSUpt:;
run;

data ROSU_long2;
    set ROSU_long;
    pt = _NAME_;
    if pt ne '';
    drop _NAME_;
run;







data AE_long;
    set ATOR_long2 ROSU_long2;

    /* Create age groups */
    if age ne . then do;
        if age < 18 then age_group="0-17";
        else if 18 <= age < 45 then age_group="18-44";
        else if 45 <= age < 65 then age_group="45-64";
        else if age >= 65 then age_group="65+";
    end;
run;

proc sql;
    create table ae_counts as
    select age_group, sex, pt,
           count(distinct primaryid) as a
    from AE_long
    group by age_group, sex, pt;
quit;

/* Step 4b: Compute total reports per strata (needed for b, c, d) */
proc sql;
    create table total_counts as
    select age_group, sex, count(distinct primaryid) as total
    from AE_long
    group by age_group, sex;
quit;



Step 5: Merge counts and compute b, c, d placeholders
data ae_counts2;
    merge ae_counts total_counts;
    by age_group sex;

    /* Approximate b = reports with drug but NOT this PT */
    b = total - a;

    /* Placeholders for c & d (to be refined with full AE matrix) */
    c = .;
    d = .;
run;

Step 6: Calculate DPA metrics (ROR, PRR, IC)
data dpa_results;
    set ae_counts2;

    /* Only calculate metrics if a>0 and b>0 */
    if a>0 and b>0 and c>0 and d>0 then do;

        /* Reporting Odds Ratio (ROR) */
        ROR = (a*d)/(b*c);
        ROR_log = log(ROR);
        ROR_se  = sqrt(1/a + 1/b + 1/c + 1/d);
        ROR_LCL = exp(ROR_log - 1.96*ROR_se);
        ROR_UCL = exp(ROR_log + 1.96*ROR_se);

        /* Proportional Reporting Ratio (PRR) */
        PRR = (a / (a+b)) / (c / (c+d));

        /* Information Component (IC, BCPNN) */
        N = a + b + c + d;
        p_obs = a / N;
        p_exp = ((a+b)*(a+c)) / (N*N);
        if p_obs>0 and p_exp>0 then IC = log2(p_obs / p_exp);
        else IC = .;

        /* EBGM placeholder for later calculation in R */
        EBGM = .;
    end;
run;

Step 7: Save the results
/* Save as SAS dataset */
data mydata.dpa_results;
    set dpa_results;
run;

/* Export as CSV for EBGM analysis in R */
proc export data=dpa_results
    outfile="/home/u64263069/faers/dpa_results.csv"
    dbms=csv
    replace;
run;

































Step 0: Load dataset and create a library
/* Set library */
libname mydata "/home/u64263069/faers";

/* Load your cleaned final dataset */
data final;
    set mydata.final_statin;
run;

Step 1: Sort dataset for transpose
proc sort data=final;
    by primaryid age sex;
run;

Step 2: Transpose Atorvastatin PTs to long format
proc transpose data=final out=ATOR_long prefix=ATORpt;
    by primaryid age sex;
    var ATORpt:; /* All columns starting with ATORpt */
run;


/* Step 0: Sort first */
proc sort data=final;
    by primaryid age sex;
run;

/* Step 1: Transpose Atorvastatin PTs (character variables) */
proc transpose data=final out=ATOR_long(drop=_LABEL_);
    by primaryid age sex;
    var ATORpt:;
run;

data ATOR_long2;
    set ATOR_long;
    /* Transposed character variable is called _NAME_ or _CHARACTER_1, check proc contents */
    pt = _NAME_; /* If character values are in _NAME_ */
    if pt ne '';
    drop _NAME_;
run;

/* Step 2: Transpose Rosuvastatin PTs */
proc transpose data=final out=ROSU_long(drop=_LABEL_);
    by primaryid age sex;
    var ROSUpt:;
run;

data ROSU_long2;
    set ROSU_long;
    pt = _NAME_;
    if pt ne '';
    drop _NAME_;
run;







data AE_long;
    set ATOR_long2 ROSU_long2;

    /* Create age groups */
    if age ne . then do;
        if age < 18 then age_group="0-17";
        else if 18 <= age < 45 then age_group="18-44";
        else if 45 <= age < 65 then age_group="45-64";
        else if age >= 65 then age_group="65+";
    end;
run;

proc sql;
    create table ae_counts as
    select age_group, sex, pt,
           count(distinct primaryid) as a
    from AE_long
    group by age_group, sex, pt;
quit;

/* Step 4b: Compute total reports per strata (needed for b, c, d) */
proc sql;
    create table total_counts as
    select age_group, sex, count(distinct primaryid) as total
    from AE_long
    group by age_group, sex;
quit;



Step 5: Merge counts and compute b, c, d placeholders
data ae_counts2;
    merge ae_counts total_counts;
    by age_group sex;

    /* Approximate b = reports with drug but NOT this PT */
    b = total - a;

    /* Placeholders for c & d (to be refined with full AE matrix) */
    c = .;
    d = .;
run;

Step 6: Calculate DPA metrics (ROR, PRR, IC)
data dpa_results;
    set ae_counts2;

    /* Only calculate metrics if a>0 and b>0 */
    if a>0 and b>0 and c>0 and d>0 then do;

        /* Reporting Odds Ratio (ROR) */
        ROR = (a*d)/(b*c);
        ROR_log = log(ROR);
        ROR_se  = sqrt(1/a + 1/b + 1/c + 1/d);
        ROR_LCL = exp(ROR_log - 1.96*ROR_se);
        ROR_UCL = exp(ROR_log + 1.96*ROR_se);

        /* Proportional Reporting Ratio (PRR) */
        PRR = (a / (a+b)) / (c / (c+d));

        /* Information Component (IC, BCPNN) */
        N = a + b + c + d;
        p_obs = a / N;
        p_exp = ((a+b)*(a+c)) / (N*N);
        if p_obs>0 and p_exp>0 then IC = log2(p_obs / p_exp);
        else IC = .;

        /* EBGM placeholder for later calculation in R */
        EBGM = .;
    end;
run;

Step 7: Save the results
/* Save as SAS dataset */
data mydata.dpa_results;
    set dpa_results;
run;

/* Export as CSV for EBGM analysis in R */
proc export data=dpa_results
    outfile="/home/u64263069/faers/dpa_results.csv"
    dbms=csv
    replace;
run;






libname mydata "/home/u64263069/faers";

data final;
    set mydata.final_statin;
run;
Step 1: Create age groups
sas
Copy code
data final2;
    set final;
    if age ne . then do;
        if age < 18 then age_group="0-17";
        else if 18 <= age < 45 then age_group="18-44";
        else if 45 <= age < 65 then age_group="45-64";
        else if age >= 65 then age_group="65+";
    end;
run;
Step 2: Transpose PTs for each drug to long format
Atorvastatin
sas
Copy code
proc sort data=final2;
    by primaryid age sex;
run;

proc transpose data=final2 out=ATOR_long(drop=_LABEL_);
    by primaryid age sex;
    var ATORpt:;
run;

data ATOR_long2;
    set ATOR_long;
    pt = COL1;
    if pt ne '';
    drop _NAME_ COL1;
    drug="Atorvastatin";
run;
Rosuvastatin
sas
Copy code
proc transpose data=final2 out=ROSU_long(drop=_LABEL_);
    by primaryid age sex;
    var ROSUpt:;
run;

data ROSU_long2;
    set ROSU_long;
    pt = COL1;
    if pt ne '';
    drop _NAME_ COL1;
    drug="Rosuvastatin";
run;
Step 3: Combine both drugs into one long dataset
sas
Copy code
data AE_long;
    set ATOR_long2 ROSU_long2;
run;







/* Step 0: Library and load */
libname mydata "/home/u64263069/faers";

data final;
    set mydata.final_statin;
run;

/* Step 1: Create age_group (only keep 65+) */
data final2;
    set final;
    if age >= 65 then age_group="65+";
run;

/* Step 2: Transpose Atorvastatin PTs */
proc sort data=final2;
    by primaryid age sex age_group;
run;

proc transpose data=final2 out=ATOR_long(drop=_LABEL_);
    by primaryid age sex age_group;
    var ATORpt:;
run;

data ATOR_long2;
    set ATOR_long;
    pt = COL1;
    if pt ne '';
    drop _NAME_ COL1;
    drug="Atorvastatin";
run;

/* Step 3: Transpose Rosuvastatin PTs */
proc transpose data=final2 out=ROSU_long(drop=_LABEL_);
    by primaryid age sex age_group;
    var ROSUpt:;
run;

data ROSU_long2;
    set ROSU_long;
    pt = COL1;
    if pt ne '';
    drop _NAME_ COL1;
    drug="Rosuvastatin";
run;

/* Step 4: Combine both drugs */
data AE_long;
    set ATOR_long2 ROSU_long2;
run;

/* Step 5: Count 'a' (reports with drug + PT, stratified) */
proc sql;
    create table counts_a as
    select drug, age_group, sex, pt,
           count(distinct primaryid) as a
    from AE_long
    group by drug, age_group, sex, pt;
quit;

/* Step 6: Total reports per (drug, age_group, sex) */
proc sql;
    create table total_counts as
    select drug, age_group, sex, count(distinct primaryid) as total
    from AE_long
    group by drug, age_group, sex;
quit;

/* Step 7: Merge counts and calculate b (c & d placeholders for now) */
data ae_counts2;
    merge counts_a total_counts;
    by drug age_group sex;

    b = total - a; /* reports with drug but not this PT */
    c = .; 
    d = .;
run;

/* Step 8: Save intermediate results */
data mydata.ae_counts2;
    set ae_counts2;
run;







/*=============== 4) Build exact 2×2 counts (a, b, c, d) ===============*/

/* a = reports WITH (drug & pt) per stratum */
proc sql;
  create table counts_a as
  select drug, age_group, sex, pt,
         count(distinct primaryid) as a
  from AE_long
  group by drug, age_group, sex, pt;
quit;

/* total_drug = all reports WITH the drug (any pt) per stratum */
proc sql;
  create table totals_drug as
  select drug, age_group, sex,
         count(distinct primaryid) as total_drug
  from AE_long
  group by drug, age_group, sex;
quit;

/* total_all = all reports across BOTH drugs per stratum */
proc sql;
  create table totals_all as
  select age_group, sex,
         count(distinct primaryid) as total_all
  from AE_long
  group by age_group, sex;
quit;

/* event_total = all reports WITH this pt (any drug) per stratum */
proc sql;
  create table totals_pt as
  select age_group, sex, pt,
         count(distinct primaryid) as event_total
  from AE_long
  group by age_group, sex, pt;
quit;

/* Merge and compute b, c, d */
proc sql;
  create table ae_2x2 as
  select a.drug, a.age_group, a.sex, a.pt,
         a.a,
         d.total_drug,
         s.total_all,
         p.event_total,
         /* b = with drug, without this PT */
         (d.total_drug - a.a) as b,
         /* c = with PT, without this drug */
         (p.event_total - a.a) as c,
         /* d = everything else in the stratum */
         (s.total_all - calculated a - calculated b - calculated c) as d
  from counts_a a
  left join totals_drug d
    on a.drug=d.drug and a.age_group=d.age_group and a.sex=d.sex
  left join totals_all s
    on a.age_group=s.age_group and a.sex=s.sex
  left join totals_pt p
    on a.age_group=p.age_group and a.sex=p.sex and a.pt=p.pt
  ;
quit;
