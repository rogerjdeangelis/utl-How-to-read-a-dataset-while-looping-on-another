# utl-How-to-read-a-dataset-while-looping-on-another
How to read a dataset while looping on another

    How to read a dataset while looping on another

    github
    https://tinyurl.com/u9y9ezd
    https://github.com/rogerjdeangelis/utl-How-to-read-a-dataset-while-looping-on-another

    Solution ib one datastep?

    Loop through table L2 selecting the first  matches ro consecutive records in table L1.
    Edge cases not wee documented?

    SAS Forum
    Very closely related to
    https://tinyurl.com/vbds76u
    https://communities.sas.com/t5/SAS-Programming/How-to-read-a-dataset-while-looping-on-another/m-p/629047

    *_                   _
    (_)_ __  _ __  _   _| |_
    | | '_ \| '_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    ;

    data l1 ;
    input (subset gender code ageclass) ($);
    cards4;
    1 M 001 20-29
    1 M 002 20-29
    1 F 003 30-39
    ;;;;
    run;quit;


    data l2 ;
    input (subset gender code ageclass) ($);
    cards4;
    2 M 004 20-29
    2 M 005 20-29
    2 M 006 20-29
    2 M 007 20-29
    2 F 008 30-39
    2 F 009 30-39
    2 F 010 30-39
    2 F 011 20-29
    ;;;;
    run;quit;

    *           _
     _ __ _   _| | ___  ___
    | '__| | | | |/ _ \/ __|
    | |  | |_| | |  __/\__ \
    |_|   \__,_|_|\___||___/

    ;

    We match on gender and ageclass.
    Keep first two matches to L2.
    Do not reuse the first two matches.
    Create assined code using L1 code


                                          |
     WORK.L1 total obs=3                  |    WORK.L2 total obs=8
                                          |                              ASSIGNED
     SUBSET    GENDER    CODE    AGECLASS |    SUBSET    GENDER    CODE    CODE    AGECLASS
                                          |
       1         M       001      20-29   |      2         M       004      001    20-29     Keep  M20-29 = M2029
                                          |      2         M       005      001    20-29     keep  M20-29 = M2029
       1         M       002      20-29   |
       1         F       003      30-39   |      2         M       006      002    20-29     Keep  F20-29 = F2029
                                          |      2         M       007      002    20-29     keep  F20-29 = F2029
                                          |
       1         F       003      30-39   |      2         F       008      003    30-39     Keep  F20-29 = F2029
                                          |      2         F       009      003    30-39     keep  F20-29 = F2029

    *            _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| '_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    ;

     WORK,WANT total obs=6

                                             ASSIGNED_
      SUBSET    GENDER    CODE    AGECLASS     CODE

        2         M       004      20-29        001
        2         M       005      20-29        001
        2         M       006      20-29        002
        2         M       007      20-29        002
        2         F       008      30-39        003
        2         F       009      30-39        003

    *
     _ __  _ __ ___   ___ ___  ___ ___
    | '_ \| '__/ _ \ / __/ _ \/ __/ __|
    | |_) | | | (_) | (_|  __/\__ \__ \
    | .__/|_|  \___/ \___\___||___/___/
    |_|
    ;

    proc datasets lib=work;
     delete want;
    run;quit;

    %let beg=1;

    data l1l2;
      set l1;
      call symputx('gender_ageclass',cats(gender,ageclass));
      call symputx('code',code);

      * use dosubl to loop through L2;
      rc=dosubl('
         %put &=beg;
         data tmp;
              set l2(firstobs=&beg);
              if cats(gender,ageclass)="&gender_ageclass";
              n+1;
              assigned_code="&code";
         run;quit;
         proc print data=tmp;
         run;quit;
         %if %nobs(tmp) ge 2 %then %do;
             proc append data=tmp(obs=2) base=want;
             run;quit;
             %let beg= %eval(&beg + 2);
         %end;
      ');

    run;quit;


