# utl-using-sas-character-functions-instead-of-regular-expressions-ie-prx
Using sas character functions instead of regular expressions ie prx

    Using sas character functions instead of regular expressions ie prx                                                       
                                                                                                                              
    github                                                                                                                    
    https://tinyurl.com/y3zlq7zs                                                                                              
    https://github.com/rogerjdeangelis/utl-using-sas-character-functions-instead-of-regular-expressions-ie-prx                
                                                                                                                              
    SAS Forum                                                                                                                 
    https://tinyurl.com/yyhzzq27                                                                                              
    https://communities.sas.com/t5/SAS-Programming/regularexpression-how-to-get-the-number-of-matches/m-p/568975              
                                                                                                                              
    Problem                                                                                                                   
                                                                                                                              
       Count the number of "ABC" words in 'Start ABC ABC ABC End'                                                             
                                                                                                                              
       Four Solutions                                                                                                         
                                                                                                                              
          1. prx  https://communities.sas.com/t5/user/viewprofilepage/user-id/18408                                           
             by Ksharp                                                                                                        
          2. countc tranwrd  n= countc(tranwrd(str,'ABC','@'),'@');                                                           
          3. scan iteration                                                                                                   
          4. catx requires consecutive ABCs                                                                                   
                                                                                                                              
                                                                                                                              
    I know the op wanted a regular expression solution but I think                                                            
    solutions with SAS charater functions are much cleaner                                                                    
                                                                                                                              
                                                                                                                              
    There are only a few places where regular expression are superior to                                                      
    SAS character functions?                                                                                                  
                                                                                                                              
      Problems for regular expressions use SAS character function                                                             
                                                                                                                              
       1. Do the two lists have any common words                                                                              
          if prxmatch('/MAY|STEVE/',"ROGER MAY PENG")>0 then put 'Strings have at least one word in common';                  
                                                                                                                              
       2. Valid SSN  (SSNs cannot begin with 666 reserved for Satan)                                                          
                                                                                                                              
          if prxmatch('/^(?!000)(?!666)(?!9)\d{3}([- ]?)(?!00)\d{2}\1(?!0000)\d{4}$/'                                         
          ,ssn)>0 then put "Valid";                                                                                           
                                                                                                                              
       3. Distinct Count of 'booboo' in 'boobooboobooboo' should be 2                                                         
          cnt=countc(prxchange('s/([booboo]){6}/*/',-1,'boobooboobooboo'),'*') ;                                              
         
             
     Usefull insights by                                                                                             
                                                                                                                    
    Paul Dorfman                                                                                                    
    sashole@bellsouth.net                                                                                           
                                                                                                                    
    Roger,                                                                                                          
                                                                                                                    
    I agree that using SAS functions is simpler and infinitely more readable than the regexen.                      
                                                                                                                    
    However, using expression:                                                                                      
                                                                                                                    
    countc (tranwrd (str, 'ABC', '@'), '@')                                                                         
                                                                                                                    
    works only if STR is guaranteed to contain no  @'s. Using, say, "00"x instead appears to be safer;              
    but it still means making assumptions about data. Moreover, it will produce incorrect results if STR            
    should contain a word where ABC repeats, such as XABCYABCZ. The latter cannot be rectified by using:            
                                                                                                                    
    countc (tranwrd (str, ' ABC ', '@'), '@')                                                                       
                                                                                                                    
    because it will fail if STR contains " ABC ABC " or " ABC ABC ABC " and so forth, from which TRANWRD            
    will not remove the last remaining " ABC " due to the                                                           
    overlapping of the trailing and leading blanks. In other words,                                                 
                                                                                                                    
    tranwrd ("Start ABC ABC ABC End", " ABC ", "") = "Start ABC End "                                               
                                                                                                                    
    while we need the result to be "Start End".                                                                     
                                                                                                                    
    So, what's the solution? Apparently, we need to TRANWRD " ABC " into "" via a series of nested calls            
    until no " ABC " remains in the string. How many nested calls do we need? Since STR can be no longer            
    than 32767, in the extreme case of STR = "ABC ABC ... ABC ABC", it contains 8192 "ABC"s and 8191                
    blanks between each "ABC" pair. Thus, we would need log2(8192)=13 nested TRANWRD call                           
    s to reduce STR to "ABC ABC" and the ultimate nested call TRANWRD(<nested call result>,"ABC ABC")               
    to get the final required blank. Yet, it still doesn't cut it, since instead of "ABC ABC ... ABC ABC"           
    we could potentially have "A A ... A A" if the specs call for counting "A"s rather than "ABC"s.                 
    In this case, we would have to have log2(16384)=14 nested calls followed by the final ne                        
    sted TRANWRD (<nested call result>,"A A").                                                                      
                                                                                                                    
    In other words, the general solution is:                                                                        
                                                                                                                    
    countw (str) -                                                                                                  
    countw (tranwrd (tranwrd (tranwrd (tranwrd (str, " ABC "), " ABC "), " ABC ") ... )))))))))))))), "ABC ABC"))   
                                                                                                                    
    Looks pretty, ain't it? Of course, to reduce the pain, the nested " ABC " calls can be macroized.               
    But I reckon it's simpler just to FCMP a DO loop with SCAN counting the occurrences of "ABC"                    
    delimited by blanks and use the resulting function thereafter.                                                  
                                                                                                                    

         
    *_                   _                                                                                                    
    (_)_ __  _ __  _   _| |_                                                                                                  
    | | '_ \| '_ \| | | | __|                                                                                                 
    | | | | | |_) | |_| | |_                                                                                                  
    |_|_| |_| .__/ \__,_|\__|                                                                                                 
            |_|                                                                                                               
    ;                                                                                                                         
                                                                                                                              
      str =  'Start ABC ABC ABC End'                                                                                          
                                                                                                                              
    *            _               _                                                                                            
      ___  _   _| |_ _ __  _   _| |_                                                                                          
     / _ \| | | | __| '_ \| | | | __|                                                                                         
    | (_) | |_| | |_| |_) | |_| | |_                                                                                          
     \___/ \__,_|\__| .__/ \__,_|\__|                                                                                         
                    |_|                                                                                                       
    ;                                                                                                                         
                                                                                                                              
     n =3                                                                                                                     
                                                                                                                              
    *          _       _   _                                                                                                  
     ___  ___ | |_   _| |_(_) ___  _ __  ___                                                                                  
    / __|/ _ \| | | | | __| |/ _ \| '_ \/ __|                                                                                 
    \__ \ (_) | | |_| | |_| | (_) | | | \__ \                                                                                 
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|___/                                                                                 
                                                                                                                              
    ;                                                                                                                         
                                                                                                                              
    *_                                                                                                                        
    / |     _ __  _ ____  __                                                                                                  
    | |    | '_ \| '__\ \/ /                                                                                                  
    | |_   | |_) | |   >  <                                                                                                   
    |_(_)  | .__/|_|  /_/\_\                                                                                                  
           |_|                                                                                                                
    ;                                                                                                                         
                                                                                                                              
                                                                                                                              
    data _null_;                                                                                                              
                                                                                                                              
    string='Start ABC ABC ABC End';                                                                                           
                                                                                                                              
    s=1;e=length(string);                                                                                                     
                                                                                                                              
    pid=prxparse("/\bABC\b/i");                                                                                               
    call prxnext(pid,s,e,string,p,l);                                                                                         
    n=0;                                                                                                                      
                                                                                                                              
    do while(p>0);                                                                                                            
     n+1;                                                                                                                     
     call prxnext(pid,s,e,string,p,l);                                                                                        
    end;                                                                                                                      
                                                                                                                              
    put n=;                                                                                                                   
                                                                                                                              
    run;                                                                                                                      
                                                                                                                              
    *____                             _         _                                                                             
    |___ \       ___ ___  _   _ _ __ | |_ ___  | |_ _ __ __ _ _ __                                                            
      __) |     / __/ _ \| | | | '_ \| __/ __| | __| '__/ _` | '_ \                                                           
     / __/ _   | (_| (_) | |_| | | | | || (__  | |_| | | (_| | | | |                                                          
    |_____(_)   \___\___/ \__,_|_| |_|\__\___|  \__|_|  \__,_|_| |_|                                                          
                                                                                                                              
    ;                                                                                                                         
                                                                                                                              
    data _null_;                                                                                                              
      str='Start ABC ABC ABC End';                                                                                            
      n= countc(tranwrd(str,'ABC','@'),'@');                                                                                  
      put n=;                                                                                                                 
    run;quit;                                                                                                                 
                                                                                                                              
    *_____                                                                                                                    
    |___ /      ___  ___ __ _ _ __                                                                                            
      |_ \     / __|/ __/ _` | '_ \                                                                                           
     ___) |    \__ \ (_| (_| | | | |                                                                                          
    |____(_)   |___/\___\__,_|_| |_|                                                                                          
                                                                                                                              
    ;                                                                                                                         
                                                                                                                              
    data _null_;                                                                                                              
      retain n 0;                                                                                                             
      str='Start ABC ABC ABC End';                                                                                            
      do i=1 to countw(str);                                                                                                  
        n = n + (scan(str,i,' ') = "ABC");                                                                                    
      end;                                                                                                                    
      put n=;                                                                                                                 
      stop;                                                                                                                   
    run;quit;                                                                                                                 
                                                                                                                              
    *_  _                 _                                                                                                   
    | || |       ___ __ _| |___  __                                                                                           
    | || |_     / __/ _` | __\ \/ /                                                                                           
    |__   _|   | (_| (_| | |_ >  <                                                                                            
       |_|(_)   \___\__,_|\__/_/\_\                                                                                           
                                                                                                                              
    ;                                                                                                                         
                                                                                                                              
    * if the ABC are consecutive;                                                                                             
    data _null_;                                                                                                              
                                                                                                                              
       retain n 0;                                                                                                            
       length chk $200;                                                                                                       
                                                                                                                              
       str='Start ABC ABC ABC End';                                                                                           
                                                                                                                              
       do i=1 to countw(str,"ABC");                                                                                           
                                                                                                                              
           chk=catx(" ",chk,"ABC");                                                                                           
           if index(str,strip(chk)) > 0 then n=n+1;                                                                           
       end;                                                                                                                   
                                                                                                                              
       put n=;                                                                                                                
                                                                                                                              
    run;quit;                                                                                                                 
                                                                                                                              
