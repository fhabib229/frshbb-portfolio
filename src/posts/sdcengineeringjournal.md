---
title: "System Design Journal"
date: "2019-04-30"
path: "/sdc-engineering-journal"
coverImage: "../images/sdc/diagrams/sdc-standings-diagram-3.png"
author: "Faris Habib"
excerpt: "Engineering Journal for the System Design Capstone Project."
tags: ["sdc", "system design capstone", "blogs"]
---

# System Design Capstone Engineering Journal

## Entries

1. [February 4th, 2019](#1)
2. [February 6th, 2019](#2)
3. [February 11th, 2019](#3)
4. [February 12th, 2019](#4)
5. [February 16th, 2019](#5)
6. [February 24th, 2019](#6)
7. [March 3rd, 2019](#7)
8. [March 14th, 2019](#8)
9. [Choosing a Database](#databases)

---
## 1

### February 4th, 2019

**1. Choosing FEC Project and Service**
- The chosen project for our SDC group is the ESPN NFL Team Page. I'll be working on the standings service that was originally made by Kevin Phung. This service renders the NFL Team standings within the different divisions of the league. There are two components to this service that are rendered to the page:

  - **Full Standings Component**: This component will render all of the NFL team standings within each sub division of the league. This component isn't rendered onto the main page but is accessed through the standings component. Sub divisions are hard coded, but the information from the database that is rendered to this page include:

    - Team name
    - Team logo
    - Wins
    - Losses
    - Ties
    - Percentage
    - Points for
    - Points against
    - Difference

  ![full standings][one]

  - **Standings Component**: This component renders one sub division of the Full Standings Component to the main page, which is always the NFC WEST division. The information from the database that gets rendered is the same as the Full Standings Component.

  ![standings][two]

**2. Choosing SQL DBMS**
- For my SQL DBMS choice I've chosen to implement a PostgreSQL database. I archived all of Kevin's MongoDB files with an underscore extension in case I'll need it again. Implementing the PostgreSQL was not difficult due to my previous knowledge in implementing one during the FEC project for my service. To utilize PostgresSQL I installed it with homebrew and used the PG client paired with KnexJS and Bookshelf to implement the database easily. I tested an insertion of the same mock data that Kevin created with my new database and it worked successfully. The next step will be to create a script that can insert 10 million records.

**3. Generating 10,000,000 records**
- I've completed the seed script that will execute the records to my SQL database. In summary I created a function that generates the data using a for loop. The only parameter supplied is how many records are to be generated. For quick record generation I used faker, but I'll need to edit the types of images that are supplied for the team logo because faker doesn't have a good method to supply team logo images. What I'll probably end up doing is hosting the sports team logos on S3 and randomize the selection so essentially each record looks more like a sport team. The number of wins,losses, and ties generated by faker also seem extreme so I'll probably tweak this later too.

- Generating 1 million records proved to be a success. It took about 2 minutes and 21 seconds to complete. At about 1.5 million records is when I get an allocation failure. I've copied the full error message below for reference.

```
<--- JS stacktrace --->

==== JS stack trace =========================================

Security context: 0xfc4822cf781 <JS Object>
    2: randomWord [/Users/fh/sdc/kevin-services-standings/node_modules/faker/lib/random.js:~116] [pc=0x30e249a6abcb] (this=0x1844f5d85be1 <a Random with map 0x1897ee7ef459>,type=0xfc482204381 <undefined>)
    3: arguments adaptor frame: 0->1
    4: generateDataSet [/Users/fh/sdc/kevin-services-standings/database/seeds/seed.js:~11] [pc=0x30e249a74b29] (this=0x22b8e5698c19 <JS Global Object>,numOf...

FATAL ERROR: CALL_AND_RETRY_LAST Allocation failed - JavaScript heap out of memory
 1: node::Abort() [/Users/fh/.nvm/versions/node/v6.13.1/bin/node]
 2: node::FatalException(v8::Isolate*, v8::Local<v8::Value>, v8::Local<v8::Message>) [/Users/fh/.nvm/versions/node/v6.13.1/bin/node]
 3: v8::internal::V8::FatalProcessOutOfMemory(char const*, bool) [/Users/fh/.nvm/versions/node/v6.13.1/bin/node]
 4: v8::internal::Factory::NewFixedArray(int, v8::internal::PretenureFlag) [/Users/fh/.nvm/versions/node/v6.13.1/bin/node]
 5: v8::internal::TypeFeedbackVector::New(v8::internal::Isolate*, v8::internal::Handle<v8::internal::TypeFeedbackMetadata>) [/Users/fh/.nvm/versions/node/v6.13.1/bin/node]
 6: v8::internal::(anonymous namespace)::Ensurefactor: test 10m records. Fails at 1.5m
reFeedbackVector(v8::internal::CompilationInfo*) [/Users/fh/.nvm/versions/node/v6.13.1/bin/node]
 7: v8::internal::(anonymous namespace)::GenerateBaselineCode(v8::internal::CompilationInfo*) [/Users/fh/.nvm/versions/node/v6.13.1/bin/node]
 8: v8::internal::(anonymous namespace)::GetUnoptimizedCodeCommon(v8::internal::CompilationInfo*) [/Users/fh/.nvm/versions/node/v6.13.1/bin/node]
 9: v8::internal::Compiler::Compile(v8::internal::Handle<v8::internal::JSFunction>, v8::internal::Compiler::ClearExceptionFlag) [/Users/fh/.nvm/versions/node/v6.13.1/bin/node]
10: v8::internal::Runtime_CompileLazy(int, v8::internal::Object**, v8::internal::Isolate*) [/Users/fh/.nvm/versions/node/v6.13.1/bin/node]
11: 0x30e2493092a7
```
- I'll need to perform some fine-tuning with my seed script. One idea I have is turning the seed function into an async function so that records are inserted one at a time, but this would require me to refactor the code that uses knex's bulkInsert method, which I find to be helpful overall. I'll need to play around some more to figure out how I can insert 10 million records in about 50 minutes.

- I successfully completed the insertion of 10m records! The official time is 24min8s56ms. They key was chaining knex's batch insert in promise format.

```
exports.seed = knex => knex('standings').del()
  .then(() => knex.batchInsert('standings', generateDataSet(1000000), 1000)
    .then(() => knex.batchInsert('standings', generateDataSet(1000000), 1000))
    .then(() => knex.batchInsert('standings', generateDataSet(1000000), 1000))
    .then(() => knex.batchInsert('standings', generateDataSet(1000000), 1000))
    .then(() => knex.batchInsert('standings', generateDataSet(1000000), 1000))
    .then(() => knex.batchInsert('standings', generateDataSet(1000000), 1000))
    .then(() => knex.batchInsert('standings', generateDataSet(1000000), 1000))
    .then(() => knex.batchInsert('standings', generateDataSet(1000000), 1000))
    .then(() => knex.batchInsert('standings', generateDataSet(1000000), 1000))
    .then(() => knex.batchInsert('standings', generateDataSet(1000000), 1000)));

```

- I met with my team member Amit and we had discussed how chaining insertions might be the key. This is because it's hard to hold so many records in any one function. I knew that the batch insert utility could insert 1m records in 1000 chunks, so I decided to try chaining ten batch inserts in that same format and success!

- With 10 million records in the database I had to make adjustments to my server code so that the API call would not fetch all 10 million records. To do this I went through Bookshelf's documentation and found that the collection had a couple methods I could utilize. In summary I had to make sure I was performing the query below each time I was making an API request for the data:

```
  SELECT * FROM STANDINGS ORDER BY ID DESC LIMIT 100;
```

- Luckily Bookshelf has the OrderBy and Query methods to help structure the API call on the Standings collections to mimic the query above.

- Work for generating 10 million records is for the most part done. Now I'll be conducting some benchmarking tests on the database before I start with my NoSQL Database.
---

## 2

### February 6th, 2019

**1. Initial Benchmarking of SQL Database**

- One of our objectives in Phase 1 is performing some simple benchmarking tests to measure the initial performance of our databases. I accomplished this by measuring 2 aspects of my SQL database:

  - **1. Measure and record the time it takes to insert 10,000,000 records**
    - This was done by creating one date function before the seed function ran, another after it ran, and calculating the difference in time between the two in milliseconds. I've posted the results below.

  ![benchmark records][three]

  - **2. Measure and record time it takes to execute a simple read query**
    - I used one of PostgreSQL's built in functions to analyze the time it took to randomly select one of 10,000,000 records. The first initial results are unfortunate as it took about 19 seconds to perform a simple read query of 1 record. The goal is to achieve a read query speeds of 50ms, so I have some work to do to try to minimize my initial time.

  ![benchmark query][four]

---

## 3

### February 11th, 2019

**1. Refactor Database to Cassandra**

- The next step of phase 1 was to implement a NoSQL DBMS for our chosen project. Since MongoDB was the original NoSQL DBMS we chose Cassandra.

- Implmenting Cassandra came with quite a few challenges, the lack of clear documentation on how to implement Cassandra with Express/NodeJS being one of them. One of the first errors I encountered while setting up Cassandra was a thrift socket error:

![cassandra thrift socket error][five]

- When loading Cassandra it will create a thrift socket protocol. Supposedly this is a type of interface that was originally used with Cassandra, but the standard nowadays is CQL or the Cassandra Query Language. Why Cassandra stil lloads this I'm not sure as I already installed CQL with the Python package manager. After some trial and error I was able to resolve this issue by installing an older Java SDK (version 1.8). I had other issues running Cassandra that weren't resolved by simply running ```brew services start cassandra``` after installation. In order to resolve these issues I had to remove lines from the ```cassandra-env.sh``` file after running ```cassandra -f```. Luckily these got resolved quickly.

- The second major issue I had when finishing installation was running CQL for the first time. By default, CQL seems to connect to ```localhost 9042``` but I ran into issues connecting to this port. After searching for solutions on the internet I tried connecting to CQL using port 9160.

![cassandra cql port error][six]

- Since I'm not using the standard port I had to think about how to make my config file connect to the right port too. Luckily with the installation of Cassandra-Driver there was a helpful method to do just that.

- I was able to succesfully insert mock data and render it to the front end. The next step is to insert 10 million records.

**2. Seeding 10,000,000 records and Benchmarking**

- Inserting 10,000,000 records had it's own set of challenges. First was figuring out the right format to write my insertion function so that I wouldn't end up with a heap memory allocation error like I did with my Postgres database. I ended up utilizing the driver's built in batch function to insert multiple records. Unfortunately the batch function could only insert 125 records at a time, so I chained several batch methods together in promises so I could insert 20,000 records at a time. From there I looped the insertion to run 500 times so that I could insert all 10m records. The first warning I encountered with this was a timeout warning:

```
(node:16353) UnhandledPromiseRejectionWarning: Unhandled promise rejection (rejection id: 1): OperationTimedOutError: The host 127.0.0.1:9160 did not reply before timeout 12000 ms
```

- Luckily this isn't fatal, and I modified my timeout settings in the ```cassandra.yaml``` file to make sure it wasn't. It took some time but I was able to successfully insert 10 million records in about 21 minutes.

- The simplest way to check for 10,000,000 records is to perform a count function in CQL, but unfortunately performing such a function is not possible because of timeout issues:

```
cqlsh:espn> select count(uid) from standings;
ReadTimeout: Error from server: code=1200 [Coordinator node timed out waiting for replica nodes' responses] message="Operation timed out - received only 1 responses." info={'received_responses': 1, 'required_responses': 1, 'consistency': 'ONE'}
```
- In order to confirm whether I had 10 million records or not, I added a logging statement to the end of my batch insertions of 20,000:

```
const startTime = new Date().getTime();
...
.then( ()=> db.batch(generateBatch(), { prepare: true})) // 20,000
.then(() => console.log(`Successfully inserted 20,000 records in ${new Date().getTime() - startTime}ms`));
```

- Since the chain of batch insertions creates 20,000 records, and the loop runs 500 times, I copied each logging statement and pasted them into an Excel sheet and summed up the rows. This confirmed that I had 10,000,000.

![cassandra 10m record confirmation][seven]

- The logging statements also served as my benchmark. At every 20,000 records, a timestamp is calculated to figure out how many milliseconds the insertion of 20,000 took. Looking at the last insertion shows a time of 1274791ms or about 21 minutes.

![cassandra benchmark][eight]

- Although this method is crude, I was able to insert the required amount of data in less than 50 minutes. The next step will be to fine-tune Cassandra to see if I can reduce that time and make my seeding code look cleaner. I'll most likely deploy my PostgerSQL and Cassandra databases to AWS EC2 so that I can perform more tests to see which comes out faster. At this point it's about the same, PostgresSQL is only about 5 minutes faster (but the code is cleaner!).

---
## 4

### February 12th, 2019

**1. Adding CRUD support for Primary Database**

- We've decided to set our primary database to be Postgres. One of the last tasks for Phase 1 is to add basic support for CRUD (Create, Read, Update, Delete) operations. This took less than an hour to implement since we didn't have to worry about adding client-side functionality to support these CRUD operations. For the full rundown on how the CRUD operations are written out for my service check out the README that's linked below.

  https://github.com/fhabib229/kevin-services-standings/blob/crud/server/CRUD.md

- The next step is to deploy the primary database to AWS EC2, though as I mentioned yesterday, I'll be deploying both databases to EC2 to perform additional tests. Before I do that I'm going to try different methods of inserting 10m records with Postgres and Cassandra, specifically inserting the data with csv files. My team has reported quick speeds for this method so I'm looking forward to implementing it for my own service and performing additional tests to confirm this.

---
## 5

### February 16th, 2019

**1. Modifying Cassandra Benchmark**

- I did a quick modification of the insertion script I used to get a more accurate benchmark of how long it takes to insert 10,000,000 records. Before I addeed 500 logging statements and pasted them into Excel to tally up if I had the right amount of records. Now I've added a counter variable that keeps track of when each batch of 20,000 records has been added. At the final batch of 20,000 (when count = 500) the script will print a message to the terminal showing the time it took to insert all 10m records. Removing the previous logs and only adding 1 reduced my data insertion time by about 4 minutes, bringing the total time to about 18 minutes.

  ![cassandra final benchmark][nine]

- Not bad at all. I probably won't be playing with csv insertions with Cassandra. My other team members did so to get all 10m records in in a shorter amount of time, but I'm happy with leaving my Cassandra database where it is at the moment since this is not the primary database choice.

- The next step is to perform some simple csv insertions with my Postgres Database since this will be my primary database choice, then I'll deploy the database, service, and proxy server to EC2.

---
## 6

### February 24th, 2019

**1. Data Insertions w/ CSV into Postgres**

- Insertion of 10,000,000 records to my primary database with a csv file did not go smooth. I was able to insert 1,000,000 records into the database in less than a minute, but adding additional lines to the csv file just created heap allocation memory errors. I also received timeout errors connecting to the Postgres Database through KnexJS while configuring my csv script.
  ![postgres csv errors][ten]
- After spending about 2-3 hours trying to solve my errors, I decided to give up on trying to insert 10m records through a csv file. I already have a working method with batch insertions, so I'll stick with that when I move to deployment and stress-testing.

**2. Deploying Service + Database to EC2**

- Deploying to AWS EC2 wasn't too difficult after the FEC project. Although we didn't explicitly set out to use EC2 in FEC, we used it as a biproduct of deploying our project to Elastic Beanstalk. This experience already made me more familiar with understanding how AWS works and what I could expect from launching my EC2 instance for the Standings service.

- Initially I created an instance with 30 GiB of storage and 100 IOPS and chose an instance with Ubuntu. Installing npm and node was relatively easy with the commands below:

```curl -sL https://deb.nodesource.com/setup_11.x | sudo -E bash -```
```sudo apt-get install -y nodejs```

- After that I had to figure out how to deploy postgres to my instance. At first I thought I could use homebrew but quickly discovered that it's easier to install postgres through Ubuntu's APT repositories. I logged in as the root user and ran the following command below to install:

```apt-get install postgresql postgresql-contrib```

- Then I cloned my service's repo using git and installed all dependencies. I ran into my first set of errors when trying to start my server in my instance.

  ![ec2 server error][eleve]

- After researching the issue I discovered that I had to reconfigure the ports in my Knex configuration files to match the ports that could be accessed through my EC2 instance.

```
//knexfile.js
module.exports = {
  development: {
    client: 'pg',
    connection: {
      host: '0.0.0.0', // ---> This was changed from localhost to 0.0.0.0
...
```

```
// config.js
const knex = require('knex')({
  client: 'pg',
  connection: {
    host: '0.0.0.0', // ---> This was changed from localhost to 0.0.0.0
...
```

- After fixing this I moved onto seeding my 10m records. This proved to be the trickiest part of deploying my instance as there were several memory allocation and timeout errors from trying to insert 1m records at a time.

  ![ec2 seeding errors][twelve]

- I quickly realized that generating 10 million unique records and inserting them all at once was not a good idea. So instead I resolved to generate 100,000 unique records and insert them 100 times, giving me 10 million records into my database but only 100,000 of which were unique. This is doable for the standings service as only 32-100 records actually get pulled to the API and rendered to the front end. This way I can still stress test my database and insert 10 million records in a timely manner. Lastly I tried increasing the volume to 500 GiB and 1000 IOPS out curiosity to see if it would insert the records faster. My final time was about 4 minutes and unfortunately increasing the volume size had little to no effect on this as I was able to create another instance with 30 GiB and 100 IOPS and still insert the records in 4 minutes.

  ![ec2 10m records][thirteen]

---
## 7

## March 3rd, 2019

**1. Stress Testing Service in Development**

- In phase 3 our goal is to stress-test our services with our primary database and 10 million records inserted. This can be accomplished by using New Relic to monitor the application and with other utilities like Artillery for load testing and functional testing.

- Setting these tools up doesn't take much work. Once I signed up for a New Relic account it's as simple as creating a configuration file in the root directory of my service and letting New Relic listen for the deployment. I used Artillery to run quick load tests too. An important note to keep in mind is that all stress tests in development are done locally through my computer, so the mileage on stress-testing performance may vary if my teammates stress test my service on their machine.

- Once New Relic was set up I ran a simple Artillery command to perform some initial benchmarks of the standings service under load. The first test was to see how the latency and RPS looked with 10 users and 1 GET request per user.

  ![1st stress test w/ Artillery][fourteen]

- As you can see the initial numbers are promising. With 10 users each sending 1 GET request to the service the median RPS turned out to be 3.8ms. These numbers were trivial so I cranked up the number of users...

  ![2nd stress test w/ Artillery][fifteen]

- The test above is an attempt to increase the number of users to 1000 and with 1 GET request per user. The numbers here are still promising with a median latency of 1.9ms. It's important to get a realistic number of users expected to login to the ESPN standings page to view the Los Angeles Ram's stats, so I decided to increase the number of users to 10,000...

  ![3rd test w/ Artillery][sixteen]

- This is when I encountered several error statements showing up in the server. When researching this error I found out that the number of files were greatly exceeding the open file limit on my Macbook. This is when I decided to decrease the number of users and increase the number of GET requests per user.

  ![4th test w/ Artillery][seventeen]

- The image above was a stress test of about 2500 users each requesting the service 25 times for a total of 62,500 scenarios. After failing at about 10,000 users I wanted to get as close to a realistic number of users as possible, so I brought it down to 5,000 users...then 4,000...3,000...and finally 2,500. My assumption is that users won't attempt to access the standings page for their team that many times, so I tried to keep the number of GET requests per user to about 20-25. Unfortunately I had to keep bringing the number of users down because of the open file limit on my machine. There may have been a way to increase the file limit size via the CLI, but I figured in a real scenario developers would probably be using better hardware to stress-test a similar service. Although, what's great about this image is that it shows a successful attempt at about 1,000 RPS which was the goal of this phase.

  ![new relic dashboard][eighteen]

- Pictured above is the throughput, error rate, and response time of the standings service while running Artillery load tests. You can see that the highest response time was at about 638ms per request and a 1 minute response time at about 8:10pm. This was when I was trying to configure the right number of users and requests per user, at this point I was attempting 5,000 users at about 20 requests per user. Other notable stats include an average throughput of about 2.75k rpm and an average error rate of about 0.0040%.


**2. Stress Testing in AWS**

- Stress testing in EC2 was about the same as stress testing in development, except I used Loader.io to facilitate the tests to my service while deployed in EC2. The results of the tests were surprising, I was able to successfully send 10,000 clients to the service in 1 minute with an average response time of about 81ms.

  ![loader.io graph][nineteen]

- The New Relic dashboard showed interesting results as well. The throughput was abour 943rpm, the error rate less than 1%, and the highest response time at 81ms, mirroring what loader.io showed.

  ![new relic ec2][twenty]

---

```
`Unhandled rejection Error: connect ECONNREFUSED 172.31.27.205:5432
at TCPConnectWrap.afterConnect [as oncomplete] (net.js:1083:14)
```

---
## 8

### March 14th, 2019

**1. Scaling The Service**

- In my last entry I claimed that I was able to send 10,000 clients to the service in 1 min with a latency of about 81ms. This is not accurate with as explained by the requirements for the project. When I ran my stress-tests in Loader.io I selected the "clients per test" test type, which means I wasn't measuring in requests per second(RPS).

- I switched the test type in Loader to "clients per second". Turns out I could only pull about 2000rps before my service errored out of the test entirely. I had to think of a way to scale my service so I could achieve 10,000 RPS, as that was the goal to shoot for in the project requirements. The most efficient way to scale the service that came to mind, would be to add additional instances of my service in EC2 and apply a load balancer to adjust the site traffic accordingly. The first step in doing this was to split my service and database into separate instances. I took this time to figure out how to create a Docker container too so I could replicate my instances easily.

**2. Splitting the Service and Database**
- Splitting up the service and database was trivial for the most part. I created another instance and setup my repo to get started. Technically I didn't need my repo in the database instance, but I was not sure what was needed in my first attempt to split my database. The only part I needed to install in my database instance was PostgreSQL and this could be done by running a simple Ubuntu command:

  ```sudo apt-get install postgresql postgresql-contrib```

- From there I just had to make sure that postgres services had started:

  ```service postgresql start```

- Next I copied the URL of my database instance and pasted it to my Knex configuration file so that my service could connect to the database.

  ```
  ...
  production: {
    client: 'postgresql',
    connection: {
      host: 'ec2-34-219-142-73.us-west-2.compute.amazonaws.com',
  ...
  ```

- The only issue I ran into was when the service had attempted to connect to the database, running into permission issues.

  ![Permission errors in database instance][twenty-one]

- I fixed this by editing the ```postgres.conf``` and ```pg_hba.conf``` files that came with installing postgres by allowing access to all users attempting to connect to the database with an encrypted password.

```
// postgres.conf file
listen_addresses = '*' // added to the end of the file
```
```
// pg_hba.conf file
host  all  all 0.0.0.0/0 md5 // added to the end of the file
```
**3. Implementing Docker Images and a Load Balancer**

- One of the goals for this project was learning how to create Docker containers. Initially I read through the dense documentation provided by online by Docker, and practiced implementing a container by following their "Getting Started" guide. I was hoping to dig deeper into docker to try and figure out how to have the container not only start my service and connect to the database, but to also seed my database; This could be done by creating a compose file. Unfortunately this proved to be time-consuming to understand and implement, so I decided to stick with the container having the ability to start my service and connect to the database.

- The minor issue I ran into was which port to expose within the container, as it couldn't be the same as the port number I was using to connect the service externally. I decided to expose my service to port ```4001``` and map that to port ```4000```.

- Next I implemented a load balancer through the EC2 dashboard to initialize multiple instances and scale my service. With my service in a Docker container all I had to do was create each instance and pull the service from Docker Hub. I could've automated this process further by utilizing AMI and Auto-Sclaing through EC2, but this would've made me forfeit my services in docker containers. Forfeiting my docker containers would've been better for performance and therefore was the smart way to go as all my other teammates had done, but I was too proud of the work I had accomplished with Docker.

  ![Creating a load balancer through ec2][twenty-three]

  ![Pulling docker to ec2 instance][twenty-two]

**4. Scaling the Service and Stress Testing**

- Up next was to scale my service by figuring out how many instances I needed to create in order to obtain 10,000 RPS. I started with 2, then 3, and so on until I discovered that 6 instances was enough to obtain 10,000 requests without terrible latency and errors.

  ![terminal of my 6 ec2 service instances and the database instance][twenty-four]

  ![successfully achieved 10krps with Loader test][twenty-five]

**5. Stress Testing the Proxy, Part II**

- With our services properly scaled to handle 10,000RPS each we attempted one more stress-test of our proxy to see if we could get 10,000RPS. Unfortunately the proxy could only handle about 1,500RPS before it errored out. We discovered that the most time-consuming services were my own and Amit's, but with the minimum requirements met and our presentation deadline approaching soon we stopped all further testing of the proxy.

  ![The most time consuming services according to New Relic][twenty-six]

---

## Databases

**Choosing a Primary Database**

- When comparing my RDBMS (PostgreSQL) and NoSQL (Cassandra) choices for this project it was not clear which would serve my service well as my primary database. I conducted some research to compare the two databases and see where each had their strengths and where they fell short.

- **PostgreSQL**

  - PostgreSQL is one of the most popular choices when it comes to RDBMSs. One of its strengths is in its transaction support. You can think of a transaction like some amount of work being done on a database. Often databases need to provide work that will allow recovery from database failures and allow for consistency when executions on that amount of work stops. One example of where this is important is in money transfers from one bank to another. A database with strong transaction support will be able to accurately store the correct amounts from each bank as it transfers the amount from one to the other even when there is a system failure. Postgres is also ACID (atomicity, consistency, isolation, durability) compliant, which means that all transactions conducted in Postgres are completed in a timely manner. Postgres and other relational engines are optimized for reading data by keeping it in memory. This means applications that are read-intensive could perform well on a RDBMS.

  - Where Postgres falls short is in scaling for big data (Though this is technically no longer the case with Postgres XL). Most RDBMS cannot exceed the capacity of a single node (which is about 1TB or less). This makes it fine for most web applications but it will fall short when dealing with a mass volume of data. PostgreSQL is also not suitable for write-intensive applications like Facebook.

- **Cassandra**

  - Cassandra is one of the top choices when it comes to NoSQL databases. Cassandra is known as a wide-column store database, which means that data in each row can vary and do not have to be structured the same way. Cassandra is designed to scale beyond a single node which makes it perfect for storing large amounts of data (1 PB or more). It achieves this by handling the swaths of data across multiple nodes or clusters, in a parallel computing structure making it so that each node is not a single point of failure.  Cassandra is also optimized for writing data, so social media applications are a perfect fit for Cassandra as many users can create posts to the application without a single hiccup.

  - Since Cassandra is a NoSQL DBMS it falls short in having normalized data. This means you can't have JOIN tables so an applications would have to make multiple queries to the database in order to obtain the information it needs. Cassandra also lacks certain abilities that RDBMS have like transforming data with a SUM() function.

- **Summary**

  - Either database would work well for my application. Since the application only runs on a single node and contains no more than 2-5 GB of data with no heavy writing operations I chose to have PostgreSQL as my primary database with Cassandra as my secondary choice.



---
[one]: ../images/sdc/images/fullstandingscomponentsnapshot.png
[two]: ../images/sdc/images/standingscomponentsnapshot.png
[three]: ../images/sdc/images/benchmark1.png
[four]: ../images/sdc/images/benchmark2.png
[five]: ../images/sdc/images/cassandraerror1.png
[six]: ../images/sdc/images/cassandraerror2.png
[seven]: ../images/sdc/images/cassandrabenchmark2.png
[eight]: ../images/sdc/images/cassandrabenchmark1.png
[nine]: ../images/sdc/images/cassandrabenchmark3.png
[ten]: ../images/sdc/images/postgrescsv.png
[eleven]: ../images/sdc/images/ec2servererror.png
[twelve]: ../images/sdc/images/ec2seederrors.png
[thirteen]: ../images/sdc/images/ec210mrecords.png
[fourteen]: ../images/sdc/images/artillery1sttest.png
[fifteen]: ../images/sdc/images/artillery2ndtest.png
[sixteen]: ../images/sdc/images/artillery3rdtest.png
[seventeen]: ../images/sdc/images/artillery4thtest.png
[eighteen]: ../images/sdc/graphs/newrelicdashboard.png
[nineteen]: ../images/sdc/graphs/loaderec2.png
[twenty]: ../images/sdc/graphs/newrelicdashboardec2.png
[twenty-one]: ../images/sdc/images/splittingdatabaseerror.png
[twenty-two]: ../images/sdc/images/dockerimagec2.png
[twenty-three]: ../images/sdc/images/ec2lb.png
[twenty-four]: ../images/sdc/images/ec27instances.png
[twenty-five]: ../images/sdc/images/10krpsstandings.png
[twenty-six]: ../images/sdc/images/timeconsumingservicesfinal.png