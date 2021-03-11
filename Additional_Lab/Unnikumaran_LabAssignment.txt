/*Yekaterina Unnikumaran
DBST652 Spring2021
Additional Lab Assignment*/

--#3 Make a copy of the STUDENT table and name it STUDENT1.
CREATE TABLE STUDENT1 AS (SELECT * FROM STUDENT); 

--#4 Define a user-defined object type data type named phone_type with attributes COUNTRY_CODE, AREA_CODE and PHONE_NUMBER
create or replace type phone_type as object(country_code varchar2(3), area_code varchar2(3), phone_number varchar2(12));

--#5 Define a user-defined VARRAY data type named Phone_List_type as an array of size three of the type phone_type
CREATE OR REPLACE TYPE PHONE_LIST_TYPE IS VARRAY(3) OF PHONE_TYPE;

--#6 Modify the table STUDENT1 by adding the attribute PHONE2 is of data type Phone_List_type
STUDENT1 ADD (PHONE2 PHONE_LIST_TYPE)

--#7 Populate the new PHONE2 column:  Use ‘001’ for the value of COUNTRY_CODE. Use the values of AREA_CODE, PHONE_NUMBER from PHONE
UPDATE STUDENT1 S SET S.PHONE2 = (phone_list_type(phone_type(('001'), (substr(S.phone,1,3)), (substr(S.phone,5,8)))));

--#8 Drop the column PHONE and rename PHONE2 as PHONE
alter table student1 drop column phone;
alter table student1 rename column phone2 to phone;

--#9 Write a query to display the values of the attributes student_ID and phone
select s.student_ID, v.* from student1 s, table (s.phone) v;

--#10 Create a type named course_T that has attributes named COURSE_NO, DESCRIPTION, ENROLL_DATE and FINAL_GRADE
create or replace type course_T as object(Course_No number(8,0), Description varchar2(50), Enroll_Date date, Final_Grade number(3,0)); 

--#11 Create an object table named course_Table_T as a table of course_T
create or replace type course_Table_T is table of course_T;

--#12 Create a table named transcript that contains the student_id and a column whose type is the table course_Table_T
create table transcript(Student_ID number(8,0), Transcript_attrib course_Table_T) nested table Transcript_attrib store as Transcript_tab;

--#13 Determine what courses have grades and update the transcript table accordingly
/* Using the following SQL, I determined the students that successfully completed the course. I also obtained the variables needed to populate the nested table and they included student_ID, course_no, Description, Enroll_Date, Final_Grade*/

select e.student_id, c.course_no, c.Description, e.enroll_date, e.final_grade from course c inner join section sc on c.course_NO = sc.course_NO inner join enrollment e on sc.section_ID = e.section_ID inner join student1 s on e.student_id=s.student_id where s.student_id =102 and e.final_grade is not null;

/* The above query yielded only data from only one student who had successfully completed the course. This student's information was inserted in the nested table as follows */
insert into transcript values(102, course_Table_T(Course_T(25,'Intro to Programming','30-JAN-07',92)));

select t.student_ID, c.* from transcript t, table(t.transcript_attrib) c;
--Notes: I ran the example statements above to understand how to pull final grades for students that were not null, 
then I ran it against the following statement, to double check that indeed only one student has a final grade:

--#14 Create and populate a table named Current_Enrollment that contains the student_id and a column whose type is the table course_Table_T. Include only those courses with no final grade
create table Current_Enrollment (student_id number(8,0), final_grade_attrib course_Table_T) nested table final_grade_attrib store as FinalGrade_tab;
