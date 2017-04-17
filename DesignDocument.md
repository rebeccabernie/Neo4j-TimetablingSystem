# Design Document - Neo4J Timetabling System

This is the Design Document for timetabling system using Neo4j, completed as part of 3rd Year Software Development module Graph Theory. The document outlines my reasoning for the general structure of the database, as well as some useful example queries.

## Structure

The best way to approach creating this database, in my opinion, was to take the main components of a timetable and set them as nodes.  Lecturers, rooms, modules, class groups and days of the week can all be searched as nodes in the database - in my own experience, nodes are somewhat easier to search for than properties on relationships.

## Relationship Logic

* Will include rough drawings/explanation of relationships *

## Useful Queries

Display all Group A Labs on a Tuesday - 

    MATCH (g:Group)-[]-(d:Day),(d:Day)-[h:HAS{group:"A"}]-(m:Module), (m:Module)-[i:IN{group: "A"}]-(r:Room) where g.group='Gr A' and d.name='Monday'  
	return g,d,m,r
	 
Display only lectures for the whole week -

	match (c:Course)-[:ON]-(d:Day)-[:HAS]-(m)-[:IN]-(r) return *
	
Display a particular lecturer's classes for the week -

	match (a)-[:ON]-(d:Day)-[:HAS]-(m:Module)-[:IN]-(r),(m:Module)<-[:TEACHES]-(l:Lecturer{name:"D Costello"}) return *


