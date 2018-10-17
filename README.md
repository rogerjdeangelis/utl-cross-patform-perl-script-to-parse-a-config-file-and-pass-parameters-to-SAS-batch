# utl-cross-patform-perl-script-to-parse-a-config-file-and-pass-parameters-to-SAS-batch
Cross patform perl script to parse a config file and pass parameters to SAS tasks

    Cross patform perl script to parse a config file and pass parameters to SAS tasks

    see github
    https://tinyurl.com/y87ncq4b
    https://github.com/rogerjdeangelis/utl-cross-patform-perl-script-to-parse-a-config-file-and-pass-parameters-to-SAS-batch

    Problem: Run two SAS programs with macro arguments in a configuration file

      1. Create two SAS programs with input and output as external arguments
           Program 1 - Create a text file
           Program 2 - Read file createsd above and write to log

      2. Build a config file that contains the input and output file names

      3. Run a perl script to parse the config file and pass the
         filenames to the two separate SAS programs


    Related to (Perl is better?)
    see github
    https://tinyurl.com/ydhuoomp
    https://communities.sas.com/t5/SAS-Enterprise-Guide/how-to-passed-parameter-in-a-ksh-file-in-order-to-execute-a-sas/m-p/505119

    *_                   _
    (_)_ __  _ __  _   _| |_
    | | '_ \| '_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    ;

    CONFIG FILE
    ===========

    This creates config file for SAS batch Processing

    data _null_;
      file  "C:\UTL\CONF.TXT";
      input;
      put _infile_;
    cards4;
    # GLOBAL SECTION

    GBL_SASEXE = c:/progra~1/sashome/x86/sasfoundation/9.4/sas.exe
    GBL_SASPGM = c:/utl/
    GBL_SASCNF = c:/utl/config.txt
    GBL_SASWORK= c:/utl/
    GBL_SASLOG = c:/utl/

    GBL_DATE   = 10AUG1999

    # MAKE FILE EXECUTE TASK 1 - MKE_FILE.SAS

    # SET MKE_SKIP TO YES IF THIS STEP DOES NOT NEED TO RUN
    MKE_SKIP      =    NO
    MKE_TITLE     =    SIMPLE CREATE TEXT FILE
    MKE_ OBJ      =    MKE_FILE
    MKE_IN_TEXT   =    PUT THIS LINE IN TEXT FILE
    MKE_OT_FILE   =    C:/UTL/DMO_FILE.TXT
    MKE_FLOP      =    ROW010_COL010

    # GET FILE EXECUTE GET_FILE.SAS

    # SET GET_SKIP TO YES IF THIS STEP DOES NOT NEED TO RUN
    GET_SKIP      =    NO
    GET_TITLE     =    SIMPLE GET TEXT FROM FILE AND SHOW DATA IN LOG
    GET_OBJ       =    GET_FILE
    GET_IN_FILE   =    C:/UTL/DMO_FILE.TXT
    GET_FLOP      =    ROW020_COL010
    GET_DATE      =    GBL_DATE
    ;;;;
    run;quit;


    TASK 1 Input one text file output another
    =========================================

    %macro mke_file
        (
         title=Simple create text file,
         obj=mke_file,

         in_text=Put this line in text file,

         ot_file=c:\temp\lod_file.txt,

         flop=ROW010_COL010
        )
        / des = "Simple create text file";

      data _null_;

        file "&ot_file";
        put "&in_text";

        putlog "Text written to &ot_file";

      run;
    %mend mke_file;
    ;;;;
    run;quit;


    TASK 2 Input output text above and output to log
    ==================================================

    data _null_;
      file  "C:\UTL\get_file.sas";
      input;
      put _infile_;
    cards4;
    %macro get_file
        (
         title=Simple get text from file show data in log,
         obj=get_file,

         date=05AUG99,

         in_file=c:\utl\dmo_file.txt,

         flop=COL010_COL010
        )
        / des = "Simple get text from file";

      data _null_;

        infile "&in_file" length=l;

        input text $varying200. l;

        put text=;
        put "&date";

       putlog "Text read from &in_file";

      run;
    %mend get_file;
    ;;;;
    run;quit;


    *
     _ __  _ __ ___   ___ ___  ___ ___
    | '_ \| '__/ _ \ / __/ _ \/ __/ __|
    | |_) | | | (_) | (_|  __/\__ \__ \
    | .__/|_|  \___/ \___\___||___/___/
    |_|
    ;

    EXECUTE THE SAS PROGRAMS
    =========================

    * create PERL script;
    data _null_;
      file "c:\utl\drv.pl";
      input;
      put _infile_;
    cards4;
    # PERL Script for conf file driven batch processing

    open(LOADCNF, 'C:/UTL/CONF.TXT' ) or die " Could not Open CONF.TXT ";

    while ( <LOADCNF> )
       {
             chomp;                             # no newline done
             s/#.*//;                           # skip comments
             s/^\s+//;                          # no leading whitespace
             s/\s+$//;                          # no trailing whitespace
             next unless length;
             my ($var,$value) = split ( /\s*=\s*/,$_,2);
             $$var = $value;
       }

       # Create a demostration file with one line of conf text
       # if conf parameter $MKE_SKIP is set to YES then this step will not be executed

       if ( $MKE_SKIP eq "NO" )

         {

           system (  $GBL_SASEXE .
                 " -sysin " . $GBL_SASPGM . 'mke_file.sas' .
                 " -nosplash "   .
                 " -icon"        .
                 " -sasautos " . $GBL_SASPGM .
                 " -log      " . $GBL_SASLOG . "mke" . $GBL_DATE . ".log" .
                 " -work     " . $GBL_SASWORK .
                 " -termstmt "               .
                     "'%mke_file("  .
                                "in_text =" . $MKE_IN_TEXT . "," .
                                "ot_file =" . $MKE_OT_FILE .
                                ")'"
                  );

          }

       # Read the file created in the previous step    note global date value
       # if conf parameter $GET_SKIP is set to YES then this step will not be executed

       if ( $GET_SKIP eq "NO" )

         {

           system (  $GBL_SASEXE .
                 " -sysin " . $GBL_SASPGM . 'get_file.sas' .
                 " -nosplash "   .
                 " -icon"        .
                 " -sasautos " . $GBL_SASPGM .
                 " -log      " . $GBL_SASLOG . "get" . $GBL_DATE . ".log" .
                 " -work     " . $GBL_SASWORK .
                 " -termstmt "               .
                     "'%get_file("  .
                                "in_file =" . $GET_IN_FILE . "," .
                                "date=" . $GBL_DATE .
                                ")'"
                  );

          }

     close(LOADCNF);
    ;;;;
    run;quit;

    filename xeq_bat pipe "perl c:/utl/drv.pl";
    data _null_;
      infile xeq_bat;
      input;
      put _infile_;
    run;

