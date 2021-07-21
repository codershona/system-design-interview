# Design A Twitter or Any Social Media like(Facebook, Instagram, Snapchat)

## Introduction
* Twitter is a kind of platform where users can do microblogging in this social media by posting and interacting via messages that is known as tweets. 
* The users needs to register to post their status, like comments and retweet the tweets.
* On the other hand, Unregistered users are unable to read , make comments and like other users posts , in this case they can only view and read those posts. 
* This social media platform has a character limitation such as a it has maximum 140 characters limitation.

## Requirement analysis 
### Functional requirements

* User should be able to post tweets.
* User should be able to follow other users.
* User should be able to see his timeline filled with top tweets from his followers

### Non-functional requirements

* Service should be highly available – You should be always able to access twitter website, read the feed and post tweets.
* Service should be reliable – there should be no data loss. Once you post a tweet successfully it shouldn’t be lost.
* Service should have minimum latency. Consistency can take a hit for latency.

### Additional requirements

* User should be able to search tweets.
* Replying to a tweet.
* Retweeting
* Trending topics
* Sending messages

## API design
 * boolean tweet(userId, tweetText, location)
 * Json populateFeed(userId)
 * boolean follow(followId)

<b> Explanation:</b>
Here I am going to define this apis. 
* Tweet APIS return params userID and tweetText and an alternativeparam location which returns a boolean. and this tweet api used for to post a tweet and returns a value that it going to explain even if the posts is succesfully post did or not. This actually implement the basic api to twwet texts.

* populateFeed returns the params in userId which returns a JSON. It is the way which gives the value of populate the timeline data of the user. It returns the json containing the tweet data.

* follow takes a param followId which returns a boolean. The follow apis sends to a follow request to the account the followId. This can also returns a boolean to display whether we can follow the request which has been sent successfully. 

## Define data model
* User : userId(PK), name, email, creationDate, lastLogin
* Tweet : tweetId(PK), data, location, creationDate
* Follow : userId1, userId2

<b> Explanation:</b>
We are going to do a simple design to store the data. We also require to save user and tweet data. Here we also require to save the relationship between the one user which follows another user. For example, User table has a unique userID name email creationDate and lastLogin. Tweet table has a unique twwetId, the tweet text location and creationDate. As, the tweet text is of the maximum 140 characters that we can be keep by the datatype as string. 

##### The question could to which database needs to be used?

* As the data is in the form of the tables that has relationships among the users that can be think of a SQL solution. SQL would be a great good solution whether not for the huge amounts of data to be processed. 

* We also needs scalability reliability and availability that needs to be NoSQL Databases. 

* The key value needs dB, documnet dB and graph dB which not to meet the requirement in our data model. 

* A good choice will be a cloumn wide database such as Cassandra. 
* As we had huge amounts of data that we could partition the data to be used by a partition key. 

* Here we can use the userId or tweetId due to ket or the best way to go with the consistent hashing with the tweetId because of the key since tweetIds are truly random. 

#### Which database to use?

* Column-wide database like Cassandra.

## Back-of-the-envelope estimations
### Question :

* If we have 1 million tweets sent each second with average tweet size 1 KB we need to calculate how much data will be generated in next 5 years?

<b>Answer :</b>

*  1 Million * 1KB = 1000000 KB in 1 second = 1000MB in 1 second = 1 GB in 1 second
*  1GB * 60 * 60 * 24 in 1 day = 1 * 3600 * 24 = 4000 GB * 25 = 100,000 GB = 100 TB in 1 day
*  In 1 year we have 365 * 100TB = 36500 TB = 40000 TB = 40 PB
*  In 5 years we have 40 PB * 5 = 200 Petabytes.
*  Considering that we should not use more that 70% of our capacity we have 200 * 100 / 70 = 2000/7 = 300 PB
*  Reads are almost 50 times more than writes. So we will read 1 GB * 50 = 50 GB in 1 second.

## High level design
* We can consider twitter as an extension of the reader/writer design problem.

#### Challenge :

#####  Suppose a user follows 500 people out of which 200 tweet daily. What are the different ways to populate the user’s timeline.

#### Solution :

* Populate based on the tweet’s creation time.
* Use some recommendation system which is based on some parameters which suggest the top 20 tweets for the user.

  Moreover, we can except the running the recommendation system which would increase the processing time. There is a way to vercome that will be we can precompute the feed and save it in a NoSQL databse. It can also increases the storage cost but it would help us in minimising the latency which would be primary non-functional requirement. 
  <br/>
  <b>Which NoSQL Databases needs to be used to save these values? </b> 
  <br/>
  ANSWER: We can also go for the some key value that is based to databse where the key will be the userId and the value will be our precomputed feed. The access would be the fast as we can only do a lookup into whe use to call the populateFeed API. 


#### Challenge :

* Our solution works perfectly fine for normal users. 
* But for celebrity users we have a problem. They have millions of followers.

#### Solution :

* Pregenerate the feed of only normal followers.

<b>Explanation:</b>
Suppose, we had a celebrity that had 10 million followers posts a tweet. If we follow in our solution, we need to add that tweet in all our 10 million runs of the recommendation system and saved it in the timeline of all those users if the tweet can turns into one among top 20. So every time a celebrity posts a tweet we needs to recompute for his all of millions of followers. If we pregenerate the feed of only normal followers and we can append the tweets of the celebrities only when the users can log into his account. 

#### Challenge :

### How to refresh the feed?

### Solution :

* Push model
* Pull model


<b>Explanation:</b>
Here we need the feed data that is sent by the user to save in our key value database. Then, we need to pregenerate this feed data for new tweets and when the user refreshers his page that gives him the new tweets.

* In the solution we can see 2 methods, the <b>push model</b> is used for the server where we can sends data to the user as soon as new tweet is available. If we imagine that we are reading a tweet and suddenly the page refreshers without asking, so that we can see the new tweet on the top. Another reason we can see that, we do not wan tto use the push model that will be consumes unnecessary bandwidth. This is a particularly as issue if the user had expensive internet connection or he is using mobile data which is limited speed. 

* In the pull model we can see that, the user can able to manually refresh the page and ask for new data. Alternatively user might be be prompted by using a NEW POSTS button to refresh the feed if new tweets are available. 

#### Challenge :

### How do we handle infinite scrolling and loading the older tweets.

### Solution :

* Let the system compute 20 additional tweets beforehand each time we load 20 tweets.
 

<p align="center">
	
<img src="https://user-images.githubusercontent.com/57604500/125806459-41fba76c-ac91-465f-bb74-afd314e40f78.png" width=556>
<br />
<h3 align="center">👨🏻‍💻 Twitter Design</h3>
</p>



##### Explanation: 
* First functional requirements is User can able to posts the tweets. The user can tweet to sent to the writer, server which would save the tweet to the database. 

* Second functional requirements that are User can able to follow the other users. The request needs to be forwarded by the writer into server so that it would make the exact same entries into the database.

    * Since we would had many tweets and follow the requests we could able to apply single responsibities priniciple and create the separate tweet servers and follow servers. This can assist us to maintain the ability in the futuer also, we can also make a change to only tweet module that we need to test and deploy only on that module. 

    * In the diagram we can observe that, only one writer server was easy to understand as well as we do not need to get overwhelmed by the number of components that is needed. 

* Third Functional Requirements will be User can able to see their timeline filled with the top tweets from his followers. 

* In the diagram we can see the high level design. we had a reader and writer server. Here if we use a single responsibility principle we could break our writer server into 2 servers tweet and follow but for the sake of simplicity we would keep it as only one. We had our main database which saves all the data. It jas a column oriented database. Here we had a feed generator thatr picks up data from the main dB and creates the feed for the user by by running a recommendation system and saves it into our key value databse which is the feed dB. Our recommendation system for each user should run whem there are new tweets available. A write request would be com eto writer server and save the data in the main database. The feed generator can monitor the main database, find the top 20 tweets and save it in the feed database. A read request will come to the reader server, if the read requirement is to get the feed then it reads from feed database and if it has to read other things it can read directly from the main database. 

## Scaling the design
* Let us see our non-functional requirements.
<p align="center">
	
<img src="https://user-images.githubusercontent.com/57604500/125809983-439cc7eb-72da-41c3-85f8-41b03d368470.png" width=556>
<br />
<h3 align="center">👨🏻‍💻 No Single point of failure principle 👨🏻‍💻</h3>
</p>

<br/>
<p align="center">
	
<img src="https://user-images.githubusercontent.com/57604500/125810772-c08cf1c4-be5d-4317-8b72-299504614522.png" width=556>
<br />
<h3 align="center">👨🏻‍💻 No Bottleneck principle 👨🏻‍💻</h3>
</p>


* Service should be highly available – We should be always able to access twitter website, read the feed and post tweets. High availability can be achieved by no single point of failure principle.
  
Moreover, we could had replica for each of our components which can had an event failure that also can able to takeover. 

If we see in our No single point of failure diagram.

* Service should be reliable – there should be no data loss. Once you post a tweet successfully it shouldn’t be lost. This can be achieved by saving redundant copies of data.

* Service should have minimum latency. Consistency can take a hit for latency – It is perfectly fine if we post a tweet and it takes up some time to show it in our followers feed i.e. there is eventual consistency.

Moreover, while doing the high level design we can introduced a feed generator to reduce latency. We can also use the no bottleneck priniciple that helps us to reduce our latency. We can see this in our no bottlenecks principle diagram. As part of this priniciple we would able to increase the number of the servers to handle the requests. We know that the amount of read and write requests we can get per second. This can procure the appropriate amount pf the servers to handle the data. Beside this, we can also introduce the caching by considering the 80:20 priniciple where the 20% data is responsible for 80% of our reads. Here we can also cache the feeds and celebrity tweets to increase the performance of our system. As we can also observe that, we have a feed cache as well as a main database cache to improve the performance. 

* We can also add caching for faster reads.
* No single point of failure principle
* No single point of failure principle
* No bottleneck principle
