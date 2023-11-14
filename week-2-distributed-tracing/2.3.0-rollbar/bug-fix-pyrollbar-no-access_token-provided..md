---
description: >-
  Credentials and authentication issues preventing the rollbar agent in our
  backend from calling home.
---

# Bug Fix: pyrollbar: No access\_token provided.

## 1. Problem

* Although the rollbar `access_token` is configured with the python variable `rollbar_access_token`, the agent isn't calling home, and no data are coming through on the Rollbar side.

<div data-full-width="true">

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

</div>

## 2. Analysis

* Something must be wrong with the variable `rollbar_access_token` in line 94 (in the above image).&#x20;
* After good dozens of minutes, I finally figured out: there are multiple `access_tokens` at varying levels in Rollbar - one at the project level (for each project) and the other at the account level.&#x20;
* I was using the account level token where I have to input the project access token.

## 3. Solution

* [x] Grab the access token from the project.&#x20;
  * [x] `export` and `gp env` ROLLBAR\_ACCESS\_TOKEN

```
export ROLLBAR_ACCESS_TOKEN="YOUR_ROLLBAR_TOKEN_FROM_PROJECT"
gp env ROLLBAR_ACCESS_TOKEN="YOUR_ROLLBAR_TOKEN_FROM_PROJECT"
```

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

However, I still get the same error that the `access_token` is not provided. Well... the error message was quite stragithforward in fact. It's been telling me to "call" the `rollbar.init()` function.

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

* [x] Reorganise the Rollbar code.
* [x] Call `init_rollbar()` under `with app.app_context():` function.

{% code title="app.py" lineNumbers="true" %}
```python
# ROLLBAR -------------------------
rollbar_access_token = os.getenv('ROLLBAR_ACCESS_TOKEN')

def init_rollbar():
    """init rollbar module"""
    rollbar.init(
        # access token
        rollbar_access_token,
        # environment name
        'production',
        # server root directory, makes tracebacks prettier
        root=os.path.dirname(os.path.realpath(__file__)),
        # flask already sets up logging
        allow_logging_basic_config=False)

    # send exceptions from `app` to rollbar, using flask's signal system.
    got_request_exception.connect(rollbar.contrib.flask.report_exception, app)


# @app.before_first_request # this decorator will break the code as it is depricated after Flask 2.2 
with app.app_context():     # use this 'with' statement instead. 
  init_rollbar()

# ROLLBAR TEST---------------------
@app.route('/rollbar/test')
def rollbar_test():
    rollbar.report_message('Hello Rollbar!', 'warning')
    return "Hello, Rollbar!"

```
{% endcode %}

## 4. Result

* The error message `pyrollbar: No access_token provided` is gone.&#x20;
* I check the Rollbar console and the spans are coming through.

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption><p>Rollbar listing the tracing entry.</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption><p>A closer look into the details of an error span.</p></figcaption></figure>
