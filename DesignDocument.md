# Design Document - Neo4J Timetabling System

This is the Design Document for timetabling system using Neo4j, completed as part of 3rd Year Software Development module Graph Theory. The document outlines my reasoning for the general structure of the database, as well as some useful example queries.


## Introduction

For this project, we were asked to design a prototype database that would store timetabling information.  Information was to relevant to Galway Mayo Institute of Technology and could be obtained through GMIT's own timetabling website or by other means.


## How Data Was Obtained

Before starting anything in the database, I gathered information from the GMIT [Timetabling Website](http://timetable.gmit.ie/sws1617/(S(315aqmvve2tdkz45b3n32z55))/default.aspx).

I selected the following, repeating for each year of the course -

	Department			Galway Campus - Dept of Computer Science & Applied Physics
	Program				G-KSOFG73 BSc in Computing in Software Development L7 Yr3 Sem 6
	Week(s)				Spring/Summer - January 2017 to June 2017
	Day(s)				Weekdays
	Time Range			Working Day (08:00 - 18:00)
	Type	 			List Timetable

	
I used the list view as it was formatted in rows and columns and allowed data to be selected as plain text, making it easier to manipulate in a text editor. List view also gave a list of lecturer names, rather than their numbers in the grid view. I ended up with a text file, containing the following information - 

	Module, Type(Lecture/Practical), Start, End, Duration, Room, Lecturer
	
All of the data can be seen in the [AllTimetablesList File](https://github.com/rebeccabernie/TimetablingSystem/blob/master/AllTimetablesList.txt).	  


## Data To Be Stored

After initially planning to do the whole Software Development course (Years 1-3, 4), I decided to stick to Year 3 to keep the database relatively small. While adding extra years would not have been difficult since I already had all the data, I felt it wouldn't have made the database any better. Course structure is extremely similar from year to year and as this is only a prototype database, I felt setting out basic structure and logic was more important than creating a very large database.

Most timetables, regardless of what college/institute they relate to, contain the same information - times, days, subjects and rooms. After extracting the data outlined above, I came to the conclusion that the following should be stored as nodes:

- Year Group (BSc Software Development Y3): Used for lectures that all groups share
- Class Groups: Used for individual group practicals
- Modules
- Days of the Week
- Lecturers
- Rooms

Time Slots, Duration and Type (Lecture or Practical) are also contained in the database, but not as nodes.


## Structure

As mentioned above, I decided to set the main components of the database as nodes. The more experience I gained with Neo4j, the more I felt that nodes were a lot easier search for than, for example, relationships.  

Modules tie everything together in any timetable, so it made sense to have them act as a link between different nodes in this database.  Lecturers relate to Modules, which in turn relate to rooms and student groups - meaning a lecturer can easily find out what module they have with what student group and in what room. 
By running the following query, the lecturer is presented with their own node, whatever subjects they teach, and the rooms they're in. The lecturer can also see what group they have in what room, for how long, and at what time by looking at the relationship properties between module and room.

	match (l:Lecturer)-[t:TEACHES]-(m:Module)-[i:IN]-(r:Room) where l.name = "I McLoughlin" return *
	
Similarly, student groups can easily find out what module they have in what room and with what lecturer.

When dealing with lectures and how to differentiate them from practicals when searching, I decided to use the Year Group node to represent the year as a whole. The entire year group has the same lectures, but not necessarily the same lab groups. I had considered simply tying all lab groups to a lecture, but felt that it made the database more cluttered and slightly less searchable. By tying the year group to lectures, you can easily search for *just* lectures or *just* practicals. For example:

	match (c:Course)-[o:ON]-(d:Day)-[h:HAS]-(m:Module)-[i:IN]-(r:Room) where h.type = "Lecture" and i.type = "Lecture" return *

I had considered storing time slots as nodes, but felt this would have made things too complicated due to the amount of time slots as well as timeslots overlapping every day. In the end I opted for making timeslots a property on relationships, along with type (lecture or practical) and duration. This makes it much easier to grab more information from the database, with smaller queries. For example, to see all Group A Labs for the week all you need to run is:

	match (g:Group{group:"Gr A"})-[o:ON]-(d:Day)-[h:HAS]-(m)-[i:IN]-(r) where h.group = "A" and i.group = "A" return *
	
*Side note; if autocomplete is turned ON in Neo4j, this query will display the relevant nodes but also those nodes' relationships, regardless of whether the type is Lecture or not.

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


