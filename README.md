# DDL for Graph Schema
 CREATE VERTEX person(PRIMARY_ID name STRING)
 
 CREATE UNDIRECTED EDGE friendship(FROM person, TO person , connect_day DATETIME)
 
 CREATE GRAPH social(person, friendship)
#  To load data Create a loading job
 USE GRAPH social

 CREATE LOADING JOB load_social FOR GRAPH social{
 
 DEFINE FILENAME file1="";
 
 DEFINE FILENAME file2="";
 
 LOAD file1 TO VERTEX person VALUES($"name",$"age")
 
 USING header="true", seperator=",";
 
 LOAD file2 to edge friendship values(40,$1,$2)USING header="true", seperator=",";}
 
 RUN LOADING JOB load_social
 
