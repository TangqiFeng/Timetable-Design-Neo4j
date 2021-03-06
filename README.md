# Neo4j database for a timetabling system
##### Tangqi Feng

## Introduction
This is a design document for a Neo4j database to store the information of a timetabling system.
It was completed as part of a third year module in the B.Sc. in Software Development at [GMIT](http://www.gmit.ie).

## What is Neo4j?

[Neo4j](https://neo4j.com/) is a [graph based](https://en.wikipedia.org/wiki/Graph_database) database management system by Neo Technology, Inc. 

![Image of Graph database structure](https://s3.amazonaws.com/dev.assets.neo4j.com/wp-content/uploads/graphdb-gve.png)

Neo4j is implemented in **Java** and accessible from software written in other languages using the [Cypher Query Language](https://en.wikipedia.org/wiki/Cypher_Query_Language) through a transactional HTTP endpoint, or through the binary 'bolt' protocol.

## How does Neo4j store data?
In Neo4j, everything is stored in the form of either an edge(Relationship), a node, or an attribute. Each node and edge can have any number of attributes. Both the nodes and edges can be labelled. Labels can be used to narrow searches.

##### Nodes & Relationships:
>The fundamental units that form a graph are nodes and relationships. In Neo4j, both nodes and relationships can contain properties. Nodes are often used to represent entities, but depending on the domain relationships may be used for that purpose as well. Apart from properties and relationships, nodes can also be labeled with zero or more labels.

>A relationship between two nodes in the graph. A relationship has a start node, an end node and a type. You can attach properties to relationships with the API specified in PropertyContainer.

![Image of Neo4j Node](https://neo4j.com/docs/2.1.8/images/graphdb-nodes-overview.svg)
![Image of Neo4j Relationship](http://www.markhneedham.com/blog/wp-content/uploads/2013/11/2013-11-23_21-43-57.png)

##### Labels:
>A label is a named graph construct that is used to group nodes into sets. All nodes labeled with the same label belongs to the same set. Many database queries can work with these sets instead of the whole graph, making queries easier to write and more efficient. A node may be labeled with any number of labels, including none, making labels an optional addition to the graph.

![Image of Label](https://neo4j.com/docs/2.1.8/images/graphdb-labels.svg)

## What information does a timetabling system need to store?
- Room Number

- day & Timeslot

- year & department & program & student groups

- subject (course name)

- Lecturers

## How to store timetabling data in Neo4j?

#### labels: 
- lecturer       
- program       
- course   
- room

#### nodes: 
- lecturer  
- program  
- subject
- room

#### relationships: 
- lecturer --TEACH_IN--> subject
- program --ATTEND--> subject
- subject --HELD_IN--> room

#### properties: 
- lecturer :{ name }
- program :{ name, year, department }
- subject :{ name, day, timeslot, studentGroup }
- room :{ name, capacity }

#### design diagram:
![Image of logical design1](https://cloud.githubusercontent.com/assets/22374434/24590969/e33e4716-17ee-11e7-8e74-4ae8680e6ad8.png)

![Image of logical design2](https://cloud.githubusercontent.com/assets/22374434/24590971/e5254ee4-17ee-11e7-96b2-139cf9587392.png)

#### design benifits:
Easy to get courses by three ways: student, stuff and room

That means: student and lecturer can get their individual course info, also can get course info by search specific room.

## How to use?
### (Note: Timetable trsting data is from GMIT-COMP_GA_KSOFG_B070306)

- get all programs:
> match (m:Program) return m;

- get all lecturers:
> match (m:Lecturer) return m;

- get all rooms:
> match (m:Room) return m;

- As a stuff, to find his individual timetable:
e.g. lecturer name: Ian Mcloughlin
> match (lecturer:Lecturer {name:'Ian Mcloughlin'})-[:TEACH_IN]->(courses:Course)-[:HELD_IN]->(room:Room)
return lecturer,courses,room;

- As a student, to find his individual timetable:
e.g. student from 'G-KSOFG73 BSc in Computing in Software Development L7 Yr 3 Sem 6','Dept of Computer Science &amp; Applied Physics'

> match (program:Program {name:'G-KSOFG73 BSc in Computing in Software Development L7', year_sem:'3 , 6', department:"Dept of Computer Science & Applied Physics"})-[:ATTEND]->(courses:Course)-[:HELD_IN]->(room:Room)
return program,courses,room;

different lab groups:  e.g. A

> match (program:Program {name:'G-KSOFG73 BSc in Computing in Software Development L7', year_sem:'3 , 6', department:"Dept of Computer Science & Applied Physics"})-[:ATTEND]->(courses:Course **{student_group:"A"}**)-[:HELD_IN]->(room:Room)
return program,courses,room; 

- Search by room with specific day/time:
e.g. room '0436 CR5'   all courses

> match (room:Room {name:'0436 CR5'})<-[:HELD_IN]-(courses:Course)<-[:TEACH_IN]-(lecturers:Lecturer)
return room,courses,lecturers

e.g. room '0436 CR5'   all courses in Monday

> match (room:Room {name:'0436 CR5'})<-[:HELD_IN]-(courses:Course **{day:'monday'}**)<-[:TEACH_IN]-(lecturers:Lecturer)
return room,courses,lecturers

e.g. room '0436 CR5'   all courses in Tuesday 14.00-16.00

> match (room:Room {name:'0436 CR5'})<-[:HELD_IN]-(courses:Course **{day:'tuesday', timeslot:'14.00-16.00'}**)<-[:TEACH_IN]-(lecturers:Lecturer)
return room,courses,lecturers

## Conclusion
Graph databases are very adept at working with not just single points of information, but also evolving relationship networks. It works particularly well when the relationships inside your data are important and your queries depend on exploring and exploiting them.
