---
title: Blackboard DDA Queries
date: 2025-03-09 09:00:00 -0600
categories: [SQL, DDA]
tags: [SQL, DDA, queries]
---

Here are some of the top Blackboard DDA Queries that have saved me time and effort.

## Generate an SIS Feed File from a Database Query

Sometimes an admin needs to make a change to multiple courses at the same time. An SIS flat feed file can be the simplest way to do that, but generating that feed file could be a pain. Well it isn't with this SQL query. In this example, I'm taking all the cross listed courses (in this system they contain an -XLM- code in the Course ID) which are in the SYSTEM dsk and move them to the appropriate DSK so they will be archived.

```SQL

SELECT CM.BATCH_UID, '|', CM.COURSE_ID, '|', DS.BATCH_UID, '|219F_COURSES'
FROM COURSE_MAIN CM
    JOIN DATA_SOURCE DS
     ON CM.DATA_SRC_PK1 = DS.PK1
WHERE DS.BATCH_UID = 'SYSTEM' 
  AND CM.COURSE_ID LIKE '219F-XLM%'

```

This generates output that will be easy to copy and paste into a text file for processing. You can change the '|' for whatever delimiter you currently use. You may also change the 219F_COURSES to the data source key name that you want to move the courses under.

## Grab Information about the Device and Location of a User

Sometimes users will have an issue accessing content, report they were unable to complete a task, etc. These events can be difficult when the user is communicating indirectly with you the admin. If you are trying to troubleshoot, you must play a game of telephone to get simple information. However, if the issue happened recently and you know the date and/or time of the event you can use the data in the auth_provider_log table to get some data that can speed up the investigation. I use this query to get information on what type of system they are using.

```SQL

select apl.*
from auth_provider_log apl
join users u on apl.username = u.user_id
join course_users cu on u.pk1 = cu.users_pk1
join course_main cm on cu.crsmain_pk1 = cm.pk1
where cm.course_id = [COURSE_ID_HERE]
and cu.role = [COURSE_USER_ROLE_HERE]
and apl.username = [BLACKBOARD_USER_ID]

/* Uncomment the line below and it will allow you to search for a specific value in the user agent string.

--and apl.user_agent like '%ANGEL Secure%'

```

The user agent string will look similar to this below.

```
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/133.0.0.0 Safari/537.36
```

I then copy and past this into a User Agent Lookup to find out the specific information. My regular go to is <a href="https://www.whatismyip.net/tools/user-agent-lookup.php" target="_blank">What Is My IP</a>.

This can also be very helpful if you are a Respondus Lockdown Browser school and need to confirm that the user is accessing the test/assessment via the Respondus Lockdown Browser application and not going through a regular web browser.