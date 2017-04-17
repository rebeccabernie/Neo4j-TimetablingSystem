# Design Document - Neo4J Timetabling System

This is the Design Document for timetabling system using Neo4j, completed as part of 3rd Year Software Development module Graph Theory. The document outlines my reasoning for the general structure of the database, as well as some useful example queries.

## Introduction

For this project, we were asked to design a prototype database that would store timetabling information.  Information was to relevant to Galway Mayo Institute of Technology and could be obtained through GMIT's own timetabling website or by other means.

## Data To Be Stored


Most timetables, regardless of what college/institute they relate to, contain the same information - times, days, subjects and rooms. I used GMIT's own [timetabling website](http://timetable.gmit.ie/sws1617/(S(anusup452nzd3yeefh4zyr45))/default.aspx) to see what basic data needed to be stored and came to the conclusion that modules, lecturers, days, times and rooms were the main components. 

## How Data Was Obtained

After initially planning to do the whole Software Development course (Years 1-3, 4), I decided to stick to Year 3 to keep the database relatively small. While adding extra years would not have been difficult (I had the data already pulled from the timetabling website), I felt it wouldn't have made it any better as the basic functionality is the same year to year. As this is only a prototype database, I felt setting out structure and logic was more important than creating a very large database.

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


