# 1. Deploy Node.js code to AWS Lambda

### Steps

1. Install the `serverless` framework globally

   ```
   npm i -g serverless
   ```

2. The serverless framework needs to access `AWS S3`, `AWS Lambda`, and `AWS CloudForMATION` so go to your root account and provide these rights to your IAM account

3. Setup your AWS secret keys in serverless

```
$ sls config credentials \ --provider aws \ --key PUBLIC_KEY \ --secret SECRET_KEY
```

- We need to add our AWS security configs to serverless, so the serverless framework can access our AWS Lambda account
- Replace the PUBLIC_KEY and the SECRET_KEY with your own creds

4. Create a boiler plate code

```
mkdir serverless-nodejs-app && cd serverless-nodejs-app
```

5. We are now left with running the create command to generate some starter code for us. This is called a serverless service

```
sls create -t aws-nodejs -n serverless-nodejs-app
```

6. Installing dependencies

```
npm init -y
npm i --save express serverless-http
```

7. Change the name of the`handler.js` file to `app.js`, and delete all the starter code inside `app.js`

```js
// app.js

const express = require("express");
const sls = require("serverless-http");
const app = express();
app.get("/", async (req, res, next) => {
  res.status(200).send("Hello World!");
});
module.exports.server = sls(app);
```

8. Modify the `serverless.yml` file

```yaml
# serverless.yml

service: serverless-nodejs-app

provider:
  name: aws
  runtime: nodejs14.x
  stage: dev
  region: eu-central-1

functions:
  app:
    handler: app.server # reference the file and exported method
    events: # events trigger lambda functions
      - http: # this is an API Gateway HTTP event trigger
          path: /
          method: ANY
          cors: true
      - http: # all routes get proxied to the Express router
          path: /{proxy+}
          method: ANY
          cors: true
```

9. Deploy the code

```
sls deploy
```

You will get a domain in your terminal if the deployment succides

10. Deploy the project in production

- Add a `secrets.json` file for environment variables

  ```json
  {
    "NODE_ENV": "production"
  }
  ```

- Add a reference for the `secrets.json` in the serverless.yml file

  ```yaml
  service: serverless-nodejs-app
  custom: # add these two lines
    secrets: ${file(secrets.json)} # reference the secrets.json file
  provider:
    name: aws
    runtime: nodejs14.x
    stage: production # make sure to change this to production
    region: eu-central-1
    environment: # add environment property
      NODE_ENV: ${self:custom.secrets.NODE_ENV}
      # reference the NODE_ENV from the secrets.json file
  functions:
    app:
      handler: app.server
      events:
        - http:
            path: /
            method: ANY
            cors: true
        - http:
            path: /{proxy+}
            method: ANY
            cors: true
  ```

- Delete the `node_modules` folder and `.serverless` folder from your project folder and run

  ```
  npm install --production
  ```

- Deploy the project again

  ```
  sls deploy
  ```

**NOTE:** There was a ROLL_BACK error when i tried to deploy the project with serverless and it was solved by using NodeJs version 14.x in the serverless.yml file

### Resource

[Resource](https://dashbird.io/blog/how-to-deploy-nodejs-application-aws-lambda/)
