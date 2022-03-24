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
 
 LOAD file2 to edge friendship values($0,$1,$2)USING header="true", seperator=",";}
 
 RUN LOADING JOB load_social
# Run some built-in queries
 USE GRAPH social

 SELECT count() FROM person
 
 SELECT count() FROM person -(friendship)->person
 
 SELECT * FROM person where primary_id=="Tom"
 
 SELECT name FROM person WHERE stat="ca"
 
 SELECT name,age FROM person where age>30
 
 SELECT * FROM person -(friendship)->person WHERE from_id=="Tom"
# Add, Install, Run parameterized query
 return 1-hop neighbour nodes
 
 CREATE QUERY hello( VERTEX <person>p)FOR GRAPH social
 
 { start={p};

  Result=SELECT tgt

  FROM Start:src-(friendship:e)->person:tgt;

 PRINT Result;
 }
 INSTALL QUERY hello
 
 RUN QUERY hello("Tom")
 
 curl -X GET 'http://localhost:9000/query..."
 
# Advanced gsql accumulators 
 2-hop neighbour nodes and their average age
 
 CREATE QUERY hello2(VERTEX <person>p)FOR GRAPH social
 {
 
 OrAccum @visited=false;
 
 AvgAccum @@avgAge;
 
 Start={p};
 
 FirstNeighbour=SELECT tgt FROM Start:src-(friendship:e)->person:tgt
 
 Accum tgt @visited+=true, src@visited+=true;
 
 SELECT Neighbours=SELECT tgt
 
 FROM FirstNeighbours-(:e)->:tgt
 
 where tgt @visited== false
 
 POST_Accum @@avgAge+=tgt.age;
 
 PRINT SecondNeighbours;
 
 PRINT @@avgAvg;}
 
 INSTALL QUERY hello2
 
 RUN QUERY hello2("Tom")
# Run Deep Link query on a billion edge scale graph using flow-control WHILE loop
 
 USE GRAPH TWITTER
 
 CREATE QUERY khop(VERTEX <MyNode>start_node,
 
 INT depth> for graph Twitter{
 
 OrAccum @visited=false;
 
 SumAccum<int> @@loop=0;

 Start={start_node};
 
 Start=SELECT v
 
 FROM Star:v
 
 Accum v.@visited=true
 
 WHILE(@@loop(depth)DO
 
 Start=SELECT v
 
 FROM Start:u-(MyEdge e)->v
 
 WHERE v.@visited==false;
 
 Accum v.@visited=true;
 
 @@loop+=1;
 
 END;
 
 PRINT Start.size();}
 
 INSTALL QUERY khop
 
 run query khop("618593",1)
 
 run query khop("618593",2)
 
 set query_timeout=100000
 
 run query khop("618593",8)
 
 run query khop("618593",12)
 
