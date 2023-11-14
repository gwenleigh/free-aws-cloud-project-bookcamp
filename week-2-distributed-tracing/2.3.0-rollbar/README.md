# 2.3.0 Rollbar

## 0. **Learning Materials**

* Video: [Week 2 - Rollbar](https://www.youtube.com/watch?v=xMBDAb5SEU4\&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv\&index=33\&ab\_channel=ExamPro)
* Andrew's repo: [week-2-rollbar](https://github.com/omenking/aws-bootcamp-cruddur-2023/tree/week-2-rollbar)
* My branch repo: [02-03-rollbar](https://github.com/mariachiinajar/aws-bootcamp-cruddur-2023-again/tree/02-03-rollbar)

### Task List

* [ ] Implement Rollbar
  * [ ] Update `requirements.txt`
  * [ ] Implement Rollbar in `app.py`
* [ ] Update the backend-container of `docker-compose`

### Env Variables

<table><thead><tr><th width="168">Micro service</th><th>Variables</th></tr></thead><tbody><tr><td>backend</td><td><ul><li><code>HONEYCOMB_API_KEY</code></li><li><code>OTEL_SERVICE_NAME</code></li><li><code>OTEL_EXPORTER_OTLP_ENDPOINT</code></li><li><code>OTEL_EXPORTER_OTLP_HEADERS</code></li><li><code>AWS_XRAY_URL</code></li><li><code>AWS_XRAY_DAEMON_ADDRESS</code></li><li><code>AWS_DEFAULT_REGION</code></li><li><code>AWS_ACCESS_KEY_ID</code></li><li><code>AWS_SECRET_ACCESS_KEY</code></li><li><strong><code>ROLLBAR_ACCESS_TOKEN</code></strong></li></ul></td></tr><tr><td>x-ray daemon</td><td><ul><li><code>AWS_ACCESS_KEY_ID</code></li><li><code>AWS_SECRET_ACCESS_KEY</code></li><li><code>AWS_REGION</code></li></ul></td></tr></tbody></table>

***

## 1. Workflow

### 1. Implement Rollbar

#### 1.1.  Update requirements.txt

[Rollbar](https://rollbar.com/) is an error monitoring platform, similar to all the other observability tools we have implemented so far - AWS X-Ray, CloudWatch and [Honeycomb](https://www.honeycomb.io/).

{% code title="requirements.txt" overflow="wrap" lineNumbers="true" %}
```diff
+ blinker
+ rollbar
```
{% endcode %}

* Now, run `pip install -r requirements.txt` so the Rollbar libraries are installed locally.

#### 1.2. Implement rollbar in app.py

Below is the original code provided by Rollbar to setup Rollbar SDK. However, the Flask server will break cause it's buggy.

{% code title="app.py" lineNumbers="true" %}
```python
from time import strftime
import os
import rollbar
import rollbar.contrib.flask
from flask import got_request_exception

@app.before_first_request
def init_rollbar():
    """init rollbar module"""
    rollbar.init(
        # access token
        'ROLLBAR_ACCESS_TOKEN',
        # environment name
        'production',
        # server root directory, makes tracebacks prettier
        root=os.path.dirname(os.path.realpath(__file__)),
        # flask already sets up logging
        allow_logging_basic_config=False)

    # send exceptions from `app` to rollbar, using flask's signal system.
    got_request_exception.connect(rollbar.contrib.flask.report_exception, app)
```
{% endcode %}

In order to add the working code to `app.py` across the file where appropriate, refer to:

* [the working code](https://github.com/mariachiinajar/aws-bootcamp-cruddur-2023-again/commit/465723776c1b9daee1799896ed7d06ccf93822a0) for `init_rollbar()`
* [app.py](https://github.com/mariachiinajar/aws-bootcamp-cruddur-2023-again/blob/02-03-rollbar/backend-flask/app.py)

Also refer to the troubleshooting log ["Bug Fix: pyrollbar: No access\_token provided."](bug-fix-pyrollbar-no-access\_token-provided..md) here.



### 2. Update the backend-container of docker-compose

Update the environment variable for the `backend-container` in `docker.compose`.

{% code title="docker-compose.yml" lineNumbers="true" %}
```diff
version: "3.8"
services:
  backend-flask:
    environment:
+      ROLLBAR_ACCESS_TOKEN: "${ROLLBAR_ACCESS_TOKEN}"
```
{% endcode %}

Then run `docker-compose`.
