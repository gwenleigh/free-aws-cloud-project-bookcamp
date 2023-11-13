# 2.2.0 CloudWatch Logs

### **Learning Materials**

* Video: [Week 2 CloudWatch Logs](https://www.youtube.com/watch?v=ipdFizZjOF4\&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv\&index=31\&ab\_channel=ExamPro)
* Andrew's repo: [week-2-cloudwatch-logs](https://github.com/omenking/aws-bootcamp-cruddur-2023/tree/week-2-cloudwatch-logs)
* My branch repo:&#x20;

### Task List

* [ ] a
* [ ] b
* [ ] c
* [ ] d



### Workflow

Add the python library `watchtower` which is a  log handler for [Amazon Web Services CloudWatch Logs](https://aws.amazon.com/blogs/aws/cloudwatch-log-service/). Then run the `pip install` command to install it in our local environment.&#x20;

{% code title="requirements.txt" overflow="wrap" lineNumbers="true" %}
```diff
+ watchtower
```
{% endcode %}

* `pip install -r requirements.txt`



Update `app.py`.

{% code title="app.py" lineNumbers="true" %}
```diff
+ import watchtower
+ import logging
+ from time import strftime
```
{% endcode %}



Add the following to the `app.py` where they belong. Refer to the actual code.&#x20;

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





{% code title="home_activities.py" lineNumbers="true" %}
```diff
from datetime import datetime, timedelta, timezone
from opentelemetry import trace

...

class HomeActivities:
  def run(logger):
    logger.info("HomeActivities")
```
{% endcode %}











