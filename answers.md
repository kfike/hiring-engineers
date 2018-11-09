## Setup:

Setup and Installing the Agent:
I used Vagrant (Ubuntu) to complete the exercise. My Datadog host map also includes a docker instance because I was curious about installing the agent on a Docker container. The setup after creating an account was pretty straight forward.

## 1) Collecting Metrics

Everything in this section was self-explanatory and found the docs easy to understand.

Note: The dd-agent status command helped me out A LOT through this particular exercise.

### a) Adding Tags
Used the docs here: <a href="https://docs.datadoghq.com/tagging/assigning_tags/?tab=hostmap">https://docs.datadoghq.com/tagging/assigning_tags/?tab=hostmap</a>

Editing the the datadog.yaml, I found the tag dictionaries, saved the file, and restarted the dd agent:

#### (screenshot)
<img src="https://github.com/kfike/hiring-engineers/blob/solutions-engineer/images/1/1/1.png" />

I confirmed that the tags showed up by looking at the UI of my host map instance:

#### (screenshot)
<img src="https://github.com/kfike/hiring-engineers/blob/solutions-engineer/images/1/1/2.png" />

### b) Install Datadog integration for my DB
Used the docs here: <a href="https://docs.datadoghq.com/integrations/postgres/">https://docs.datadoghq.com/integrations/postgres/</a>

I installed Postgres on my machine and went through the steps of setting up the Postgres user for datadog and edited the Postgres.d/conf.yaml.

After restarting the dd-agent, I ran the agent status and received an “ok” from the status check:

#### (screenshot)
<img src="https://github.com/kfike/hiring-engineers/blob/solutions-engineer/images/1/2/1.png" />

After confirming confirming an “ok” from my status check, I went back to the UI to confirm it was showing up:

#### (screenshot)
<img src="https://github.com/kfike/hiring-engineers/blob/solutions-engineer/images/1/2/2.png" />


### c) Custom Agent called “my_metric”
Used the docs here: <a href="https://docs.datadoghq.com/developers/agent_checks/?tab=agentv6">https://docs.datadoghq.com/developers/agent_checks/?tab=agentv6</a>

Created a file called my_metric.py and my_metric.yaml. Confirmed that the metric was showing up:

#### (screenshot)
<img src="https://github.com/kfike/hiring-engineers/blob/solutions-engineer/images/1/3/1.png" />

#### (screenshot)
<img src="https://github.com/kfike/hiring-engineers/blob/solutions-engineer/images/1/3/2.png" />

Note: I have to admit that this was the first time writing Python and I used the library “random” to generate the random number to send. I hadn’t downloaded the necessary library (your docs told me to but I was just being lazy).

Again, this is where the dd-agent status command helped me out. Running the command let me know that that library wasn’t there and I needed to pull it in.

### d) Send the metric every 45 seconds

I used my my_metric.yaml to set the min_collection_interval to 45, looked at my map in the UI and looked like it was sending every 45 seconds

#### (screenshot)
<img src="https://github.com/kfike/hiring-engineers/blob/solutions-engineer/images/1/4/1.png" />

### Summary
I find setup to be one of the crucial decisions in deciding a third party provider because robust integrations always seem to have a hidden cost.

Having worked with third party integrations in the past, most of them require paid trainings just to get it up an running. More importantly, it costs money in developer time. After "training" that developer to customize the app, what happens if they leave the company? The process starts over training other developers and increasing the cost. If a problem exists that is too niched, a company might need to bring in an expensive consultant to solve it.

Datadog's integration seemed the complete opposite. The docs were easy to read and I was up in running within minutes. The setup felt native, changing config files that any ops person would know how to change. Custom metrics were as easy as importing a library and integrating with your Python api.

Using native a toolset enables developers to customize the application the way they want and speaks a language they already know and understand. The result is on going customization anyone on the team can develop, all the while saving your company hundreds of thousands of dollars.

## 2) Visualizing Data

Completed Dashboard: <a href="https://app.datadoghq.com/dash/959266/my-metric-4">https://app.datadoghq.com/dash/959266/my-metric-4</a>

### Using the API

a) created my custom metric

b) anomoly function with my db metric (I used “postgresql.max_connections”)

c) roll up from with my custom metric

I used these docs:

API Timeboards: <a href="https://docs.datadoghq.com/api/?lang=python#timeboards">https://docs.datadoghq.com/api/?lang=python#timeboards</a>

Anomaly: <a href="https://docs.datadoghq.com/monitors/monitor_types/anomaly/">https://docs.datadoghq.com/monitors/monitor_types/anomaly/</a>

Rollup: <a href="https://docs.datadoghq.com/graphing/functions/rollup/">https://docs.datadoghq.com/graphing/functions/rollup/</a>

Note: I used a ruby script
```
  require 'dogapi'
  require 'byebug'

  api_key = '' # left these out bc the repo is public
  app_key = ''

  dog = Dogapi::Client.new(api_key, app_key)

  title = 'My Metric 4'
  description = 'Scope over my host'
  graphs = [{
    "definition" => {
        "events" => [],
        "requests" => [
          {"q" => "avg:my_metric{*}"},
          {"q": "avg:my_metric{*}.rollup(sum, 60)" },
          {"q" => "anomalies(avg:postgresql.max_connections{*}, 'basic', 2)"}
        ],
        "viz" => "timeseries"
    },
    "title" => "My Metric Scoped over Ubuntu Xenial"
  }]
  template_variables = [{
    "name" => "ubuntu-xenial",
    "prefix" => "ubuntu-xenial",
    "default" => "host:ubuntu-xenial"
  }]

  dog.create_dashboard(title, description, graphs, template_variables)
```

### d) Accessing in the UI and sending to myself using @ notation

#### (screenshot)
<img src="https://github.com/kfike/hiring-engineers/blob/solutions-engineer/images/2/snapshot email.png" />

Bonus: Noticed that my anomaly function was pretty empty, so it’s technically not detecting anything. If I had a db receiving inputs, it would be displaying an algorithmic function to detect an unusual amount of max connections

### Summary
The Datadog UI is the highlight of the entire application. Having been a New Relic user over the last 7-8 years, it's hard for me not to compare the two.

All developers know that log data is a necessary evil and spacing monitoring visualizations help it make sense. The problem with most monitoring applications come straight out the box and give you graphs that are predefined by the application (this is my direct shot at New Relic - haha). The issue is all infrastructure is different and changes constantly.

The UI that worked a few years doesn't necessary work today and the result having to look at multiple logs/graphs. Another result is using multiple providers to measure different metrics, doubling/tripling the cost of monitoring.

Datadog seems to take the approach of: Configure the data you want to monitor and custom create dashboards you want to visualize. Again, the customization of the dashboards enables Ops teams to create dashboards they want to see. The power in this (and cost savings) exists in the ability to quickly see log data graphed in metrics that make sense. Adding multiple metrics to one dashboard gives teams quicker analysis verses looking over endless logs. One problem with a database might take a developer hours to locate costing the company time and money.

One more note: My guess is that most teams initially use a third party integration because they have specific pain points they're trying to solve. The product teams of these providers are knowledgeable of these pain points and make sure the solution is front and center. Being that the monitoring space is a competitive market, a huge sell for me would be Datadog's ability to build customizable UIs for any future metric I hadn't thought of. Simply put, the customization of the UI doesn't limit measuring ability but gives my team thousands of possible features.


## 3) Monitoring Data

I used the UI: <a href="https://app.datadoghq.com/monitors#create/metric">https://app.datadoghq.com/monitors#create/metric</a>

### a) Create warning, alerting, no data thresholds

#### (screenshot)
<img src="https://github.com/kfike/hiring-engineers/blob/solutions-engineer/images/3/1.png" />

### b) Different messages

#### (screenshot)
<img src="https://github.com/kfike/hiring-engineers/blob/solutions-engineer/images/3/2.png" />

### c) Send an email

#### (screenshot)
<img src="https://github.com/kfike/hiring-engineers/blob/solutions-engineer/images/3/3.png" />

Bonus:

#### (screenshot)
<img src="https://github.com/kfike/hiring-engineers/blob/solutions-engineer/images/3/bonus/Screen Shot 2018-10-24 at 11.13.13 PM.png" />

#### (screenshot)
<img src="https://github.com/kfike/hiring-engineers/blob/solutions-engineer/images/3/bonus/Screen Shot 2018-10-24 at 11.13.38 PM.png" />

### Summary
An Ops team worst enemy is the false notification that doesn't meaning anything spamming their email box. This type of noise hides other important alerts.

In my experience, false alarms primarily come from:

- using multiple providers for monitoring and teams not giving access to configure other team's providers
- lack customization of the monitor

Again, the monitoring tool allows a developer to get the correct alert and to know immediately how serious the issue is. The user interface allows for easy configuration to specifically notify the correct metric at the correct time. This prevents bloated email inboxes with useless warnings and missing important messages.


## 4) Collecting APM Data:

I used the flask app example and the docs for the APM setup:

#### (screenshot)
<img src="https://github.com/kfike/hiring-engineers/blob/solutions-engineer/images/4/1.png" />

Dashboard: <a href="https://app.datadoghq.com/dash/959266/my-metric-4">https://app.datadoghq.com/dash/959266/my-metric-4</a>

### Summary
The most impressive feature of Datadog's APM app is how easily it integrates with custom dashboards. My impression was that it would be standalone, similar to other infrastructure monitors that have added it. One can monitor memory issues on the infrastructure side and cross check it with stack tracing. For example, one could monitor database and see dashboard with metrics on stack traces performing connections to that database. Seeing this displayed in one dashboard would save me a massive amount of time vs having to check multiple logs/graphs.

The direct result is a massive amount of time/cost savings.

## 5) Final Question:

I'd love to use Datadog to measure baby monitor frequency data. The monitor could send frequency to datadog so that it could monitor consistency. Ideally, Datadog would be able to tell if a baby woke up and alert parents.
