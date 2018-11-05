# utl-length-of-longest-common-word-in-two-sentences-text-AI
Length of longest common word in two sentences text AI.
    Length of longest common word in two sentences text AI

      Two Solutions

         1. Datastep (requires fcmp function to pop first word and reduce string)

         2. HASH  (I made slight modifications to Bart's excellent solution)
            Bartosz Jablonski's
            yabwon@gmail.com


    https://tinyurl.com/ycyljcqw
    https://github.com/rogerjdeangelis/utl-length-of-longest-common-word-in-two-sentences-text-AI

    related post
    https://github.com/rogerjdeangelis/utl_words_in_common_in_two_sentences

    https://tinyurl.com/y8x2nbql
    https://communities.sas.com/t5/New-SAS-User/Check-the-longest-substring-two-strings-have-in-common/m-p/510444

    INPUT
    =====
                                                                       | RULES
     WORK.HAVE total obs=4                                             |
                                                                       |
       STR1                        STR2                                | COMMON           LONGEST
                                                                       |
       Company ABCDE ABC WXYZ      ABCDE is the organisation ABC WXYZ  | ABCDE ABC WXYX     ABCDE
       MyBusiness                  MyBusiness Ltd.                     |
       Governmental agency FGHIJ   G. A. FGHIJ etc.                    |
       Made Up Company Name        "Made Up" Name                      |
                                                                       |

    EXAMPLE OUTPUT
    --------------

    WORK.WANT_DATASTEP total obs=4

      INTERSECT         MAXWRD        CNTWRDS

      ABCDE ABC WXYZ    ABCDE            3
      MyBusiness        MyBusiness       1
      FGHIJ             FGHIJ            1
      Name              Name             1



    PROCESS
    =======

    1. Datastep (requires fcmp function to pop first word odd a list and reduce string)
    -----------------------------------------------------------------------------------

       * Requires FCMP function below;

       data want_datastep;

        length pop1st intersect maxWrd $200 ;
        call missing(pop1st,intersect);

        set have;

        lenWrd=0;
        do wrds=1 to countw(str1);
          call utl_pop(str1,pop1st,"first");
          if indexw(str2,pop1st)>0 then do;
             intersect=catx(' ',intersect,pop1st);
             if lenWrd < length(pop1st) then do; lenWrd = length(pop1st); maxWrd=pop1st; end;
          end;
        end;

        cntWrds=countw(intersect);
        keep intersect cntWrds maxWrd;

       run;quit;


    2. HASH  (I made slight modifications to Bart's excellent solution - Bartosz Jablonski's)

       data want;

       dcl hash h () ;
         h.definekey ("word") ;
         h.definedata ("word", "count") ;
         h.definedone ();

       do until(eof);
       h.clear();

       set have end = eof;

       count = 0;
       do i=1 to countw(str1);
        word = scan(str1, i);
        if h.find() then h.add();
       end;

       do i=1 to countw(str2);
        word = scan(str2, i);
        if h.find() = 0 then do; count = 1; h.replace(); end;
       end;


       declare hiter ih('h');
       length common_words $ 200 maxWrd $ 200;
       count_of_common=0;
       lenWrd=0;
       common_words = "";

       _rc_ = ih.first();

       do while(_rc_ = 0);
           if count then do;
                           if lenWrd < length(word) then do; lenWrd = length(word); maxWrd=word; end;
                           common_words = catx(" ", common_words, word);
                           count_of_common +1;
                         end;
           _rc_ = ih.next();
       end;
       output;
       end;

       stop;
       keep str1 str2 common_words count_of_common maxW;
       run;

    *                _              _       _
     _ __ ___   __ _| | _____    __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \  / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/ | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|  \__,_|\__,_|\__\__,_|

    ;

    data have;
    informat str1 str2 $64.;
    input str1 & str2 &;
    cards4;
    Company ABCDE ABC WXYZ  ABCDE is the organisation ABC WXYZ
    MyBusiness  MyBusiness Ltd.
    Governmental agency FGHIJ  G. A. FGHIJ etc.
    Made Up Company Name  "Made Up" Name
    ;;;;
    run;quit;


    * __
     / _| ___ _ __ ___  _ __    _ __   ___  _ __
    | |_ / __| '_ ` _ \| '_ \  | '_ \ / _ \| '_ \
    |  _| (__| | | | | | |_) | | |_) | (_) | |_) |
    |_|  \___|_| |_| |_| .__/  | .__/ \___/| .__/
                       |_|     |_|         |_|
    ;


    options cmplib=work.funcs;
    proc fcmp outlib=work.funcs.temp;
    Subroutine utl_pop(string $,word $,action $);
        outargs word, string;
        length word $200;
        select (upcase(action));
          when ("LAST") do;
            call scan(string,-1,_action,_length,' ');
            word=substr(string,_action,_length);
            string=substr(string,1,_action-1);
          end;

          when ("FIRST") do;
            call scan(string,1,_action,_length,' ');
            word=substr(string,_action,_length);
            string=substr(string,_action + _length);
          end;

          otherwise put "ERROR: Invalid action";

        end;
