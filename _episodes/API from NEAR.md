---
title: "Downloading data with with NEAR using API"
teaching: 5 min
exercises: 0
questions: "How to download data from website using API?"
objectives:

keypoints:
- "Postman, Python, API"
---




## Introduction
- Sometime, we need to use API to download the data from website when there are so many files and options to download.
- You can download data from Twitter, YouTube, Google etc using API. However you will need to register with the provider and get the API key or Bearer Token.
- This is an actual project where we will be downloading data from near website vista.um.co, with given username/password and API token.

## POSTMAN

- Postman is an API platform for building and using API
- In order to use Postman, you need to register an account and login with that:

![image](https://user-images.githubusercontent.com/43855029/168138485-100c60bb-ffe6-4270-91e0-88bcf5c6130d.png)

- If you are new to Postman, you will need to create Workspace to save your API query:

![image](https://user-images.githubusercontent.com/43855029/168138620-ea97ac73-a6ee-4130-81e0-bd6cd50b84c9.png)

- Once you have My Workspace, click on the + to open up the new workspace:

![image](https://user-images.githubusercontent.com/43855029/168138778-adf0a92d-e308-46a8-bd89-1f18780099f7.png)

- The following workspace opened:

![image](https://user-images.githubusercontent.com/43855029/168138850-0c2e8a6e-e50e-4fe6-b6fa-15326784cc6c.png)

- Here I will be downloading near data using shapefile (in zip format, created in other post)

- The approach that I am using is POST instead of GET.
- I also need to paste in the endpoint for the POST location and Authorization key in the Headers tab:
The endpoint is downloaded from NEAR DATA API given from their company in PDF, in the section of Create Job with Shapefile:

```bash
https://uberretailapi.uberads.com/v1/uberretailapi/createJobWithFile
```

- In Authorization tab, change **Type** to Bearer Token and insert Token value given by NEAR to Token box


![image](https://user-images.githubusercontent.com/43855029/168139485-2c7984e4-5099-4086-aee1-1ed12a970c63.png)

- The most important task is the Body section, in the form data, there are 2 files that you need to insert:

    + **polygonFile** with file type is **File**. The browse button will appear for you to upload the ziped shapefile
    + **jsonRequest** with file type **Text**. Detail of json file is below:

```json
{"pipReportType":"PIN_REPORT",
"reportName":"F17_2021Q1",
"polygonInputOptions": { "polygonFormat": "ESRI_SHAPEFILE_ZIP",
"polygonNameAliasElement": "PageName" },
"startDateTime": "2021-01-01 00:00:00",
"endDateTime": "2021-03-31 23:59:59"
}
```

![image](https://user-images.githubusercontent.com/43855029/168343025-30961336-e99a-4d72-94d0-3e5ee1c69319.png)


Note that: the reportName can be changed to match with the input shapefile.
The **polygonNameAliasElement="PageName"** is fixed with the shapefile variable names
The start and end DateTime can be altered

- Once everything is specified, hit Send then the job is submitted

![image](https://user-images.githubusercontent.com/43855029/168140571-8eb5be8a-88fb-43d1-ae92-ef80d0151186.png)

- Go back to Vista page and you will see the job submitted:
![image](https://user-images.githubusercontent.com/43855029/168140674-733120ad-5299-42d0-9558-6b7bef2cd2d5.png)


## Python
- Postman is free and simple to use to download data. However, it still requires manual import the shapefile and name changed for every download.
- Python script is generated to support mass downloading.
- To get the python script from Postman click on the link to open code snippet:

![image](https://user-images.githubusercontent.com/43855029/168147268-ab5b3f1b-496b-4e7a-9dcd-30ee77a3945e.png)

- In order to do the automation, we need to modify the input information, such as report name, zip file name, zip file location 
- Here we do everything in ManeFrame M2 supercomputer, the GIS shapefiles are uploaded to home directory.
- Following is the python code:

```python
import requests
import os
import json

url = "https://uberretailapi.uberads.com/v1/uberretailapi/createJobWithFile"
headers = {'Authorization': 'Bearer xxxxxxx'}    
j=0
n=1
dir1 = '/home/tuev/Projects/Makris/GIS/zip/test/'
listfile = os.listdir(dir1)
while j<=len(listfile)-1:    
    dict1 = dict({"pipReportType":"PIN_REPORT","reportName":f"{listfile[j]}","polygonInputOptions": { "polygonFormat": "ESRI_SHAPEFILE_ZIP","polygonNameAliasElement": "PageName" },"startDateTime": "2021-01-01 00:00:00","endDateTime": "2021-03-31 23:59:59"})
    payload = {'jsonRequest':str(dict1).replace("'",'"')}
    files=[('polygonFile',(f"{listfile[j]}",open(f'{dir1}{listfile[j]}','rb'),'application/zip'))]    
    response = requests.request("POST", url, headers=headers, data=payload, files=files)            
    if "True" in str(json.loads(response.text).values()):
        print("Succeeded. Submitting job to download ",listfile[j])
        j+=1
        n=1
    else:
        print("Failure. Resubmitting job ", listfile[j], " ", n,  " times")
        n+=1
```

- Note that due to the near server, sometime not being able to import GIS file, so we need to resubmit if it failed. It is represented as the for while loop.

## Download the submitted jobs:

Using OnDemand web portal (Remote Desktop) and open Firefox, 
Go to near.com website and signin using Nicos's username & password then to retrieve the submitted jobs:

```
https://vista.um.co/users/sign_in
```

Check if your job has spawned the report or not to download to M2 directory:

![image](https://user-images.githubusercontent.com/43855029/196506189-515b42bd-127d-4f0a-a0d3-c32b886cd66d.png)
