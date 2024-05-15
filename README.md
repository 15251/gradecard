# Setup
1. Create a repo from the [15251 fork of the Gradecard template](https://github.com/15251/gradecard). Name this repo `gradecard-<university>-<course>-<your semester>` (e.g. gradecard-cmu-15251-s24).
2. Clone the repo to the machines of everyone who will be managing student cards.
3. You will need Python 3.7+. Install it if you don’t have it.
4. Create a virtual environment in the repo folder with `python3 -m venv env`
5. To enter the virtual environment, `source env/bin/activate`
6. Install dependencies: `pip install -r requirements.txt`
	- Note: If you are using Python 3.10+, change PyInquirer package to `InquirerPy` in `requirements.txt` before pip installing.
### Google Cloud Platform setup
1. Follow [these steps](https://developers.google.com/workspace/guides/create-project#create_a_new_google_cloud_platform_gcp_project) to create a new Google Cloud Platform project. Feel free to use a personal Google account as CMU may restrict your ability to create a project using your CMU account. 'Gradecard University Course Semester' and `gradecard-<university>-<course>-<semester>` are recommended defaults for the project name and project ID.
2. Follow [these steps](https://developers.google.com/workspace/guides/enable-apis) to enable Google Sheets API and Google Drive API for your project.
4. Follow [these steps](https://developers.google.com/workspace/guides/configure-oauth-consent) to configure the OAuth consent screen. Do not worry if you cannot make the app internal. Do not worry about adding or removing scopes. Add whoever wants to use Gradecard as test users. Make sure to use university emails (CMU emails) here.
5. Follow [these steps](https://developers.google.com/workspace/guides/create-credentials#create_a_oauth_client_id_credential) to create an OAuth client ID credential. Choose 'Desktop app' as your application type. 'Gradecard Client' is the recommended default for the client name.
6. Download your client ID and secret as JSON and store them as `credentials.json` in the project directory. This JSON is gitignored and should not be pushed to the Github repo. Distribute the JSON to others in another manner (slack, email, etc.).
7. Run python auth.py and complete the authentication flow ignoring Google's safety warnings. These show up because the app has not been verified by Google.
	- This creates a `token.json` file in your root directory. The token expires pretty frequently. If it does, just remove the file from the directory and run python auth.py again.
### Google Drive setup
1. Create a new folder called `<semester> Gradecard` (e.g. S24 Gradecard). Navigate inside it.
2. Create a new sheet called `Gradecard`. This is where the course staff will manage grades. Feel free to use this example as a starting point ([CMU 15-251 Gradecard]()).
3. Create a new sheet called `Student Card Template`. This is the base sheet that is used to create student cards for each student. Feel free to use this example below as a starting point ([CMU 15-251 Student Card Template]())
4. Create a new folder called `Student Cards`. This is where Student Cards for each student will be populated.

Here is an overview of the sheets:
1. The `Gradecard` spreadsheet is the gradebook.
	1. This contains a record of all scores, attendance, and anything else you would like to manage.
	2. The `export` tab in this spreadsheet contains all the data that is actually pushed to student cards, and so should be spot checked before every sync. Also, make sure that you only include columns that you intend students to see on their student cards.
	3. The `roster` tab is where the student roster will be populated.
	4. All other tabs can be customized to fit your course structure. 
	5. An example of this spreadsheet can be found here. Again, `export` and `roster` are the only required tabs.
2. The `Student Card Template` spreadsheet is the base of every student’s card.
	1. Data synced appears in the `Data` tab in columns A and B, where column A contains keys and column B contains values. This data will come directly from the `export` tab.
	2. You can customize `Dashboard` and `Scores` tabs such that it is helpful and clear to students.	
Now, update `secrets.py` with the `GRADECARD_SPREADSHEET_ID`, `BASE_STUDENT_SPREADSHEET_ID`, and `STUDENT_CARDS_FOLDER_ID`. You may push this update to Github. Gradecard is now set up.
# Creating Student Cards
### Adding a roster
You will need the roster in csv format.
1. Place roster in `roster` folder of the repo
2. Run `python main.py`
3. In CLI, select `Add Students`
4. Select roster to add
**You should now see the roster in the `roster` tab on `Gradecard`**. When new students join the course, simply add the updated roster and run the above steps again. Note, this will add new students, but won't delete students who dropped the course.
### Creating Cards
You are now ready to create student cards.
1. Run `python main.py`
2. In the CLI, choose `Create Cards`
3. Gradecard will now create and share cards with students, place them in the Student Cards folder, and populate the `export` sheet
4. Verify that you are satisfied that cards have been created. These should all be blank.
5. Run `python main.py` again
6. In the CLI, choose `Update Views`
7. Choose to update both sheets (`Dashboard` and `Scores`)
8. Press enter when asked which students’ views to update: this indicates all students.
The last 4 steps are the steps to do whenever you update the base card. This syncs all students’ views to match that of the base card.
### Emailing Students
In the menu, under Extensions, you should be able to access Apps Script. If not, get it as an add-on. The code used to send emails to students is included below in case the script is not copied.

```
function sendEmails() {
 var exportSheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("export");
 var sSheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("s");
 var firstRow = 3
 var lastRow = 3 // TODO

 for (var r = firstRow; r <= lastRow; r++) {
   var address = exportSheet.getRange(r, 2).getValue();
   var name = sSheet.getRange(r-1, 3).getValue();
   var subject = "[15-251] Your Student Card" // TODO change this to your course number
   var ssid = exportSheet.getRange(r, 3).getValue();
   var message = "Hi "+name+",\n\nYou can find your 15-251 Student Card here: https://docs.google.com/spreadsheets/d/"+ssid+".\n\nThis will contain an overview of your performance in 15-251, including recitation attendance, homework scores, and exam scores! This card will not be updated in real time; expect to see updates about once a week.\n\nBest,\n15-251 Course Staff"; // TODO change this

   // MailApp.sendEmail(address, subject, message);
   console.log(address);
   console.log(message);
 }
}
```
Change lastRow as needed to include all students. Change message as you see fit. Run with console logs to make sure emails look good. When ready, uncomment the sendEmail line and run. Note, firstRow and lastRow are row indices in the export sheet.
# Syncing Student Cards
Now, you should have Student Cards set up and shared with students. Here are the steps to sync data from `Gradecard` to student cards:
1. First, make sure you have spot-checked the data in the `export` tab in `Gradecard`. This data will be the only thing that is pushed to student cards. 
	1. Note that only columns with non-empty body will be pushed, so columns with only a header will not be pushed.
	2. Do NOT edit columns A through E
2. Run `python main.py`
3. In CLI, select `Sync Data`
4. Here are some options to select which student cards to update
	1. blank: all student cards will be synced
	2. andrew ID (e.g. `gyeongwk`): only this student's card will be synced
	3. andrew ID followed by `...` (e.g. `gyeongwk...`): this student and all following students' cards will be synced (according to the order in `export` tab)

### Contact Info
Please contact 15-251 course staff or Professor Ada (aada@cs.cmu.edu) with any questions on how to set up Gradecard for your course.
