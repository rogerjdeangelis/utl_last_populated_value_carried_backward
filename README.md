# utl_last_populated_value_carried_backward
Last populated value backwards.  Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.

    Last populated value backwards

      Three solutions  (assumes last value in group is populated)

           1. Datastep set point backwards
           2. HASH (elegant - good use of hash)
           3. R locf and locb (last value carried backwards)
           4. see SAS Forum for descending sort solution

    https://tinyurl.com/ybv4wtck
    https://communities.sas.com/t5/SAS-Procedures/repeat-observation-by-id/m-p/490130

    Hash solution Novinosrin profile
    https://communities.sas.com/t5/user/viewprofilepage/user-id/138205

    INPUT
    =====

      SD1.HAVE total obs=12    |  RULES
                               |
       MONTH_                  |
      BEGIN_DT    ID      X    |  NewX
                               |
      01JAN2013    1     .     |  0.15
      01FEB2013    1     .     |  0.15
      01MAR2013    1     .     |  0.15
      01APR2013    1    0.15   |  0.15  carry backwards

      01MAY2013    1    0.90   |  0.90
      01JUN2013    1    0.34   |  0.34

      01JUL2013    2     .     |  0.09
      01AUG2013    2     .     |  0.09
      01SEP2013    2    0.09   |  0.09  carry backwards

      01OCT2013    2    0.28   |  0.28
      01NOV2013    2    0.31   |  0.31
      01DEC2013    2    0.79   |  0.79

     EXAMPLE OUTPUT
     --------------

     SD1.WANT total obs=12

       MONTH_
      BEGIN_DT     ID      X     NEWX

      01JAN2013     1     .      0.15
      01FEB2013     1     .      0.15
      01MAR2013     1     .      0.15
      01APR2013     1    0.15    0.15
      01MAY2013     1    0.90    0.90
      01JUN2013     1    0.34    0.34
      01JUL2013     2     .      0.09
      01AUG2013     2     .      0.09
      01SEP2013     2    0.09    0.09
      01OCT2013     2    0.28    0.28
      01NOV2013     2    0.31    0.31
      01DEC2013     2    0.79    0.79


    PROCESS
    =======

     1. Datastep set point backwards

        data want;

        * read table backwards and retain non missing;
        if _n_=0 then do;
          %let rc=%sysfunc(dosubl('
              data havBac;
                 do pt=nobs to 1 by -1;
                    set sd1.have point=pt nobs=nobs;
                    if x ne . then newX=x;
                    output;
                 end;
                 stop;
              run;quit;
          '));
        end;

        do pt=nobs to 1 by -1;
           set havBac point=pt nobs=nobs;
           output;
        end;
        stop;

        run;quit;


     2. HASH (elegant - good use of hash)

        data _null_;
        dcl hash H (ordered:'y') ;
           h.definekey  ("_n_") ;
           h.definedata ('month_begin_dt','id','x') ;
           h.definedone () ;

        do _n_=nobs to 1 by -1;
           set have point=_n_ nobs=nobs;
           if not missing(x) then _x=x;  * hold non missing;
           else x=_x;
           h.add();
        end;
        h.output(dataset:'want');
        stop;
        run;


     3. R locf and locb (working code - one liner)

        have$locb<-na.locf(have$X, fromLast = TRUE);


    OUTPUT
    ======

     SD1.WANT total obs=12

       MONTH_
      BEGIN_DT     ID      X     NEWX

      01JAN2013     1     .      0.15
      01FEB2013     1     .      0.15
      01MAR2013     1     .      0.15
      01APR2013     1    0.15    0.15
      01MAY2013     1    0.90    0.90
      01JUN2013     1    0.34    0.34
      01JUL2013     2     .      0.09
      01AUG2013     2     .      0.09
      01SEP2013     2    0.09    0.09
      01OCT2013     2    0.28    0.28
      01NOV2013     2    0.31    0.31
      01DEC2013     2    0.79    0.79


    *                _               _       _
     _ __ ___   __ _| | _____     __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \   / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/  | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|   \__,_|\__,_|\__\__,_|

    ;

    options validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.have;
    input month_begin_dt :mmddyy10. id x :best32.;
    format month_begin_dt date9.;
    cards4;
    1/1/2013 1 .
    2/1/2013 1 .
    3/1/2013 1 .
    4/1/2013 1 0.15
    5/1/2013 1 0.90
    6/1/2013 1 0.34
    7/1/2013 2 .
    8/1/2013 2 .
    9/1/2013 2 0.09
    10/1/2013 2 0.28
    11/1/2013 2 0.31
    12/1/2013 2 0.79
    ;;;;
    run;quit;


    *____              _       _   _
    |  _ \   ___  ___ | |_   _| |_(_) ___  _ __
    | |_) | / __|/ _ \| | | | | __| |/ _ \| '_ \
    |  _ <  \__ \ (_) | | |_| | |_| | (_) | | | |
    |_| \_\ |___/\___/|_|\__,_|\__|_|\___/|_| |_|

    ;

    %utl_submit_r64('
    library(xts);
    library(haven);
    have<-read_sas("d:/sd1/have.sas7bdat");
    have$locf<-na.locf(have$X,na.rm = FALSE);
    have$locb<-na.locf(have$X, fromLast = TRUE);
    save.image("d:/rda/work.Rdata");
    ');

    * create sas dataset from R dataframe;
    proc iml;
      submit / R;
      load("d:/rda/work.Rdata");
      endsubmit;
      run importdatasetfromr("work.want_r","have");
    run;quit;

    proc print data=want_r;
    run;quit;

                          Forward and Backward
       MONTH_
     BEGIN_DT    ID      X     LOCF    LOCB

    01JAN2013     1     .       .      0.15
    01FEB2013     1     .       .      0.15
    01MAR2013     1     .       .      0.15
    01APR2013     1    0.15    0.15    0.15
    01MAY2013     1    0.90    0.90    0.90
    01JUN2013     1    0.34    0.34    0.34
    01JUL2013     2     .      0.34    0.09
    01AUG2013     2     .      0.34    0.09
    01SEP2013     2    0.09    0.09    0.09
    01OCT2013     2    0.28    0.28    0.28
    01NOV2013     2    0.31    0.31    0.31
    01DEC2013     2    0.79    0.79    0.79




