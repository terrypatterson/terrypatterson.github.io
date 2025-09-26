---
title: Managing Enrollments with Google Sheets and Rest APIs (Part 1)
date: 2025-01-23 09:00:00 -0600
categories: [Python, Rest APIs, Enrollments]
tags: [documentation, enrollments, rest api, python, scripting]
---

# Introduction

 In Fall 2022, a team that manages the enrollment of tutors, supplemental instructors, and academic coaches approached the department with the request to help them enroll these users into their courses at the start of the coming Spring 2023 term. They had been keeping their enrollments in a Google spreadsheet (aka Google Sheets) and wanted to know if we could process this file. I was working on my Python certification so automating this with a Python script seemed like a great way to test out my python scripting skills and complete their task.

<div style="border: 1px solid; padding:10px; font-style: italic; font-weight: bold;">
Note: This is a living document, some information maybe incorrect or incomplete. Please contact me if you have any updates and/or corrections.<br /><br />
I use Artifical Intelligence services (ChatGPT, Claude, Microsoft Copilot, Google Gemini, and  Perplexity) to help me plan, develop, and troubleshoot documentation and scripts shared on this site.
</div><br />

Here is a sample of a csv that came out of their Google Sheet.

```
Course ID, User,Course Role,Availability Status, Last Updated,[DATETIME],Last Run,[DATETIME]
BLACKBOARD-COURSE-ID-001,j1234,T,Y,,,,
BLACKBOARD-COURSE-ID-002,j1234,T,Y,,,,
BLACKBOARD-COURSE-ID-003,J1234,T,Y,,,,
BLACKBOARD-COURSE-ID-004,j1234,T,Y,,,,
BLACKBOARD-COURSE-ID-005,j1234,T,Y,,,,
```

## My Initial Script

So my first pass at this process was pretty elementary.

1. Accept a filename
2. Create a log to collect any errors / data
3. Strip the extra whitespace
4. Turn each line into a value array
5. Update the Availability Status to use the full word (Yes/No) instead of Y or N.
6. Use the bbrest library to create a connection to the Blackboard instance
7. Check to see that the course exists and is enabled in Blackboard
8. Check to see that the user exists and is enabled in Blackboard
9. Check to see if the user is current enrolled in the course
10. Update the enrollment to ensure it matches what is in the file
11. Add the enrollment if it doesn't exist and ensure it matches the file
12. Log every enrollment and its status to a log file.
13. Count all the various success and failures.
14. Output the result of the process to the log file and the terminal

Once the process completed it would output something like this.

```
The script has completed.
INFO: 142 records were processed.
SUCCESS: 136 records were processed successfully.
FAIL: 6 records failed during processing.
CREATE: 0 enrollments created in the process.
UPDATE: 136 enrollments update in the process.

Script finished processing at: 07/15/2024 14:45:00.
```

## Review of the Script

First the script configures the bbrest library to connect to the appropriate instance. This is also when I created the log file for the script.

```python

now = datetime.now()

from bbrest import BbRest
key = [REST API KEY]
secret = [REST API SECRET]
learnfqdn = [BLACKBOARD INSTANCE]
bb = BbRest(key, secret, "https://"+ str(learnfqdn) +"")  # Does a lot! Get the system version, pull in the functions from dev portal, 2-legged authentication w/ caching of token.

# Create a log file for the day's run
# Create the file name.
log_file_name = "bb-rest-enrollment-testing-log-" + now.strftime("%Y-%m-%d") + ".txt"
log_file_in_sub = "logs\\" + log_file_name

```

Now the script will prompt the user to put in the filename for processing. This was the easiest way for me to do it. I had plenty of examples from class that used this workflow, but later you will see how I made improvements. Also note I logged the start of the process.

```python

name = input("Enter file:")
textRow = open(name)
log = open(log_file_in_sub, "a")
## debug_log = open("bb-rest-enrollment-testing-debug.txt", "a")
dt_start_string = now.strftime("%m/%d/%Y %H:%M:%S")
log_start_timedate = "Script was started at: " + dt_start_string + '.\n\n'
log.write(log_start_timedate)

```

After my initial testing, I found that the Rest API integration didn't like any capital letters in the user ID. It seemed that forcing the team to always put in lower case letters in the user ID might be an annoyance so I wrote a function that would convert any upper case letter at the start of the user ID to a lower case one.

```python

def first_lower(variable):
  """
  This function takes a variable and looks at it to see if it starts with an uppercase character and changes the character to lowercase.

  Args:
    variable: The variable to be modified.

  Returns:
    The variable with the first character changed to lowercase.
  """

  if variable[0].isupper():
    return variable[0].lower() + variable[1:]
  else:
    return variable

```

Next, I knew the enrollments would be large, so I needed to create a way to keep track of any errors that happened. So I classified these categories so I could keep track of the various statuses. Here I set them out as variables.

```python

# Setting a counter to know how many enrollments were processed.
totalCount = 0
successCount = 0
errorCount = 0
createCount = 0
updateCount = 0

```

Now here's where the rubber meets the road. This area of the script cleans up and prepares the row of text for processing from the file. 

```python

for line in textRow:

    # Read a line into the script and remove any trailing characters.
    line = line.rstrip()

    # Check to see the line isn't blank.
    if len(line) < 1:
        continue
    
    # Create an array from the line splitting at the commas.
    value = line.split(',')
    
    # Take the value array and assign it to variables.
    course = value[0]
    userid = value[1]
    role = value[2]
    available = value[3]

    # Setting datasource to use SYSTEM for the enrollments.
    dataSource = "_2_1"

    # Fix userID to make sure it has a lowercase letter.
    userid = first_lower(userid)

    # Check to see if available is 'Y' or 'N' and update to full word

    if available == 'Y' or available == 'Yes' or available == 'YES':
        available = "Yes"
    elif available == 'N' or available == 'No' or available == 'NO':
        available = "No"
    else:
        logEvent = 'File contains incorrect characters in the availability column. Stopping...\n\n'
        log.write(logEvent)
        print(logEvent)
        break

```

**Author's Note:** *I will just state here that I think I did a good job of documenting and commenting the script so someone could follow along and understand what was going on.*

Now the script begins to process the row and use the bbrest library.

```python

    # Check to see if Rest API bearer token is expired.
    s = bb.is_expired()
    if s == True:
        # Refresh bearer token
        ref = bb.refresh_token()

    # Check to see if the course exists and if it's disabled.
    courseCheck = bb.GetCourse(courseId=course)
    courseCheckStatus = courseCheck.status_code
    
    # 404 code means the course doesn't exist in Blackboard. 
    if courseCheckStatus != 200:
        # Logging the event.
        logEvent = course + ' does not exist in Blackboard or isn\'t properly formatted in the feed file. Skipping enrollment...\n\n'
        log.write(logEvent)
        totalCount = totalCount + 1
        errorCount = errorCount + 1

    else:
        courseOutput = courseCheck.json()
        courseAvail = courseOutput['availability']['available']

        if (courseAvail == 'Disabled'):
            # Logging the event
            logEvent = course + ' has been disabled in Blackboard. Enrollments are not allowed. Skipping enrollment for ' + userid + '.\n\n'
            log.write(logEvent)
            totalCount = totalCount + 1
            errorCount = errorCount + 1

        else:
            userCheck = bb.GetUser(userId=userid)
            userCheckStatus = userCheck.status_code

            if userCheckStatus != 200:
                logEvent = userid + ' does not exist in Blackboard or isn\'t properly formatted in the feed file. Skipping enrollment...\n\n'
                log.write(logEvent)
                totalCount = totalCount + 1
                errorCount = errorCount + 1
        
```

The script above check to see if the course existed and wasn't disabled. It also checked to see if the user id exists in the Blackboard instance too.

Once that was done, we need to see if the user id is enrolled already in the course. Some instructors would manually add the users to their courses out of habit and I didn't want to stop that process. So I needed to check for this, because the script was also used to unassign enrollments in courses. 


```python
            # Check the status code from the Membership result and either create the membership or update it.
            enrollCheck = bb.GetMembership(courseId=course,userId=userid)
            enrollCheckStatus = enrollCheck.status_code

            # 404 status means membership doesn't exist, so create the membership in Blackboard.
            if enrollCheckStatus == 404:
                r = bb.CreateMembership(courseId=course,userId=userid,payload={"dataSourceId": dataSource,"availability": {"available": available},"courseRoleId": role})
                statusCode = r.status_code
                totalCount = totalCount + 1
                successCount = successCount + 1
                createCount = createCount + 1

                # Logging the membership creation event.
                logEvent = userid + ' has been enrolled as ' + role + ' in the course ' + course + ' with the availability of ' + available + '.\nBlackboard provided the following response status code ' + str(statusCode) + '.\n\n'
                log.write(logEvent)

            # 200 status means membership exists, so update the membership in Blackboard.
            elif enrollCheckStatus == 200:
                r = bb.UpdateMembership(courseId=course,userId=userid,payload={"dataSourceId": dataSource,"availability": {"available": available},"courseRoleId": role})
                statusCode = r.status_code
                totalCount = totalCount + 1
                successCount = successCount + 1
                updateCount = updateCount + 1

                # Logging the membership update event.
                logEvent = userid + ' has been enrolled as ' + role + ' in the course ' + course + ' with the availability of ' + available + '.\nBlackboard provided the following response status code ' + str(statusCode) + '.\n\n'
                log.write(logEvent)

            else:
                enrollOutput = enrollCheck.json()
                enrollCheckMsg = enrollOutput['message']
                print('Error: ' + str(enrollCheckStatus) + ' was provided from Blackboard when searching for ' + course + ' with the message \"' + enrollCheckMsg + '\". \n Skipping record: ' + str(value) + '.')
                totalCount = totalCount + 1
                errorCount = errorCount + 1

finalOutput = 'The script has completed.\nINFO: ' + str(totalCount) + ' records were processed.\nSUCCESS: ' + str(successCount) + ' records were processed successfully.\nFAIL: ' + str(errorCount) + ' records failed during processing.\nCREATE: ' + str(createCount) + ' enrollments created in the process.\nUPDATE: ' + str(updateCount) + ' enrollments update in the process.\n\n'
log.write(finalOutput)

# datetime object containing current date and time
now = datetime.now()

dt_end_string = now.strftime("%m/%d/%Y %H:%M:%S")
log_end_timedate = "Script finished processing at: " + dt_end_string + '.\n\n'
log.write(log_end_timedate)
log.close()
print(log_end_timedate)
print(finalOutput)



```

Once every row was processed, I output the end of the process and the counts for the process to the log file and the terminal window.

## Issues with the first version of this script

The first iteration of this script did the basics, but there were issues.

- Capitalized Letters in the username cause failure (break fix applied)
- File had to directly point to the absolute file path for the file.
- Had to remove extra commas by hand instead of removing them via a function in the script.

Part 2 reviews the second version of this script [Managing Enrollments with Google Sheets and Rest APIs (Part 2)](https://terrypatterson.github.io/posts/managing-enrollments-via-google-sheets-and-rest-apis-pt2/)