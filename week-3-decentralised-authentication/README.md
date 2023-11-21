# Week 3 Decentralised Authentication

## **0. Learning Materials**

* Video: [FREE AWS Cloud Project Bootcamp (Week 3) - Decentralized Authentication](https://www.youtube.com/watch?v=9obl7rVgzJw\&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv\&index=40\&ab\_channel=ExamPro)
* Andrew's repo: [week-3-again](https://github.com/omenking/aws-bootcamp-cruddur-2023/tree/week-3-again)
* My branch repo: [03-decentralized-authentication](https://github.com/mariachiinajar/aws-bootcamp-cruddur-2023-again/tree/03-decentralized-authentication)
  * My commits:&#x20;
    * [Implemented Amplify in the frontend](https://github.com/mariachiinajar/aws-bootcamp-cruddur-2023-again/commit/4309a4c77341c9be928268f12efe826a1b014742)

### &#x20;Task List

* [ ] 1.1. Create Cognito Userpool in AWS Cognito
* [ ] 1.2. Integrate Cognito to the Frontend
  * [ ] 1.2.1. Setup: install packages
  * [ ] 1.2.2. Embed AWS Amplify in the Frontend application

### Env Variables

<table><thead><tr><th width="168">Micro service</th><th>Variables</th></tr></thead><tbody><tr><td>backend</td><td><ul><li><code>HONEYCOMB_API_KEY</code></li><li><code>OTEL_SERVICE_NAME</code></li><li><code>OTEL_EXPORTER_OTLP_ENDPOINT</code></li><li><code>OTEL_EXPORTER_OTLP_HEADERS</code></li><li><code>AWS_XRAY_URL</code></li><li><code>AWS_XRAY_DAEMON_ADDRESS</code></li><li><code>AWS_DEFAULT_REGION</code></li><li><code>AWS_ACCESS_KEY_ID</code></li><li><code>AWS_SECRET_ACCESS_KEY</code></li><li><code>ROLLBAR_ACCESS_TOKEN</code></li></ul></td></tr><tr><td>frontend</td><td><ul><li><strong><code>REACT_APP_PROJECT_REGION</code></strong></li><li><strong><code>REACT_APP_AWS_COGNITO_REGION</code></strong></li><li><strong><code>REACT_APP_AWS_USER_POOLS_ID</code></strong></li><li><strong><code>REACT_APP_CLIENT_ID</code></strong></li></ul></td></tr><tr><td>x-ray daemon</td><td><ul><li><code>AWS_ACCESS_KEY_ID</code></li><li><code>AWS_SECRET_ACCESS_KEY</code></li><li><code>AWS_REGION</code></li></ul></td></tr></tbody></table>

***

## 1. Workflow

### 1.1. Create Cognito Userpool in AWS Cognito

You could tailor the Cognito userpool configurations during creation based on your needs. I have listed the configurations Andrew used for this Cruddur project beneath the image.&#x20;

<figure><img src="../.gitbook/assets/image (60).png" alt="" width="375"><figcaption></figcaption></figure>

**Step 1.**

* Cognito user pool sign-in options: :white\_check\_mark:User name, :white\_check\_mark:Email

**Step 2.**&#x20;

* **Password Policy**
  * **Password policy mode**: :radio\_button: Cognito defaults&#x20;
  * **Multi-factor authentication**: :radio\_button: No MFA
  * **User account recovery**
    * **Self-service account recovery**: :white\_check\_mark: Enable self-service account recovery - Recommended
    * **Delivery method for user account recovery messages**: :radio\_button:  Email only

**Step 3.**

* **Self-service sign-up**: Self-registration: :white\_check\_mark: Enable self-registration
* **Attribute verification and user account confirmation**
  * Cognito-assisted verification and confirmation
    * :white\_check\_mark: Allow Cognito to automatically send messages to verify and confirm - Recommended
  * Attributes to verify: :radio\_button:Send email message, verify email address
*   #### Verifying attribute changes

    :white\_check\_mark: Keep original attribute value active when an update is pending - Recommended

    * **Active attribute values when an update is pending**: :radio\_button:Email address
* **Required attributes**:

**Step 4**&#x20;

* **Email: Email provider**: :radio\_button: Send email with Cognito

**Step 5**&#x20;

* **User pool name**: `cruddur-user-pool`&#x20;
* :no\_entry: DO NOT Use the Cognito Hosted UI Initial app
* **Initial app client: App type**: :radio\_button: Public client

Then create the pool.



### 1.2. Integrate Cognito to the Frontend

#### 1.2.1. Setup: install packages

Install the `aws-amplify` npm library in the frontend. Make sure that you are at the root of the frontend application.

```bash
cd frontend-react-js
npm i aws-amplify --save
```

* `--save`: save the library to the `package.json`.

After installing, make sure that the package is included in the updated `package.json`.&#x20;

<figure><img src="../.gitbook/assets/image (63).png" alt="" width="442"><figcaption></figcaption></figure>

#### 1.2.2. Implement AWS Amplify in the Frontend application.

:desktop: Coding time! Now let's plant `Amplify` in our frontend application.

{% code title="App.js" overflow="wrap" lineNumbers="true" fullWidth="true" %}
```jsx
import { Amplify } from 'aws-amplify';

Amplify.configure({
  "AWS_PROJECT_REGION": process.env.REACT_APP_AWS_PROJECT_REGION,
  "aws_cognito_region": process.env.REACT_APP_AWS_COGNITO_REGION,
  "aws_user_pools_id": process.env.REACT_APP_AWS_USER_POOLS_ID,
  "aws_user_pools_web_client_id": process.env.REACT_APP_CLIENT_ID,
  "oauth": {},
  Auth: {
    // We are not using an Identity Pool
    // identityPoolId: process.env.REACT_APP_IDENTITY_POOL_ID, // REQUIRED - Amazon Cognito Identity Pool ID
    region: process.env.REACT_APP_AWS_PROJECT_REGION,           // REQUIRED - Amazon Cognito Region
    userPoolId: process.env.REACT_APP_AWS_USER_POOLS_ID,         // OPTIONAL - Amazon Cognito User Pool ID
    userPoolWebClientId: process.env.REACT_APP_CLIENT_ID,   // OPTIONAL - Amazon Cognito Web Client ID (26-char alphanumeric string)
  }
});
```
{% endcode %}

* **lines 4-14**: `process.env` is a way for JavaScript to access the env variables stored locally and load them into a React application. A bit more about it [here](./#2.2.-process.env).&#x20;

#### 1.2.3. Update the checkAuth function HomeFeedPage.js

Now, replace the existing `checkAuth()` function that uses `Cookie` with the following:

{% code title="HomeFeedPage.js" lineNumbers="true" fullWidth="true" %}
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

* :maple\_leaf: **line 3**: _(optional) by default is `false`._&#x20;
  * :maple\_leaf: _If set to true, this call will send a request to Cognito to get the latest user data._
* :maple\_leaf: **line 7**: what `Auth.currentAuthenticatedUser()` returns will become `cognito_user` in line 8.
* **lines 9-12**: _(optional)_ which will be then used to populate the user's data.&#x20;

#### 1.2.4. Implement Cognito Auth in ProfileInfo.js

Replace the `signOut()` function with Cookie with a new version that uses Cognito's `Auth.signOut()`.

{% code title="ProfileInfo.js" lineNumbers="true" %}
```javascript
import { Auth } from "aws-amplify";

  const signOut = async () => {
    try {
      await Auth.signOut({ global: true });
      window.location.href = "/"
    } catch (error) {
      console.log('error signing out: ', error)
    }
  }
```
{% endcode %}

#### 1.2.5. Implement Cognito Auth in SignIn.js







## 2. Discussion: AWS Amplify

### 2.1. Amplify

AWS Amplify is a comprehensive set of tools and services to simplify the process of building secure and scalable web and mobile applications with a wide range of features which include but not limited to:&#x20;

* **Authentication** (we are using this one in week 3)
* API management&#x20;
* Analytics and monitoring&#x20;
* Hosting Frontend libraries and UI components

<figure><img src="../.gitbook/assets/image (61).png" alt=""><figcaption><p>Random moment from Week 3 livestreaming :D</p></figcaption></figure>

### 2.2. process.env

* In JavaScript, `process.env` is an object that provides access to environment variables within a Node.js application.
  * You have to create a separate file named `.env` and store your local variables there.
  * Make sure that the `.env` file is located in the React project's root directory.
  * You can name the `.env` files depending on your development phase as in:
    * `.env.development`
    * `.env.production`, etc.
  * `REACT_APP_`: in order for React to auto-load the variables, we need this prefix before every variable we use.

### 2.3. package-lock.json

* "_`package-lock.json` is automatically generated for any operations where npm modifies either the `node_modules` tree, or `package.json`. It describes the exact tree that was generated, such that subsequent installs are able to generate identical trees, regardless of intermediate dependency updates_." (npm, [package-lock.json](https://docs.npmjs.com/cli/v10/configuring-npm/package-lock-json))
* The `package-lock.json` file is automatically generated and updated by `npm` whenever a new package is installed or the versions of existing packages are changed.&#x20;
  * It's recommended to commit the `package-lock.json` file together with the `package.json` file so we can save any team members (or your future self) [some trouble](bug-fix-export-auth-imported-as-auth-was-not-found-in-aws-amplify.md).
