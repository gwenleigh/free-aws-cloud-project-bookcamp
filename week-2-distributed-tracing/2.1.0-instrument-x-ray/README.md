---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# 2.1.0 Instrument X-Ray

## 0. **Learning Materials**

* Video: [Week 2 Instrument XRay](https://www.youtube.com/watch?v=n2DTsuBrD\_A\&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv\&index=30\&ab\_channel=ExamPro)
* Andrew's repo: [week-2-xray](https://github.com/omenking/aws-bootcamp-cruddur-2023/tree/week-2-xray)
* My branch repo: [02-01-instrument-xray](https://github.com/mariachiinajar/aws-bootcamp-cruddur-2023-again/tree/02-01-instrument-xray)
  * My commits:
    * [#9](https://github.com/mariachiinajar/aws-bootcamp-cruddur-2023-again/issues/9) [Instrumented X-ray (and container)](https://github.com/mariachiinajar/aws-bootcamp-cruddur-2023-again/commit/1b98ea63c3238ab5a0016b99174028e305edc01c)
    * [#9](https://github.com/mariachiinajar/aws-bootcamp-cruddur-2023-again/issues/9) [Implemented custom x-ray segment](https://github.com/mariachiinajar/aws-bootcamp-cruddur-2023-again/commit/def86e4f46123ebe9189d0010d785c3429fb024f)

### Task List

* [ ] Install AWS SDK
* [ ] Add X-ray daemon to Docker Compose
  * [ ] Add backend variables to Docker Compose

### Env variables

<table><thead><tr><th width="168">Micro service</th><th>Variables</th></tr></thead><tbody><tr><td>backend</td><td><ul><li><code>HONEYCOMB_API_KEY</code></li><li><code>OTEL_SERVICE_NAME</code></li><li><code>OTEL_EXPORTER_OTLP_ENDPOINT</code></li><li><code>OTEL_EXPORTER_OTLP_HEADERS</code></li><li><code>AWS_XRAY_URL</code></li><li><code>AWS_XRAY_DAEMON_ADDRESS</code></li></ul></td></tr><tr><td>x-ray daemon</td><td><ul><li><strong><code>AWS_ACCESS_KEY_ID</code></strong></li><li><strong><code>AWS_SECRET_ACCESS_KEY</code></strong></li><li><strong><code>AWS_REGION</code></strong></li></ul></td></tr></tbody></table>

***

## 1. Workflow

### 1.1 Install AWS SDK

* Install AWS SDK.

<pre class="language-diff" data-title="requirements.txt"><code class="lang-diff"><strong>+ aws-xray-sdk
</strong></code></pre>

* Then run:
  * `pip install -r requirements.txt`
* Create a json file for samling rules.
  * This is necessary because the X-Ray tracing can become expensive very quickly so we are reducing the amount of tgracing analysis by sampling only a limited amount.

```
cd aws
code xray.json
```



{% code title="xray.json" %}
```json
{
    "SamplingRule": {
        "RuleName": "Cruddur",
        "ResourceARN": "*",
        "Priority": 9000,
        "FixedRate": 0.1,
        "ReservoirSize": 5,
        "ServiceName": "backend-flask",
        "ServiceType": "*", 
        "Host": "*",
        "HTTPMethod": "*",
        "URLPath": "*",
        "Version": 1
    }
}
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>







```json
aws xray create-group \
    --group-name "Cruddur" \ 
    --filter-expression "service(\"$FLASK_ADDRESS\" {fault OR error}"
```

<div data-full-width="true">

<figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

</div>

<div data-full-width="true">

<figure><img src="../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

</div>



### 1.2 Add X-ray daemon to DC (Docker Compose)

{% code title="docker-compose.yml" lineNumbers="true" %}
```yaml
...
  xray-daemon:
    image: "amazon/aws-xray-daemon"
    environment:
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
      AWS_REGION: "us-east-1"
    command:
      - "xray -o -b xray-daemon:2000"

...
```
{% endcode %}

* **line 9**: run `xray` on port 2000.&#x20;
  * `-o`: local mode. :maple\_leaf: _By default, X-Ray thinks that it's running on an AWS EC2 instance and tries to grab the metadata from the instance_. The `-o` tag instructs X-ray to not check for EC2 instance metadata.
  * `-b`: Overrides default UDP address (127.0.0.1:2000).

### 1.2.1 Add backend variables in DC

Every time we add a container definition to the `docker-compose` file or we use env variables in the code base, it is important to add the env variables in the `docker-compose` file.

{% code title="docker-compose.yml" lineNumbers="true" fullWidth="true" %}
```
        AWS_XRAY_URL: "*4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
        AWS_XRAY_DAEMON_ADDRESS: "xray-daemon:2000"
```
{% endcode %}

<div data-full-width="true">

<figure><img src="../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

</div>

* We can [commit](https://github.com/omenking/aws-bootcamp-cruddur-2023/commit/1b98ea63c3238ab5a0016b99174028e305edc01c) the implementation at this point.&#x20;

## 1.2 Troubleshooting



<div data-full-width="true">

<figure><img src="../../.gitbook/assets/image (10) (1).png" alt=""><figcaption><p>The Docker container seems to be running. Inside the contianer, I find that the AWS credentials are not correctly injected into the container. </p></figcaption></figure>

</div>

<div data-full-width="true">

<figure><img src="../../.gitbook/assets/image (11) (1).png" alt=""><figcaption><p>I get no data on X-Ray console. Something must be wrong. </p></figcaption></figure>

</div>

<figure><img src="../../.gitbook/assets/image (12) (1).png" alt=""><figcaption><p>No credentials provided</p></figcaption></figure>

<div data-full-width="true">

<figure><img src="../../.gitbook/assets/image (13) (1).png" alt=""><figcaption><p>Hardcoded the credential values into the docker-compose file did the magic <br>(This is a very poor thing to do in terms of security. I'm doing it this way only because I'm testing things out!).</p></figcaption></figure>

</div>

<figure><img src="../../.gitbook/assets/image (14) (1).png" alt=""><figcaption><p>The span data are coming through to X-ray on AWS Cloud.</p></figcaption></figure>







##

## 2. Discussion&#x20;

### 2.1 AWS X-Ray

<figure><img src="../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

* In our Bootcamp context, this X-ray is another container running alongside our application (the other containers - `backend-flask`, `frontend-react-js`, `dynamodb-local`, and `db`).
* **X-Ray** collects the data from our local machine, and send them in batches to AWS using the AWS APIs.&#x20;

### 2.2 Middleware

* _Middleware is a type of_ [_computer software_](https://en.wikipedia.org/wiki/Computer\_software) _programme that provides services to software applications beyond those available from the_ [_operating system_](https://en.wikipedia.org/wiki/Operating\_system)_. It can be described as "software glue"_ ([Wikipedia](https://en.wikipedia.org/wiki/Middleware))_._
*

