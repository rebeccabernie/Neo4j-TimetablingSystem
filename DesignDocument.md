# Design Document - Neo4J Timetabling System

This is the Design Document for timetabling system using Neo4j, completed as part of 3rd Year Software Development module Graph Theory. The document outlines my reasoning for the general structure of the database, as well as some useful example queries.

## Introduction

For this project, we were asked to design a prototype database that would store timetabling information.  Information was to relevant to Galway Mayo Institute of Technology and could be obtained through GMIT's own timetabling website or by other means.

## How Data Was Obtained

After initially planning to do the whole Software Development course (Years 1-3, 4), I decided to stick to Year 3 to keep the database relatively small. While adding extra years would not have been difficult (I had the data already pulled from the timetabling website), I felt it wouldn't have made it any better as the basic functionality is the same year to year. As this is only a prototype database, I felt setting out structure and logic was more important than creating a very large database.  

On the GMIT [Timetabling Website](http://timetable.gmit.ie/sws1617/(S(315aqmvve2tdkz45b3n32z55))/default.aspx), I selected the following -

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

Most timetables, regardless of what college/institute they relate to, contain the same information - times, days, subjects and rooms. After extracting the data as explained above, I came to the conclusion that the following would be stored as nodes:

- Year Group (BSc Software Development Y3): Used for lectures that all groups share
- Class Groups: Used for individual group practicals
- Modules
- Days of the Week
- Lecturers
- Rooms

Time Slots, Duration and Type (Lecture or Practical) are also contained in the database, but not as nodes.


## Structure

As mentioned above, I decided to set the main components of the database as nodes. The more experience I gained with Neo4j, the more I felt that nodes were a lot easier search for than, for example, relationships.  

Modules tie everything together in any timetable, so it made sense to have them act as a link between different nodes in this database.  Lecturers relate to Modules, which in turn relate to rooms and student groups - meaning a lecturer can easily find out what module they have with what student group and in what room. Similarly, student groups can easily find out what module they have in what room and with what lecturer.

When dealing with lectures and how to differentiate them from practicals when searching, I decided to use the Year Group node to represent the year as a whole. The entire year group has the same lectures, but not necessarily the same lab groups. I had considered simply tying all lab groups to a lecture, but felt that it made the database more cluttered and slightly less searchable. By tying the year group to lectures, you can easily search for *just* lectures or *just* practicals.

I had considered storing time slots as nodes, but felt this would have made things too complicated due to the amount of time slots as well as timeslots overlapping every day.

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


