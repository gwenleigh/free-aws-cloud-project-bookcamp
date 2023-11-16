---
description: Observability and tracing
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

# Week 3 Resources

### **Week 3 Live stream**

* [OpenTelemetry](https://opentelemetry.io/)
* [HoneyComb in Python](https://docs.honeycomb.io/getting-data-in/opentelemetry/python-distro/): sending log data to your HoneyComb environment
* HoneyComb trace - [who am I](https://honeycomb-whoami.glitch.me/)
*

### 2.1.0 Instrument XRay

* StackOverFlow: [What are the best practises for setting up x-ray daemon?](https://stackoverflow.com/questions/54236375/what-are-the-best-practises-for-setting-up-x-ray-daemon)
* Docker: [amazon/aws-xray-daemon](https://hub.docker.com/r/amazon/aws-xray-daemon)
  * Github: [aws / aws-xray-daemon](https://github.com/aws/aws-xray-daemon)
* [AWS Xray](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/xray.html): boto3 XRay client documentation
  * Github: [AWS XRay SDK Py11thon](https://github.com/aws/aws-xray-sdk-python)
  * [Boto3](https://github.com/boto/boto3): Python AWS SDK
  * [Segment Dynamic Naming](https://docs.aws.amazon.com/xray-sdk-for-python/latest/reference/configurations.html#segment-dynamic-naming)

### 2.2.0 CloudWatch Logs

* Python
  * [watchtower](../week-1-dockerise-app/): a log handler for [Amazon Web Services CloudWatch Logs](https://aws.amazon.com/blogs/aws/cloudwatch-log-service/).
  * AWS: [Amazon CloudWatch Logs endpoints and quotas](https://docs.aws.amazon.com/general/latest/gr/cwl\_region.html)

### 2.3.0 Rollbar

* [Rollbar](https://rollbar.com/): error monitoring platform
  * [Rollbar SDK configuration reference](https://docs.rollbar.com/docs/python-configuration-reference)
* StackOverFlow: [flask-deprecated-before-first-request-how-to-update](https://stackoverflow.com/questions/73570041/flask-deprecated-before-first-request-how-to-update/74629704#74629704)

### 2.4.0 X-Ray Subsegments

* Olga Timofeeva's blog: [How to add custom X-Ray Segments for Containerised Flask Application](https://olley.hashnode.dev/aws-free-cloud-bootcamp-instrumenting-aws-x-ray-subsegments)
* [Sending segment batch failed with: NoCredentialProviders: no valid provider #59](https://github.com/aws/aws-xray-daemon/issues/59)
* **AWS**&#x20;
  * [AWS X-Ray concepts](https://docs.aws.amazon.com/xray/latest/devguide/xray-concepts.html#xray-concepts-segments)
  * [Generating custom subsegments with the X-Ray SDK for Python](https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-python-subsegments.html)

### Brilliant

* Josh Hargett's blog: [the amazing summary of the entire week 3](https://awstip.com/week-3-aws-cloud-bootcamp-a9efd2b51f59)
