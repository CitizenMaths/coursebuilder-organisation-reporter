# coursebuilder-organisation-reporter
A cron job that provides organisations/partners with learner progress reports which are in the form of google sheets

##Important:

Before using this code, you will need to enable the google sheets api for your project in the google cloud console, as well as creating a default app engine service account so we can talk to the google sheets you specify.

Following Step 1, located here https://developers.google.com/sheets/api/quickstart/python, will help you get started. Please note this does not show how to create a service account though.

Also, the google sheets that would need to the data pushed to it would require the app engine service account id to be given edit permissions in each sheet itself.

We also start adding data at row number 3 in the sheet. This can be changed in the code, but it does leave a couple of rows at the top of the sheet for headers and information about what the data in the column represents.

Another prerequisite is for you to manually create an Entity in the datastore called OrganisationReporter (the kind) for your courses namespace, with the following fields:
-For the key identifier, select custom name, and then enter the name for this record
-end_row_number - which is an Integer, set it to 0, and make sure it is not indexed
-organisation - which is a String, set it to the organisations key or id, and tick indexed
-sheet_name - which is a String, set this to the worksheet name in the google sheet which the data needs to be entered into, untick indexed
-spreadsheet_id - which is a String, set this to google sheet id (for example in the URL for this google sheet https://docs.google.com/spreadsheets/d/1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgvE2upms/edit, you would need to enter 1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgvE2upms part in this field) where the data needs to pushed to, tick indexed

##Modifications to cron.yaml 

**Lines #22-24** - Added the URL and schedule to run the organisation reporter cron job

##Modifications to the code at models/models.py

###Additional classes added:

**Lines #1292-1325** - `class OrganisationReporter(BaseEntity)`

The OrganisationReporter class is the model that is used to retrieve information about the Organisation from the datastore. The fields that are stored include the id of the organisation to look for when searching students, 
as well as the target google sheet and worksheet name.

##Modifications to the code at models/progress.py

###Additional methods added:

**Line #1002** - `get_unit_percent_complete_by_title(self, student)` added to class `UnitLessonCompletionTracker`

This method returns a dictionary with each unit's completion with the key being the unit title, and the value being a percentage represented by a float with the value between 0.0 (which is 0%) and 1.0 (which is 100%).
For example, the value represented by 0.45 is 45% in reality.

##Modifications to the code at modules/courses/courses.py

###Additional Code to pre-existing methods

**Line #97** in method `register_module`

`[(lessons.OrganisationReporterCronHandler.URL, lessons.OrganisationReporterCronHandler)],`

The URL and class with the cron action in it is added to the routes in coursebuilder, so when the URL is called, it runs the cron code.

##Modifications to the code at modules/courses/lessons.py

###Additional classes added:

**Lines #595-724** - `class OrganisationReporterCronHandler(utils.AbstractAllCoursesCronHandler)`

This class contains the cron action code that is run when the URL is called from the cron scheduler. This class contains comments.
