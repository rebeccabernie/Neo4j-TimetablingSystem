# Design Document - Timetabling System Using Neo4j 

> Author - 
> Rebecca Kane, G00320698
> Third Year BSc Software Development Student

This is the Design Document for timetabling system using Neo4j, completed as part of 3rd Year Software Development module Graph Theory. The document outlines my reasoning for the general structure of the database, as well as some useful example queries.


## Introduction

For this project, we were asked to design a prototype database that would store timetabling information.  Information was to be relevant to Galway Mayo Institute of Technology and could be obtained through GMIT's own [Timetabling Website](http://timetable.gmit.ie/) or by other means.

When running any query in Neo4j that filters relationship type, make sure autocomplete is off. If not, the correct nodes will display all of their relationships, not just the filtered ones.


## How Data Was Obtained

Before starting anything in the database, I gathered information from the GMIT [Timetabling Website](http://timetable.gmit.ie/) under "Academic Year 2016/17" and "Programmes".

I selected the following, repeating for each year of the course -

	Department			Galway Campus - Dept of Computer Science & Applied Physics
	Program				G-KSOFG73 BSc in Computing in Software Development L7 Yr3 Sem 6
	Week(s)				Spring/Summer - January 2017 to June 2017
	Day(s)				Weekdays
	Time Range			Working Day (08:00 - 18:00)
	Type	 			List Timetable

	
I used the list view as it was formatted in rows and columns and allowed data to be selected as plain text, making it easier to manipulate in a text editor. List view also gave a list of lecturer names rather than their staff number, unlike the grid view. I ended up with a text file formatted by tabs, containing the following information - 

	Module	 Type(Lecture/Practical)	Start	End		Duration	Room	Lecturer
	
Grid view was used for obtaining information on room sizes because it displays capacity with each entry, unlike list view which does not display room capacities.
	
All of the data can be seen in the [AllTimetablesList File](https://github.com/rebeccabernie/TimetablingSystem/blob/master/Resources/AllTimetablesList.txt).


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

## Creating the Database
 
I started by created nodes in groups - year group, class groups, lecturers, rooms, modules, and days, in that order. All information stored in the AllTimetablesList file is formatted with tabs, so it was quick and easy generate queries. For example, to generate rooms I started off with a blank query with a row for each room:

	CREATE (a:Room {name: "", campus:"", capacity: ""})
		,(b:Room {name: "", campus:"", capacity: ""})
	      ,(c:Room {name: "", campus:"", capacity: ""})
		  
		 etc, etc

I then used vertical selection in Notepad++ (Ctrl + Alt + Shift) to copy and paste all the data into this empty query. The process was much the same for the other nodes.  

I then moved on to creating relationships. The simplest relationships in the database are those between Lecturer and Module, so I created those first using the following query:

	MATCH (l:Lecturer {name:"I McLoughlin"}), (m:Module {name: "Graph Theory"})
	Create (l)-[:TEACHES]->(m)
	
Match finds the lecturer and module, and create adds a relationship (*from* Lecturer *to* Module) with the label "TEACHES" between the two. 

Once lecturers and modules had been tied together, I moved onto student groups. I decided to create relationships day by day because the AllTimetablesList file goes by days - this meant I'd be less likely to miss anything by going in the same order. These relationships were more complicated than Lecturer-Module ones, and were created as follows:

	MATCH (c:Course{cName: "BSc in Software Development"}), (d:Day {name: "Monday"}), (m:Module {name: "Database Management Systems"}), (r:Room {name:"0994"})
	Create (c)-[:ON]->(d),(d)-[:HAS{time: "10am", type: "Lecture", group:"All", duration:"1HR"}]->(m),(m)-[:IN{time: "10am", type: "Lecture", group:"All", duration:"1HR"}]->(r)
	
This query finds the specified Year Group, Day, Module and Room. It then ties Year Group to Day (with the label "ON"), then Day to Module (labelled as "HAS"), and finally the Module to the Room (labelled "IN"). In English, the above relationship looks like:  
__BSc in Software Development -On- Monday -Has- Database Management Systems -In- 0994__  
I related nodes in this way because when searching a timetable, the most common order is by course and then by day - it wouldn't make much sense if someone tried to search by day and *then* narrow it down to their course. I also added properties on the IN and HAS relationships - *:HAS{time: "10am", type: "Lecture", group:"All", duration:"1HR"}* - to allow for filtering when searching. Both relationship types have the same properties.

I ran into issues when trying to query a room's availability on a particular day, so I had to add a day property to IN and HAS relationships. I decided the easiest way to do this was search relationships by ID and edit from there. I found this solution for how to delete a relationship by ID on a [StackOverflow Question](http://stackoverflow.com/questions/28185623/simple-way-to-delete-a-relationship-by-id-in-neo4j-cypher), and simply edited it to set a property instead :

	Match ()-[r]-() Where ID(r)=303 
	set r.day="Wednesday"

This finds a relationship between two nodes where the ID = 303, and sets a day property on that relationship to "Wednesday".

After creating all nodes, I discovered that the timetable had been changed at stages. This meant I had entered duplicates, so I needed to delete old relationships. I did this by running:

	Match ()-[r]-() Where ID(r)=200
	Delete r

This works the same way as the SET query above, except it deletes the relationship. I also used this when deleting all instances of a specific relationship type, for example ()-[r:HAS]-() will delete all instances of relationships labelled HAS. This came in very useful when I was figuring out the best way to relate different nodes.

I kept a detailed account of all queries I used when creating the database - the text file can be found [here](https://github.com/rebeccabernie/TimetablingSystem/blob/master/Resources/queries). All of the above queries can be found in the file.
	
## Structure and Relationship Logic

As mentioned above, I decided to set the main components of the database as nodes. The more experience I gained with Neo4j, the more I felt that nodes were a lot easier search for than, for example, relationships. Node colours are as follows:  

- Orange: Year Group
- Green: Class Group
- Yellow: Days of the Week
- Blue: Modules 
- Purple: Lecturers
- Red: Rooms

Modules/subjects tie everything together in any school or college timetable, so it made sense to have them act as a link between different nodes in this database.  Lecturers relate to Modules, which in turn relate to rooms and student groups - meaning a lecturer can easily find out what module they have with what student group and in what room. 
By running the following query, the lecturer is presented with their own node, whatever subjects they teach, and the rooms they're in. The lecturer can also see what group they have, in what room, for how long, and at what time by looking at the relationship properties between module and room.

	match (l:Lecturer)-[t:TEACHES]-(m:Module)-[i:IN]-(r:Room) where l.name = "I McLoughlin" return *
	
![Lecturer Search](https://github.com/rebeccabernie/TimetablingSystem/blob/master/Resources/QueryScreenshots/lecturerclasses.png "Lecturer Search")
	
Similarly, student groups can easily find out what module they have in what room and with what lecturer.

When dealing with lectures and how to differentiate them from practicals when searching, I decided to use the Year Group node to represent the year as a whole. The entire year group has the same lectures, but not necessarily the same lab groups. I had considered simply tying all lab groups to a lecture, but felt that it would have made the database more cluttered and slightly less searchable. By tying the year group to lectures, you can easily search for *just* lectures or *just* practicals. For example:

	match (c:Course)-[o:ON]-(d:Day)-[h:HAS]-(m:Module)-[i:IN]-(r:Room) where h.type = "Lecture" and i.type = "Lecture" return *
	
![Just Lectures](https://github.com/rebeccabernie/TimetablingSystem/blob/master/Resources/QueryScreenshots/alllectures.png "Just Lectures")

I had considered storing time slots as nodes, but felt this would have made things too complicated due to the amount of time slots as well as time slots overlapping every day. In the end I opted for making timeslots a property on relationships, along with type (lecture or practical) and duration. This makes it much easier to grab more information from the database, with smaller queries. For example, to see all Group A Labs for the week all you need to run is:

	match (g:Group{group:"Gr A"})-[o:ON]-(d:Day)-[h:HAS]-(m)-[i:IN]-(r) where h.group = "A" and i.group = "A" return *
	
![Group A Labs](https://github.com/rebeccabernie/TimetablingSystem/blob/master/Resources/QueryScreenshots/groupAlabs.png "Group A Labs")
	
Searching rooms is also an integral part of any timetable, so I felt having the ability to display when a room is occupied was important. Unfortunately there's no clear way of displaying when a room is free, but you can display all instances where the room is occupied. For example, if you want to see when room 0994 is occupied on a Wednesday, run the following query:

	match (r:Room)-[i:IN]-(m:Module) where i.day="Wednesday" and r.name="0994" return *
	
![Room Availablility](https://github.com/rebeccabernie/TimetablingSystem/blob/master/Resources/QueryScreenshots/roomdeetswednesday.png "Room Availability")
	
Hovering over a relationship between a room and a module gives information on time, which means you can find a free room by elimination. 


## Conclusion

While creating the database initially proved challenging, I got more and more used to it as time went on. Actually gathering data was one of the more difficult elements as it involved finding the information, formatting it somehow, and generating queries from it. Creating the database itself was relatively straightforward - once or twice I had to rethink the design and add properties here and there, but for the most part the process flowed quite well.

Displaying the whole database at once can be somewhat cluttered but if searches are filtered well, Neo4j automatically displays information well. Colour coding was a great asset when testing queries, as it gave a quick idea as to whether or not the query was in fact displaying what I wanted. It also helped a lot in figuring out the general efficiency of the database because I could easily see if different nodes looked like they should or shouldn't be related.

By recording all queries used, I've made it easy to expand this database in the future - it would just be a case of pulling data from the timetabling website and generating queries in the same way I did for this prototype. Any queries I used, whether for creating, editing or searching, were all very quick -  mostly 2 to 10 milliseconds but very occassionally up to 80/90 milliseconds. The only issue I ran into with speed was when I would forget to clear the slides - if I had 30/40 slides in the queue then I found the whole console would get slow, but this is human error more than anything else. As a whole, Neo4j executes varying queries extremely quickly.

To summarise, this project has highlighted the advantages of using Neo4j as a database, and also made me more comfortable with it and the Cypher language. I'm sure there are more efficient ways of storing this data or creating and querying the data, but I felt my design plans and logic were all very reasonable.









































