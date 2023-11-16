---
coverY: 0
layout:
  cover:
    visible: true
    size: full
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: false
  pagination:
    visible: true
---

# 2.2.0 CloudWatch Logs

## **0. Learning Materials**

* Video: [Week 2 CloudWatch Logs](https://www.youtube.com/watch?v=ipdFizZjOF4\&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv\&index=31\&ab\_channel=ExamPro)
* Andrew's repo: [week-2-cloudwatch-logs](https://github.com/omenking/aws-bootcamp-cruddur-2023/tree/week-2-cloudwatch-logs)
* My branch repo: [02-02-cloudwatch-logs](https://github.com/mariachiinajar/aws-bootcamp-cruddur-2023-again/tree/02-02-cloudwatch-logs)
  * My commits:
    * [#12](https://github.com/mariachiinajar/aws-bootcamp-cruddur-2023-again/issues/12) [Implemented CloudWatch agent in backend](https://github.com/mariachiinajar/aws-bootcamp-cruddur-2023-again/commit/ebca395a2851244edea48a93a4e2b994884c96b0)
    * [#12](https://github.com/mariachiinajar/aws-bootcamp-cruddur-2023-again/issues/12) [Minor bug fix on home\_activities](https://github.com/mariachiinajar/aws-bootcamp-cruddur-2023-again/commit/2601929374e0dada7b10556bfd2294dc0dbb4d2c)

### Task List

* [ ] 1\. Implement a CloudWatch agent in backend
  * [ ] 1.1. Plant a logger in the flask server `app`.
  * [ ] 1.2. Set logger in the service (`home_activities`)
* [ ] 2\. Update the backend docker container

### Env Variables

<table><thead><tr><th width="168">Micro service</th><th>Variables</th></tr></thead><tbody><tr><td>backend</td><td><ul><li><code>HONEYCOMB_API_KEY</code></li><li><code>OTEL_SERVICE_NAME</code></li><li><code>OTEL_EXPORTER_OTLP_ENDPOINT</code></li><li><code>OTEL_EXPORTER_OTLP_HEADERS</code></li><li><code>AWS_XRAY_URL</code></li><li><code>AWS_XRAY_DAEMON_ADDRESS</code></li><li><strong><code>AWS_DEFAULT_REGION</code></strong></li><li><strong><code>AWS_ACCESS_KEY_ID</code></strong></li><li><strong><code>AWS_SECRET_ACCESS_KEY</code></strong></li></ul></td></tr><tr><td>x-ray daemon</td><td><ul><li><code>AWS_ACCESS_KEY_ID</code></li><li><code>AWS_SECRET_ACCESS_KEY</code></li><li><code>AWS_REGION</code></li></ul></td></tr></tbody></table>

***

## 1. Workflow

### 1. Implement a CloudWatch agent in backend

#### 1.1. Plant a logger in the flask server app.

The python library `watchtower` is a log handler for [Amazon Web Services CloudWatch Logs](https://aws.amazon.com/blogs/aws/cloudwatch-log-service/). Run `pip install` to install it in local.&#x20;

{% code title="requirements.txt" overflow="wrap" lineNumbers="true" %}
```diff
+ watchtower
```
{% endcode %}

* `pip install -r requirements.txt`

Update `app.py` by adding the following snippets across the file. Refer to [the actual code here](https://github.com/mariachiinajar/aws-bootcamp-cruddur-2023-again/blob/02-02-cloudwatch-logs/backend-flask/app.py).&#x20;

{% code title="app.py" lineNumbers="true" %}
```diff
+ import watchtower
+ import logging
+ from time import strftime
```
{% endcode %}

{% code title="app.py" lineNumbers="true" %}
```python
# CONFIGURE LOGGER TO USE CLOUDWATCH
LOGGER = logging.getLogger(__name__)
LOGGER.setLevel(logging.DEBUG)
console_handler = logging.StreamHandler()
cw_handler = watchtower.CloudWatchLogHandler(log_group='cruddur') # set a log group within AWS CloudWatch. The log group will be called "cruddur"
LOGGER.addHandler(console_handler)
LOGGER.addHandler(cw_handler)
LOGGER.info("test log")

## CLOUDWATH -----
@app.after_request
def after_request(response):
  timestamp = strftime('[%Y-%b-%d %H:%M]')
  LOGGER.error("%s %s %s %s %s %s", timestamp, request.remote_addr, request.method, request.scheme, request.full_path, response.status)
  return response
```
{% endcode %}

{% code title="app.py " lineNumbers="true" %}
```diff
@app.route("/api/activities/home", methods=['GET'])
+ def data_home(logger=LOGGER):
    data = HomeActivities.run()
    return data, 200
```
{% endcode %}

#### 1.2. Set logger in the service (home\_activities)

* [Andrew's code](https://github.com/omenking/aws-bootcamp-cruddur-2023/compare/main...week-2-cloudwatch-logs)

{% code title="home_activities.py" lineNumbers="true" %}
```diff
from datetime import datetime, timedelta, timezone
from opentelemetry import trace

...

class HomeActivities:
  def run():
+    logger.info("HomeActivities")

...
```
{% endcode %}



### 2. Update the backend docker container

The following env variables will be used by `watchtower`.

{% code title="docker-compose.yml" lineNumbers="true" %}
```diff
version: "3.8"
services:
  backend-flask:
    environment:
      ...
      AWS_XRAY_DAEMON_ADDRESS: "xray-daemon:2000"
+     AWS_DEFAULT_REGION: "${AWS_DEFAULT_REGION}"
+     AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
+     AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
    build: ./backend-flask
    ...
```
{% endcode %}

Now our app is ready to test the CloudWatch agent in action.&#x20;

* Run `docker-compose`&#x20;
  * then hit some backend endpoints by clicking around in the frontend
* Check your AWS CloudWatch console.

<div data-full-width="true">

<figure><img src="../../.gitbook/assets/image (52).png" alt=""><figcaption><p>Log streams are coming through. </p></figcaption></figure>

</div>
