# Part 1. Foundations of Data Systems

Let's think about how to design a data system :)

## Chapter 1. Reliable, Scalable, and Maintainable Applications

Many applications of this kind need to:
  Store data => databases
  Remeber the result of an expensive operation => caches
  Allow users to search data by keyword or filter => search indexes
  Send a message to another process, to be handled asynchronously => ***stream processing***
  Periodically crunch a large amount of accumulated data => *** batch processing ***
 
One possible architecture for a data system that combines several components.
![Architecture](https://github.com/pysherlock/Designing-Data-Intensive-Applications-Note/blob/master/image.png)

### Reliability, Scalability, Maintainability

1. Reliability

Netflix Chaos Engineer: increase the rate of faults by triggering them deliberately, so that the fault-tolerance machinery is continually exercised and tested.

Faults:
  * Hardware Faults: ***Hard disks crash, RAM becomes faulty, the power grid has a blackout, someone unplugs the wrong network cable.***
  
  As long as you can restore a backup onto a new machine fairly quickly, the downtime in case of failure is not catastrophic in most applications.
  multi-machine redundancy => applications for high availability
  
  Amazon Web Services is designed to prioritize flexibility and elasticity over single-machine reliability, so it is common that one VM become unavailable without warning
  
  A system which can tolerate machine failure can be pathched one node at a time, without downtime of the entire system
  
  * Software Errors: 
  
  More failures, hard to anticipate. Like bugs, using up shared resource
  
  * Human Errors: caused by operators
  
2. Scalability

System's ability to cope with increased load

  * Describing Load
 Load parameters: parameters should be considered while increasing the load
  
Example: Twitter

a. When a user requests their home timeline, look up all the people they follow, find all the tweets for each of those users, and merge them (sorted by time)
![Simple relational schema for implementing a Twitter home timeline](https://github.com/pysherlock/Designing-Data-Intensive-Applications-Note/blob/master/tweet1.jpg)

b. Maintain a cache for each user’s home timeline—like a mailbox of tweets for each recipient user. 
When a user posts a tweet, look up all the people who follow that user, and insert the new tweet into each of their home timeline caches. The request to read the home timeline is then cheap, because its result has been computed ahead of time.
![Twitter’s data pipeline for delivering tweets to followers](https://github.com/pysherlock/Designing-Data-Intensive-Applications-Note/blob/master/tweet2.jpg)

However, the downside of approach 2 is that posting a tweet now requires a lot of extra work. Some users have over 30 million followers. This means that a single tweet may result in over 30 million writes to home timelines
Tries to deliver tweets to followers within five seconds - is a significant challenge.

The final twist of the Twitter anecdote: 
  Most users' tweets continue to be fanned out to home timelines at the time when they are posted. but a small number of users with a very large number of followers are excepted from this fan-out.
  Tweets from any celebrities that a user may follow are fetched separately and merged with the user's home timeline when it is read, like in approach 1.

  * Desribing Performance
 Batch processing: Throughput - the number of records can be processed per second
 
 Online systems: service's response time. Average is not a good metric for response time. Use ***percentiles***, ***median***
 
 ***p50***: half of user requests are served in less than one thershold response time. The threshold is p50
 p95: 95%, p99: 99%
 
***Tail latencies***: high percentiles of response times are important because they affect users' experience
For example, the customers with slowest request are often those who have made many purchases => most valuable customers
 
***Queueing delays***: small number of slow requests hold up the processing of subsequent requests => ***head-of-line blocking***. 
  So Even if those subsequent requests are fast to process on theserver, the client will see a slow overall response time due to the time waiting for the prior request to complete.
  
3. Maintainability
