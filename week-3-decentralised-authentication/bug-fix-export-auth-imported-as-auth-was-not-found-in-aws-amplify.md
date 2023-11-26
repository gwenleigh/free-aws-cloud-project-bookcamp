---
coverY: 0
layout:
  cover:
    visible: true
    size: full
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

# Bug Fix: export 'Auth' (imported as 'Auth') was not found in 'aws-amplify'

## 1. Problem

<div data-full-width="true">

<figure><img src="../.gitbook/assets/스크린샷 2023-11-19 140323.png" alt=""><figcaption></figcaption></figure>

</div>

* **line 8**: even though the code is there, the frontend container would still through the `undefined` error:&#x20;

{% code title="HomeFeedPage.js" lineNumbers="true" %}
```javascript
   const checkAuth = async () => {
    Auth.currentAuthenticatedUser({
      bypassCache: false
    })
    .then((user) => {
      console.log('user', user);
      return Auth.currentAuthenticatedUser()
    }).then((cognito_user) => {
      setUser({
        display_name: cognito_user.attributes.name,
        handle: cognito_user.attributes.preferred_username
      })
    })
    .catch((err) => console.log(err));
  }
```
{% endcode %}

## 2. Analysis

* It turns out, the upgraded version of `Amplify` was out. It's been more than 6 months since this Week 3 phase of development was out. As I'm starting all over again, I automatically got the latest version of 6.x.x when I ran `npm i`.

## 3. Solution

* [ ] Use the specific version Andrew used (`5.x.x`):

```diff
-    "aws-amplify": "^6.0.2",
+    "aws-amplify": "^5.0.16",
```

## 4. Result

* Amplify works correctly so throws the corresponding error correctly in the frontend.&#x20;
  * The Dev console error was gone.

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>
