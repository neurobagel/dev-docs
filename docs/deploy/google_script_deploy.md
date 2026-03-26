---
title: Google App script deployment
tags: [Infrastructure]

---

We use a google app script to handle the uploads from some of the sub-communities.
This script is embedded in the [beta version of the annotation tool](https://beta-annotate.neurobagel.org/).
It operates under the neurobagel google account.

??? tip "The Google Script"

    **NOTE**: when copy pasting for use, ensure to update the `UPLOAD_PASSWORD` and `MAIN_FOLDER_ID`.


    ```js
    const MAIN_FOLDER_ID = '';
    function doPost(e) {
    try {
        const data = JSON.parse(e.postData.contents);

        // ACTION: GET SITES (List Folders)
        if (data.action === 'getSites') {
        const mainFolder = DriveApp.getFolderById(MAIN_FOLDER_ID);
        const folders = mainFolder.getFolders();
        const siteNames = [];
        while (folders.hasNext()) {
            siteNames.push(folders.next().getName());
        }
        // Sort alphabetically
        siteNames.sort();

        return ContentService.createTextOutput(
            JSON.stringify({
            status: 'success',
            sites: siteNames,
            })
        ).setMimeType(ContentService.MimeType.JSON);
        }

        // ACTION: UPLOAD FILE
        const UPLOAD_PASSWORD = '***';
        if (data.password !== UPLOAD_PASSWORD) {
        return ContentService.createTextOutput(
            JSON.stringify({
            status: 'auth_failed',
            message: 'Incorrect Password',
            })
        ).setMimeType(ContentService.MimeType.JSON);
        }

        const filename = data.filename;
        const content = data.content;
        const description = data.description || '';
        const folderName = data.folderName; // The site name, e.g., "Site A"

        // 1. Get the Main Folder
        const mainFolder = DriveApp.getFolderById(MAIN_FOLDER_ID);
        let targetFolder = mainFolder;
        // 2. If a specific folder name (Site) is provided, find or create it
        if (folderName) {
        const folders = mainFolder.getFoldersByName(folderName);
        if (folders.hasNext()) {
            targetFolder = folders.next();
        } else {
            targetFolder = mainFolder.createFolder(folderName);
        }
        }

        // 3. Check for existing file
        const existingFiles = targetFolder.getFilesByName(filename);
        const fileExists = existingFiles.hasNext();
        // If checkExists is true and file exists, return CONFLICT (don't write yet)
        if (data.checkExists && fileExists) {
        return ContentService.createTextOutput(
            JSON.stringify({
            status: 'conflict',
            message: 'File already exists',
            })
        ).setMimeType(ContentService.MimeType.JSON);
        }
        // 4. Create or Update file
        let file;
        if (fileExists) {
        // Update existing file
        file = existingFiles.next();
        file.setContent(content);
        file.setDescription(description);
        } else {
        // Create new file
        file = targetFolder.createFile(filename, content, MimeType.PLAIN_TEXT);
        file.setDescription(description);
        }
        // 5. Handle Comments (Optional)
        if (data.commentsContent) {
        const commentsFolderName = 'comments';
        let commentsFolder;
        const cFolders = targetFolder.getFoldersByName(commentsFolderName);
        if (cFolders.hasNext()) {
            commentsFolder = cFolders.next();
        } else {
            commentsFolder = targetFolder.createFolder(commentsFolderName);
        }
        // Comments filename: same as base filename but .txt
        const commentsFilename = filename.replace(/\.json$/i, '') + '.txt';
        const existingComments = commentsFolder.getFilesByName(commentsFilename);

        if (existingComments.hasNext()) {
            // Update the first file found (Preserve ID)
            const commentsFile = existingComments.next();
            commentsFile.setContent(data.commentsContent);
        } else {
            // Create new if none exist
            commentsFolder.createFile(commentsFilename, data.commentsContent, MimeType.PLAIN_TEXT);
        }
        }

        return ContentService.createTextOutput(
        JSON.stringify({
            status: 'success',
            fileId: file.getId(),
            url: file.getUrl(),
            folder: targetFolder.getName(),
            folderId: targetFolder.getId(),
        })
        ).setMimeType(ContentService.MimeType.JSON);
    } catch (error) {
        return ContentService.createTextOutput(
        JSON.stringify({
            status: 'error',
            message: error.toString(),
        })
        ).setMimeType(ContentService.MimeType.JSON);
    }
    }
    function doOptions(e) {
    return ContentService.createTextOutput('');
    }
    ```

## Deployment

Here are the steps you need to take to deploy or update the Google script.

Prep

1. Log into the google account you want to run this script as
   1. you should use the neurobagel account
   2. Seb has the login credentials
2. Go to [https://script.google.com/](https://script.google.com/).
3. You should already have access to [the `nb_annotation_community_dict_uploader` project](https://script.google.com/d/1M_7e4OR9KZHDWNr3qEMo0hg9sLcQ5yyiLSAVDwT_pkNE8_4x_C1hQog0/edit?usp=sharing) - which is the E-PD backend.

Make a new project

1. Click on "new project"
2. Copy the App script into the editable area
3. Change the values for the GDrive folder and change the password secrets
4. Share the project with the rest of the NB maintainers (but leave it on "restricted")

(Re-)Deploying the script

1. Click `Deploy > Manage deployments`
2. Click the `Edit` button (pen looking icon) in the modal that opens
3. Make a new version (or first deploy), give it a name
4. Under "execute as", use your user (e.g. the Neurobagel account)
5. Under "who has access", set "Anyone".
   1. If you set "Only myself" then nobody but you can use the backend
   2. If you set "Anyone with a Google Account", then we might as well just go full OAuth because users will have to log in first
6. Click deploy, make a note of the "Web App URL"
   1. You will pass this as an environment variable to the frontend
   2. The .env variable is called `NB_GOOGLE_APPS_SCRIPT_URL`
