---
title: Panopto LTI Migration Documentation - Getting Started
date: 2024-05-09 09:00:00 -0600
categories: [LTI Migration, Panopto]
tags: [LTI, panopto, getting-started]
---

# Panopto LTI Migration

*Note: These paragraphs contain my documentation notes, comments and information. They may not be fully formed or may use old data. I work to keep this information as up to date as possible. If you have questions about something use the links to contact me. This is a work in progress.*

## Getting Started

This document covers the May 2024 migration of my institution from the Panopto Building block integration to the LTI integration. 

In April 2024 Blackboard notified Panopto institutions that the Panopto building block integration would be ending life on June 30, 2024. This caused us to start the migration from the building block to LTI.

## Author's Recommendations

So if you are looking through this documentation, you either a. want to know how this was done or b. you are me and I'm trying to review what I did. (Hi future me, you did your best so just breathe.) If you are the former here are some recommendations to review.

#1. Just tell Blackboard to extend support for the building block. This might seem like the most idiotic option, but this should be done. Anthology/Blackboard has provided little guidance for the Panopto migration tool. Panopto doesn't have a consistent set of recommendations across their support teams based on conversations with other institutions who are dealing with other Panopto support personnel. This is a dumpster fire floating down a flooded street.

**TLDR:** Tell Blackboard and Panopto to extend support. If Panopto or Anthology isn't getting screwed in this situation, guess what. You are going to be screwed. 

![GIF Image of a fire in a dumpster floating down a flooded area](assets/img/posts/panopto-lti/dumpter-fire-flood.gif)

#2. Ok. You aren't going to do number 1? *sigh* Ok then, well first understand the impact of this migration. Issues you should look at include:

- Panopto Video Quiz Usage (which will break)
- Panopto Mashups (which will break)
- Panopto Links (which will break too) *Are you noticing a pattern here?*

If you previously used course copy v1 and any course copies you will know that the videos reside in the Panopto video folder of the source course and not in the destination course. This will make fixing any of the videos that are broken very difficult. As of this posting, we are still trying to figure out how we will be able to fix this issue.

**TLDR:** Shit is going to break, understand how widespread this will be and how hard it will be to fix.

#3. Educate yourself on the differences between v1 and v2 of course copy in Panopto

- Version 1 creates permission links to the video file where it resides in the source course.
- Version 2 creates a reference video that resides in the destination course.

While version 2 would appear to be the preferred option, Panopto states that version 2 course copies can take up to 24 hours. Any access before this (by the way no notification is provided to the user to tell them that the Panopto course copy has completed) will be accessing the source video (its not clear if the permissions allow access, more testing required) so analytics will show access to the source video and not the video within the course.

**TLDR:** Users who enable Course Copies V2 should tell users they should not allow student access until 24 hours after course copy.

#4. Panopto videos for course ingestion should always reside within the course. If users copied the content from their personal folder (user folder) the permissions will not be applied in the same way with the LTI integration. No reference copies appear to be working. (More testing required, but just know this is an issue.)

**TLDR:** Panopto videos that will be available to students must be in the course Panopto folder or permissions might not be fixed. 



## Documentation from Panopto

Here's the documentation from Panopto. This information is generic so let's review the specifics.

Link: LTI 1.3 Documentation from Panopto - [https://support.panopto.com/s/article/How-to-Set-Up-a-Blackboard-Ultra-Integration-with-LTI-1-3](https://support.panopto.com/s/article/How-to-Set-Up-a-Blackboard-Ultra-Integration-with-LTI-1-3)


## Creation of the Panopto Rest API User in Blackboard

### REST API Integration User
	
    Example username: panopto_restapi
    First Name: Panopto
    Last Name: REST API User

### System Role for Panopto REST API User:
	
    Role Name: Panopto REST API Role
    Role ID: PANOPTO-REST-API-ROLE
    Description: Role for Panopto REST API User as part of LTI 1.3 Deployment

### Allowed Permissions:
    
    Administrator Panel (Users) > Users > Edit > View Organization Enrollments
    Administrator Panel (Courses) > Terms
    Administrator Panel (Courses) > Courses
    Administrator Panel (Users) > Users
    Administrator Panel (Users) > Users > Edit > View Course Enrollments
    Course/Organization (Content Areas) > Adaptive Release > View
    Course/Organization Control Panel (Customization) > Properties

### During REST API Integration Creation the following must be set:
    
    End-User Access: Select Yes.
    Authorized To Act As User:  Select Yes.



## Overview of Panopto LTI 1.3 Documentation 

These are the same configuration settings that our institution selected. These might not match what your Panopto person recommended so feel free to push back on the tech to provide more information and guidance.

**Instance Name (Required):** Blackboard-Instance-Name-LTI (as example) *This can be whatever you wish.*

**Application Key:** (Auto Generated)

**Friendly Description (Required):** *custom description*

**Server Name (Required):** your-instance.blackboard.com

**Federated Sign-out URL (Optional):**

**Role Mappings (name1:creator;name2:viewer):**

These are for custom role mappings for those who should have access or be able to see Panopto content. If you don't have custom course roles, ignore. If not, review and make changes as you see fit.

**Create Folders On Sign-in (Optional):** *Recommended by Panopto Tech because it would create user folders when they login*

**Parent Folder Name (Required):** Folder-Name-for-Provider *We set this flag to have this be the same as the parent folder used for the Building Block*

**Suppress Access Permission Sync on LTI Link:** *Recommended Unchecked*

**Prepend Course Code:** *Recommended Checked* - *Adds a Course ID to the prefix of the Panopto course folder name, for example "Introduction to Meteorology" becomes "Spring-2024-METR-1003-001 - Introduction to Meteorology"*

**Sign Out of Provider on Panopto Sign-Out (Optional):** *Recommended Unchecked* *This would sign the user out of Blackboard if they sign out of Panopto*

**Bounce Page Blocks iframes:** *Recommended Unchecked*

**Default Sign-in Option (Optional):**

**Personal Folders for Users (Optional):** 

**Unify user accounts from this provider:** *Recommended Checked* *Our instance does use unified accounts so this is checked*

**Sync permissions for this provider on Sign-In from other providers:** *Unchecked*

**Show this in Sign-in Dropdown:** *Unchecked* *Personal preference of the institution*

**Course copy version:** *Checked* *This is an important issue to deal with as the course copy v2 versus the course copy v1 do change their processes. Review the recommendations about this configuration*

There are more configurations, but if you work with Panopto support they will configure the rest of this instead of requiring you to make these configurations. 

## Panopto Placement Creation

Panopto LTI Placements will automatically be created by the LTI 1.3 Integration. Here are the default names that were used by our Panopto support tech.

![A screenshot of the LTI placements created for the Panopto integration within Blackboard](assets/img/posts/panopto-lti/panopto-placements.png)

Some notes / recommendations on the placement names. 

The placements that come from the LTI 1.3 integration are comparable to the ones in the building block. However there are some variations...

1. The default embed tool is called, "Panopto Video Embed Tool". Generally the term "embed" means displaying the video within the web page. This placement doesn't do that. It only creates a link. When used in the text editor, the video will be embedded. This is a change in workflow and important to document. This is why we changed the name of the tool to "Panopto Video Link / Embed Tool"

2. The Panopto Video Submission Tool will not break according to our testing. However, instructors should now just create an assignment. This might be confusing as the Panopto Student Submission Tool will appear in the Create Content area in Original courses and in the Content Market within Ultra courses. Also, the tool no longer populates the instructions for students on how to submit a video assignment and the old instructions that were injected into the Assignment text box editor are now incorrect and should be changed.



## Panopto LTI Migration Testing Plan

We generated 30+ courses to do our testing (see the course creation and enrollment files below as templates). These were used to create content using the building block and then see how we would have to manipulate the content after the migration. After the migration, we highlighted the following items for testing and validation:

Test Previous links created with the following:

Original
- [ ] Panopto Video Embed
- [ ] Panopto Video Link
- [ ] Panopto Video Quiz
- [ ] Panopto Student Submission Assignment
- [ ] Panopto Embed in Text Editor (Sample Item)
- [ ] Panopto Course Tool Application Link

Ultra
- [ ] Panopto Video Embed
- [ ] Panopto Video Link
- [ ] Panopto Video Quiz
- [ ] Panopto Student Submission Assignment
- [ ] Panopto Embed in Text Editor (Sample Item)
- [ ] Panopto Course Tool Application Link

Our goal was to not only confirm the LTI placements were working and figure out what content that used the building block wasn't, but to also document the following:

- Any changes in workflows / additional steps
- Any changes in how items display


**Blackboard Course Creation and Enrollment Template Files for Download**

[Panopto Course Creation Template - Text File](https://github.com/terrypatterson/terrypatterson.github.io/blob/b6322625d61e1e3bd02bbcd03b3127315fc9fa54/docs/panopto-lti/panopto-course-creation-lti.txt)

[Panopto Course Enrollment Template - Text File](https://github.com/terrypatterson/terrypatterson.github.io/blob/b6322625d61e1e3bd02bbcd03b3127315fc9fa54/docs/panopto-lti/panopto-course-enrollment-lti-template.txt)


### Course Testing Processes

The 30+ courses were created to do various levels of testing regarding how instructors who did course copies before and after the migration might be impacted by the change. Each course had it's own task.

>PANOPTO-LTI-TESTING-001: The items below were created using the Panopto Connector building block
>- Panopto Course Tool Application Link (In Course Menu)
>- Under Course Documents
>    - Panopto Video Embed
>    - Panopto Video Link
>    - Panopto Video Quiz
>    - Panopto Student Submission Assignment
>    - Panopto Embed in Text Editor (Sample Item)
>    - Panopto Embed Video code without LTI
>    - Panopto Video Link without LTI
>    - Panopto Embed Video in Text Editor within an Announcement
>
>PANOPTO-LTI-TESTING-002: The items from PANOPTO-LTI-TESTING-001 were copied to 002 before the migration to see how course copied content prior to the migration would be effected.
>
>PANOPTO-LTI-TESTING-003: The same content in PANOPTO-LTI-TESTING-001 was recreated in this section so we had a backup for testing if something went wrong with the first section.

The next section, we went into our environment and use a database query to find courses that heavily utilized the various tools (Video Embed, Video Link, and Video Quiz) to use a more "real-life" situation for testing.

>PANOPTO-LTI-TESTING-004: We copied (using Blackboard Course Copy) a course that heavily utilized the Panopto Video Embed tool from the building block to see how it would be impacted. This was done pre LTI migration.
>
>PANOPTO-LTI-TESTING-005: We copied (using Blackboard Course Copy) a course that heavily utilized the Panopto Video Link tool from the building block to see how it would be impacted. This was done pre LTI migration.
>
>PANOPTO-LTI-TESTING-006: We copied (using Blackboard Course Copy) a course that heavily utilized the Panopto Video Quiz tool from the building block to see how it would be impacted. This was done pre LTI migration.
>

The next section of courses we used the same three source course sections, but we exported out of those courses and imported the contents into the following courses. These were done pre LTI migration.

>PANOPTO-LTI-TESTING-007
>
>PANOPTO-LTI-TESTING-008
>
>PANOPTO-LTI-TESTING-009

The next six course sections we repeated the same course copy and course import processes from the same source courses, however these were done after (or post) the LTI migration.

>PANOPTO-LTI-TESTING-010
>
>PANOPTO-LTI-TESTING-011
>
>PANOPTO-LTI-TESTING-012
>
>PANOPTO-LTI-TESTING-013
>
>PANOPTO-LTI-TESTING-014
>
>PANOPTO-LTI-TESTING-015



## Testing Notes

Please note that the notes below are raw and may not have fully fleshed out information at this time. They are a work in progress.

- [ ] Test logins via the hosted site and via Blackboard to make sure all is well.
- [ ] Test provisioning a new course folder to ensure you want the option Prepend Course Code set to True.
- [ ] Test the tools in the test courses.
- [ ] Test all the new tools.
- [ ] Test disabling tools from Tools section to see if the white glove treatment will still work.
- [ ] Test making quiz tool unavailable (if necessary) from placements to see if it the white glove treatment will still work.

**Information:** Creation of the Panopto Folder now happens when you click on the Panopto Course Tool Link Folder. The three page workflow with the building block has changed to provisioning the folder when you click the LTI Link.

**Issue:** When in the Panopto Course Folder - Clicking the close button in the upper right-hand corner throws an error.

**Problem:** The Panopto Video Embed Tool does not embed the video. It only creates a link within the course based on testing.

What is the proper way to create a Video Embed within the Blackboard Learn application? If embedded within a course item and copied, will the permissions sync over for the embed within the course item.

**Information:** New workflow on quiz creation process

**Issue:** Panopto Quiz defaults to 100, would like to have the ability to set a points possible in the workflow

**Information:** Panopto Student Submission - Change from Video link to Video Embed within the Create Submission. 

**Issue:** Student submission videos don’t appear to reside in the student’s Panopto My Folder, this doesn’t appear to be the case. (Follow-up: Review student’s My Folder requirements / needs with Mario)

**Issue:** Capture window did not auto-close after upload. (See screenshot below) Document, that is expected behavior.

**Issue:** Panopto Student Submission Tool appears within the Build Content area which it shouldn’t as it should only be available under the plus icon in the Text Editor. How can we properly hide that?

**Issue:** Course Copy v2 - Delays in the course copy process? Seems to have taken over 20 minutes for a course copy to appear in the backend of Panopto, should those delays be expected and shouldn’t those 

Video was uploaded into 18 and copied via the item copy feature within Blackboard to section 21. Video does not exist in the 21 Panopto folder nor did the permissions get updated to allow 18 folder content to be accessed by Section 21 viewers. 

Made the Blackboard Building Block Unavailable

Links in Panopto LTI Testing 001 all worked successfully

Copied Panopto LTI Testing 001 to section 025
- Quiz Link breaks
- Embedded Panopto Videos and Panopto Video Links permissions not there
- Content was not copied


Course Copies made before Panopto LTI Migration
- Quiz Links break
- Embedded Panopto Videos and Panopto Video Links permissions are not there for users
- Referenced content copies don’t reside in the destination course. Support teams will have to manually copy content from Panopto folder in source course over to destination course.
- Student Submissions Still Work

Course Copies made after Panopto LTI Migration
- Quiz Links break
- Embedded Panopto Videos and Panopto Video Links permissions are not there for users
- Panopto copied referenced Panopto content from source course into the destination course’s Panopto folder.
- Student Submissions Still Work
