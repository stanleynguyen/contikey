# Contikey
Contikey is a web app for friends to share articles with each other. 

![](https://d2mxuefqeaa7sj.cloudfront.net/s_9CB1673BEAD3890D5658EDF501AE18932A0C7DC40F9FBC7BCB12770995C03873_1512643113561_Screen+Shot+2017-12-07+at+18.37.53.png)

### Main functionality
Users create channels on topics they are interested in, and post articles to these channels. Users can also subscribe to other users’ channels, and the app’s home page will show a feed of the most recently posted articles.

### Implementation
Contikey uses MariaDB for the database, Django for the backend, and React for the frontend.

## ER Diagram
![](https://d2mxuefqeaa7sj.cloudfront.net/s_D25635E5CBEB07B7D384E4A726E1F745A416101BCAAC840F8945253224A56D6C_1512565581430_dbProjERD.png)


Note: All Entities and Relationships have a created_at attribute as well.

## Relational Schema

### Tables

```sql
CREATE TABLE `article` (
  `article_id` int(11) NOT NULL AUTO_INCREMENT,
  `url` varchar(500) NOT NULL,
  `caption` varchar(500) DEFAULT NULL,
  `preview_image` varchar(500) DEFAULT NULL,
  `preview_title` varchar(100) DEFAULT NULL,
  `preview_text` varchar(500) DEFAULT NULL,
  `channel_id` int(11) DEFAULT NULL,
  `shared_from_article_id` int(11) DEFAULT NULL,
  `created_at` timestamp NOT NULL DEFAULT current_timestamp(),
  `num_words` int(11) DEFAULT NULL,
  PRIMARY KEY (`article_id`),
  FOREIGN KEY (`channel_id`) REFERENCES `channel` (`channel_id`) ON DELETE SET NULL ON UPDATE CASCADE,
  FOREIGN KEY (`shared_from_article_id`) REFERENCES `article` (`article_id`) ON DELETE SET NULL ON UPDATE CASCADE
)
```
```sql
CREATE TABLE `channel` (
  `channel_id` int(11) NOT NULL AUTO_INCREMENT,
  `user_id` int(11) DEFAULT NULL,
  `title` varchar(100) NOT NULL,
  `description` varchar(500) NOT NULL,
  `created_at` timestamp NOT NULL DEFAULT current_timestamp(),
  `num_subscribers` int(11) NOT NULL DEFAULT 0,
  PRIMARY KEY (`channel_id`),
  FOREIGN KEY (`user_id`) REFERENCES `user` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE
)
```
```sql
CREATE TABLE `channel_tags` (
  `channel_id` int(11) NOT NULL,
  `tag_id` int(11) NOT NULL,
  PRIMARY KEY (`channel_id`,`tag_id`),
  FOREIGN KEY (`channel_id`) REFERENCES `channel` (`channel_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  FOREIGN KEY (`tag_id`) REFERENCES `tag` (`tag_id`) ON DELETE CASCADE ON UPDATE CASCADE
)
```
```sql
CREATE TABLE `comment` (
  `comment_id` int(11) NOT NULL AUTO_INCREMENT,
  `comment_text` varchar(500) DEFAULT NULL,
  `user_id` int(11) NOT NULL,
  `article_id` int(11) NOT NULL,
  `created_at` timestamp NOT NULL DEFAULT current_timestamp(),
  PRIMARY KEY (`comment_id`),
  FOREIGN KEY (`user_id`) REFERENCES `user` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  FOREIGN KEY (`article_id`) REFERENCES `article` (`article_id`) ON DELETE CASCADE ON UPDATE CASCADE
)
```
```sql
CREATE TABLE `notification` (
  `notification_id` int(11) NOT NULL AUTO_INCREMENT,
  `user_id` int(11) NOT NULL,
  `created_at` timestamp NOT NULL DEFAULT current_timestamp(),
  `type` varchar(20) DEFAULT NULL,
  `is_read` tinyint(1) DEFAULT 0,
  `article_id` int(11) DEFAULT NULL,
  `type_user_id` int(11) DEFAULT NULL,
  `type_id` int(11) DEFAULT NULL,
  `channel_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`notification_id`),
  FOREIGN KEY (`article_id`) REFERENCES `article` (`article_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  FOREIGN KEY (`channel_id`) REFERENCES `channel` (`channel_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  FOREIGN KEY (`user_id`) REFERENCES `user` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  FOREIGN KEY (`type_user_id`) REFERENCES `user` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE
)
```
```sql
CREATE TABLE `tag` (
  `tag_id` int(11) NOT NULL AUTO_INCREMENT,
  `label` varchar(50) NOT NULL,
  `image` varchar(500) NOT NULL,
  PRIMARY KEY (`tag_id`)
)
```
```sql
CREATE TABLE `user` (
  `user_id` int(11) NOT NULL AUTO_INCREMENT,
  `facebook_id` varchar(20) NOT NULL,
  `email` varchar(100) DEFAULT NULL,
  `name` varchar(100) DEFAULT NULL,
  `photo` varchar(500) DEFAULT NULL,
  `bio` varchar(500) DEFAULT NULL,
  `created_at` timestamp NOT NULL DEFAULT current_timestamp(),
  PRIMARY KEY (`user_id`),
  UNIQUE KEY `facebook_id` (`facebook_id`)
)
```
```sql
CREATE TABLE `user_follows_channel` (
  `user_id` int(11) NOT NULL,
  `channel_id` int(11) NOT NULL,
  `created_at` timestamp NOT NULL DEFAULT current_timestamp(),
  PRIMARY KEY (`user_id`,`channel_id`),
  FOREIGN KEY (`user_id`) REFERENCES `user` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  FOREIGN KEY (`channel_id`) REFERENCES `channel` (`channel_id`) ON DELETE CASCADE ON UPDATE CASCADE
)
```
```sql
CREATE TABLE `user_follows_tag` (
  `user_id` int(11) NOT NULL,
  `tag_id` int(11) NOT NULL,
  PRIMARY KEY (`user_id`,`tag_id`),
  FOREIGN KEY (`user_id`) REFERENCES `user` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  FOREIGN KEY (`tag_id`) REFERENCES `tag` (`tag_id`) ON DELETE CASCADE ON UPDATE CASCADE
)
```
```sql
CREATE TABLE `user_friends` (
  `user_id` int(11) NOT NULL,
  `friend_id` int(11) NOT NULL,
  `created_at` timestamp NOT NULL DEFAULT current_timestamp(),
  PRIMARY KEY (`user_id`,`friend_id`),
  FOREIGN KEY (`user_id`) REFERENCES `user` (`user_id`) ON DELETE CASCADE,
  FOREIGN KEY (`friend_id`) REFERENCES `user` (`user_id`) ON DELETE CASCADE
)
```
```sql
CREATE TABLE `user_likes_article` (
  `user_id` int(11) NOT NULL,
  `article_id` int(11) NOT NULL,
  `likeStatus` tinyint(4) DEFAULT NULL CHECK (`likeStatus` = 1 or `likeStatus` = -1),
  `created_at` timestamp NOT NULL DEFAULT current_timestamp() ON UPDATE current_timestamp(),
  PRIMARY KEY (`user_id`,`article_id`),
  FOREIGN KEY (`user_id`) REFERENCES `user` (`user_id`) ON DELETE CASCADE ON UPDATE CASCADE,
  FOREIGN KEY (`article_id`) REFERENCES `article` (`article_id`) ON DELETE CASCADE ON UPDATE CASCADE
)
```
```sql
CREATE TABLE `view` (
  `view_id` int(11) NOT NULL AUTO_INCREMENT,
  `timestamp` timestamp NOT NULL DEFAULT current_timestamp() ON UPDATE current_timestamp(),
  `user_id` int(11) DEFAULT NULL,
  `article_id` int(11) NOT NULL,
  PRIMARY KEY (`view_id`),
  FOREIGN KEY (`user_id`) REFERENCES `user` (`user_id`) ON DELETE SET NULL ON UPDATE CASCADE,
  FOREIGN KEY (`article_id`) REFERENCES `article` (`article_id`) ON DELETE CASCADE ON UPDATE CASCADE
)
```

### Triggers

We used triggers to:

1. create notifications when someone comments on or likes a user’s article, or subscribes to a user’s channel
2. update num_subscribers for each channel when someone subscribes to or unsubscribes from it

```sql
CREATE TRIGGER comment_notification AFTER INSERT ON comment
FOR EACH ROW
INSERT INTO notification (type, article_id,type_id, user_id, is_read, type_user_id)
VALUES ('comment', NEW.article_id, NEW.article_id, (
     SELECT user_id FROM channel WHERE channel_id IN (
         SELECT channel_id FROM article WHERE article_id = NEW.article_id)), false, NEW.user_id);
```
```sql
CREATE TRIGGER like_notification AFTER INSERT ON user_likes_article
FOR EACH ROW
INSERT INTO notification (type, type_id, article_id, user_id, is_read, type_user_id)
VALUES ('like', NEW.article_id, NEW.article_id, (
     SELECT user_id FROM channel WHERE channel_id IN (
         SELECT channel_id FROM article WHERE article_id = NEW.article_id)), false, NEW.user_id);
```
```sql
CREATE TRIGGER delete_like_notification AFTER DELETE ON user_likes_article
FOR EACH ROW
DELETE FROM notification
WHERE
    type = 'like'
AND type_id = OLD.article_id
AND article_id = OLD.article_id
AND type_user_id = OLD.user_id
AND user_id IN (
    SELECT user_id from channel WHERE channel_id IN (
         SELECT channel_id FROM article WHERE article_id = OLD.article_id))
```
```sql
CREATE TRIGGER follow_notification AFTER INSERT ON user_follows_channel
FOR EACH ROW
INSERT INTO notification (type, type_id, channel_id, user_id, is_read, type_user_id)
VALUES ('channel', NEW.channel_id, NEW.channel_id, (
     SELECT user_id from channel WHERE channel_id = NEW.channel_id), false, NEW.user_id);
```
```sql
CREATE TRIGGER subscribe_channel AFTER INSERT ON user_follows_channel
FOR EACH ROW
BEGIN
SET @subbed_chnn = (SELECT channel_id FROM user_follows_channel WHERE created_at = (SELECT max(created_at) FROM user_follows_channel));
UPDATE channel SET num_subscribers = num_subscribers + 1 WHERE channel_id = @subbed_chnn;
END;
```
```sql
CREATE TRIGGER unsubscribe_channel AFTER DELETE ON user_follows_channel
FOR EACH ROW
UPDATE channel SET num_subscribers = num_subscribers - 1 WHERE channel_id = OLD.channel_id;
```
---
## Implementation Details

### Features (numbers corresponding to the bookstore features):


**1. Registration** 

Users log in with Facebook and create an account in our database.

- Frontend: uses Facebook Login and sends the Facebook access token to the backend
- Backend: verifies the access token and retrieves the user’s email, name and photo from Facebook API
- Database: inserts user info into user table
- Django’s session module is used to persist sessions

```python
# insert new user details
cursor.execute("""
    INSERT INTO user (facebook_id, name, email, photo)
    VALUES (%s, %s, %s, %s)""",
    [data.get('facebook_id'), data.get('name'), data.get('email'), data.get('photo')])
cursor.execute("SELECT LAST_INSERT_ID()")
```
---

**2. Subscribing to tags** 

After registration, users can choose tags to follow.

- Database: stored in user_follows_tag table, with foreign keys referencing user_id and tag_id

```python
# insert record for user following tag
cursor.execute("""
    INSERT INTO user_follows_tag (user_id, tag_id)
    VALUES (%s, %s)
""",[user_id, tag_id])
```

![](https://d2mxuefqeaa7sj.cloudfront.net/s_9CB1673BEAD3890D5658EDF501AE18932A0C7DC40F9FBC7BCB12770995C03873_1512643659131_Screen+Shot+2017-12-07+at+18.44.51.png)

---

**3. User profile** 

Viewing a user’s profile page shows all of that user’s channels, articles, friends and subscribed channels. 

```python
# return all of a user's articles
cursor.execute("""
    SELECT * FROM article WHERE channel_id in
        (SELECT channel_id FROM channel WHERE user_id= %s);
""", [user_id])
```
---

**4. User activity log**

Users can also view their own activity log (created channels, liked articles, posted articles, posted comments, made a friend, followed a channel, liked an article) ordered by the time the action was carried out.

```python
# return all of a user's activity, ordered by date
cursor.execute("SET @user_id = %s;", [user_id])
cursor.execute("""
SELECT * FROM (
    SELECT channel_id, NULL as article_id, NULL as comment_id,
        NULL as friend_id, NULL as followed_channel_id,
        NULL as liked_article_id, created_at
        FROM channel WHERE user_id = @user_id
    UNION ALL
    SELECT NULL as channel_id, article_id, NULL as comment_id,
        NULL as friend_id, NULL as followed_channel_id,
        NULL as liked_article_id, created_at
        FROM article WHERE channel_id in
            (SELECT channel_id from channel where user_id = @user_id)
    UNION ALL
    SELECT NULL as channel_id, NULL as article_id, comment_id,
        NULL as friend_id, NULL as followed_channel_id,
        NULL as liked_article_id, created_at
        FROM comment WHERE user_id = @user_id
    UNION ALL
    SELECT NULL as channel_id, NULL as article_id, NULL as comment_id,
        friend_id, NULL as followed_channel_id,
        NULL as liked_article_id, created_at
        FROM user_friends WHERE user_id = @user_id
    UNION ALL
    SELECT NULL as channel_id, NULL as article_id, NULL as comment_id,
        NULL as friend_id, channel_id as followed_channel_id,
        NULL as liked_article_id, created_at
        FROM user_follows_channel WHERE user_id = @user_id
    UNION ALL
    SELECT NULL as channel_id, NULL as article_id, NULL as comment_id,
        NULL as friend_id, NULL as followed_channel_id,
        article_id as liked_article_id, created_at
        FROM user_likes_article WHERE user_id = @user_id
    ) T1
ORDER BY created_at;
""")
```

![](https://d2mxuefqeaa7sj.cloudfront.net/s_9CB1673BEAD3890D5658EDF501AE18932A0C7DC40F9FBC7BCB12770995C03873_1512647482501_Screen+Shot+2017-12-07+at+19.46.08.png)

---

**5. New channel** 

Users can create a new channel by entering a title, description and tags.

- Database: store new channel record in channel table and tags in channel_tags table

```python
# insert channel
cursor.execute('INSERT INTO channel (user_id,title,description) VALUES (%s,%s,%s)', [user_id,title,description])
cursor.execute('SELECT last_insert_id()')
```
```python
# insert tags records
tags_insert_str = ''
for tag in tags:
    tags_insert_str += '('+last_insert+','+tag+'),'
cursor.execute('INSERT INTO channel_tags(channel_id, tag_id) VALUES ' + tags_insert_str[:-1])
```
![](https://d2mxuefqeaa7sj.cloudfront.net/s_9CB1673BEAD3890D5658EDF501AE18932A0C7DC40F9FBC7BCB12770995C03873_1512644774074_Screen+Shot+2017-12-07+at+19.01.09.png)

---

**6. Subscribing to a channel** 

Users can subscribe to or unsubscribe from channels, and that channel’s number of subscribers will change automatically.

- Database: Store the new subscription in the user_follow_channel table, while a trigger updates the channel table.


For creating/deleting a subscription record:

```python
# insert follow record
cursor.execute('INSERT INTO user_follows_channel (user_id, channel_id) VALUES (%s,%s)', [user_id,channel_id])

# remove follow record
cursor.execute('DELETE FROM user_follows_channel WHERE user_id = %s AND channel_id = %s', [user_id,channel_id])
```

As the frontend requires the number of subscribers very frequently, we used a num_subscribers column with triggers instead of executing a count on user_follows_channel as it is more efficient.

```python
# trigger to increment num_subscribers when a user follows a channel
migrations.RunSQL("""
    CREATE TRIGGER subscribe_channel AFTER INSERT ON user_follows_channel
    FOR EACH ROW
    BEGIN
    Set @subbed_chnn = (SELECT channel_id FROM user_follows_channel WHERE created_at = (SELECT max(created_at) FROM user_follows_channel));
    UPDATE channel SET num_subscribers = num_subscribers + 1 WHERE channel_id = @subbed_chnn;
    END;
""",
"DROP TRIGGER subscribe_channel"
)
```
Trying new things in SQL.

```python
# trigger to decrement num_subscribers when a user unfollows a channel
migrations.RunSQL("""
    CREATE TRIGGER unsubscribe_channel AFTER DELETE ON user_follows_channel
    FOR EACH ROW
    UPDATE channel SET num_subscribers = num_subscribers - 1 WHERE channel_id = OLD.channel_id;
""",
"DROP TRIGGER unsubscribe_channel"
)
```
![](https://d2mxuefqeaa7sj.cloudfront.net/s_9CB1673BEAD3890D5658EDF501AE18932A0C7DC40F9FBC7BCB12770995C03873_1512647610280_Screen+Shot+2017-12-07+at+19.53.15.png)

---

**7. Post article**

Users can post articles to one of their channels. They enter a URL and caption. The datetime will is automatically recorded and the article id incremented.

- Backend: we wrote an information scraper to get the details we needed from the URL the user keys in.

```python
scraper = metascrapy.Metadata()
#attempts to get information from the given url
scraper.scrape(url)
preview_image = scraper.image 
#Attempts to encode in utf-8 for database's use.
preview_title = scraper.title.encode('utf-8') 
preview_text = scraper.description.encode('utf-8') 
```

Following that, creating a new article is a simple affair.

```python
# create a new article with given values
cursor.execute('INSERT INTO article(channel_id,url,caption,preview_image,preview_title,preview_text,num_words, shared_from_article_id) VALUES (%s,%s,%s,%s,%s,%s,%s,%s)', [channel_id,url,caption,preview_image,preview_title,preview_text,num_words,shared_from_article_id])
```

![](https://d2mxuefqeaa7sj.cloudfront.net/s_9CB1673BEAD3890D5658EDF501AE18932A0C7DC40F9FBC7BCB12770995C03873_1512646490620_Screen+Shot+2017-12-07+at+19.34.22.png)

---

**8. Comments and likes**

Users can like or comment on articles.

- Modifying likes on articles:

```python
# insert a new like
cursor.execute('INSERT INTO user_likes_article(article_id,user_id) VALUES (%s,%s)', [article_id,user_id])
# delete a like
cursor.execute('DELETE FROM user_likes_article WHERE article_id = %s AND user_id = %s', [article_id,user_id])
```

- Modifying comments on articles:

```python
# insert a new comment
cursor.execute('INSERT INTO comment(article_id,user_id,comment_text) VALUES (%s,%s,%s)', [article_id,user_id,comment_text])
```

That these two (as well as a handful of other entities in the system) commonly come with other data, and at the same time may or may not exist led to interesting ways of checking them.

```python
# check if a specified user likes a specified article
cursor.execute("SELECT exists(SELECT 1 FROM user_likes_article WHERE user_id = %s AND article_id = %s) liked", [user_id, article_id])
```

![](https://d2mxuefqeaa7sj.cloudfront.net/s_9CB1673BEAD3890D5658EDF501AE18932A0C7DC40F9FBC7BCB12770995C03873_1512647117328_Screen+Shot+2017-12-07+at+19.44.47.png)

---

**9. Search**

Users can search for channels (by title), articles (by preview_title) and users (by name).

Held in 3 separate endpoints for user, channel and article separately.

```python
cursor.execute("SELECT * FROM user WHERE name LIKE %s;", [username])
cursor.execute("SELECT * FROM channel WHERE title LIKE %s;", [title])
cursor.execute("SELECT * FROM article WHERE preview_title LIKE %s;", [title])
```

![](https://d2mxuefqeaa7sj.cloudfront.net/s_9CB1673BEAD3890D5658EDF501AE18932A0C7DC40F9FBC7BCB12770995C03873_1512647667811_Screen+Shot+2017-12-07+at+19.54.11.png)

---

**10. Notifications** 

Users are shown a list of notifications when they click on the notifications button. Notifications are created for user A when another user B subscribes to one of A’s channels, or likes or comments on one of A’s articles.

  - Notification table stores A and B’s user ids, the notification type, id of the article/channel involved, and if the notification has been read.

```python
# retrieve list of a user's notifications
cursor.execute("SELECT * FROM notification WHERE user_id = %s;", [user_id])

# set the read status of notification after it is read
cursor.execute("""UPDATE notification SET is_read = %s WHERE notification_id = %s""",[is_read, notification_id])
```

- Triggers are used to create all notifications - on comment or like of an article, and subscription to a channel.

```python
# trigger for creating 'new comment' notifications
migrations.RunSQL("""
    CREATE TRIGGER comment_notification AFTER INSERT ON comment
    FOR EACH ROW
    INSERT INTO notification (type, article_id, user_id, is_read)
    VALUES ('comment', NEW.article_id, NEW.user_id, false)
""")

# trigger for removing 'new like' notifications when the like is deleted
migrations.RunSQL("""
    CREATE TRIGGER delete_like_notification AFTER DELETE ON user_likes_article
    FOR EACH ROW
    DELETE FROM notification
    WHERE
        type = 'like'
    AND type_id = OLD.article_id
    AND article_id = OLD.article_id
    AND type_user_id = OLD.user_id
    AND user_id IN (
        SELECT user_id from channel WHERE channel_id IN (
             SELECT channel_id FROM article WHERE article_id = OLD.article_id))
""")
```

![](https://d2mxuefqeaa7sj.cloudfront.net/s_9CB1673BEAD3890D5658EDF501AE18932A0C7DC40F9FBC7BCB12770995C03873_1512646910636_Screen+Shot+2017-12-07+at+19.41.11.png)

---

**11. Feed** 

Logged in users will see articles posted by channels they are subscribed to, with the most recent articles appearing at the top. 

```python
# return articles from channels followed by the user
cursor.execute('SELECT * FROM article WHERE channel_id IN(SELECT channel_id FROM user_follows_channel WHERE user_id = %s) ORDER BY created_at DESC', [user_id])
```

If the user is not logged in, the feed will then return a list of all the articles in every channel sorted by date of creation.

```python
# return most recent articles from all channels
cursor.execute('SELECT * FROM article ORDER BY created_at DESC')
```

![Left: Feed, Right: Channel recommendations](https://d2mxuefqeaa7sj.cloudfront.net/s_9CB1673BEAD3890D5658EDF501AE18932A0C7DC40F9FBC7BCB12770995C03873_1512646024932_Screen+Shot+2017-12-07+at+19.04.15.png)

---

**12. Channel recommendation** 

Users can view channels recommended for them on the home page. This returns a list of channels that the user is not subscribed to, but have tags that the user follows.

- The logic to acquire a logged in user’s recommended channel was broken up into two steps - to intersect the channels the user has not followed and the channels that have tags that the user liked.

```python
# user's not-followed channels
cursor.execute("SELECT channel_id FROM channel c LEFT JOIN user_follows_channel ufc USING(channel_id) WHERE ufc.user_id != %s OR ufc.user_id IS NULL", [user_id])   
     
# user's like-tagged channels
cursor.execute("SELECT DISTINCT channel_id FROM channel_tags WHERE tag_id IN(SELECT tag_id FROM user_follows_tag WHERE user_id = %s)", [user_id])
```

From the channel_id’s obtained we then query the database for the details necessary for the front end: channel_info, tags of the channel, the owner of the channel, number of subscribers on the channel, and if the user is subscribed to the channel (This value would be False, as the nature of recommendation is to recommend unfollowed channels anyway).

```python
# gets the channel information, including subscriber count
cursor.execute("SELECT * FROM channel WHERE channel_id = %s", [i])
    
# gets the tags of the channel
cursor.execute("SELECT * FROM tag WHERE tag_id IN(SELECT tag_id FROM channel_tags WHERE channel_id = %s)", [i])
    
# gets the user who owns the channel
cursor.execute("SELECT * FROM user WHERE user_id=(SELECT user_id FROM channel WHERE channel_id = %s)", [channel_id])
```

Of course, over the course of the system’s use users may be recommended *many* channels. We thus paginated the list of recommended channels before returning them.

```python
try:
    user_id = request.session['user_id']
    queryset = sql.get_user_recommend(user_id)
except:
    queryset = sql.get_nologin_recommend()
    
# instantiates a channel paginator        
paginator = channel_paginator()

# which paginates the given queryset based on Django's PAGE_SIZE setting
result_page = paginator.paginate_queryset(queryset,request)

# serializes the results for response to caller
serializer = recommend_serializer(result_page, many=True)
return Response({'channel':serializer.data},status=status.HTTP_200_OK)
```
----------


**13. Explore** 

The explore page lists the 10 most popular articles (based on number of likes) this month.

- Select the 10 articles with the most user_likes_article records within the last month, ordered by number of records

```python
cursor.execute("""
    SELECT * FROM article WHERE article_id IN (
        SELECT article_id FROM user_likes_article 
        WHERE TIMESTAMPDIFF(DAY, created_at, NOW()) < 31 
        GROUP BY article_id 
        ORDER BY count(*) DESC 
        LIMIT 10);
""")
```
---
