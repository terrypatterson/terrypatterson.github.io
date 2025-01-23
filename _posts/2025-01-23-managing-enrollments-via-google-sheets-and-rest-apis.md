---
title: Managing Enrollments with Google Sheets and Rest APIs
date: 2025-01-23 09:00:00 -0600
categories: [Enrollments, Rest APIs, Python]
tags: [documentation, enrollments, rest api, python, scripting]
---

# Managing Blackboard Enrollments using Google Sheets and Rest API via Python Script

## Introduction

 In Fall 2022, a team that manages the enrollment of tutors, supplemental instructors, and academic coaches approached the department with the request to help them enroll these users into their courses at the start of the coming Spring 2023 term. They had been keeping their enrollments in a Google spreadsheet (aka Google Sheets) and wanted to know if we could process this file. I was working on my Python certification so automating this with a Python script seemed like a great way to test out my python scripting skills and complete their task.

**Note: This is a living document, some information maybe incorrect or incomplete. Please contact me if you have any updates and/or corrections.**

Here is a sample of a csv that came out of their Google Sheet.

```
Course ID, User,Course Role,Availability Status, Last Updated,[DATETIME],Last Run,[DATETIME]
BLACKBOARD-COURSE-ID-001,j1234,T,Y,,,,
BLACKBOARD-COURSE-ID-002,j1234,T,Y,,,,
BLACKBOARD-COURSE-ID-003,J1234,T,Y,,,,
BLACKBOARD-COURSE-ID-004,j1234,T,Y,,,,
BLACKBOARD-COURSE-ID-005,j1234,T,Y,,,,
```

## My Intial Script

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