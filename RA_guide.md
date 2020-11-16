## Step by step process to BIDs-ifying data and validating that it worked
 
 
**Some notes before you start. These are the variables you'll need to change in order to run each script**



| Variable   | Definition     |  Example  |
|----|:-----:|-------------:|
| ${sub} | This is the subject ID. This will change depending on what subject you're working on, this will always be in the form INET###. Please note that sometimes you'll have to put quotation marks around the subject. | INET001| INET001|
| ${ses} | This is the session. This will change depending on what session you're working with, this will always be in the form of a number 1, 2, 3. Please note that sometimes you'll have to put quotation marks around the session. | 1 |


  **Follow along using the QC sheet**
  
  ### Downloaded
  * Download the files off of Nunda, DO NOT UNZIP
  
  ### iNet2BIDS_all
  * Open the terminal type in the following

  
  
  
   ``` cd /Volumes/GRATTONLAB/iNetworks/BIDS/Nifti/.bidsignore ```
   
   **NOTE if you have not done so before you will need to give the script permissions**
     
```chmod 777 iNet2BIDS_allRDSS```
   
   
  * Run the following **change based on your subject and session**
  
  
  ``` ./iNet2BIDS_allRDSS ${sub} ${ses} ```
  
  
  *If you get an error that the file path was not found type*
    ``` ./iNet2BIDS_allRDSS.sh ${sub} ${ses} ```
  
  ex
  
  
   ``` ./iNet2BIDS_allRDSS INET001 1 ```
   
   
 * Fill out the QC sheet with the number 1 to indicate the script is running
 ## After the script is complete go through this verification process 
 ### fix_jsons
* Verify the script worked, go to the json website and select the phase diff json file in the fmap folder, look for the section IntendedFor. This should have a list of paths of all the functional scans 
* https://jsoneditoronline.org/
* ex "IntendedFor": [
        "ses-3/func/sub-INET001_ses-3_task-rest_run-05_bold.nii.gz", 
        "ses-3/func/sub-INET001_ses-3_task-mixed_run-02_bold.nii.gz", 
        "ses-3/func/sub-INET001_ses-3_task-ambiguity_run-02_bold.nii.gz", 
        "ses-3/func/sub-INET001_ses-3_task-mixed_run-01_bold.nii.gz", 
        "ses-3/func/sub-INET001_ses-3_task-ambiguity_run-01_bold.nii.gz", 
        "ses-3/func/sub-INET001_ses-3_task-rest_run-03_bold.nii.gz", 
        "ses-3/func/sub-INET001_ses-3_task-rest_run-06_bold.nii.gz", 
        "ses-3/func/sub-INET001_ses-3_task-rest_run-08_bold.nii.gz", 
        "ses-3/func/sub-INET001_ses-3_task-rest_run-01_bold.nii.gz", 
        "ses-3/func/sub-INET001_ses-3_task-slowreveal_run-02_bold.nii.gz", 
        "ses-3/func/sub-INET001_ses-3_task-rest_run-04_bold.nii.gz", 
        "ses-3/func/sub-INET001_ses-3_task-slowreveal_run-01_bold.nii.gz", 
        "ses-3/func/sub-INET001_ses-3_task-rest_run-07_bold.nii.gz", 
        "ses-3/func/sub-INET001_ses-3_task-rest_run-02_bold.nii.gz"
    ]
### pydeface
* Verify the anatomicals were defaced
* Open the terminal and go to the anat folder


``` cd /Volumes/GRATTONLAB/iNetworks/BIDS/Nifti/sub-${sub}/${ses}/anat ```


ex


  ``` cd /Volumes/GRATTONLAB/iNetworks/BIDS/Nifti/sub-INET001/ses-3/anat ```
  
  
  ``` afni ```
  
  
* This will open the afni program and show you the anatomical scan if one exists, click on overlay and select the T1 file. The image should NOT have a face on it. 
  
### bids_valid
* Verify the script worked, open the subjects session in the Nifti folder it should look bids appropriate 
* Then go to this website and select the entire Nifti folder, select to ignore warnings and Nifti headers. This will take awhile to load and will output whether the files are BIDs approved.  
* http://bids-standard.github.io/bids-validator/
  
  
  
  
  
### What to do if fmaps phasediff file wasn't created 
* Login to Nunda and determine what scan number the phase diff file is for the sub/ses you are working on. To find look for scan titled gre_field_mapping with Image Type ORIGINAL/PRIMARY/P/ND 
* Open a terminal and go to the Nifti directory 

    ``` cd /Volumes/GRATTONLAB/iNetworks/BIDS/Nifti ```
    
 * Type dcm2bids but ONLY the scan number you found on Nunda
 
 Ex for INET002 session 4, fmap from scan 6 based on nunda
 
   ``` dcm2bids -d /Volumes/GRATTONLAB/iNetworks/BIDS/DICOM/sub-INET002/INET002_4/SCANS/6/DICOM -p INET002 -s 4 -c /Volumes/GRATTONLAB/iNetworks/BIDS/Nifti/.bidsignore/config.json --clobber  --forceDcm2niix```
   
  The converter will only go into that particular scan which will take significantly less time then rerunning the entire shell script (<5min) once complete go into the folder to see if phasediff files exist
  
  * If do exist
  
  You will have to run an additional script to apply some of what would have otherwise been applied during the inet2bids shell script
  
  Ex
  
``` cd /.bidsignore ```
    
    
``` python -c "exec(open('fix_jsonsRDSS.py').read()); fix_jsons('INET002', '4')" ```
    
   * If phasediff file does not exist
   
 Run this (with the appropriate sub/ses
 
    ``` dcm2bids -d /Volumes/GRATTONLAB/iNetworks/BIDS/DICOM/sub-INET002/INET002_4/SCANS/6/DICOM -p INET002 -s 4 -c /Volumes/GRATTONLAB/iNetworks/BIDS/Nifti/.bidsignore/config3.json --clobber  --forceDcm2niix```
    
 Check to see if phasediff exists and run python code above
 
 
   
   
   
   Contact Alexis for further debugging
