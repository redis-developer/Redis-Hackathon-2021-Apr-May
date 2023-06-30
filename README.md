<!--
Author: Mohamed Fazaary A S
Associate Consultant @ Servian
 -->


This is an implementation ofRedis Searchleveraging Exact Match, Prefix based searching and Fuzzy search features


# Why do we need this app?
A lot of times, we need to search keyWords in our dataset. It is super hard to go through each dataset when we exactly don't remember where they keyWord and data that we search for resides.

Our Dataset also contains different Column lengths, which makes it very hard if not impossible to execute a single search across all datasets and retrieve all possible keyword matches [Fuzzy, Prefix or Exact Matches] from all of those dataset without crashing our code editor or stalling our UI.

This project gives a portal where we can manage our datasets and execute searches across all of those datasets and provides a one-stop platform

# How does this app work, functionally?
All you need to do as a user is to upload your Dataset. Within seconds, you are good to execute your search.
You can also keep adding datasets and *non-duplicated* search results will be available on request.

# Which datasets are supported?
At the moment, we only support CSV datasets. It is preferred to upload a CSV that has headers.

# How does this app works, Technically?

Architecture

![image](https://user-images.githubusercontent.com/20859772/118287326-fc1f6a00-b4f0-11eb-8ac8-05732da09b00.png)

Once you upload a CSV Dataset, it is then goes through post-processing. This includes

* You create a new user. - This is stored in Redis Cache. For convenience, A user is created at the time of starting this app to get a head start of this boring process.
* Login to application - If the credentials match, a jwt token is created and maintained in Redis using jwt-redis package.
* User uploads a CSV dataset.
* Once the upload is completed and stored in datasource/ of this app, next steps such as creating the indexes for every column of the dataset using [hset] and
  [FT.CREATE] is performed- Helps with Exact Word matches
* Creates a suggestion dictionary for every row in the dataset - Helps with Fuzzy and Prefix based searching [FT.SUGADD]
* Within seconds of uploading the dataset, all keys are indexed and suggestion dictionaries are populated and notifies the user in the frontend.
* The user in the frontend dispatches a request to obtain search results.
* Every keyString is then processed against [FT.SEARCH], [FT.SUGGET] and [FT.SUGGET,FUZZY] commands and finally a payload is then sent to frontend for the user to visualize.
* Matching key-string in the payload is highlighted to the user to better locate their search text.

# Spin up Environment
 - This app requires you install Docker and Git
 - Clone the repository
   git clone https://github.com/Fazaarycode/Redis-Hackathon-2021-Apr-May.git
 - Change Directory: cd Redis-Hackathon-2021-Apr-May/
 - *Run* `docker-compose up`
 - Once docker containers have spun up, You should have frontend running on http://localhost:3000/
 - Backend running http://localhost:4000/
 - A user is already created for you.
 - Click on Login in the home page and pass
   * email: user@runtimeterror.com
   * password: 123
- You should now be able to Access the search page where you can upload CSV dataset and begin searching!
- For any questions, watch the video from the URL linked to this README.md

# Technologies Used
* ReactJS
* NodeJS
* Docker containerization
* Redis Search
   - Commands
     - FT.SEARCH
     - FT.CREATE
     - FT.DROPINDEX
     - FT.SUGADD
     - FT.SUGGET
     - SET
     - GET
* Redis Cache
* A few npm packages

# Bring down environment

Make sure you are in the project directory.
Run `docker-compose down`

# Troubleshooting steps

If your server/datasource has CSV datasets files added but you aren't seeing any search results in UI, then what might have happened is the Keys that were created during your previous session must be erased when the container did or you ran `docker-compose down`
Quick walkaround is start fresh and that is really simple. Just remove the .CSV files inside `server/datasource` directory and refresh your UI and add new dataset, it should work fine.
The reason Keys aren't persisted as vMounts is to provide a fresh feel during Application showcases.

This is a complete implementation from scratch. Hope you like it!
If you have any questions, comments, recommendations, jobs, or partnership with Servian, let me know through the email we registered.

# Test URLs (Backend only), Make sure you pass your cookies
* Allow origin of your client inside server/index.js settings.

* User registration
curl --location --request POST 'http://localhost:4000/user-registration' \
--header 'Content-Type: application/json' \
--data-raw '{"userName":"123","email":"alex.plywood@tuts.com","companyName":"123","password":"123","confirmPassword":"123"}'


* User login

curl --location --request POST 'http://localhost:4000/user-login' \
--header 'Content-Type: application/json' \
--data-raw '{"email":"alex.plywood@tuts.com","password":"123"}
'
* User logout
curl --location --request POST 'http://localhost:4000/user-logout' \
--header 'Content-Type: application/json' \
--data-raw '{
    "userName": "alex.plywood@tuts.com"
}'

* File Upload - Make sure your file exists and adjust the Url. ` Make sure you pass your cookies `

curl --location --request POST 'http://localhost:4000/upload-csv' \
--form 'fileUpload=@"~/Fazaary/Desktop/shorterimdb.csv"'

* Get KeyStrings Results - Returns Exact, Prefix and Fuzzy based results ` Make sure you pass your cookies `
curl --location --request GET 'http://localhost:4000/auto-complete-results?keyString=169

# Screenshots

Welcome Page

![image](https://user-images.githubusercontent.com/20859772/118283179-b5c80c00-b4ec-11eb-85f5-9d40256bf31b.png)

Client Login

![image](https://user-images.githubusercontent.com/20859772/118283241-c37d9180-b4ec-11eb-9bb2-4a769cbf1d76.png)


Search Page Landing

![image](https://user-images.githubusercontent.com/20859772/118283290-d2fcda80-b4ec-11eb-9303-44025bbed4ad.png)

Upload file

![image](https://user-images.githubusercontent.com/20859772/118283366-ea3bc800-b4ec-11eb-8e90-adcbd5f890a9.png)

Search One dataset

![image](https://user-images.githubusercontent.com/20859772/118283635-32f38100-b4ed-11eb-9c1c-2543f3ec7cbc.png)

Redis LookUp
At this point, we can have a look at our Indexes: They are dynamically added

Like so:
![image](https://user-images.githubusercontent.com/20859772/118284384-fbd19f80-b4ed-11eb-9037-47f626b143ef.png)

Individual records looks like:

![image](https://user-images.githubusercontent.com/20859772/118283801-5f0f0200-b4ed-11eb-82f4-2a56753bee3c.png)

Along side, there resides our JWT token and our user in cache:

JWT:
![image](https://user-images.githubusercontent.com/20859772/118284027-9da4bc80-b4ed-11eb-89e4-340da200eb2a.png)

User:
![image](https://user-images.githubusercontent.com/20859772/118284069-aac1ab80-b4ed-11eb-9ce7-fec4a8bc7e83.png)

Suggestion Dictionary entry:
Based on Headers from our uploaded Dataset

![image](https://user-images.githubusercontent.com/20859772/118284795-6e427f80-b4ee-11eb-89d0-9094ffd446d2.png)

Retrieving Fuzzy based results from our Suggestion Dictionary
![image](https://user-images.githubusercontent.com/20859772/118284954-a1850e80-b4ee-11eb-87a0-0928085d2399.png)


# Have a good day, Stay safe.
