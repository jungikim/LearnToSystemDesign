
# Content Recommendation

## Overview
 - Youtube
 - assumptions (guess-timate)
   - num active users per 24h: ~100M
   - num new videos per 24h: ~1M
   - total num videos: ten billion? 1M per day * 365 * 20yrs
   - video:
     - avg length: ~10 minutes
     - avg size (assuming only one resolution/encoding): 10MB
     - metadata (few megabytes?):
       - created/updated/.. dates
       - uploaded by
       - description
       - num views (total, by segment)
       - num votes (up/down)
       - captions
   - comments
      - num should vary:
         - longer it's been on the platform
         - content length
         - num views
      - median: 10 comments?
      - average: 50 comments?

   - creator / channel
     - description
     - videos
     - subscribed users

   - an average user
     - plays 10 videos
     - time/purpose
       - day-time (live news/educational/kids/elderly?)
       - evening-time (everything else? recreational)
       - hard to categorize and make assumptions
         - they will probably not work. Let the data tell.
       - let's say, throughout the day, any kind of video
     - user info we can gather/use
       - age group
       - gender?
       - region
       - default language
       - subscribed channels
       - view history
         - view dates
         - comments
         - thumbs-up list
         - thumbs-down list


 - Metrics
   - Business-oriented
     - on recommented videos
       - click rate
       - play time
       - thumbs-up
       - comment (and its sentiment) 
     - over all user engagement 
       - change in active users per day
       - change in avg num/length of views
       - hard to interpret the contributing factor?


   - Recommendation system performance
     - hit-or-miss vs follow-up measure over time
       - user sees the recommendation and clicks one of the suggested -> success
       - or success measured num played after shown / num suggested
     - do we need to care about recommended/played order?
       - depending on the platform, users see multiple videos at once
         - 2~5 videos on a phone
         - 5~20 videos on a PC
         - there is still a natural order in which the user see the suggestions
           - in most cultures top-to-bottom, left-to-right
           - right-to-left? (Arabic, Japanese)
         - unsure
       - play order tells the most about the user interest
         - I myself would click first the most interesting one
         - do we expect users to go back to the recommended list and look again?


- Goal/continuous monitoring
  - increase user engagement
  - many ways to do so,
  - one example:
    - view rate of recommended videos within some time period of being suggested
      - near-complete view: 80% of total length
      - period: 1 day
      - Top K suggested: {5,10,20} videos
    -> Precision@Top5, @10, @20


## Feature

### Data Model
- User
  - userid
  - age group (category)
  - gender
  - location (country/region)
  - language

- Creator/Channel
  - channelid

- Video
  - videoid
  - channelid
  - created date
  - description
  -

- Channel Subscription
  - (key, userid, channelid, joined date)

- Video-User-View
  - (key, userid, videoid, view date, play time)
  - a user can view the same video multiple times

- Video-Caption

- Video-Vote
 - (key, videoid, userid, vote: {up/down})

- Video-User-Comment
  - (key, video id, user id, comment, up vote, down vote)




## Video-User-View seems to be the most important and useful

- disneyplus/hulu/netflix/amazon prime has:
  - trending contents (most viewed)
  - because you watched these ... (content-based)
  - users who like (genre/topic) watched these...
  - continue to watch (series, didn't watch the entire content, ...)

- User interest modeling:
  - similar users -> video
  - similar videos -> video

 - List of trending videos/channels

 - Complete/Incomplete watches
  - complete watches should be removed/pulled down in the rank
  - incomplete watches 
     - if stopped at the beginning -> poor indication
     - stopped in the middle -> higher chance at continuing


## Recommendation approach
- Video x User matrix -> get distributional similiarity
  - matrix is sparse, but seems intractable (100Million x 10Billion)

- collaborative
  - similar viewers -> videos watched by similar users
- content-based
  - previously watched video -> video with similar topic/genre, by same creator,
- prior prob / general interest
  - popularity by age group, gender, region, language
  - subscribed channels
  - (some services ask you about genres you are interested etc..)


All three factors can be merged.

P(Vrec | U),
  marginalized over all Usimilar of U
   Sigma_Usim [ P(Vwatched | Usim) * P(Usim | U ) ]
  marginalized over Vwatched of U
   Sigma_Vsim [ P(Vrec | Vwatched) * P(Vwatched | U)]

P(Vrec) ~= global statistics of Vrec over all Videos
   could be from past N hours or N days

P(Vrec, U) = P(Vrec) * P(Vrec | U) / P(U) (but P(U) is given)



## Algorithms - User-User, Video-Video similarity

### Unsupervised clustering
- k-means
  - training needed
  - faster inference
  - feasible for 100M users and 
- k-nn
  - no training 
  - slower inference

- as a user->video search task using a shared feature space


### feature representation


### similarity measure (U-U or V-V)
- feature vector or embedding of some sort -> cosine similarity (other/better measures?)
- FAISS: speed up embedding similarity with GPU
- if this is bottleneck, reduce search space with pre-computed K-means
  - or just consider certain factor combinations such as (age group, region, gender) as a cluster


## Multi-step vs Single-step

- Retrieve candidates -> Re-Rank
- Retrieve 

## Post-processing
- filter out already watched videos
- filter based on age, region restriction


## Batch-processed and Cached for each user
- certain number of times per day
  - 100 videos per user
  - twice a day


