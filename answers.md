

 

Table of Contents
Prerequisites - Setup the environment	2
Create an instance in AWS	2
Installing the agent	2
Collecting Metrics	4
Add tags in the Agent config file and show us a screenshot of your host and its tags on the Host Map page in Datadog.	4
Install a database on your machine (MongoDB, MySQL, or PostgreSQL) and then install the respective Datadog integration for that database.	5
Create a custom Agent check that submits a metric named my_metric with a random value between 0 and 1000	6
Change your check's collection interval so that it only submits the metric once every 45 seconds.	7
Bonus Question	8
Visualizing Data	9
Utilize the Datadog API to create a Timeboard	9
Once this is created, access the Dashboard from your Dashboard List in the UI	14
Bonus Question	15
Monitoring Data	15
Create a new Metric Monitor that watches the average of your custom metric (my_metric) and will alert if it’s above the following values over the past 5 minutes	15
Bonus Question	19
Collecting APM Data	20
Initial Setup	21
APM Setup	22
Bonus Question	25
Final Question	25
Extra Information	26

 

Prerequisites - Setup the environment

Create an instance in AWS 

I deployed an t2.micro EC2 instance with os: Ubuntu 20.04 (focal) 
in the London region, created a new key pair for this instance to be able to SSH into it. 
While creating the instance I created a security group and allow SSH traffic on port 22. 
And enabled the auto-assign for public IP. 

 


Installing the agent 

To installed the agent I accessed the EC2 instance via SSH. And followed the datadog agent installation guide from the web interface. Which is executing the following command in the console 

DD_AGENT_MAJOR_VERSION=7 DD_API_KEY=<API_KEY> DD_SITE="datadoghq.eu" bash -c "$(curl -L https://s3.amazonaws.com/dd-agent/scripts/install_script.sh)"

 


I also deployed a couple other instances to try different things.

•	VM via Vagrant – Ubuntu 20.10
•	VM via Fusion – Ubuntu 20.10 
•	AWS EC2 – Amazon Linux 

I added some tags to the vagrant Ubuntu machine directly in the configuration file, and then added some tags to the EC2 linux machine via the UI to see the difference. 

 
Vagrant machine with tags added in the Agent’s config file


 
EC2 instance with tags added directly in the UI


Collecting Metrics

Add tags in the Agent config file and show us a screenshot of your host and its tags on the Host Map page in Datadog.

From the datadog documentation: There are a few different ways to add tags to your instances. 

 

For this exercise I have followed the “Configuration Files” method, which consists on adding the tags directly in the datadog agent’s configuration file. 

The configuration file for Ubuntu are under the following path: 
/etc/datadog-agent/datadog.yaml

 


After adding the tags it is necessary to restart the agent. 
sudo service datadog-agent restart 

Then I ran the agent status check to verify that the changes had taken place
sudo datadog-agent status 

 


Install a database on your machine (MongoDB, MySQL, or PostgreSQL) and then install the respective Datadog integration for that database.

I decided to install mysql for ubuntu: 

sudo apt-get install mysql-server

And then installed the integration following the mysql installation guide. In this case my mysql deployment is version 8.0.23, so the steps required were: 

mysql> CREATE USER 'datadog'@'localhost' IDENTIFIED WITH mysql_native_password by '<UNIQUEPASSWORD>';

mysql> ALTER USER 'datadog'@'localhost' WITH MAX_USER_CONNECTIONS 5;


 

Then restarted the agent and ran the agent status check. 

In the console then navigated to “Dashboards” and looked for the “MySQL – Overview” Dashboard and verified that data was being gathered. 

 



Create a custom Agent check that submits a metric named my_metric with a random value between 0 and 1000.

In order to create a custom metric there are also a few available methods to send metrics to Datadog: 
-Custom Agent Check 
-DogStatsD
-PowerShell
-AWS Lambda
-Datadog’s HTTP API

For this exercise it was requested to create a custom agent check, and in order to create this custom check in the agent I followed the documentation at: https://docs.datadoghq.com/developers/metrics/agent_metrics_submission/?tab=count  
 

Restarted the agent and ran the agent status check, and looked for the newly created custom agent check. 

 


Change your check's collection interval so that it only submits the metric once every 45 seconds. 

In my_metric.d/ folder, modify the configuration file named my_metric.yaml with the collection interval set to 45 seconds

 


Bonus Question: Can you change the collection interval without modifying the Python check file you created?

Yes. It is possible to change the collection interval in two possible ways. The first one being modifying the python check file, as done in the previous step. 

Or alternatively: It can also be changed directly the Datadog UI.

Procedure:
-Navigate to metrics -> Summary
-In the search bar input the name of the metric 
-Then in the metadata section click on “Edit”, and modify the interval

 


Visualizing Data

Utilize the Datadog API to create a Timeboard that contains:
•	Your custom metric scoped over your host.
•	Any metric from the Integration on your Database with the anomaly function applied.
•	Your custom metric with the rollup function applied to sum up all the points for the past hour into one bucket

Initial setup
Documentation about the API
https://docs.datadoghq.com/api/latest/ 

I set up postman and imported the collection.And set the the api_key and application_key in the “Datadog Authentication” environment. 

Then validated the key and that I was able to reach the service using the Authentication call. 

 

Now that the it is all set up and I am able to interact with the API the next step it’s to create the the Timeboard. For this I followed the documentation at: https://docs.datadoghq.com/api/latest/dashboards/ 

To create a new dashboard it is necessary to use the following API call: 
POST https://api.datadoghq.eu/api/v1/dashboard

And as guide there is the following Example JSON:

{
  "description": "string",
  "is_read_only": false,
  "layout_type": "string",
  "notify_list": [],
  "template_variable_presets": [
    {
      "name": "string",
      "template_variables": [
        {
          "name": "string",
          "value": "string"
        }
      ]
    }
  ],
  "template_variables": [
    {
      "default": "my-host",
      "name": "host1",
      "prefix": "host"
    }
  ],
  "title": "",
  "widgets": [
    {
      "definition": "[object Object]",
      "id": "integer",
      "layout": {
        "height": 0,
        "width": 0,
        "x": 0,
        "y": 0
      }
    }
  ]
}

This is an example of how the body of the request should look like.  Not being familiar with the tool I was not sure of the possibilities or what to put in the JSON in order to reflect what is being request. So I went to the datadog UI and used to look at creating the dashboard to export the JSON. Which I then added to the body of the request, and successfully created via API.

•	Your custom metric scoped over your host.
{
    "viz": "timeseries",
    "requests": [
        {
            "q": "max:my_metric{*}",
            "type": "line",
            "style": {
                "palette": "purple",
                "type": "solid",
                "width": "thin"
            },
            "on_right_yaxis": false
        }
    ],
    "yaxis": {
        "scale": "linear",
        "min": "auto",
        "max": "auto",
        "includeZero": true,
        "label": ""
    },
    "markers": []
}

•	Any metric from the Integration on your Database with the anomaly function applied

Metric: mysql.performance.cpu_time 

{
    "viz": "timeseries",
    "requests": [
        {
            "q": "anomalies(avg:mysql.performance.cpu_time{*}, 'basic', 2)",
            "type": "line",
            "style": {
                "palette": "purple",
                "type": "solid",
                "width": "normal"
            },
            "on_right_yaxis": false
        }
    ],
    "yaxis": {
        "scale": "linear",
        "min": "auto",
        "max": "auto",
        "includeZero": true,
        "label": ""
    },
    "markers": []
}

•	Your custom metric with the rollup function applied to sum up all the points for the past hour into one bucket

{
    "viz": "query_value",
    "requests": [
        {
            "q": "max:my_metric{*}.rollup(sum, 3600)",
            "aggregator": "sum",
            "conditional_formats": [
                {
                    "comparator": ">",
                    "palette": "white_on_red"
                },
                {
                    "comparator": ">=",
                    "palette": "white_on_yellow"
                },
                {
                    "comparator": "<",
                    "palette": "white_on_green"
                }
            ]
        }
    ],
    "autoscale": true,
    "precision": 2
}
 


The entire JSON

{
  "title": "Visualizing Data",
  "description": "## Visualizing Data\n\nIncludes: Custom metric over host, Metric on the DB, Custom metric with rollup function \n",
  "widgets": [ 
		… 
	      ],
  "template_variables": [],
  "layout_type": "ordered",
  "is_read_only": false,
  "notify_list": [],
  "id": "66f-vy9-aqx"
}


Create new timeboard API Call

 


Create new timeboard response

 


Then verified that the timeboard was created by checking in the UI


 

Dashboard public url: https://p.datadoghq.eu/sb/478y64jbxcdcxv9i-f0c844c10bcbe0b3922a5e33da046a3b 




Once this is created, access the Dashboard from your Dashboard List in the UI:

•	Set the Timeboard's timeframe to the past 5 minutes
 

•	Take a snapshot of this graph and use the @ notation to send it to yourself.
 

Bonus Question: What is the Anomaly graph displaying?

From the datadog Anomally function definition: 

Anomaly detection is an algorithmic feature that identifies when a metric is behaving differently than it has in the past

Anomaly detection - With the default option (above or below) a metric is considered to be anomalous if it is outside of the gray anomaly band. Optionally, you can specify whether being only above or below the bands is considered anomalous.

Monitoring Data

Since you’ve already caught your test metric going above 800 once, you don’t want to have to continually watch this dashboard to be alerted when it goes above 800 again. So let’s make life easier by creating a monitor.

Create a new Metric Monitor that watches the average of your custom metric (my_metric) and will alert if it’s above the following values over the past 5 minutes:

•	Warning threshold of 500
•	Alerting threshold of 800
 

•	And also ensure that it will notify you if there is No Data for this query over the past 10m.
 


Please configure the monitor’s message so that it will:
•	Send you an email whenever the monitor triggers.
 

•	Create different messages based on whether the monitor is in an Alert, Warning, or No Data state.
•	Include the metric value that caused the monitor to trigger and host ip when the Monitor triggers an Alert state.

 

{{#is_alert}}Alert! my_metric is over 800: value {{value}} with host_ip: {{host.ip}}{{/is_alert}} 
{{#is_warning}}Warning! my_metric is over 500: value {{value}}{{/is_warning}} 
{{#is_no_data}}No data received{{/is_no_data}} 
{{#is_recovery}}my_metrics is OK{{/is_recovery}} 
@efraingamboab@gmail.com

•	When this monitor sends you an email notification, take a screenshot of the email that it sends you

Alert 

The alert wasn’t triggering, so I changed the random values to be higher so the average would be over 800.

 

After making this change the alert was finally triggered 

 

Looking at the message received it is not displaying the host_ip. From what I understand reading the documentations it takes it from the agent config file in the tags. Since I haven’t specified it there it’s not displaying

Testing with the {{host.name}} instead, but it didn’t work either. 







Warn

 

Recovered

 











No data received 
 


Bonus Question: Since this monitor is going to alert pretty often, you don’t want to be alerted when you are out of the office. Set up two scheduled downtimes for this monitor:

One that silences it from 7pm to 9am daily on M-F

In order to do this it’s necessary to schedule downtime.

The process is quite straight forward: 
-Navigate to “Monitor” -> “Manage Downtime”
-Select the “manage downtime” tab 
-Click on “Schedule downtime”

 


One that silences it all day on Sat-Sun
 


Make sure that your email is notified when you schedule the downtime and take a screenshot of that notification

 

Collecting APM Data

Deploy the Flask app with Python 


from flask import Flask
import logging
import sys

# Have flask use stdout as the logger
main_logger = logging.getLogger()
main_logger.setLevel(logging.DEBUG)
c = logging.StreamHandler(sys.stdout)
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
c.setFormatter(formatter)
main_logger.addHandler(c)

app = Flask(__name__)

@app.route('/')
def api_entry():
    return 'Entrypoint to the Application'

@app.route('/api/apm')
def apm_endpoint():
    return 'Getting APM Started'

@app.route('/api/trace')
def trace_endpoint():
    return 'Posting Traces'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port='5050')


Initial Setup

In order to deploy the given Flask App in my EC2 instance there are some necessary previous steps to take to have the environment ready. 

These are: 

Install virtualenv 

$ sudo apt-get install python-virtualenv
$ mkdir datadogflask
$ cd datadogflask
$ sudo apt-get install python3-venv
$ python3 -m venv venv
$ . venv/bin/activate
$ pip install Flask


Create the python file “datadogflaskapp.py”

In the virtual environment 
(venv) ubuntu@ip-172-31-41-67:~/datadogflask$ export FLASK_APP=datadogflaskapp.py
(venv) ubuntu@ip-172-31-41-67:~/datadogflask$ flask run –-port 5050

 

And the datadogflaskapp is running. 
Now it’s time to implement APM


APM Setup

In the dashboard navigate to “APM” 
Then in the getting started section select “Host Based”. And follow the instructions

The agent in this case is already installed. 

Select the language. In this case python


In the virtual environment 
(venv) ubuntu@ip-172-31-41-67:~/datadogflask$ DD_SERVICE="datadogflaskapp" DD_ENV="prod" DD_LOGS_INJECTION=true ddtrace-run python datadogflaskapp.py


 

Now the application is running. 

Since I had already install the GUI for my ubuntu machine I went to the browser and navigated to the url: 
http://0.0.0.0/5050/ 
http://0.0.0.0/5050/api/apm 
http://0.0.0.0/5050/api/trace

 


Now in the datadog UI. It is possible to see the below screen under “APM”. 
And the datadogflasapp listed and reporting data. 

 

 
 



 


Bonus Question: What is the difference between a Service and a Resource?


•	Services groups together endpoints, queries, or jobs for the purpose of scaling instances. All services are listed under Service List and can be visually seen as in a Micro-service format under Service Map.

•	Resources represent a particular domain of a Customer Application.They could typically be an instrumented web endpoint, database query, or background job. Each resource has its own Resource page with trace metrics scoped to the specific endpoint.

Final Question
Is there anything creative you would use Datadog for?

I live in Barcelona where there currently is a big initiative for a greener city. So one use case that comes to mind is tracking the number cars in the low emissions area. 
There are cameras checking the license plates of all vehicles entering that area. If this was publicly available can be monitored. 

Another idea would be during the winter season check the mountain and slopes snow site to know if it’s good to god skiing. Also compare it with previous year’s data and use it to determine global warming rates. 





Extra Information

Mysql DB

I follow this guide: https://www.digitalocean.com/community/tutorials/a-basic-mysql-tutorial

Mysql datadog user: 
CREATE USER 'datadog'@'localhost' IDENTIFIED BY 'datadog1!'


Mongo DB

Simultenously I tried to implement also the mongodb integration. So I set up a mongo DB deployment following this guide: 

https://www.digitalocean.com/community/tutorials/how-to-install-mongodb-on-ubuntu-18-04-source

Then imported a DB

Following the steps explained in this guide: 
https://www.digitalocean.com/community/tutorials/how-to-import-and-export-a-mongodb-database-on-ubuntu-20-04

And then followed the integration steps

db.createUser({
  "user": "datadog",
  "pwd": "datadog1!",
  "roles": [
    { role: "read", db: "admin" },
    { role: "clusterMonitor", db: "admin" },
    { role: "read", db: "local" }
  ]
})

 


Custom metric 
In order to understand how the custom metric works and the what the different functions do I created the metric exactly as it was in the example. 
 

And obtained the following result. 
 

import random

from datadog_checks.base import AgentCheck

__version__ = "1.0.0"

class MyClass(AgentCheck):
    def check(self, instance):
        self.count(
            "example_metric.count",
            10,
            tags=["env:dev","metric_submission_type:count"],
        )
        self.count(
            "example_metric.decrement",
            -5,
            tags=["env:dev","metric_submission_type:count"],
        )
        self.count(
            "example_metric.increment",
            14,
            tags=["env:dev","metric_submission_type:count"],
        )
        self.rate(
            "example_metric.rate",
            3,
            tags=["env:dev","metric_submission_type:rate"],
        )
        self.gauge(
            "example_metric.gauge",
            random.randint(0, 10),
            tags=["env:dev","metric_submission_type:gauge"],
        )
        self.monotonic_count(
            "example_metric.monotonic_count",
            2,
            tags=["env:dev","metric_submission_type:monotonic_count"],
        )

        # Calling the functions below twice simulates
        # several metrics submissions during one Agent run.
        self.histogram(
            "example_metric.histogram",
            random.randint(0, 1000),
            tags=["env:dev","metric_submission_type:histogram"],
        )
        self.histogram(
            "example_metric.histogram",
            random.randint(0, 1000),
            tags=["env:dev","metric_submission_type:histogram"],
        )


import random

from datadog_checks.base import AgentCheck

__version__ = "1.0.0"

class MyClass(AgentCheck):
    def check(self, instance):
        self.gauge(
            "my_metric",
            random.randint(0, 1000),
            tags=["env:dev","metric_submission_type:gauge"],

gauge()
This function submits the value of a metric at a given timestamp. If called multiple times during a check’s execution for a metric only the last sample is used
histogram()
This function submits the sample of a histogram metric that occurred during the check interval. It can be called multiple times during a check’s execution, each sample being added to the statistical distribution of the set of values for this metric.

Then I settled for the gauge option. 





Visualizing Data
I encountered difficulties using postman to interact with the API. Specifically around authentication. 
I changed the environment variables with the same API key I used for the integrations, and it wasn’t working. Also checked that the URL was the correct one (datadoghq.eu).

 

And it was always returning code 403. 
 
I generated a new API key but that didn’t worked either. The reasoning behind this was that I believed that I thought that since the key was being used in the integrations it might’ve had an impact on using the same key in postman. 
I tried using the web version of postman and it started working.
