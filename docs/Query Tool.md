---
title: Query Tool
tags: [Frontend]

---

Code: https://github.com/neurobagel/query-tool

Running version: https://query.neurobagel.org/

## Example data
The query tool has two files that a user can download:

1. A .tsv with the datasets the user has found in their query and selected
2. A .tsv with all of the subjects in the selected datasets that also match the query

Example of the dataset.tsv:

| DatasetID                       | DatasetName      | PortalURI                             | NumMatchingSubjects | AvailableImageModalities
| :---------------                | :------------- | :------------------------------------ | :------------------ | ---------------
| `http://neurobagel.org/vocab/001` | First Dataset  | https://openneuro.org/datasets/ds0001 | 142                 | [T1, T2, DWI]
| `http://neurobagel.org/vocab/002` | Great Dataset  | https://openneuro.org/datasets/ds0002 | 2                   | [T1, Flow]

- `DatasetID`: required - dataset uuid
- `DatasetName`: required - human readable name of the dataset
- `PortalURI`: optional - link to website about dataset
- `NumMatchingSubjects`: required - number of subjects matching query inside this dataset
- `AvailableImageModalites`: aggregate variable - list of unique available imaging modalities for the dataset

Example of a subject .tsv:


| DatasetID                       | SubjectID | Age | Sex    | Diagnosis | Assessment | SessionID | SessionPath | NumSessions | Modality      |
| :------------------------------ | :-------- | :-- | :----- | :-------- | :-------- | :-------- | :-------- | :------------------------------- | :------------ |
| `http://neurobagel.org/vocab/001` | sub-01    | 60  | Male   | PD        | MMSE      | ses-01    | /data/BIDS/ds0001/sub-01/ses-01/ | 2 | [T1, T2, DWI] |
| `http://neurobagel.org/vocab/001` | sub-01    | 60  | Male   | PD        | MMSE      | ses-02    | /data/BIDS/ds0001/sub-01/ses-02/ | 2 | [T1, EEG]     |
| `http://neurobagel.org/vocab/001` | sub-02    | 40  | Female | CTL       | MMSE      | ses-01    | /data/BIDS/ds0001/sub-02/ses-01/ | 2 | [T1]          |
| `http://neurobagel.org/vocab/002` | sub-02    | 40  | Female | CTL       | UPRSIII   | ses-02    | /data/BIDS/ds0002/sub-02/ses-02/ | 2 | [EEG, T2]     |
| `http://neurobagel.org/vocab/002` | sub-01    | 20  | Male   | CTL       | UPRSIII   | ses-01    | /data/BIDS/ds0002/sub-01/ses-01/ | 2 | [T1]          |

- `DatasetID`: required - dataset uuid, not persistent across different graphs. It is identical across output files and can be used as the key to join the two output files.
- `SubjectID`: required - human readable name / label for the subject
- `Age`: optional - if available: the age as float (according to Neurobagel). Otherwise `""` empty string
- `Sex`: optional - if available: the sex as string (according to Neurobagel). Otherwise `""` empty string
- `Diagnosis`: optional  - if available: array of string values that represent diagnoses for a session (according to Neurobagel).
- `Assessment` : optional - if available: array of string values that represent assessments for a session.
- `SessionID`: required (?) - the human readable name of the session
- `SessionPath`: required - the file system path or datalad id of the session directory and the files in it
- `NumSessions`: aggregate variable - int number of total available sessions for a subject
- `Modality`: required (? unclear, what if no imaging?) - array of string values that represent imaging modalities available in this session


## Troubleshooting the query tool
When the query tool internally encounters a problem, it will display a generic error of "Oops, something went wrong" in the UI.
To try to pinpoint the problem from the browser side, open the **Developer tools** in your browser and check for errors in the JavaScript **Console** tab as well as in the **Network** tab. 
Since the **Network** panel only logs network activity while it is open, in the query tool page next to the panel, resubmit the query that previously resulted in the error. Then, click on the request that appears in the Network log to inspect the headers.

### A note on CORS errors
When there is a problem with how the API/graph or query tool was deployed, you may encounter errors when using the query tool that commonly take the form of a CORS error. 
In the **Console**, this might look like error messages that contain `...blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.`

This can, but _does not necessarily_ mean that the issue is with the allowed origins on the API side. CORS errors may appear more generically when there is a problem on the client side (or server side) that results in a failure in attempting a cross-origin request to the API.
Similarly, these errors can arise when the response from the server simply does not having the expected header due to some reason (not limited to the actual origin being disallowed).
