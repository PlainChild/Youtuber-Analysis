# **YouTube Channel Performance Analysis**  
**An analysis of Indonesia's Top 100 YouTubers by subscribers and their recent engagement metrics**  

---

## **Objectives**  
The Marketing Team wanted to find out which popular YouTubers were the best fit for their marketing campaign. Therefore, the Marketing Team asked us to find the top 20 Indonesian YouTubers with the most subscribers, and find out their latest performance metrics. 

The Head of Marketing also asked the Dashboard to highlight which channels have high audience engagement and high potential reach. So, the Marketing Team can choose which YouTubers to promote with.

---

### **Metrics & Measures**  
| Metric | Description | 
|--------|-------------|
| **Subscribers** | Total channel followers. Measures their popularity |
| **Total Recent Views** | Total views from last 20 videos. Indicates current audience interest |
| **Total Recent Likes** | Total likes from last 20 videos. Measures positive engagement |
| **Total Recent Comments** | Total comments from last 20 videos. Indicates audience interaction |
| **Views per Video** | Average views per recent video. Helps assess consistency |
| **Engagement Rate** | `(Likes + Comments) / Views * 100`. Measures how much the audience interacts |
| **Upload Frequency** | Daily, Weekly, or Monthly. Determines their posting frequency|

---

### **Charts on Dashboard**  
| Chart | Description | 
|--------|-------------|
| **KPI Cards** | Showing Views per Video, Total Recent Likes, and Engagement Rates |
| **Column Chart** | Showing Top 20 Youtubers with the most Subscribers |
| **Treemap Chart** | Showing Total Recent Views of Top 20 Most Subscribed Youtubers |
| **Table** | Showing Rank, Channel Name, and Upload Frecuency of Top 20 Most Subscribed Youtubers |

---

## **Data Source**  
By using "Top 100 Social Media Influencers 2024 Countrywise" on kaggle. We can get access to channel' name, username, or ID of the top Indonesia youtuber. The dataset provides structured information about the top 100 influencers from various countries globally. 

We only focus on Columns:
1. Rank
2. Name = Channel' name, username, or ID
3. Followers = Total subscribers of 2024

        Link: https://www.kaggle.com/datasets/bhavyadhingra00020/top-100-social-media-influencers-2024-countrywise
        or
        File: assets/Dataset/youtube_data_indonesia.csv

---

## **Workflow** 
1. Open new Excel and Import the Data Source
![Image](https://github.com/user-attachments/assets/8d774655-1ba2-4dab-85a0-21349c39a66a)   
- Choose File Origin as "650001: Unicode (UTF-8)"

![Image](https://github.com/user-attachments/assets/30e70b83-3aae-4396-827a-7cb57ff862e7)

2. Check for Data missing on "Name" columns 
- Find & Select -> Go To Special -> Blank
![Image](https://github.com/user-attachments/assets/fb923bc2-4cf8-4936-8fb6-880550e046a8)

3. Check for Data duplication on "Name" columns 
![Image](https://github.com/user-attachments/assets/01393357-440b-4d78-aa04-c650472c3ed1)

    This will highlight Cell that has duplication

4. Create 2 new columns: Channel Name & Channel Username (Due to "Name" column consist of both channel's name and username that separated with "@")
- Formula for "Channel Name" =LEFT([@NAME];FIND("@";[@NAME])-1)
- Formula for "Channel Username" =RIGHT([@NAME];LEN([@NAME])-FIND("@";[@NAME]))

5. Check for data error
- Find & Select -> Go To Special -> Formula -> Error
![Image](https://github.com/user-attachments/assets/7a5d2b9a-db08-47dd-95b9-d4ab5b76793e)

6. So in total, there are 4 columns we are focusing into: Rank, Channel Name, Channel Username, and Followers

        File: assets/Dataset/Excel-youtube_data_indonesia.csv

7. Using Youtube Data API, we can access Channel's recent videos to calculate their recent performance.
- Iam using Google Colab to run the Python Script of Youtube Data API

        File= assets/Script/Python Script of Youtube Data API
- Connect the runtime of Google Colab
![Image](https://github.com/user-attachments/assets/4d650e81-65d0-43e1-b52e-b8352c641da3)
- CTRL + M + B -> create new code cell, and Copy the script of Python API into the code cell
- Input your API Key here:

        line 6  API_KEY = " "
- How many recent videos we request. I request 20 videos as it is enough to measure recent channel's perfomance
(You can request more videos, but Youtube has limit quota on using API. So, if you have a lot of Youtube Data API Quota, you can request a lot of recent videos for better stats.) 

        line 52  maxResults=20

8. So, we just add more columns into the dataset: Total Recent Likes, Total Recent Views, Total Recent Comments, Earliest Video Date, Newest Video Date.

        File: assets/Dataset/Python-youtube_data_indonesia.csv
- Those 5 columns are stats based on 20 recent videos we request using API.
- Earliest Video Date & Newest Video Date are for calculatinf Upload Frequency.

9. I am using Microsoft SQL Server Management System to create theivew table
- Use the Query of View Table of Youtube Data

        File: assets/Script/Query of View Table of Youtube Data
- On Step 3 or after creating the Database and using it, Import Flat File of the dataset into SQL Server
![Image](https://github.com/user-attachments/assets/229b5aaa-686d-4bf2-ac40-863bc859bdf2)
- On Modify Columns, set Followers, Earliest and Newest Date as NVARCHAR. The reason are: Followers has "M" at the end of value, and both date can have "N/A" as value due to previous Python API script.
![Image](https://github.com/user-attachments/assets/488e09e2-a7d1-452b-a011-60a8dc928600)
- Continue on the SQL Query, including: Removing Rows that has "N/A" in either Earliest or Newest Video Date, Removing "M" in Follower, and creating the View table.
![Image](https://github.com/user-attachments/assets/6f90ea4f-f276-48e5-a7ab-7440443753de)

        This means that Channel is deleted or hidden, so we do not need it
-  View Table consist of 7 Columns needed for the dashboard: Rank, Channel Name, Followers, Total Likes, Total Comments, Total Views, Earliest Date, and Newesst Data

10. I am using Power BI to create the dashboard. Select a data source: SQL Server.
- Input SSMS Server Name
![Image](https://github.com/user-attachments/assets/7d9d657e-5388-481b-a78d-e4d751110da5)
- Select the view table

11. Create 3 new Measures
![Image](https://github.com/user-attachments/assets/32943b9f-582a-45e5-befd-70e851dca04b)
- Views per Video. Formula:

        VIEWS PER VIDEO = DIVIDE(SUM('View-youtube_data_indonesia'[TOTAL_RECENT_VIEWS]), 20) -- View-youtube_data_indonesia = Table name & TOTAL_RECENT_VIEWS = column name
- Engagement Rate. Formula:

        ENGAGEMENT RATE = 
                        DIVIDE(
                                SUM('View-youtube_data_indonesia'[TOTAL_RECENT_COMMENTS])+
                                SUM('View-youtube_data_indonesia'[TOTAL_RECENT_LIKES]), 
                                SUM('View-youtube_data_indonesia'[TOTAL_RECENT_VIEWS])
                        )
- Upload Frequency. Formula:

        UPLOAD FREQUENCY = 
        VAR BetweenDay = DATEDIFF(
                                MIN('View-youtube_data_indonesia'[EARLIEST_VIDEO_DATE]), 
                                MAX('View-youtube_data_indonesia'[NEWEST_VIDEO_DATE]),
                                DAY
                        )
        VAR AvgInterval = DIVIDE(BetweenDay, 19) -- Since we take 20 recent videos, therefore 20-1 = 19
        RETURN
            SWITCH(
                    TRUE(),
                    AvgInterval <= 2, "Daily", -- upload a video every 2 days or less is considered daily 
                    AvgInterval >= 17, "Monthly", -- upload a video every more than 2 weeks+2 days is considered monthly
                    "Weekly"
            )

12. Structure the Dashboard:
- Three Cards for: Views per Video, Total Recent likes, and Engagement Rate.
- Column chart with: Channel Name on X-axis and Followers on Y-axis. Then, Filter Channel Name with Filter Type Top 20, and By Value Followers
- Table with: Rank, CHannel Name, and Upload Frequency. Then, Filter Channel Name with Filter Type Top 20, and By Value Followers
- Treemap with: Channel Name as Category and Total Recent Views as Value.  Then, Filter Channel Name with Filter Type Top 20, and By Value Followers

---

### **Result**
![Image](https://github.com/user-attachments/assets/3bfba81f-f1e5-42a6-86fd-20aa8daf92e9)

    File: assets/Youtuber Dashboard.pbix

---


## **Conclussion**
### **Findings**
1. "Ricis Official" leads as most subscribed youtuber with 30.6M subscribers, followed closely by "AH" (29.8M) and "Rans Entertainment" (24.2M)
2. "Arif Muhammad" shows the highest potential reach (2.9M Views per Video) despite ranking 18th in subscribers.
3. "Deddy Corbuzier" who is on 8th place and "Zuni and Family" who is on 11th place also demonstrate exceptional reach with 2.7M and 1.4M views per video respectively
4. "Naisa Alifia Yuriza (N.A.Y)" on 10th, "BUDI01 GAMING" on 19th, and "Baim Paula" on 6th place in subscribers. Three of them show the highest Engagement Rate (3,6%)
5.  Daily Uploaders dominate the ranks (12 out of 20)

### **Insights**
1. Channels with the high engagement don't necessarily have high reach, suggesting different channel's strategies like, Deddy Corbuzier focuses on mass appeal while Baim Paula focuses on dedicated community.
2. Children's channels have massive subscriber bases but near-zero engagement due to Youtube disable comment feature on Kid's Channel or the Channel itself disables both like and comment features. Also, children don't typically interact and ony watch.
3. All top 5 channels upload daily, suggesting regular content is key to maintaining large audiences.

### **Recommendation**
**1. Deddy Corbuzier**
- Massive reach (2.68M views per video - 2nd highest)
- High absolute engagement (1.17M likes - highest)
- Daily uploads mean frequent content opportunities
- Broad appeal across demographics

**2.Naisa Alifia Yuriza (N.A.Y)**
- Exceptional 3.60% engagement rate (tied for highest)
- Strong views per video (1.24M - 4th highest)
- High total likes (889k - 3rd highest)

**3.Ria Ricis**
- Highest Subscribers count
- Strong engagement (3.47%)

