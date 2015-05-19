## Introduction

For a project I had done recently, I was required to create a distributed application using NodeJS which exposed a bunch of REST APIs which could be used by clients. Since the application required large amount of persistent data, I added [MySQL](https://www.npmjs.com/package/mysql "npm MySQL") to the [EARN stack](https://www.airpair.com/express/posts/earn-stack "EARN Stack") and used [RabbitMQ](https://www.rabbitmq.com/ "RabbitMQ") for queuing the messages. 

##Problem
I wanted to optimize the application by caching the data such that the response time for the REST calls was less and that the solution should be generic and non-intrusive so that I don’t have to handle it each time I added a new module to the application.

##Hurdle
The application was storing 2 types of data: 
- First was data related to entities/actors of the application like profile details of clients or details of the products etc. 
- Second was the data that resulted because of relation between two or more entities/actors or because of other attributes of the entities/actors. 

For this post, let's call the former *Simple* Data and latter *Complex* data.

## Solution
### Simple Data
For *Simple* data, I used Redis for caching and stored data in a HashMap with the *id* as key and other attributes as field-value pairs. Since data access for *simple* data was done using *id* throughout the application, extracting details from cache became very simple and gave a good improvement in terms of response time.

### Complex Data
For *Complex* data, I was not able to identify correct key for storing in the cache as there can be multiple ways in which it can be accessed and there is no universal identifier for the same (as was for *simple* data). So for these, I decided I would use SQL caching:

Now the first way I found was to use SQL caching provided by MySQL server itself. Although this technique was effective, generic and didn’t require any changes in the code, it still required a trip to the MySQL server to fetch the results.

So to avoid that trip to MySQL server, I thought of implementing a query cache of my own using Redis and it's HashMap. The idea was to store the complex SQL query as the **key** (which becomes the unique identifier) and the result as it's **value** in a HashMap so that the next time that query is fired from the application, the result can be fetched directly from the Redis database instead of making a trip to the MySQL server. Also, to make this a generic and non-intrusive implementation, the cache operations were done by creating a MySQL helper function and all the database operations where done using that function only.


## Conclusion
Since the idea was clear, the implementation became very clear and I had to just take care of some exceptional cases:
1. Cache only the *SELECT* queries
2. Do not cache Login/Authentication related queries
3. Have a defined cache expiration strategy and test it thoroughly. *(very important!)*

The above strategy not only gave me a better response time, it also reduced a number of trip to MySQL server which I think in case of a distributed application environment is a big plus.

####Happy Coding!