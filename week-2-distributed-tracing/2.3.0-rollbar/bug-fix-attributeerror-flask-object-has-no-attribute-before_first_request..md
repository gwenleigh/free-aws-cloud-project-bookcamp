# Bug Fix: AttributeError: 'Flask' object has no attribute 'before\_first\_request'.

## 1. Problem

<div data-full-width="true">

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption><p>The backend is broken. The backend container stopped and exited.</p></figcaption></figure>

</div>

* The backend is broken, the frontend is unable to fetch data from the backend.
* The backend docker container is down.

## 2. Analysis

* The backend docker container log shows that something is wrong with the flask decorator.&#x20;
*   The `@app.before_first_request` decorator ensures that the `init_rollbar()` function is executed before the first request to the application.&#x20;

    * Suitable for one-off initialization tasks in Flask

    <figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption><p>Python is misleading here. @app.before_first_request is correct if ever, because it is for one-off task that has to execute prior to any request.</p></figcaption></figure>

## 3. Solution

* [ ] &#x20;The decorator `before_first_request` is depricated after Flask 2.2.&#x20;
  * [ ] Use `app.app_context()` instead.
  * [ ] When indent the whole `def init_rollbar()` function into the `with` statement.

{% code title="app.py" lineNumbers="true" %}
```python
with app.app_context():
  def init_rollbar():
      """init rollbar module"""
      rollbar.init(
          # access token
          'YOUR_ACCESS_TOKEN'
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

## 4. Result

* The Flask server is up and running again (the backend container is running!)
* The rollbar endpoint `/rollbar/test` is working.&#x20;

<div data-full-width="true">

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

</div>
