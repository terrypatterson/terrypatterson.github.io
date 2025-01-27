---
title: Managing Enrollments with Google Sheets and Rest APIs (Part 2)
date: 2025-01-23 09:00:00 -0600
categories: [Python, Rest APIs, Enrollments]
tags: [documentation, enrollments, rest api, python, scripting]
---

![Image of the Wong Ranch from Futurama. A two story manison sits in the red dirt of Mars in the background. In the foreground, a fence with a square arch holds a sign above the entrance saying "Wong Ranch" a sign hanging from the arch says "You've Come to the Wong Place"](assets/img/posts/python-rest-api-enroll/wong-ranch.png)

## Last Time at the Documentation Ranch...

 In Fall 2022, a team that manages the enrollment of tutors, supplemental instructors, and academic coaches approached the department with the request to help them enroll these users into their courses at the start of the coming Spring 2023 term. They had been keeping their enrollments in a Google spreadsheet (aka Google Sheets) and wanted to know if we could process this file. I was working on my Python certification so automating this with a Python script seemed like a great way to test out my python scripting skills and complete their task.

## Items to Improve

 The first iteration met the basic needs/requirements for the team. Enrollments were processed, but the interface was "clunky" and not processing everything easily. Things could be done better so I wanted to make some improvements.

 1. Improve the way the system feeds a file into the script. Would be great if the script could just grab the file from Google.
 2. Remove the trailing commas that come over in each line.

**Note: This is a living document, some information maybe incorrect or incomplete. Please contact me if you have any updates and/or corrections.**

## Script Review

The first thing I need to do is call on several libraries that will help me navigate the issue of getting to the file without requiring the script runner to input the absolute filepath. 

```python

import requests
import pathlib
import os
import shutil
import pandas as pd
from datetime import datetime

# Update the following value when terms change, "Spring 2024" for example.

term = "TERM YEAR"

# datetime object containing current date and time
now = datetime.now()

```
This part of the script remains the same. The bbrest library has been a great library for this script, much appreciation and respect to [Matthew Deakyne](https://github.com/mdeakyne) for his work on developing and supporting this library. The picture below is myself and Matthew at Blackboard DevCon in 2019 with our buddy, Mark Reynolds sneaking into the picture.

[![A photo of three white males, Terry Patterson and Matthew Deakyne lean on each other back to back while another man, Mark Reynolds sneaks into the photo from the left side ](https://live.staticflickr.com/65535/48353180187_42aa56b3e0_w.jpg)](https://www.flickr.com/photos/terrypatterson/48353180187/in/album-72157709819185206)

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

This is the first location where this second version of the script differs from the first. If you download the .csv file from a Google Sheet, you will get a format that contains the name of the Google Sheet workbook and then the name of the sheet. The Google Sheets workbook each contains a sheet named for the term. So the file will always look like the example below when downloaded. Of course, this is telling the script where to find the downloaded file and then to make a copy of it for reference in case the script fails. 

The script then holds a copy of the file while it processes it.


```python

downloaded_enrollment_csv = "Tutor _ Embedded Coach Enrollments - " + term + ".csv"
enrollment_input_dir = "C:\\Users\\username\\Downloads"
enrollment_feed_dir = "D:\\python-scripts\\uploaded-files"
downloaded_full_enrollment_csv = os.path.join(enrollment_input_dir, downloaded_enrollment_csv)
enrollment_preprocess_dir = "D:\\python-scripts\\processing"

```

This section has two functions. The first checks to make sure the first letter in the username is lowercase and fixes it if the letter is uppercase. The second is the one that will move the csv file from the enrollment_input_dir value to the new directory and removes and whitespace within the filename.


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


def move_csv(csv_filepath, new_directory):
    """Reads a CSV file, saves a copy to a new directory, and reports success.

    Args:
        csv_filepath: The full path to the CSV file.
        new_directory: The directory where the copy should be saved.
    """

    try:
        # Read the CSV using pandas
        data = pd.read_csv(csv_filepath)

        # Get filename and create the new filepath
        old_filename = os.path.basename(csv_filepath)
        new_filename = old_filename.replace(" ", "_")
        new_filepath = os.path.join(new_directory, new_filename)

        # Create the new directory if it doesn't exist
        os.makedirs(new_directory, exist_ok=True)

        # Save the DataFrame (contents of CSV) to the new location
        data.to_csv(new_filepath, index=False)

    except FileNotFoundError:
        print(f"Error: CSV file not found at '{csv_filepath}'")
    except OSError as e:
        print(f"Error saving CSV: {e}")
    
    os.remove(downloaded_full_enrollment_csv)
    
    return new_filepath

```
By now you may notice that this version also uses functions to manage the various events and processes as part of the script. It hopefully shows my evolution in programming with Python. This function will remove the headers and extra commas in the first line of the csv file. This one took some time and I had to use some AI to help me better understand the best process to do this. I found that AI did help, but then once I saw the code, I took the time to review and explain to myself what it was doing. 

```python

def process_csv(input_filename, output_directory, output_filename):
    """
    Processes a CSV file:
        1. Removes the first line (header).
        2. Strips ",,," from each remaining line.
        3. Saves the processed file with a new name in a different directory.

    Args:
        input_filename (str): The full path to the input CSV file.
        output_directory (str): The path to the output directory.
        output_filename (str): The desired filename for the processed CSV.
    """

    os.makedirs(output_directory, exist_ok=True)  # Create the output directory

    with open(input_filename, 'r') as infile, \
         open(os.path.join(output_directory, output_filename), 'w', newline='') as outfile:
        next(infile)  # Skip the header line
        for line in infile:
            new_line = line.replace(",,,,", "")
            outfile.write(new_line)

    os.remove(input_filename)


```
This last function moves the entire enrollment process in the first version under a function which also calls the other inner functions to complete the process. This code is quite lengthy.


```python

def process_enrollments(downloaded_enrollment_csv):

    log = open(log_file_in_sub, "a")
    ## debug_log = open("bb-rest-enrollment-testing-debug.txt", "a")
    now = datetime.now()
    dt_start_string = now.strftime("%m/%d/%Y %H:%M:%S")
    log_start_timedate = "Script was started at: " + dt_start_string + '.\n\n'
    log.write(log_start_timedate)
    
    downloaded_cleaned_enrollment_csv = move_csv(downloaded_full_enrollment_csv, enrollment_preprocess_dir)
    
    enrollment_input_file = os.path.join(enrollment_input_dir, downloaded_cleaned_enrollment_csv)
    enrollment_process_dir = os.path.abspath("D:\\bb-tutor-enrollments\\uploaded-files")
    enrollment_processed_file = "bb-tutor-enroll-" + now.strftime("%m%d%Y") + ".csv"
    enrollment_processed_filepath = os.path.join(enrollment_process_dir, enrollment_processed_file)
    
    process_csv(enrollment_input_file, enrollment_process_dir, enrollment_processed_file)
    
    
    textRow = open(enrollment_processed_filepath, 'r')
    
    # Setting a counter to know how many enrollments were processed.
    totalCount = 0
    successCount = 0
    errorCount = 0
    createCount = 0
    updateCount = 0

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
    
    
def main():
    process_enrollments(downloaded_enrollment_csv)
    
main()

```


