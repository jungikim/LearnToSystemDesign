# User-Generated Content Feed Update

On platforms such as Twitter, Facebook, Instagram, and LinkedIn, you have "friend" connections and you get the updates from your connections.
How should this service be designed?


## assumptions
- Twitter
  - connection is directional (follower, followee?)
  - twit size: max 140 char (utf-8; up to 640 bytes)
  - 140 limit also includes links to media files
  - users can be an individual or an organization (including influential individuals, more tweets, more connections)
- Guestimates
  - total number of users: 1B
  - num users active per month: 100M
  - num tweets per user per month:
    - median: 5 tweets
    - avg: 10 tweets
    - (assuming left-skewed distribution)
  - num followers per user:
    - individual:  avg 30?
    - org, influential people: avg 10M?
  - num follows per user:
    - individual:  avg 30? (assuming symmetric connections for a regular user)
    - org, influential people: 100?
  - how many read-only accounts? (like myself): 100M? 500M? 1B?
  - 500M ~ 1B new tweets per month * 640 bytes = 300~600 gigabytes of tweet per month
  - media size can vary (are there many tweets with media files) and for now, assume they are stored in CDNs
  - I am assuming traffic is steady (as it is serviced globally) most of the time, but there can be peaks (current events, celebrity, ...)
- focusing around "feed" part


## Requirements
### functional
- user can update 
- user can create/delete a tweet
- user can subscribe to another user
- user has a list of followers (people the user subscribes to)
  - user can add another user to the followers list
  - user can remove a user from the followers list
- user has a list of follows (people who follow the user)
  - user cannot modify the follows list?
- on "feed" page, user sees 100 most recent tweets from the users subscriptions
- user can request tweets from subscriptions for different periods of time (100 older tweets or from an earlier period, one week/month/)

### non-functional
- availability - mimimum down time, twitter is a also a platform for people to get news
- consistency - how fast should a new tweet be available to all of the users?
  - own tweets - right away
  - users who follow - within a reasonable time (few minutes?)
- fetching N (100?) most recent tweets from subscription should be very quick
- fetching older tweets should work within a reasonable time (few seconds max)


## Data model

Users: 1B entries x 512K bytes? -> 500 gigabytes
```
uid
e-mail
phone number
age gender
location
language settings
passwd encrypted (separate table for old ones?)
```

Tweets: 500 gigabytes of new tweets per month;  20 years x 500GB/month * 12 month -> 120 Terabytes total
```
(tid, uid, date, message)
```

User1-Follows-User2: 4bytes each (4 byte int ~ 4 billion) = 8 bytes ; 1B users * 30 folloers per user -> 30B entries; 240 gigabytes
```
(uid1, uid2)
```

Feed-cache: 
```
(user, list of tids)
```

## Components

- DBs - Users, User1-Follows-User2, Tweets
  - partitioned, duplicated
  - priority on read speed

- feed cache DB / NoSQL
  - user (key) -> recent tweets from follows (value: list of tids)
  - parititoned, duplicated

- new tweet service
  - when user A creates a new tweet, it triggers feed cache update for all users who follow user A

- feed cache update service
  - receives uid, tid, date
  - retrieves followers of uid
    - for all followers, update feed of follower (key) with (uid, tid, date)
    - follow list can be very small (ordinary user) or very long (organiation, influential user)
    - distribute and load balance



# WIP; @TODO
1. how do we make sure the requirements are met
2. what techniques do we need for DB availability/consistency
3. How should the DBs be partitioned? - consideration for influential users and tweets?
4. what techniques for read-centric DB duplication
5. feed cache update - how to handle concurrent writes 





