The Data Engineering Project
Objective
The project phase is intended to allow you to showcase some of the skills and knowledge you have acquired over the past few weeks. You will create applications that will Extract, Transform and Load data from a prepared source into a data lake and warehouse hosted in AWS. Your solution should be reliable, resilient and (as far as possible) deployed and managed in code.

By the end of the project, you should have:

written a number of applications in Python that interact with AWS and database infrastructure and manipulate data as required
remodelled data into a data warehouse hosted in AWS
demonstrated that your project is well monitored and that you can measure its performance
deployed at least some of the project using a scripted solution.
Your solution should demonstrate your knowledge of Python, SQL, database modelling, AWS, good operational practices and Agile working.

The Minimum Viable Product (MVP)
The project is open ended and could include any number of features, but at a minimum you should seek to deliver the following:

Two S3 buckets (one for ingested data and one for processed data). Both buckets should be structured and well-organised so that data is easy to find.
A Python application that ingests all tables from the totesys database (details below). The data should be saved in files in the "ingestion" S3 bucket in a suitable format. The application must:
operate automatically on a schedule
log progress to Cloudwatch
trigger email alerts in the event of failures
follow good security practices (for example, preventing SQL injection and maintaining password security)
A Python application that remodels data into a predefined schema suitable for a data warehouse and stores the data in parquet format in the "processed" S3 bucket. The application must:
trigger automatically when it detects the completion of an ingested data job
be adequately logged and monitored
A Python application that loads the data into a prepared data warehouse at defined intervals. Again the application should be adequately logged and monitored.
All Python code should be thoroughly tested.

As much as possible of the project should be deployed via a scripted solution. The deployment scripts can be written as bash or Python code.

You should be able to demonstrate that a change to the source database will be reflected in the data warehouse within 30 minutes at most.

Enhancing Your Product
The MVP can be enhanced in a number of ways.

You can use GitHub Actions to deploy and update the infrastructure automatically when you commit code.
You can maintain a schema registry or data catalogue, which contains the schema of the data you ingest from the database. Using this, you can check that incoming data has the required structure. If there is any anomaly (eg the database has been changed in some way), you can perform a failure action, such as redirecting the data to some sort of default destination (sometimes called a dead letter queue).
Refactor your code to use the SQLAlchemy library to interact with the database in an object-oriented way.
Possible Extensions
There are a number of ways to extend the project.

Ingest data from a file source - eg another S3 bucket. We can provide JSON files in a remote S3 bucket that can be fetched at intervals.
Ingest data from an external API - eg you could retrieve relevant daily foreign exchange rates from https://freeforexapi.com/Home/Api. You can use the requests library to make the request and then save the results in S3.
Set up an API server to provide a real time report on payments for the finance department. The API could have the following endpoints:
/payments/last5 # gets last five scheduled payments, their amounts and counterparty
/payments/next5 # gets next five scheduled payments
The flask library is the rough equivalent of the Node Express package. An alternative is fastapi. You will need to host the API on an EC2 instance. Obviously (!), all infrastructure should be deployed as code...

Technical Details
The primary data source for the project is a moderately complex (but not very large) database called totesys which is meant to simulate the back end data of a commercial application. Data is inserted and updated into this database several times a day. (The data itself is entirely fake and meaningless, as a brief inspection will confirm.)

Each project team will be given read-only access credentials to this database. The ERD for the database is detailed here.

To host your solution, each team will be given access to a special AWS sandbox that will stay open for 120 hours and will allow you to work throughout the week without interruption. However, at the end of the 120 hours the sandbox will expire. You will need to rebuild all your infrastructure again the following week. Therefore, it is in your own interest that you are able to script the creation of the resources so that they can be rebuilt as quickly and efficiently as possible.

In addition, you will be given credentials for a data warehouse hosted in the Northcoders AWS account. The data will have to be remodelled for this warehouse into three overlapping star schemas. You can find the ERDs for these star schemas:

"Sales" schema
"Purchases" schema
"Payments" schema
The overall structure of the resulting data warehouse is shown here.

Required Components
You need to create:

A job scheduler to run the ingestion job. AWS Eventbridge is the recommended way to do this. Since data has to be visible in the data warehouse within 30 minutes from being written to the database, you need to schedule a your job to check for changes much more frequently.
An S3 bucket which will act as a "landing zone" for ingested data.
A Python application to check for the changes to the database tables and ingest any new or updated data. It is strongly recommended that you use AWS Lambda as your computing solution. It is possible to use EC2, but it will be much harder to create event-driven jobs, and harder to log events in Cloudwatch. The data should be saved in the "ingestion" S3 bucket in a suitable format. Status and error messages should be logged to Cloudwatch.
A Cloudwatch alert should be generated in the event of a major error - this should be sent to email or Slack.
A second S3 bucket for "processed" data.
A Python application to transform data landing in the "ingestion" S3 bucket and place the results in the "processed" S3 bucket. The data should be transformed to conform to the warehouse schema (see below). The job should be triggered by either an S3 event triggered when data lands in the ingestion bucket, or on a schedule. Again, status and errors should be logged to Cloudwatch, and an alert triggered if a serious error occurs.
A Python application that will periodically schedule an update of the data warehouse from the data in S3. As before
The tables to be ingested from the totesys source database are:

tablename
counterparty
currency
department
design
staff
sales_order
address
payment
purchase_order
payment_type
transaction
The list of tables in the complete warehouse is:

tablename
fact_sales_order
fact_purchase_orders
fact_payment
dim_transaction
dim_staff
dim_payment_type
dim_location
dim_design
dim_date
dim_currency
dim_counterparty
The structure of your "processed" S3 data should reflect these tables.

Note that data types in some columns may have to be changed...
