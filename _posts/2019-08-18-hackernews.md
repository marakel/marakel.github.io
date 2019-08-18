---
title: "Analyzing Posts from Hacker News"
date: 2019-08-18
tags: [python, data analysis]

---
## Analyzing Posts from Hacker News
Some background information: Hacker News is a social news website focusing on computer science and entrepreneurship that contains posts that are upvoted or downvoted by popularity. The dataset I am using has been reduced to about 20,000 rows. (Posts were removed that did not receive any comments). 

I am specifically analyzing "Ask HN" and "Show HN" post types. I am looking to determine whether each post type (Ask HN or Show HN) receives more comments on average. I will also be looking at whether the time the posts are created has an effect on the amount of comments it recevies. 

This is a simple project I am using to practice some basic data analysis in python. My later projects will utilize more of the python libraries and some prediction techniques.


My first step is to upload the csv as a list of the rows. I am also removing the first row of the data, which are the headers.
```python
from csv import reader
opened_file = open("hacker_news.csv")
read_file = reader(opened_file)
hn = list(read_file)

hn = hn[1:]
headers = hn[0]
headers
```


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


By calculating the average for each type of post we can see that Ask Posts in our sample receive about 14 comments while Show Posts only receive about 10 comments. Taking into account this large difference, I will focus on only Ask Posts for the rest of the analysis. 
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


I am now trying to determine if we can maximize the amount of comments an Ask HN Post receives by creating it at a certain time. My first step was to find the amount of Ask Posts created in each hour of the day and the number of comments each has. Then I calculated the average amount of comments for each post at each hour of the day. 

Note: The times listed can be converted to EST. For example, 13:00 is really 1pm EST. I determined this reading through the documentation for the dataset. I decided to leave the times in teh original format.
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

```python
#Calculating the Average Number of Comments for Ask HN Posts by Hour

avg_hour = []

for hour in comments_hour:
    avg_hour.append([hour, comments_hour[hour] / counts_hour[hour]])

avg_hour
```


The rest of the analysis will focus on sorting and printing the values I calulated.
```python
swap_avg_hour = []
for row in avg_hour:
    swap_avg_hour.append([row[1], row[0]])  
print(swap_avg_hour)

sorted_swap = sorted(swap_avg_hour, reverse=True)
sorted_swap
```

```python
#Printing the 5 hours with the highest average comments.
print("Top 5 Hours for 'Ask HN' Comments")
for avg, hr in sorted_swap[:5]:
    print(
        "{}: {:.2f} average comments per post".format(
            dt.datetime.strptime(hr, "%H").strftime("%H:%M"),avg))
```

#Conclusion
Based on this analysis, a post should be categorized as an "Ask HN" Post and be created between 15:00 and 16:00 (Which is 3:00pm - 4:00pm EST) to maximize the amount of comments it receives. It should be noted that I excluded any posts that received no comments, so this conclusion was came to only from data that included posts with comments. It would be more accurate to conclude that of the posts that received comments, Ask HN Posts received more on average and Ask Posts created between 15:00 and 16:00 recevied the most comments.

