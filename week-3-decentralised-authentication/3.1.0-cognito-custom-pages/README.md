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

# 3.1.0 Cognito Custom Pages

## **0. Learning Materials**

* Video:&#x20;
* Andrew's repo:&#x20;
* My branch repo:&#x20;

### &#x20;Task List

* [ ] a
* [ ] b
* [ ] c
* [ ] d

***

## 1. Workflow

### 1.1 Test user account.&#x20;

<figure><img src="../../.gitbook/assets/image (64).png" alt=""><figcaption></figcaption></figure>



```bash
aws cognito-idp admin-set-user-password \
  --user-pool-id <your-user-pool-id> \
  --username <username> \
  --password <password> \
  --permanent
```

<figure><img src="../../.gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>



## 2. Discussion

