# Design Document - Neo4J Timetabling System

This is the Design Document for timetabling system using Neo4j, completed as part of 3rd Year Software Development module Graph Theory. The document outlines my reasoning for the general structure of the database, as well as some useful example queries.


## Introduction

For this project, we were asked to design a prototype database that would store timetabling information.  Information was to relevant to Galway Mayo Institute of Technology and could be obtained through GMIT's own timetabling website or by other means.

When running any query in Neo4j that filters relationship type, make sure autocomplete is off. If not, the correct nodes will display all of their relationships, not just the filtered ones.


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
	
Grid view was used for obtaining information on room sizes as capacity is displayed with each entry, unlike list view which does not display room capacities.
	
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

Time Slots, Duration and Type (Lecture or Practical) are also contained in the database, but not as nodes. Days of the week are nodes, but can also be searched through relationship properties.
Module nodes contain the Moodle Code for the course, obtained from [GMIT's LearnOnline](https://learnonline.gmit.ie/).  
Lecturer nodes contain information such as title and department, pulled from the [Staff Directory](https://www.gmit.ie/staff-directory).  
Room nodes contain their location (campus) and capacity.

I recorded all queries used to create nodes and add relationships in a text file, which can be found [here](https://github.com/rebeccabernie/TimetablingSystem/blob/master/queries).

## Structure

As mentioned above, I decided to set the main components of the database as nodes. The more experience I gained with Neo4j, the more I felt that nodes were a lot easier search for than, for example, relationships.  

Modules tie everything together in any timetable, so it made sense to have them act as a link between different nodes in this database.  Lecturers relate to Modules, which in turn relate to rooms and student groups - meaning a lecturer can easily find out what module they have with what student group and in what room. 
By running the following query, the lecturer is presented with their own node, whatever subjects they teach, and the rooms they're in. The lecturer can also see what group they have in what room, for how long, and at what time by looking at the relationship properties between module and room.

	match (l:Lecturer)-[t:TEACHES]-(m:Module)-[i:IN]-(r:Room) where l.name = "I McLoughlin" return *
	
![Lecturer Search](https://lh3.googleusercontent.com/vfK1pzdYyJnkcGNaAAFs9kGUUqY26N3IdxqgchpWr34QlUFXCnt9df3nB600Ts2HGq22b1hyBNt-FCdUoV6Z3MC1QgHuwZ-ZCyuPIp-SrEh2uNPamE1T6d1Fk6KAJIxDh08vC_lXvuLs-NQE1FbWY2NRUSm1JyARMJzhqkW3xu2IvZ7G0TcYG0zVIkNUehdRCpbVw6yJEYaG-Ei82jghCOt6P9Yq1ndw_WrEx-lH9MnmWSfGxoTxFJPgqEA2S0034IunwqEqlTTa1ykRyo0M2oarnjVLMRMmxI_JBhxWeXsQlBPy7WXfVRqkijQmZoqOUUdYq0BTek2ORpELzklZaF3wVGvau_SfMQBiXumxtoKZvNN5cZx825V575cbeUp6TEr_MTapGOlSjeKGgJAZ3-ahPfF-f7PjqTE0-qBAPHcDpY-xe6ER_3goRJVU7JOcvrp6raEgM2XV3DRFTOvoBzVOgUC0r6VBz3THv_T1um7yaK3MNsII061JzNN3glPe2_TZrvZOHAZYEe-mDQ2APa3LuHD5i7Y0LGzo6jVTlxKtBdQvMRXUApBQSHof3fj4g6RyHIiluBpWDSL3iwu36iQegUO-TKonT7YAiIGk_E90Gg0=w1064-h631-no "Lecturer Search")
	
Similarly, student groups can easily find out what module they have in what room and with what lecturer.

When dealing with lectures and how to differentiate them from practicals when searching, I decided to use the Year Group node to represent the year as a whole. The entire year group has the same lectures, but not necessarily the same lab groups. I had considered simply tying all lab groups to a lecture, but felt that it made the database more cluttered and slightly less searchable. By tying the year group to lectures, you can easily search for *just* lectures or *just* practicals. For example:

	match (c:Course)-[o:ON]-(d:Day)-[h:HAS]-(m:Module)-[i:IN]-(r:Room) where h.type = "Lecture" and i.type = "Lecture" return *

Both :IN and :HAS relationships have the same properties in order to allow for filtering.

I had considered storing time slots as nodes, but felt this would have made things too complicated due to the amount of time slots as well as timeslots overlapping every day. In the end I opted for making timeslots a property on relationships, along with type (lecture or practical) and duration. This makes it much easier to grab more information from the database, with smaller queries. For example, to see all Group A Labs for the week all you need to run is:

	match (g:Group{group:"Gr A"})-[o:ON]-(d:Day)-[h:HAS]-(m)-[i:IN]-(r) where h.group = "A" and i.group = "A" return *
	
Searching rooms is also an integral part of any timetable, so I felt having the ability to display when a room is occupied was important. Unfortunately there's no clear way of displaying when a room is free, but you can display all instances where the room is occupied. For example, if you want to see when room 0994 is occupied on a Wednesday, run the following query:

	match (r:Room)-[i:IN]-(m:Module) where i.day="Wednesday" and r.name="0994" return *
	
Hovering over a relationship between a room and a module gives information on time, which means you can find a free room by elimination. 
	

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


