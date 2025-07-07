---
title: Finding and Identifying Cross Linked Course Content
date: 2025-07-02 09:00:00 -0600
categories: [Cleanup, Orphaned Content, Cross Linked]
tags: [documentation, course removal]
---

# Introduction

Course deletions have always been a difficultly in Blackboard. If you were self-hosted and didn't have enough database or application server power, your system could go down with the simple deletion of some courses. (I know, I did it.). Then even in managed hosting or SaaS environments, deleting thousands of courses could delay course copies or imports for hours while the task workers processed these requests. So how can you delete courses without having either impact? This is why I wrote this script.

**Note: This is a living document, some information maybe incorrect or incomplete. Please contact me if you have any updates and/or corrections.**

So this was built to delete courses in small batches so it wouldn't overtax the environment nor block the processing of other tasks such as course copies or imports. It also removed me from having to babysit the process so I could enjoy weekends and/or vacations.



## Running the Query

To find cross-linked content within courses, we use the following query. Note that a major part of the query is commented out. This query is too big to run as a complete process on our database. So we are going to break each part out into sections. 

```SQL

select 
	content.course_pk1
	, content.course_id
	, case 
		when position('internal/' in xyf_files.full_path) > 0 then '/'||split_part(xyf_files.full_path, '/', 2)||'/'||split_part(xyf_files.full_path, '/', 3)||'/'||split_part(xyf_files.full_path, '/', 4)||'/' 
		when position('courses/' in xyf_files.full_path) > 0 or position('orgs/' in xyf_files.full_path) > 0 then '/'||split_part(xyf_files.full_path, '/', 2)||'/'||split_part(xyf_files.full_path, '/', 3)||'/' 
		else '/'||split_part(xyf_files.full_path, '/', 2)||'/' 
		end as course_home
	, count(*) as embed_count
	, content.cnthndlr_handle as embed_type
from (
	--select course_main.pk1 as course_pk1, course_contents.pk1 as content_pk1, 'Attached' as cnthndlr_handle, course_contents.title as content_title, course_main.course_id, files.file_name as xid
	--	from course_contents_files
	--	left outer join files on course_contents_files.files_pk1 = files.pk1
	--	left outer join course_contents on course_contents_files.course_contents_pk1 = course_contents.pk1
	--	left outer join course_main on course_contents.crsmain_pk1 = course_main.pk1 
	--union
	--	select course_main.pk1 as course_pk1, course_contents.pk1 as content_pk1, 'Embedded Content' as cnthndlr_handle, course_contents.title as content_title, course_main.course_id, array_to_string(regexp_matches(course_contents.main_data, 'bbcswebdav([^\s"''<\)]+)','g'), '') as xid
	--	from course_contents
	--	left outer join course_main on course_contents.crsmain_pk1 = course_main.pk1
	--	where course_contents.main_data like '%bbcswebdav%'
	--union
	--	select course_main.pk1 as course_pk1, announcements.pk1 as content_pk1, 'Announcement' as cnthndlr_handle, announcements.subject as content_title, course_main.course_id, array_to_string(regexp_matches(announcements.announcement, 'bbcswebdav([^\s"''<\)]+)','g'), '') as xid
	--	from announcements
	--	left outer join course_main on announcements.crsmain_pk1 = course_main.pk1
	--	where announcements.announcement like '%bbcswebdav%'
	--union
	--	select course_main.pk1 as course_pk1, qti_asi_data.pk1 as content_pk1, 'Assessment' as cnthndlr_handle, qti_asi_data.title as content_title, course_main.course_id, array_to_string(regexp_matches(encode(qti_asi_data.data::bytea, 'escape'), 'bbcswebdav([^\s"''<\)]+)','g'), '') as xid 
	--	from qti_asi_data 
	--	left outer join course_main on qti_asi_data.crsmain_pk1 = course_main.pk1  
	--	where encode(data::bytea, 'escape') like '%bbcswebdav%' 
	--      and course_main.data_src_pk1 IN (PK1) 
	--union
	--	select course_main.pk1 as course_pk1, forum_main.pk1 as content_pk1, 'Discussion Forum' as cnthndlr_handle, forum_main.name as content_title, course_main.course_id, array_to_string(regexp_matches(forum_main.description, 'bbcswebdav([^\s"''<\)]+)','g'), '') as xid
	--	from forum_main
	--	left outer join conference_main on forum_main.confmain_pk1 = conference_main.pk1 
	--	left outer join course_main on conference_main.crsmain_pk1 = course_main.pk1 
	--	where forum_main.description like '%bbcswebdav%'
	--union
	--	select course_main.pk1 as course_pk1, msg_main.pk1 as content_pk1, 'Discussion Post' as cnthndlr_handle, msg_main.subject as content_title, course_main.course_id, array_to_string(regexp_matches(msg_main.msg_text, 'bbcswebdav([^\s"''<\)]+)','g'), '') as xid
	--	from msg_main
	--	left outer join forum_main on msg_main.forummain_pk1 = forum_main.pk1 
	--	left outer join conference_main on forum_main.confmain_pk1 = conference_main.pk1 
	--	left outer join course_main on conference_main.crsmain_pk1 = course_main.pk1 
	--	where msg_main.msg_text like '%bbcswebdav%'
	--union
	--	select course_main.pk1 as course_pk1, blogs.pk1 as content_pk1, 'Blog or Journal' as cnthndlr_handle, blogs.title as content_title, course_main.course_id, array_to_string(regexp_matches(blogs.description, 'bbcswebdav([^\s"''<\)]+)','g'), '') as xid
	--	from blogs
	--	left outer join course_main on blogs.crsmain_pk1 = course_main.pk1 
	--	where blogs.description like '%bbcswebdav%'
	--union
	--	select course_main.pk1 as course_pk1, blog_entry.pk1 as content_pk1, 'Blog Entry' as cnthndlr_handle, blog_entry.title as content_title, course_main.course_id, array_to_string(regexp_matches(blog_entry.description, 'bbcswebdav([^\s"''<\)]+)','g'), '') as xid
	--	from blog_entry 
	--	left outer join blogs on blog_entry.blog_pk1 = blogs.pk1 
	--	left outer join course_main on blogs.crsmain_pk1 = course_main.pk1 
	--	where blog_entry.description like '%bbcswebdav%'
	--union
	--	select course_main.pk1 as course_pk1, '0' as content_pk1, 'Banner' as cnthndlr_handle, 'Banner' as content_title, course_main.course_id, case when substr(course_main.banner_url,1,1) = '/' then course_main.banner_url else '0' end as xid
	--	from course_main 
	--	where course_main.banner_url is not null
) as content
left outer join (
	select *
	from dblink('dbname=DATABASE_cms_doc host=localhost sslmode=require user=USERNAME password=PASSWORD', 'select xyf_files.entry_id, xyf_urls.full_path from xyf_files left outer join xyf_urls on xyf_files.file_id = xyf_urls.file_id')
	as xyf_files(entry_id int, full_path text)
) as xyf_files on content.xid = '/xid-'||cast(xyf_files.entry_id as text)||'_1'
group by content.course_pk1, content.course_id, case when position('internal/' in xyf_files.full_path) > 0 then '/'||split_part(xyf_files.full_path, '/', 2)||'/'||split_part(xyf_files.full_path, '/', 3)||'/'||split_part(xyf_files.full_path, '/', 4)||'/' when position('courses/' in xyf_files.full_path) > 0 or position('orgs/' in xyf_files.full_path) > 0 then '/'||split_part(xyf_files.full_path, '/', 2)||'/'||split_part(xyf_files.full_path, '/', 3)||'/' else '/'||split_part(xyf_files.full_path, '/', 2)||'/' end, content.cnthndlr_handle

```

Each run, we want to remove the -- from in front of a query within the from ( ). In this example we want to only run the query that will help us find cross-linked content that is attached within courses. So we convert this section:

```SQL

	--select course_main.pk1 as course_pk1, course_contents.pk1 as content_pk1, 'Attached' as cnthndlr_handle, course_contents.title as content_title, course_main.course_id, files.file_name as xid
	--	from course_contents_files
	--	left outer join files on course_contents_files.files_pk1 = files.pk1
	--	left outer join course_contents on course_contents_files.course_contents_pk1 = course_contents.pk1
	--	left outer join course_main on course_contents.crsmain_pk1 = course_main.pk1 

```

to this

```SQL

	select course_main.pk1 as course_pk1, course_contents.pk1 as content_pk1, 'Attached' as cnthndlr_handle, course_contents.title as content_title, course_main.course_id, files.file_name as xid
		from course_contents_files
		left outer join files on course_contents_files.files_pk1 = files.pk1
		left outer join course_contents on course_contents_files.course_contents_pk1 = course_contents.pk1
		left outer join course_main on course_contents.crsmain_pk1 = course_main.pk1 

```

We only comment that section out of the query. Now run the query. Once complete, you should only get the values associated with the attached files in the course. will appear like this.

| Course PK1 | Course ID | Course Home Directory | Count of Items | Type of Content |
| ---------- | --------- | --------------------- | -------------- | --------------- |
|937640|8251-000000-EMSP-2260-001|/courses/8251-000000-EMSP-2260-001/|11|Attached|
|912160|4567-000000-DFTG-1433-DLS-16wk|/courses/4567-000000-DFTG-1433-DLS-16wk/|7|Attached|
|879799|1237-000000-UXUI-1370-004|/courses/1237-000000-UXUI-1370-004/|15|Attached|
|933083|123R-000000-GERM-2311-001|/courses/123R-000000-GERM-2311-001/|8|Attached|
|920993|593F-000000-CHEM-2123-002|/courses/593F-000000-CHEM-2123-002/|9|Attached|
|888005|964S-000000-ITSC-2337-003|/courses/964S-000000-ITSC-2337-003/|37|Attached|

So with these results, how can you tell what content is cross-linked. That's a good question. If the Course ID does not match the Course Home Directory, the content is cross-linked. We will manipulate this content later in the steps, but make sure your output comes out like this.

When you run each section, save the output into a folder (such as 2024-cross-linked-content-data) as a CSV file.

![Screenshot of a list of files in Windows file manager](assets/img/posts/cross-linked-content/cross-linked-content-data-image.png)

Once you complete each process you should have the following files:

- announcements-results.csv
- discussion-forum-data.csv
- discussion-post-data.csv
- blog-journal-data.csv
- blog-entry-data.csv
- course-banner-data.csv
- attached-content-data.csv
- embedded-content-data.csv
- assessments-data.csv

Note that the assessments portion of the query are much more taxing on the database, so you will need to break your query into data sources pk1 values. This can be found by running the following query.

```SQL

select pk1, batch_uid
from data_source

```

Select your pk1 values for all the data_source_keys that contain courses.

## Manipulating the Data

So now we have the data, but we need to make some slight changes to make it so that we can find files in courses that have a different course home directory. 

What we want to do is convert this...

| Course PK1 | Course ID | Course Home Directory | Count of Items | Type of Content |
| ---------- | --------- | --------------------- | -------------- | --------------- |
|937640|8251-000000-EMSP-2260-001|/courses/8251-000000-EMSP-2260-001/|11|Attached|
|912160|4567-000000-DFTG-1433-DLS-16wk|/courses/4567-000000-DFTG-1433-DLS-16wk/|7|Attached|
|879799|1237-000000-UXUI-1370-004|/courses/1237-000000-UXUI-1370-004/|15|Attached|
|933083|123R-000000-GERM-2311-001|/courses/123R-000000-GERM-2311-001/|8|Attached|
|920993|593F-000000-CHEM-2123-002|/courses/593F-000000-CHEM-2123-002/|9|Attached|
|888005|964S-000000-ITSC-2337-003|/courses/964S-000000-ITSC-2337-003/|37|Attached|

into this...

| Course PK1 | Course ID | Course Content Folder | Course Home Directory | Count of Items | Type of Content |
| ---------- | --------- | --------------------- | --------------------- | -------------- | --------------- |
|937640|8251-000000-EMSP-2260-001|8251-000000-EMSP-2260-001|/courses/8251-000000-EMSP-2260-001/|11|Attached|
|912160|4567-000000-DFTG-1433-DLS-16wk|4567-000000-DFTG-1433-DLS-16wk|/courses/4567-000000-DFTG-1433-DLS-16wk/|7|Attached|
|879799|1237-000000-UXUI-1370-004|1237-000000-UXUI-1370-004|/courses/1237-000000-UXUI-1370-004/|15|Attached|
|933083|123R-000000-GERM-2311-001|123R-000000-GERM-2311-001|/courses/123R-000000-GERM-2311-001/|8|Attached|
|920993|593F-000000-CHEM-2123-002|593F-000000-CHEM-2123-002|/courses/593F-000000-CHEM-2123-002/|9|Attached|
|888005|964S-000000-ITSC-2337-003|964S-000000-ITSC-2337-003|/courses/964S-000000-ITSC-2337-003/|37|Attached|

To do this, we need to open each CSV file in Microsoft Excel. Once open in the program. Highlight the column that contains the Course Home Directory, right click and select Copy, then right click again and select "Insert Copied Cells" which will insert the column to the left of the Course Home Directory column. 

Select this entire new column as we now need to remove the forward slashes and /courses/ from the cells so we have just the name of the folder. This will make it easier to compare and determine what content is not located within the home directory. Select Find & Select from the Home tab in the Excel program. Select the Replace tab. Input /courses/ into the Find what box and leave the Replace with box empty. Click options, set Within to Sheet and Search to By Columns. Then click on Replace All and this should only change the content within the column. Now follow the same steps, only use just a forward slash.

This will present you with a clean column that you should now title course_content_folder. 

Save the file as a .csv file. You may wish to append the word -clean to the end of the file.

Note that some older courses might have /internal/courses/COURSE_ID/ so make sure to search for any /internal/ values that might be in the results.

## Adding the Data to a Database

Now that you have this cleaned data, you want to be able to find out what course IDs might contain or rely on cross-linked content. The easiest way to do that is to turn each of these .csv files into a table in a SQLite database. An SQLite database is a simple file based database that can just reside in your local file system.  Here is how to create a SQLite datbase or open one in DBeaver.

Setting Up DBeaver for SQLite

Step 1: Download and Install DBeaver

1. Visit the DBeaver official website.

2. Download the appropriate version for your operating system.

3. Install DBeaver following the on-screen instructions.

Step 2: Download SQLite Driver

1. Open DBeaver.

2. Click on Database > New Database Connection.

3. Search for SQLite and select it.

4. If the SQLite driver is not already installed, click Download to install it.

Step 3: Connect to an SQLite Database

1. In the New Database Connection wizard: Enter the path to your SQLite database file or create a new one. Provide a name for your connection.
2. Click Finish to complete the connection setup.

Now that you have the database created, open the database and expand it. Underneath the database will be several folders. Right-click on the Tables folder. Select Import Data from the menu. This will allow you to create a table from the .csv files you have generated. Step through the Data Transfer Wizard.

1. Select Import Source as CSV. Click Next.
2. Find your CSV file using the Select Input Files window.
3. Under Importer settings, make sure that the , character appears as the Column delimiter. Click Next.
4. Expand the target container so you can see the source and target columns for the table. Change the target table name to be clc_[embed_type]. For example, if this is the file with announcements, the table name would be clc_announcements.
5. Click Proceed and your table will be generated.

Repeat these steps for all your .csv files. This will give you several tables like the ones shown below.

![Screenshot of a list of tables in DBeaver](assets/img/posts/cross-linked-content/database-tables.png)

## Generating the Queries and Data to Review

Now all of our data resides in a small and easy to query database of tables. So how do we find the courses that have cross linked content. We can simply query them. I already have a set of queries, let's go over one of them to find how its generated.

```SQL

SELECT *
FROM clc_announcement
WHERE course_id != course_content_folder
and (course_id LIKE '222F%' OR course_id LIKE '223F%' OR course_id LIKE '223S%' OR course_id LIKE '223U%' OR course_id LIKE '224F%' OR course_id LIKE '224S%' OR course_id LIKE '224U%')
and (course_content_folder LIKE '213F%' OR course_content_folder LIKE '213S%' OR course_content_folder LIKE '213U%' OR course_content_folder LIKE '214F%' OR course_content_folder LIKE '214S%' OR course_content_folder LIKE '214U%' OR course_content_folder LIKE '214F%' OR course_content_folder LIKE '214S%' OR course_content_folder LIKE '214U%' OR course_content_folder LIKE '215F%' OR course_content_folder LIKE '215S%' OR course_content_folder LIKE '215U%' OR course_content_folder LIKE '216F%' OR course_content_folder LIKE '216S%' OR course_content_folder LIKE '216U%' OR course_content_folder LIKE '217F%' OR course_content_folder LIKE '217S%' OR course_content_folder LIKE '217U%' OR course_content_folder LIKE '218F%' OR course_content_folder LIKE '218S%' OR course_content_folder LIKE '218U%' OR course_content_folder LIKE '219F%' OR course_content_folder LIKE '219S%' OR course_content_folder LIKE '219U%' OR course_content_folder LIKE '220F%' OR course_content_folder LIKE '220S%' OR course_content_folder LIKE '220U%' OR course_content_folder LIKE '221F%' OR course_content_folder LIKE '221S%' OR course_content_folder LIKE '221U%' OR course_content_folder LIKE '222S%' OR course_content_folder LIKE '222U%')

```

The first three lines of this query should be rather easy to review. Find me all rows in clc_announcement where the course_id value doesn't match the course_content_folder value. This will find content that is listed in the course_id but the file resides in the course_content_folder of a different course. 

The lines after that are a bit crazy. The first AND statement wants to find all the course_ids of courses that are currently active and not scheduled to remove. This means that these courses rely on the content and if any cross-linked content appears within them. It must be saved and not removed.

The next line looks for any courses that are eligible to be removed from the system. If active courses are using cross-linked content from courses scheduled for removal, then it must be retained.

These values will need to be appended to include the past year for both removal and active courses. This is as simple as adding the following

```SQL
OR course_content_folder LIKE '222F%'
```

```SQL
OR course_id LIKE '225S%'
```

If you are looking for a specific course, you can run the following query.

```SQL
SELECT *
FROM clc_embedded_content
WHERE course_id != course_content_folder and (course_id LIKE '224F-123345-PSYC-2301-001' OR course_content_folder LIKE '224F-123345-PSYC-2301-001')
```

This content will allow you to query and find any cross-linked content. 