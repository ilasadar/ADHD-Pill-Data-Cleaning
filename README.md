# ADHD-Pill-Data-Cleaning
# Merging and cleaning ADHD pill data (using SAS)

/*reading in the adhd patient data of the excel file*/                                                                                  
proc import                                                                                                                             
datafile='E:\Stat 465\Final Project\Data.csv'                                                                                           
dbms=csv                                                                                                                                
out= adhd                                                                                                                               
replace;                                                                                                                                
guessingrows = 28;                                                                                                                      
run;                                                                                                                                    
                                                                                                                                        
/*reading in the pill data of the excel file*/                                                                                          
proc import                                                                                                                             
datafile='E:\Stat 465\Final Project\pill.csv'                                                                                           
dbms=csv                                                                                                                                
out= pill                                                                                                                               
replace;                                                                                                                                
run;                                                                                                                                    
                                                                                                                                        
/*transpose the adhd patient data*/                                                                                                     
proc transpose data = adhd                                                                                                              
out = transpose;                                                                                                                        
var _1 - _107; * variables I am transposing;                                                                                            
id screennum; * naming the new variables from the values of the screennum column;                                                       
run;                                                                                                                                    
                                                                                                                                        
/*clean up the pill data*/                                                                                                              
data newPill;                                                                                                                           
set pill;                                                                                                                               
bottleNumber = bottle_number; * rename bottle_number in order to merge w/ the other dataset later;                                      
keep bottleNumber arm;                                                                                                                  
run;                                                                                                                                    
                                                                                                                                        
/*clean up adhd patient data*/                                                                                                          
data newTranspose;                                                                                                                      
set transpose;                                                                                                                          
bottleNumber = input(bottlenum, 2.); * convert bottlenum to numeric;                                                                    
drop _NAME_ bottlenum;                                                                                                                  
run;                                                                                                                                    
                                                                                                                                        
/*sort adhd patient data in order to merge with pill data*/                                                                             
proc sort data = newTranspose;                                                                                                          
by bottleNumber;                                                                                                                        
run;                                                                                                                                    
                                                                                                                                        
/*merge adhd patient data and pill data*/                                                                                               
data manipulated;                                                                                                                       
merge newTranspose newPill;                                                                                                             
by bottleNumber;                                                                                                                        
run;                                                                                                                                    
                                                                                                                                        
/*cleaning up the final dataset*/                                                                                                       
data clean (drop = adat_pat_text2-adat_pat_text6 rating1);                                                                              
set manipulated;                                                                                                                        
if patnum ^= "0"; * gets rid of patients who didn't have more than one visit;                                                           
                                                                                                                                        
array parents{5} adat_par_text2-adat_par_text6; * array of the parents ratings;                                                         
array weight{6} weight1-weight6; * array of the weights by dates;                                                                       
array date{6} examdate1-examdate6; * array of the exam dates;                                                                           
array rating{6}; * array of the appetite ratings by data in a numeric context;                                                          
                                                                                                                                        
/*classifying the appetitie ratings in a numeric context*/                                                                              
do i = 2 to 6;                                                                                                                          
        if parents[i-1] = "Very Good" then rating[i] = 5;                                                                               
        if parents[i-1] = "Good" then rating[i] = 4;                                                                                    
        if parents[i-1] = "Fair" then rating[i] = 3;                                                                                    
        if parents[i-1] = "Poor" then rating[i] = 2;                                                                                    
        if parents[i-1] = "Very Poor" then rating[i] = 1;                                                                               
                                                                                                                                        
        if parents[i-1] = "" then do; * finds the last exam date of the patient;                                                        
                lastNum = i-1;                                                                                                          
                i = 6;                                                                                                                  
        end;                                                                                                                            
        else if i = 6 then lastNum = 6;                                                                                                 
end;                                                                                                                                    
                                                                                                                                        
weightDiff = weight[lastNum] - weight[1]; * gets the weight difference between the first and last exam data;                            
ratingDiff = rating[lastNum] - rating[2]; * gets the appetite rating difference between the first and last exam data;                   
                                                                                                                                        
/*change appetite rating difference to a categorical variable*/                                                                         
if ratingDiff >= 0 then appetite = "better/same";                                                                                       
else appetite = "worse";                                                                                                                
                                                                                                                                        
/*change weight difference to a categorical variable*/                                                                                  
if weightDiff >= 0 then changeInWeight = "better/same";                                                                                 
else changeInWeight = "worse";                                                                                                          
                                                                                                                                        
if previousadhdtx = "" then previousadhdtx = "N"; * changes missing values to "N" so we can use this file in JMP;                       
                                                                                                                                        
keep patnum age gender previousadhdtx arm appetite changeInWeight weightDiff; * keeps the important variable;                           
                                                                                                                                        
run;                                                                                                                                    
                                                                                                                                        
proc print;                                                                                                                             
run;                                                                                                                                    
                                                                                                                                        
/*export the dataset*/                                                                                                                  
proc export data=clean                                                                                                                  
outfile='E:\Stat 465\Final Project\CleanData.csv'                                                                                       
dbms=csv                                                                                                                                
replace;                                                                                                                                
run;
