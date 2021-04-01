# Prerequisites - Setup the environment

## Create an instance in AWS 

I deployed an t2.micro EC2 instance with os: Ubuntu 20.04 (focal) 
in the London region, created a new key pair for this instance to be able to SSH into it. 
While creating the instance I created a security group and allow SSH traffic on port 22. 
And enabled the auto-assign for public IP. 

 <img width="452" alt="image" src="https://user-images.githubusercontent.com/18196749/112824238-96833580-908a-11eb-9aad-5855b2211d97.png">


## Installing the agent 

To installed the agent I accessed the EC2 instance via SSH. And followed the datadog agent installation guide from the web interface. Which is executing the following command in the console 

DD_AGENT_MAJOR_VERSION=7 DD_API_KEY=<API_KEY> DD_SITE="datadoghq.eu" bash -c "$(curl -L https://s3.amazonaws.com/dd-agent/scripts/install_script.sh)"

 
<img width="270" alt="image" src="https://user-images.githubusercontent.com/18196749/112824268-9f740700-908a-11eb-8ea5-158cb51b0e3c.png">


I also deployed a couple other instances to try different things.

•	VM via Vagrant – Ubuntu 20.10
•	VM via Fusion – Ubuntu 20.10 
•	AWS EC2 – Amazon Linux 

I added some tags to the vagrant Ubuntu machine directly in the configuration file, and then added some tags to the EC2 linux machine via the UI to see the difference. 

<img width="369" alt="image" src="https://user-images.githubusercontent.com/18196749/112824298-a8fd6f00-908a-11eb-9462-004de0502ff5.png">

Vagrant machine with tags added in the Agent’s config file

<img width="369" alt="image" src="https://user-images.githubusercontent.com/18196749/112824332-af8be680-908a-11eb-808a-d28b9b729913.png">
 
EC2 instance with tags added directly in the UI


# Collecting Metrics

## Add tags in the Agent config file and show us a screenshot of your host and its tags on the Host Map page in Datadog.

From the datadog documentation: There are a few different ways to add tags to your instances. 

 <img width="276" alt="image" src="https://user-images.githubusercontent.com/18196749/112824360-b9154e80-908a-11eb-9032-1bdc9c9c8403.png">


For this exercise I have followed the “Configuration Files” method, which consists on adding the tags directly in the datadog agent’s configuration file. 

The configuration file for Ubuntu are under the following path: 
/etc/datadog-agent/datadog.yaml

 <img width="270" alt="image" src="https://user-images.githubusercontent.com/18196749/112824393-c599a700-908a-11eb-9092-da4ac5de3991.png">

After adding the tags it is necessary to restart the agent. 
sudo service datadog-agent restart 

Then I ran the agent status check to verify that the changes had taken place
sudo datadog-agent status 

<img width="270" alt="image" src="https://user-images.githubusercontent.com/18196749/112824426-ccc0b500-908a-11eb-9c32-a5afd89a2180.png">


## Install a database on your machine (MongoDB, MySQL, or PostgreSQL) and then install the respective Datadog integration for that database.

I decided to install mysql for ubuntu: 

sudo apt-get install mysql-server

And then installed the integration following the mysql installation guide. In this case my mysql deployment is version 8.0.23, so the steps required were: 

mysql> CREATE USER 'datadog'@'localhost' IDENTIFIED WITH mysql_native_password by '<UNIQUEPASSWORD>';

mysql> ALTER USER 'datadog'@'localhost' WITH MAX_USER_CONNECTIONS 5;


 <img width="270" alt="image" src="https://user-images.githubusercontent.com/18196749/112824448-d34f2c80-908a-11eb-8328-60ac71166daa.png">

Then restarted the agent and ran the agent status check. 

In the console then navigated to “Dashboards” and looked for the “MySQL – Overview” Dashboard and verified that data was being gathered. 

<img width="452" alt="image" src="https://user-images.githubusercontent.com/18196749/112824477-dcd89480-908a-11eb-8606-72c434a3c944.png">

## Create a custom Agent check that submits a metric named my_metric with a random value between 0 and 1000.

In order to create a custom metric there are also a few available methods to send metrics to Datadog: 
-Custom Agent Check 
-DogStatsD
-PowerShell
-AWS Lambda
-Datadog’s HTTP API

For this exercise it was requested to create a custom agent check, and in order to create this custom check in the agent I followed the documentation at: https://docs.datadoghq.com/developers/metrics/agent_metrics_submission/?tab=count  
 

<img width="304" alt="image" src="https://user-images.githubusercontent.com/18196749/112824525-ee21a100-908a-11eb-8681-e729314d87e4.png">

Restarted the agent and ran the agent status check, and looked for the newly created custom agent check. 

<img width="303" alt="image" src="https://user-images.githubusercontent.com/18196749/112824551-f4b01880-908a-11eb-8744-017b3a547e41.png">

Change your check's collection interval so that it only submits the metric once every 45 seconds. 

In my_metric.d/ folder, modify the configuration file named my_metric.yaml with the collection interval set to 45 seconds

<img width="270" alt="image" src="https://user-images.githubusercontent.com/18196749/112824567-f974cc80-908a-11eb-9c4f-41f782e7f74d.png">


## Bonus Question: Can you change the collection interval without modifying the Python check file you created?

Yes. It is possible to change the collection interval in two possible ways. The first one being modifying the python check file, as done in the previous step. 

Or alternatively: It can also be changed directly the Datadog UI.

Procedure:
-Navigate to metrics -> Summary
-In the search bar input the name of the metric 
-Then in the metadata section click on “Edit”, and modify the interval

<img width="452" alt="image" src="https://user-images.githubusercontent.com/18196749/112824601-0691bb80-908b-11eb-8d63-d8354910bf95.png">

# Visualizing Data

Utilize the Datadog API to create a Timeboard that contains:
•	Your custom metric scoped over your host.
•	Any metric from the Integration on your Database with the anomaly function applied.
•	Your custom metric with the rollup function applied to sum up all the points for the past hour into one bucket

## Initial setup
Documentation about the API
https://docs.datadoghq.com/api/latest/ 

I set up postman and imported the collection.And set the the api_key and application_key in the “Datadog Authentication” environment. 

Then validated the key and that I was able to reach the service using the Authentication call. 

<img width="452" alt="image" src="https://user-images.githubusercontent.com/18196749/112824615-0f828d00-908b-11eb-9eb2-e891dc9677a3.png">

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

## •	Your custom metric scoped over your host.
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

## •	Any metric from the Integration on your Database with the anomaly function applied

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

## •	Your custom metric with the rollup function applied to sum up all the points for the past hour into one bucket

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
 
<img width="452" alt="image" src="https://user-images.githubusercontent.com/18196749/112824654-1c06e580-908b-11eb-8334-2df69a0ad3e0.png">

### The entire JSON

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


## Create new timeboard API Call

<img width="377" alt="image" src="https://user-images.githubusercontent.com/18196749/112824711-2a550180-908b-11eb-9956-3a02b4c10aac.png">

Create new timeboard response

<img width="452" alt="image" src="https://user-images.githubusercontent.com/18196749/112824745-32ad3c80-908b-11eb-8eba-a3c6846427ff.png">

Then verified that the timeboard was created by checking in the UI

<img width="452" alt="image" src="https://user-images.githubusercontent.com/18196749/112824766-380a8700-908b-11eb-94cf-375bff6885f7.png">

Dashboard public url: https://p.datadoghq.eu/sb/478y64jbxcdcxv9i-f0c844c10bcbe0b3922a5e33da046a3b 


Once this is created, access the Dashboard from your Dashboard List in the UI:

•	Set the Timeboard's timeframe to the past 5 minutes

<img width="452" alt="image" src="https://user-images.githubusercontent.com/18196749/112824786-3fca2b80-908b-11eb-9ca3-9738865df1a0.png">

•	Take a snapshot of this graph and use the @ notation to send it to yourself.

<img width="409" alt="image" src="https://user-images.githubusercontent.com/18196749/112824801-448edf80-908b-11eb-9269-150785d807e9.png">

## Bonus Question: What is the Anomaly graph displaying?

From the datadog Anomally function definition: 

Anomaly detection is an algorithmic feature that identifies when a metric is behaving differently than it has in the past

Anomaly detection - With the default option (above or below) a metric is considered to be anomalous if it is outside of the gray anomaly band. Optionally, you can specify whether being only above or below the bands is considered anomalous.

# Monitoring Data

Since you’ve already caught your test metric going above 800 once, you don’t want to have to continually watch this dashboard to be alerted when it goes above 800 again. So let’s make life easier by creating a monitor.

## Create a new Metric Monitor that watches the average of your custom metric (my_metric) and will alert if it’s above the following values over the past 5 minutes:

•	Warning threshold of 500
•	Alerting threshold of 800

<img width="452" alt="image" src="https://user-images.githubusercontent.com/18196749/112824822-4bb5ed80-908b-11eb-8319-b4eeb0029de8.png">

•	And also ensure that it will notify you if there is No Data for this query over the past 10m.

<img width="464" alt="image" src="https://user-images.githubusercontent.com/18196749/112824836-507aa180-908b-11eb-981f-4122928781f5.png">

Please configure the monitor’s message so that it will:
•	Send you an email whenever the monitor triggers.

<img width="452" alt="image" src="https://user-images.githubusercontent.com/18196749/112824853-55d7ec00-908b-11eb-9818-0bb6bbe4039a.png">

•	Create different messages based on whether the monitor is in an Alert, Warning, or No Data state.
•	Include the metric value that caused the monitor to trigger and host ip when the Monitor triggers an Alert state.

<img width="452" alt="image" src="https://user-images.githubusercontent.com/18196749/112824866-5bcdcd00-908b-11eb-9a7e-b236aa49ff7c.png">

{{#is_alert}}Alert! my_metric is over 800: value {{value}} with host_ip: {{host.ip}}{{/is_alert}} 
{{#is_warning}}Warning! my_metric is over 500: value {{value}}{{/is_warning}} 
{{#is_no_data}}No data received{{/is_no_data}} 
{{#is_recovery}}my_metrics is OK{{/is_recovery}} 
@efraingamboab@gmail.com

•	When this monitor sends you an email notification, take a screenshot of the email that it sends you

**Alert**

The alert wasn’t triggering, so I changed the random values to be higher so the average would be over 800.

 

After making this change the alert was finally triggered 

 

Looking at the message received it is not displaying the host_ip. From what I understand reading the documentations it takes it from the agent config file in the tags. Since I haven’t specified it there it’s not displaying

Testing with the {{host.name}} instead, but it didn’t work either. 

**Warn**
<img width="452" alt="image" src="https://user-images.githubusercontent.com/18196749/112825008-8a4ba800-908b-11eb-81cb-330d7dc28365.png">

**Recovered**
<img width="452" alt="image" src="https://user-images.githubusercontent.com/18196749/112825047-946da680-908b-11eb-9f65-c0d2371a97c1.png">

**No data received**
<img width="452" alt="image" src="https://user-images.githubusercontent.com/18196749/112825083-9df70e80-908b-11eb-9892-4ed360c2022d.png">

##Bonus Question: Since this monitor is going to alert pretty often, you don’t want to be alerted when you are out of the office. Set up two scheduled downtimes for this monitor:

**One that silences it from 7pm to 9am daily on M-F**

In order to do this it’s necessary to schedule downtime.

The process is quite straight forward: 
-Navigate to “Monitor” -> “Manage Downtime”
-Select the “manage downtime” tab 
-Click on “Schedule downtime”
<img width="312" alt="image" src="https://user-images.githubusercontent.com/18196749/112825101-a3ecef80-908b-11eb-9d51-e65266701c2b.png">

**One that silences it all day on Sat-Sun**
<img width="312" alt="image" src="https://user-images.githubusercontent.com/18196749/112825120-abac9400-908b-11eb-9d16-f6ffeb3e7702.png">

Make sure that your email is notified when you schedule the downtime and take a screenshot of that notification
<img width="274" alt="image" src="https://user-images.githubusercontent.com/18196749/112825142-b109de80-908b-11eb-9966-f8d47c967e23.png">


# Collecting APM Data

## Deploy the Flask app with Python 

from flask import Flask
import logging
import sys

Have flask use stdout as the logger
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


## Initial Setup

In order to deploy the given Flask App in my EC2 instance there are some necessary previous steps to take to have the environment ready. 

These are: 

## Install virtualenv 

$ sudo apt-get install python-virtualenv
$ mkdir datadogflask
$ cd datadogflask
$ sudo apt-get install python3-venv
$ python3 -m venv venv
$ . venv/bin/activate
$ pip install Flask


## Create the python file “datadogflaskapp.py”

In the virtual environment 
(venv) ubuntu@ip-172-31-41-67:~/datadogflask$ export FLASK_APP=datadogflaskapp.py
(venv) ubuntu@ip-172-31-41-67:~/datadogflask$ flask run –-port 5050

<img width="270" alt="image" src="https://user-images.githubusercontent.com/18196749/112825269-db5b9c00-908b-11eb-9fc9-81759da6e9ec.png">

And the datadogflaskapp is running. 
Now it’s time to implement APM

# APM Setup

In the dashboard navigate to “APM” 
Then in the getting started section select “Host Based”. And follow the instructions

The agent in this case is already installed. 

Select the language. In this case python

In the virtual environment 
(venv) ubuntu@ip-172-31-41-67:~/datadogflask$ DD_SERVICE="datadogflaskapp" DD_ENV="prod" DD_LOGS_INJECTION=true ddtrace-run python datadogflaskapp.py

<img width="270" alt="image" src="https://user-images.githubusercontent.com/18196749/112825316-e6aec780-908b-11eb-8147-cd57f3a811f9.png">

Now the application is running. 

Since I had already install the GUI for my ubuntu machine I went to the browser and navigated to the url: 
http://0.0.0.0/5050/ 
http://0.0.0.0/5050/api/apm 
http://0.0.0.0/5050/api/trace

<img width="411" alt="image" src="https://user-images.githubusercontent.com/18196749/112825337-eca4a880-908b-11eb-92c0-0856ccadf5ed.png">

Now in the datadog UI. It is possible to see the below screen under “APM”. 
And the datadogflasapp listed and reporting data. 

<img width="452" alt="image" src="https://user-images.githubusercontent.com/18196749/112825365-f3332000-908b-11eb-9ec8-9c67a808cd82.png">
<img width="452" alt="image" src="https://user-images.githubusercontent.com/18196749/112825383-f6c6a700-908b-11eb-98cf-8813aceafc62.png">
<img width="452" alt="image" src="https://user-images.githubusercontent.com/18196749/112825397-fa5a2e00-908b-11eb-8319-002d0c2771f4.png">
<img width="452" alt="image" src="https://user-images.githubusercontent.com/18196749/112825411-fdedb500-908b-11eb-8161-ae991b6ebba1.png">

# Bonus Question: What is the difference between a Service and a Resource?


•	Services groups together endpoints, queries, or jobs for the purpose of scaling instances. All services are listed under Service List and can be visually seen as in a Micro-service format under Service Map.

•	Resources represent a particular domain of a Customer Application.They could typically be an instrumented web endpoint, database query, or background job. Each resource has its own Resource page with trace metrics scoped to the specific endpoint.

# Final Question
Is there anything creative you would use Datadog for?

I live in Barcelona where there currently is a big initiative for a greener city. So one use case that comes to mind is tracking the number cars in the low emissions area. 
There are cameras checking the license plates of all vehicles entering that area. If this was publicly available can be monitored. 

Another idea would be during the winter season check the mountain and slopes snow site to know if it’s good to god skiing. Also compare it with previous year’s data and use it to determine global warming rates. 


# Extra Information

## Mysql DB

I follow this guide: https://www.digitalocean.com/community/tutorials/a-basic-mysql-tutorial

Mysql datadog user: 
CREATE USER 'datadog'@'localhost' IDENTIFIED BY 'datadog1!'


## Mongo DB

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

<img width="270" alt="image" src="https://user-images.githubusercontent.com/18196749/112825463-0d6cfe00-908c-11eb-8779-e9400316b2f7.png">

Custom metric 
In order to understand how the custom metric works and the what the different functions do I created the metric exactly as it was in the example. 
 
<img width="239" alt="image" src="https://user-images.githubusercontent.com/18196749/112825472-1231b200-908c-11eb-8c78-b143a8d55a0d.png">

And obtained the following result. 
 
<img width="452" alt="image" src="https://user-images.githubusercontent.com/18196749/112825487-165dcf80-908c-11eb-951f-a48c7ed21483.png">


gauge()
This function submits the value of a metric at a given timestamp. If called multiple times during a check’s execution for a metric only the last sample is used
histogram()
This function submits the sample of a histogram metric that occurred during the check interval. It can be called multiple times during a check’s execution, each sample being added to the statistical distribution of the set of values for this metric.

Then I settled for the gauge option. 

# Visualizing Data
I encountered difficulties using postman to interact with the API. Specifically around authentication. 
I changed the environment variables with the same API key I used for the integrations, and it wasn’t working. Also checked that the URL was the correct one (datadoghq.eu).

<img width="452" alt="image" src="https://user-images.githubusercontent.com/18196749/112825557-32fa0780-908c-11eb-94b8-aa67024c6343.png">

And it was always returning code 403. 
 
<img width="452" alt="image" src="https://user-images.githubusercontent.com/18196749/112825572-37bebb80-908c-11eb-8ce0-3eb23711f600.png">

I generated a new API key but that didn’t worked either. The reasoning behind this was that I believed that I thought that since the key was being used in the integrations it might’ve had an impact on using the same key in postman. 
I tried using the web version of postman and it started working.
