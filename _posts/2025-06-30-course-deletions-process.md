---
title: Deleting Courses from Blackboard Without Causing Issues
date: 2025-06-30 09:00:00 -0600
categories: [Python, Selenium, Deleting]
tags: [documentation, course removal, selenium, python, scripting]
---

# Introduction

Course deletions have always been a difficultly in Blackboard. If you were self-hosted and didn't have enough database or application server power, your system could go down with the simple deletion of some courses. (I know, I did it.). Then even in managed hosting or SaaS environments, deleting thousands of courses could delay course copies or imports for hours while the task workers processed these requests. So how can you delete courses without having either impact? This is why I wrote this script.

<div style="border: 1px solid; padding:10px; font-style: italic; font-weight: bold;">
Note: This is a living document, some information maybe incorrect or incomplete. Please contact me if you have any updates and/or corrections.<br /><br />
I use Artifical Intelligence services (ChatGPT, Claude, Microsoft Copilot, Google Gemini, and  Perplexity) to help me plan, develop, and troubleshoot documentation and scripts shared on this site.
</div><br />

So this was built to delete courses in small batches so it wouldn't overtax the environment nor block the processing of other tasks such as course copies or imports. It also removed me from having to babysit the process so I could enjoy weekends and/or vacations.

## Generating the course deletion files

So first you must know that at Austin Community College, we delete courses once a year. That's between 16-18k of courses removed from the system. To do that we have a data integration specifically for this process and we have to first migrate all courses from their original DSK and move them to the Course Deletion DSK (think of this like moving your files to the Recycle Bin). 

```

221F-29005-COURSE-002|221F_COURSE|COURSE_DELETION
221F-29746-COURSE-046|221F_COURSE|COURSE_DELETION
221F-26919-COURSE-002|221F_COURSE|COURSE_DELETION

```

The query looks like this...

```sql
select cm.batch_uid||'|' || ds.batch_uid || '|COURSE_DELETION'
from course_main cm
    join data_source ds
     on cm.data_src_pk1 = ds.pk1
where ds.batch_uid = '224S_COURSE'

```

Once moved to the new DSK, you can generate the course deletion file. Here is the query to run.

```sql
select cm.batch_uid, cm.course_id, cm.course_name
from course_main cm
	join data_source ds
	 on cm.data_src_pk1 = ds.pk1
where ds.batch_uid LIKE 'COURSE_DELETION'

```

This will generate a huge file. We want to split that file up into smaller chunks. This can be done with a simple python script. 


```python
def split_file(input_file, output_dir, lines_per_file=100):
  """Splits a text file into multiple files with a specified number of lines.

  Args:
    input_file: Path to the input text file.
    output_dir: Path to the directory where output files will be saved.
    lines_per_file: Number of lines per output file.
  """

  with open(input_file, 'r') as f:
    lines = f.readlines()

  file_counter = 1
  line_counter = 0
  current_file_lines = ["EXTERNAL_COURSE_KEY|COURSE_ID|COURSE_NAME\n"]  # Start with the header

  for line in lines:
    current_file_lines.append(line)
    line_counter += 1

    if line_counter == lines_per_file:
      output_file = f"{output_dir}/COURSE_DELETION_{file_counter:04d}.txt"
      with open(output_file, 'w') as outfile:
        outfile.writelines(current_file_lines)
      file_counter += 1
      line_counter = 0
      current_file_lines = ["EXTERNAL_COURSE_KEY|COURSE_ID|COURSE_NAME\n"]  # Reset with header

  # Write any remaining lines to a final output file
  if current_file_lines:
    output_file = f"{output_dir}/COURSE_DELETION_{file_counter:04d}.txt"
    with open(output_file, 'w') as outfile:
      outfile.writelines(current_file_lines)


if __name__ == "__main__":
  input_file = "big-list-course-deletions.txt"  # Replace with your input file path
  output_dir = "SPLIT-FILES-DIR"  # Replace with your output directory path
  split_file(input_file, output_dir)

```

## Processing the Course Deletion Files

So now you have about 60-80+ files that have 100 rows of courses to be removed from Blackboard. But how to remove these in batches and without sending them all at once. That's where a series of functions working with Selenium can make this happen.

This script, called auth.py, launches Selenium and logs into the Blackboard Instance using a local username and password.

```python
# Set your domain here
my_domain = "https://instance-name.blackboard.com"

# Set your username
user_id = "Insert Local Username Here"
user_password = "Insert Local Password Here"

from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.common.exceptions import NoSuchElementException
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.desired_capabilities import DesiredCapabilities
import time, base64, os

def login():  

    username = os.getlogin()

    # Pulls up the course manager
    chrome_options = webdriver.ChromeOptions()
    chrome_options.add_argument("--start-maximized")
    chrome_options.add_argument("--ignore-certificate-errors")
    chrome_options.add_argument("--ignore-ssl-errors")
    chrome_options.add_argument("--ignore-certificate-errors-spki-list")
    chrome_options.add_argument("--user-data-dir=C:\\Users\\" + username + "\\AppData\\Local\\Google\\Chrome")    chrome_options.add_argument("--profile-directory=Profile 4")
    chrome_options.add_argument("--remote-allow-origins=*")
    chrome_options.add_argument("--headless=new")
    #chrome_options.add_argument("--incognito");

    chrome_options.set_capability("pageLoadStrategy", "normal") #-- full page load
    #chrome_options.set_capability("pageLoadStrategy", "eager") #-- wait for the DOMContentLoaded event
    #chrome_options.set_capability("pageLoadStrategy", "none") #-- return immediately after html content downloaded

    chrome_options.add_argument("enable-automation")
    chrome_options.add_argument("--window-size=1920,1080")
    chrome_options.add_argument("--no-sandbox")
    chrome_options.add_argument("--disable-extensions")
    chrome_options.add_argument("--dns-prefetch-disable")
    chrome_options.add_argument("--disable-gpu")
    chrome_options.add_argument("--disable-cache")
    chrome_options.add_argument("--log-level=3")


    # Initiate Driver
    #service = Service(r"C:/BrowserDriver/123/chromedriver.exe")
    driver = webdriver.Chrome(options=chrome_options)
    #driver = webdriver.Chrome()
    driver.delete_all_cookies


    """
    driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()))
    """
    # Load Login Page
    driver.get(my_domain+"/webapps/login/?action=default_login")
    #print("9")
    
    if find_element_by_id(driver, "agree_button"):
        time.sleep(2)
        driver.execute_script("document.getElementById('agree_button').click()")

    if find_element_by_id(driver, "user_id"):
        driver.find_element(By.ID, "user_id").send_keys(user_id)
        driver.find_element(By.NAME, "password").send_keys(user_password)
        driver.execute_script("document.getElementById('entry-login').click()")
    
    print("=================\nLog In\n=================")

    return driver

def logout(driver):
    # Logout
    driver.get(my_domain+"/webapps/login/?action=logout")

    # Wait
    time.sleep(2)

    # Quit Browser
    driver.quit()

    print("=================\nLog Out\n=================")

def find_element_by_id(driver, element_id):
    try:
        driver.find_element(By.ID, element_id)
    except NoSuchElementException:
        return False
    return True


```

This script handles logging in and out of the Blackboard instance. You will just need to provide an admin username and password and the URL of the Blackboard instance. This bigger one does much of the work and also calls the auth.py one to run commands. I'm going to break this one up by functions.

```python
from auth import *
import time
import datetime
import shutil
import os
import subprocess
import requests
import urllib3
import smtplib
import json
import logging
from email.mime.text import MIMEText
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import Select, WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.common.exceptions import TimeoutException

x = datetime.datetime.now()

time_now = x.strftime("%H") + "-" + x.strftime("%M")

log_folder = x.strftime("%Y") + "-" + x.strftime("%m") + "-" + x.strftime("%d")

home_folder = "DIRECTORY"

log_folder_path = home_folder + "logs\\" + log_folder

log_filename = home_folder + "logs\\" + log_folder + "\\course_deletion_log_" + time_now + '.txt'

if not os.path.exists(log_folder_path):
  os.makedirs(log_folder_path)
else:
  print(f"Log File Already Exists: {log_folder_path}")

if not os.path.exists(log_filename):
  with open(log_filename, 'w') as f:
    print(f"Log File Created: {log_filename}")
else:
  print(f"Log File Already Exists: {log_filename}")
  
logging.basicConfig(filename=log_filename, level=logging.INFO,
                    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s')

```
The part of the script above was simply to configure constants and setup logging.

```python
def print_file_contents(filename):
    """
    Opens a file and prints its contents to the console.

    Args:
        filename (str): The name or path of the file to open.
    """

    try:
        # Open the file in read mode ('r')
        with open(filename, 'r') as file:
            # Read the entire contents of the file
            contents = file.read()
            # Print the contents
            print(contents)
            file.close()

    except FileNotFoundError:
        logging.info(f"Error: File '{filename}' not found.")
```

This function opens the file and prints it out. This was part of testing the script.

```python
def stop_process_if_done(file_path):
    """Checks if the first line of a file contains the word 'Done'.

    Args:
        file_path (str): The path to the file.

    Returns:
        bool: True if the first line contains 'Done', False otherwise.
    """
    with open(file_path, 'r') as file:
        logging.info(file_path + " was opened and checked to see if the file contains the word done. This would stop the process from running again if cron'ed or on task scheduler.")
        first_line = file.readline()
        logging.info(file_path + " first line states " + first_line)
        logging.info("stop_process_if_done function completed.")
        file.close()
        return "Done" in first_line

```

Once the process completes, it inserts the word "Done" in a file. This function checks to see if that exists and stops the script. This is helpful if you have it cron'd.

```python

def process_complete(file_name):
    """Writes the word 'Done' to a file.

    Args:
        file_name (str): The name of the file to write to.
    """

    # Option 1: Overwrite an existing file or create a new one
    logging.info("Based on the file in next_file_to_process.txt not existing, the script is writing to the file the word Done which will stop any cron'd or task scheduled events from running the process again.")
    with open(file_name, 'w') as f:
        f.write("Done")
    logging.info("process_complete function has completed.")

```

The function above writes the word "Done" in the file.

```python

def file_exists(file_path):
    """Checks if a file exists.

    Args:
        file_path (str): The path to the file.

    Returns:
        bool: True if the file exists, False otherwise.
    """
    logging.info("This function is checking to see if the file exists and returns that information to the parent function.")
    return os.path.exists(file_path)

```

A simple function to just be able to check if a file exists within a specific location. 

```python

def send_email_notify(code):
    
    SENDER_EMAIL = 'SENDER_EMAIL'
    RECIPIENT_EMAILS = ['EMAIL_01','EMAIL_02','EMAIL_03']
    SUBJECT = 'Course Deletion Script Alert'

    currently_processing_file_loc = home_folder + "current_file_processing.txt"
    current_file_being_processed = read_filename_from_textfile(currently_processing_file_loc)

    logging.info("Script has called the Notify function that will send an email to my text email address to update me regarding an issue with the script or completion.")

    messages = {
    1: "The script has sent " + current_file_being_processed + " to Blackboard Staging for course removal.",
    2: "The script reported a failure and has stopped running. Please review logs.",
    3: "The script cannot find " + current_file_being_processed + ". It appears the process has completed. The script has stopped.",
    4: "Too many courses are in waiting status so the script will not send new courses."}

    """Args:
        code (int): The code determining the message to send.
        recipient_email (str): The email address of the recipient.
        subject (str, optional): The subject of the email. Defaults to "Message from Python".
    """

    if code not in messages:
       logging.info(f"Invalid code: {code}")
       logging.info("The function has failed due to an improper code being passed to the function." + str(code))
       return

    message_body = messages[code]

    message = MIMEText(message_body)
    message['Subject'] = SUBJECT
    message['From'] = SENDER_EMAIL
    message['To'] = '; '.join(RECIPIENT_EMAILS)

    try:
        # Use SMTP_SSL for secure connection
        with smtplib.SMTP('SMTP-SERVER', 25) as server:
            server.ehlo()  
            #server.sendmail(message['From'], message['To'], message.as_string())
            server.send_message(message)
        logging.info('Email sent successfully!')
        logging.info("Notify function was able to successfully send a notification to the script author.")
    except Exception as e:
        logging.info(f'Error sending email: {e}')
        logging.info("Notify function had a failure when trying to send the notification to the script author.")

def push_message(code):
	token = "PUSHBULLET_TOKEN"
	url = "https://api.pushbullet.com/v2/pushes"
	headers = {"content-type": "application/json", "Access-Token": token}
		
	currently_processing_file_loc = home_folder + "\\course-deletion-status\\current_file_processing.txt"
	current_file_being_processed = read_filename_from_textfile(currently_processing_file_loc)
     
	logging.info("Script has called the Notify function that will send an email to my text email address to update me regarding an issue with the script or completion.")

	if code == 1:

		title = "Course Deletion Script Alert"
		messageBody = "The script has sent " + current_file_being_processed + " to Blackboard Staging for course removal."
		data_send = {"body":messageBody,"title":title,"type":"note","device_iden":"ujyL4J2SukKsjDz2agNPzM"}
		_r = requests.post(url, headers=headers, data=json.dumps(data_send))

	elif code == 2:
    
		title = "Course Deletion Script Failure"
		messageBody = "The script reported a failure and has stopped running. Please review logs."
		data_send = {"body":messageBody,"title":title,"type":"note","device_iden":"ujyL4J2SukKsjDz2agNPzM"}
		_r = requests.post(url, headers=headers, data=json.dumps(data_send))

	elif code == 3:
    
		title = "Course Deletion Script Completion Alert"
		messageBody = "The script cannot find " + current_file_being_processed + ". It appears the process has completed. The script has stopped."
		data_send = {"body":messageBody,"title":title,"type":"note","device_iden":"ujyL4J2SukKsjDz2agNPzM"}
		_r = requests.post(url, headers=headers, data=json.dumps(data_send))

	elif code == 4:
    
		title = "Course Deletion Script Waiting Alert"
		messageBody = "Too many courses are in waiting status so the script will not send new courses."
		data_send = {"body":messageBody,"title":title,"type":"note","device_iden":"ujyL4J2SukKsjDz2agNPzM"}
		_r = requests.post(url, headers=headers, data=json.dumps(data_send))
    
	else:
		logging.info(f"Invalid code: {code}")
		logging.info("The function has failed due to an improper code being passed to the function." + str(code))
		return
```
The functions above send emails and push messages to notify me the status of the script. If anything fails, the notifications will inform me to fix the problem or problems.

```python
def course_deletions_waiting(driver):
    
    logging.info("Script has logged into the Blackboard instance. It will now check to see how many course deletions are in waiting status.")

    # Replace with the actual URL of your webpage
    url = "https://instance.blackboard.com/webapps/blackboard/execute/admin/displaySystemTasks"

    # Set up your WebDriver (adjust the path if needed)
    driver.get(url)

    logging.info("Function has loaded the system tasks page and will now try to search for Course Deletions in waiting status.")

    try:
        # Find and select the first dropdown element
        operation_dropdown = Select(driver.find_element(By.ID, "ftypeFilterInput"))  # Replace with the ID of your dropdown
        operation_dropdown.select_by_visible_text("Course Delete Queued Operation")

        logging.info("Function found Course Delete Queued Operation in the drop down.")

        # Find and select the second dropdown element
        status_dropdown = Select(driver.find_element(By.ID, "fstatusFilterInput"))  # Replace with the ID of your dropdown
        status_dropdown.select_by_visible_text("Waiting")

        logging.info("Function found Waiting status in the drop down.")

        # Find and click the "Go" button
        go_button = driver.find_element(By.CLASS_NAME, "genericButton")  # Replace with the ID of your button 
        go_button.click()

        logging.info("Go button was clicked to search for the number of course deletions waiting.")

        show_all_waiting = driver.find_element(By.ID, "listContainer_showAllButton")
        show_all_waiting.click()


    except NoSuchElementException:
        logging.info("Function did not find any Course Deletion Queued Operations in Waiting status and therefore we will assign the number 0 to the value as none are waiting. This will allow the script to send the next batch of course deletions.")
        return_value = 0
        return return_value

    
    try:
        # Wait for results to load (adjust the timeout if needed)
        WebDriverWait(driver, 10).until(
            EC.presence_of_all_elements_located((By.CLASS_NAME, "smallCell"))  # Replace with appropriate locator
        )

        # Find all the result elements
        result_elements = driver.find_elements(By.CLASS_NAME, "smallCell") 
        logging.info("Function returned results and now the function will count how many are in waiting status.")

        # Print the total number of results
        #logging.info("Total results:", len(result_elements))
        result_val = len(result_elements)
        logging.info("According to the function " + str(result_val) + " courses are still waiting to be deleted. If this is below 16, the system will kick off another process.")
        return_value = int(result_val)
        return return_value
    
    except NoSuchElementException:
        logging.info("Function failed to find any information and therefore we will set the return_value to 0")
        return_value = 0
        return return_value
    
    except TimeoutException:
        logging.critical("Timeout Exception when collecting web scrape data.")
        send_email_notify(2)
        #push_message(2)
        return_value = 999

```

The course_deletions_waiting function is rather lengthy, but it is an important part of the monitoring portion of the script. This function has the script use selenium to login to the Blackboard instance and go to the system tasks page. Then has the page only display the "Course Delete Queued Operation" tasks that are in the "Waiting" status. Then the function will count the number of tasks in the waiting status. Note that the page only displays up to 25 at a time. That's important, because the function will count those waiting tasks. It will then pass the number back to the script. If there are no waiting tasks for the deletion operation, the number will be 0. If it completely fails to load the page, it will return 999 which will kill the script and report an issue.

```python

def check_number(number):
    """Checks if a number is less than 16 and prints a message accordingly."""
    logging.info("Function takes the value from course_deletions_waiting function and does a compare analysis.")
    if number < 16:
        status = "ready"
        logging.info("Function states that there are fewer than 16 course deletions in waiting status and the system is ready to accept another feed.")
        return status
    elif number == 999:
        status = "error"
        logging.info("Function has recieved a status code of " + status + " and will therefore quit the script.")
    else:
        status = "waiting"
        logging.info("Function states that there are 16 or more course deletions in waiting status and no files should be submitted for deletion at this time.")
        return status

```

The check_number function takes the value returned in the course_deletions_waiting function and compares it to a static value. In this case if more than 16 deletion waiting tasks are queued, the system will pass a waiting status. If it's less than 16 it passes a ready status. That value will go to another function which will either process more course deletions, or just end the script. This value can be changed, but don't use a number higher than 25 as that is the maximum number of tasks displayed on a single page.

```python

def copy_and_overwrite(source_file, destination_file):
    try:
        # Read the contents of file_a
        with open(source_file, 'r') as source:
            data = source.read()
            source.close()

        # Overwrite the contents of file_b
        with open(destination_file, 'w') as destination:
            destination.write(data)
            destination.close()

        logging.info("File contents copied successfully!")

    except FileNotFoundError as e:
        logging.info(f"Error: File not found ({e})")  

    logging.info("Function has taken " + source_file + " and moved it to " + destination_file + ".") 

```

This function just copies the content of a file into an existing file.

```python

def create_file_and_write_line(filename, text_to_write):
    """Creates a new file and writes a line of text into it.

    Args:
        filename (str): The name of the file to create.
        text_to_write (str): The text to write into the file.
    """
    logging.info("Function creates a file and inputs data into the first line.")

    try:
        with open(filename, 'w') as file:  # Open the file in write mode ('w')
            file.write(text_to_write)  # Write the line and add a newline
            file.close()
        logging.info("Function has opened " + filename + " with the following information.\n" + text_to_write)
    except OSError as e:
        fun_error = f"Error creating or writing to file: {e}"
        logging.info("Function was unable to create and/or write to a file. Error: " + fun_error)

```

While the process is running, there needs to be some way to keep track of where the course deletion status. Such as how many files have been processed and if there was a failure, where to pick up from. Normally you would use a database such as SQLite, but I wanted it simple. In the folder that contains this script, there are three sub folders. These are:

- course-deletion-files - This folder contains the text files that will be uploaded to your Blackboard instance.
- course-deletion-status - This folder holds three files...
    - current_file_processing.txt - This file contains the last file that was uploaded to the Blackboard instance.
    - next_file_to_process.txt - This file contains the next file that the script should process.
    - response.txt - This contains the response from the Blackboard instance when the file is uploaded to the SIS Framework endpoint
- logs - This folder contains the logs for every run of the script.

The previous function, create_file_and_write_line will take the name of the file the script is processing and write it to the current_file_processing.txt file. This only happens if the file doesn't exist, at the beginning of the process.

```python

def update_to_next_file(file_path):
    """Reads a file, increments the number in a specific line, and rewrites the file.

    Args:
        file_path (str): The path to the file you want to modify.
    """

    logging.info("Function opens " + file_path + " and will update the file to increase the number by one.")

    with open(file_path, 'r') as file:
        logging.info("Function opens file and reads the file.\n File Path is: " + file_path)
        first_line = file.readline()
        #print("First line reads: " + first_line)
        array = first_line.split("_")
        #print(array)
        sec_array = array[2].split(".")
        #print(sec_array)
        old_number = int(sec_array[0])
        #print(old_number)
        old_number_w_z = f"{old_number:04d}"
        #print(old_number_w_z)
        new_number = old_number + 1
        #print(new_number)
        new_number_w_z = f"{new_number:04d}"
        #print(new_number_w_z)
        new_file = "COURSE_DELETION_" + new_number_w_z + ".txt"
        #print(new_file)
        file.close()

    logging.info("Function has generated " + first_line + " to now replace it with " + new_file + ".")

    os.remove(file_path)
    with open(file_path, 'w') as write_file:
        write_file.write(new_file)
        write_file.close()
    
    logging.info("Function update_to_next_file complete.")

    new_file_abs = home_folder + '\\course-deletion-status\\%s' % (new_file)

    #print_file_contents(new_file_abs)

    #return number

```

The update_to_next_file function will be called when the script processes a new course deletion file. The function then increases the number in the text file, next_file_to_process.txt.

```python

def read_filename_from_textfile(textfile_path):
    """Reads a filename from a text file.

    Args:
        textfile_path (str): The path to the text file.

    Returns:
        str: The filename found within the text file, or None if no filename is found.
    """
    logging.info("Function reads the next file from the text file " + textfile_path + ".")

    try:
        with open(textfile_path, 'r') as file:
            for line in file:
                # Basic filename check (you might want a more robust check)
                if '.' in line:  
                    return line.strip()  # Remove any leading/trailing whitespace
                    logging.info("File name found in the text file " + line + ".")
                else:
                    return  # No filename found if we reach here
            file.close()

    except FileNotFoundError:
        logging.info("File in the text file was not found. Script assumes we need to create the file so the process will now create the file and insert the first filename.")
        inital_filename = home_folder + "course-deletion-status\\next_file_to_process.txt"
        text_to_write = "COURSE_DELETION_0001.txt"
        create_file_and_write_line(inital_filename, text_to_write)
        logging.info(f"Error: Text file not found at '{textfile_path}'")
        return text_to_write

```

This function basically just opens the next_file_to_process.txt file and reads the content into the system. If the function doesn't find fhe file, it will create it and set the first value to be COURSE_DELETION_0001.txt.


```python

def send_file_curl():

    process_status_file = home_folder + "course-deletion-status\\next_file_to_process.txt"
    current_file_processing = home_folder + "course-deletion-status\\current_file_processing.txt"

    logging.info("Function to send the file to Blackboard is about to begin.")

    # Settings
    text_file = read_filename_from_textfile(process_status_file)
    logging.info("Called read_filename_from_textfile and it states the file is: " + text_file)
    text_file_path = home_folder + 'course-deletion-files\\%s' % (text_file)
    logging.info(text_file_path)
    
    urllib3.disable_warnings()
    file_status = file_exists(text_file_path)

    logging.info("Checking to see if file exists using file_exists function.")
    logging.info(text_file_path)

    if file_status:
        logging.info("Function passed the check to make sure the file exists within the course deletion folder.")
        # Execute the cURL command using subprocess
        logging.info("Function will now set headers and run a post command to send the file up to Blackboard.")
        try:
            headers = {
                'Content-Type': 'text/plain',
            }

            with open(text_file_path, 'rb') as f:
                data = f.read()

            response = requests.post(
                'https://instance.blackboard.com/[COURSE_DELETION_ENDPOINT]',
                headers=headers,
                data=data,
                verify=False,
                auth=('USERNAME','PASSWORD'),
            )

            with open(home_folder + 'course-deletion-status\\response.txt', 'a') as f:
                output = f.write(response.text)
            logging.info("Function was able to upload the file to Blackboard. Response from Blackboard is: " + str(output))
            logging.info(process_status_file)
            logging.info(current_file_processing)
            copy_and_overwrite(process_status_file, current_file_processing)
            # print(output)
            update_to_next_file(process_status_file)
            return output
        except subprocess.CalledProcessError as e:
            post_error = e.stderr.decode('utf-8')
            logging.info("Posting to Blackboard has failed with an error. \n" + str(post_error))
            return
    else:
        process_complete(process_status_file)
        send_email_notify(3)

```

The send_file_curl function will send the flat feed file to the Blackboard SIS endpoint using a secure username/password. This is done via a curl command and the response from the endpoint is captured in the response.txt file and also copied in the logs. If this process fails, an email is generated.

```python

def main():

    check_status_file = home_folder + "course-deletion-status\\next_file_to_process.txt"
    if stop_process_if_done(check_status_file):
        logging.info("Stopped at Step 1")
        return
    else:
        driver = login()
        number = course_deletions_waiting(driver)
        sys_status = check_number(number)
        if sys_status == "ready":
            send_file_curl()
            logging.info("Script has finished successfully.")
            send_email_notify(1)
            push_message(1)
        elif sys_status == "waiting":
            #send_email_notify(4)
            #push_message(4)
            logging.info("Script has finished successfully.")
            return
        elif sys_status == "error":
            logging.info("Script has failed and is quitting.")
            send_email_notify(2)
            push_message(2)
            return
        else:
            send_email_notify(2)
            push_message(2)
        logout(driver)

main()

```

Finally, this calls all our sub-functions during the main call and processes them. You can see how the process runs logically.

## Steps to run the script

1. Generate the large single text file for course deletion. (See query-to-create-course-deletion-list.sql)
2. Download the large file to the course-deletion-file-split folder and rename is course-deletion-feed-file-complete.txt
3. Using Notepad, find and replcate "," with | or whatever delimiter value you wish to use.
4. Delete all files inside SPLIT-FILES folder
5. Open a command line and cd into D:\Jobs\bb-course-deletion-process\course-deletion-file-split\
6. Run python command .\generate_files.py
7. Once complete, check SPLIT-FILES folder to see that the files were generated.
8. Copy all the files from the SPLIT-FILES folder into the course-deletion-files folder under the instance folder (production or staging) that you are running the process. Feel free to delete any files within the directory prior to copying.
9. In the instance folder, edit auth.py in Notepad
10. Update lines 5 and 6 with a local admin username and password. Save and close the file.
11. In the instance folder, edit course-deletion-process.py in Notepad
12. Update line 111, to make sure that the correct emails are listed there. 
13. If you have changed the integration being used by the course deletion process, update lines 425 with the new endpoint and 429 with the new username and password.
14. In the subdirectory, course-deletion-status, remove the files current_file_processing.txt and next_file_to_process.txt
15. You can now manually run the script by going to a command line, cd into the directory D:\Jobs\bb-course-deletion-process\[instance-name]\ and then run the command .\course-deletion-process.py


## Running the course deletion script in Powershell via Task Scheduler

1. Go to D:\Jobs\bb-course-deletion-process and find the Powershell script, course-deletion-scheduled-process.ps1.
2. Before enabling it, edit the file in Notepad and review line 7 to make sure it is pointing to the correct script instance you wish to use. Edit and save any changes you may need to make.
3. Using Task Scheduler, create a scheduled task to run every 15-30 minutes. Have the task run the following command. powershell.exe -file "D:\Jobs\bb-course-deletion-process\course-deletion-scheduled-process.ps1"
4. Save the changes and schedule the task to run

Review logs in the instance folder, subdirectly logs to see if any issues are happening.