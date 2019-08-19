---
title: "Analyzing Posts from Hacker News"
date: 2019-08-18
tags: [python, data analysis]

---
This is a simple project I am using to practice some basic data analysis in python. My future projects will utilize more of the python libraries and some prediction techniques.

Some background information: Hacker News is a social news website focusing on computer science and entrepreneurship that contains posts that are upvoted or downvoted by popularity. The dataset I am using has been reduced to about 20,000 rows. Posts were removed that did not receive any comments.

I am specifically analyzing "Ask HN" and "Show HN" post types (HN just stands for Hacker News). I am looking to determine whether each post type (Ask HN or Show HN) receives more comments on average. I will also be looking at whether the time the posts are created has an effect on the amount of comments it recevies. 



My first step is to upload the csv as a list of the rows. I am also removing the first row of the data which are the headers.
```python
from csv import reader
opened_file = open("hacker_news.csv")
read_file = reader(opened_file)
hn = list(read_file)

hn = hn[1:]
headers = hn[0]
headers
```
`
['12224879',
 'Interactive Dynamic Video',
 'http://www.interactivedynamicvideo.com/',
 '386',
 '52',
 'ne0phyte',
 '8/4/2016 11:52']
`




I am then separating the Ask HN and Show HN Posts into different lists to make them easier to analyze.
```python
#Extracting AskHN and ShowHN Posts
askhn_posts = []
showhn_posts = []
other_posts = []
for row in hn:
    title = row[1]
    if title.lower().startswith("ask hn"):
        askhn_posts.append(row)
    elif title.lower().startswith("show hn"):
        showhn_posts.append(row)     
    else:
        other_posts.append(row)

print(len(askhn_posts))
print(len(showhn_posts))
print(len(other_posts))
```
`
1744
1162
17194
`




By calculating the average for each type of post we can see that Ask HN Posts in our sample receive about 14 comments while Show HN Posts only receive about 10 comments. Taking into account this large difference, I will focus on only Ask HN Posts for the rest of the analysis. 
```python
#Calculating Average Number of Comments for each Post Type
total_askhn_comments = 0
for row in askhn_posts:
    total_askhn_comments += int(row[4])
avg_askhn_comments = total_askhn_comments / len(askhn_posts)
print(avg_askhn_comments)

total_showhn_comments = 0
for row in showhn_posts:
    total_showhn_comments += int(row[4])
avg_showhn_comments = total_showhn_comments / len(showhn_posts)
print(avg_showhn_comments)
```
`
14.038417431192661
10.31669535283993
`




I am now trying to determine if we can maximize the amount of comments an Ask HN Post receives by creating it at a certain time. My first step was to find the amount of Ask HN Posts created in each hour of the day and the number of comments each has. Then I calculated the average amount of comments for each post at each hour of the day. 

Note: The times listed can be converted to EST. For example, 13:00 is really 1pm EST. I determined this reading through the documentation for the dataset. I decided to leave the times in the original format.
```python
#Finding Amount of Posts/Comments by Hour Created
import datetime as dt

results_list = []
for row in askhn_posts:
    results_list.append([row[6], int(row[4])])

comments_hour = {}
counts_hour = {}
date_format = "%m/%d/%Y %H:%M"
for row in results_list:
    date = row[0]
    comment = row[1]
    time = dt.datetime.strptime(date, date_format).strftime("%H")
    if time in counts_hour:
        comments_hour[time] += comment
        counts_hour[time] += 1
    else:
        comments_hour[time] = comment
        counts_hour[time] = 1

comments_hour
```
`
{'09': 251,
 '13': 1253,
 '10': 793,
 '14': 1416,
 '16': 1814,
 '23': 543,
 '12': 687,
 '17': 1146,
 '15': 4477,
 '21': 1745,
 '20': 1722,
 '02': 1381,
 '18': 1439,
 '03': 421,
 '05': 464,
 '19': 1188,
 '01': 683,
 '22': 479,
 '08': 492,
 '04': 337,
 '00': 447,
 '06': 397,
 '07': 267,
 '11': 641}
`




```python
#Calculating the Average Number of Comments for Ask HN Posts by Hour

avg_hour = []

for hour in comments_hour:
    avg_hour.append([hour, comments_hour[hour] / counts_hour[hour]])

avg_hour
```
`
[['09', 5.5777777777777775],
 ['13', 14.741176470588234],
 ['10', 13.440677966101696],
 ['14', 13.233644859813085],
 ['16', 16.796296296296298],
 ['23', 7.985294117647059],
 ['12', 9.41095890410959],
 ['17', 11.46],
 ['15', 38.5948275862069],
 ['21', 16.009174311926607],
 ['20', 21.525],
 ['02', 23.810344827586206],
 ['18', 13.20183486238532],
 ['03', 7.796296296296297],
 ['05', 10.08695652173913],
 ['19', 10.8],
 ['01', 11.383333333333333],
 ['22', 6.746478873239437],
 ['08', 10.25],
 ['04', 7.170212765957447],
 ['00', 8.127272727272727],
 ['06', 9.022727272727273],
 ['07', 7.852941176470588],
 ['11', 11.051724137931034]]
 `




The rest of the analysis will focus on sorting and printing the values I calulated.
```python
swap_avg_hour = []
for row in avg_hour:
    swap_avg_hour.append([row[1], row[0]])  
print(swap_avg_hour)

sorted_swap = sorted(swap_avg_hour, reverse=True)
sorted_swap
```
`
    [[5.5777777777777775, '09'], [14.741176470588234, '13'], [13.440677966101696, '10'], [13.233644859813085, '14'], [16.796296296296298, '16'], [7.985294117647059, '23'], [9.41095890410959, '12'], [11.46, '17'], [38.5948275862069, '15'], [16.009174311926607, '21'], [21.525, '20'], [23.810344827586206, '02'], [13.20183486238532, '18'], [7.796296296296297, '03'], [10.08695652173913, '05'], [10.8, '19'], [11.383333333333333, '01'], [6.746478873239437, '22'], [10.25, '08'], [7.170212765957447, '04'], [8.127272727272727, '00'], [9.022727272727273, '06'], [7.852941176470588, '07'], [11.051724137931034, '11']]
    [[38.5948275862069, '15'],
    [23.810344827586206, '02'],
    [21.525, '20'],
    [16.796296296296298, '16'],
    [16.009174311926607, '21'],
    [14.741176470588234, '13'],
    [13.440677966101696, '10'],
    [13.233644859813085, '14'],
    [13.20183486238532, '18'],
    [11.46, '17'],
    [11.383333333333333, '01'],
    [11.051724137931034, '11'],
    [10.8, '19'],
    [10.25, '08'],
    [10.08695652173913, '05'],
    [9.41095890410959, '12'],
    [9.022727272727273, '06'],
    [8.127272727272727, '00'],
    [7.985294117647059, '23'],
    [7.852941176470588, '07'],
    [7.796296296296297, '03'],
    [7.170212765957447, '04'],
    [6.746478873239437, '22'],
    [5.5777777777777775, '09']]
`




```python
#Printing the 5 hours with the highest average comments.
print("Top 5 Hours for 'Ask HN' Comments")
for avg, hr in sorted_swap[:5]:
    print(
        "{}: {:.2f} average comments per post".format(
            dt.datetime.strptime(hr, "%H").strftime("%H:%M"),avg))
```
`
    Top 5 Hours for Ask HN Comments
    15:00: 38.59 average comments per post
    02:00: 23.81 average comments per post
    20:00: 21.52 average comments per post
    16:00: 16.80 average comments per post
    21:00: 16.01 average comments per post
`




# Conclusion
Based on this analysis, a post should be categorized as an "Ask HN" Post and be created between 15:00 and 16:00 (Which is 3:00pm - 4:00pm EST) to maximize the amount of comments it receives. It should be noted that I excluded any posts that received no comments, so this conclusion was only from data that included posts with comments. It would be more accurate to conclude that of the posts that received comments, Ask HN Posts received more comments on average and Ask HN Posts created between 15:00 and 16:00 received the most comments.

