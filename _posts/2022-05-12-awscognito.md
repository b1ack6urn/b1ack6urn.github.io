---
title: "Super Admin Privilege Escalation from Misconfigured AWS Cognito"
date: 2022-05-12
categories:
  - VAPT
tags:
  - Penetration Testing
  - Privilege Escalation
  - AWS Cognito
---
### Scope of Work
The Scope of Work and its Details are Redacted under NDA.
{: .notice--danger}

### Introduction
The Web Application was an Exam Portal where the faculties can create/schedule Online Exams, invite and assign tests to the students. Students can then login and give their respective exams.

The Portal **did not** provide a feature to update any of our details. We need to request the Admins for any updation.

---
### AWS Cognito

I intercepted the Login request and saw the unusual headers `X-Amz-Cf-Pop`, `X-Amz-Cf-Id` which upon a simple Google Search gave results that the AWS Cognito is set in place for authentication.

Now some background on how AWS Cognito works.

|![AWS Cognito](https://miro.medium.com/max/1400/1*5Lwi-RU4Sq1DYvKn91dukg.png)|
|:--:|
|Image taken from appsecco.com|

1. In order to login to the Application, the Cognito's Login page is presented.
2. On Login, the credentials are checked against the User Pool.
3. On Successful validation, the Server gives back the JWT, which then is used to connect with the Application.

```
HTTP/2 200 OK
[...]{
    "AuthenticationResult":    
        {
            "AccessToken":"[REDACTED]",
            "ExpiresIn":3600,
            "IdToken":"[REDACTED]",
            "RefreshToken":"[REDACTED]",
            "TokenType":"Bearer"
        },
        "ChallengeParameters":
        {            
        }
}
```

So, the login flow in the app I was testing was as follows.

- Login to the App, a POST request is sent to Amazon Cognito
- If creds are valid, Amazon Cognito provides the tokens.



---
### Exploitation

Having worked with AWS before I knew that this token can be used in `awscli`. And so I was able to fetch user information from USER POOL like:

```bash
aws --no-verify-ssl cognito-idp get-user 
--region <[redacted]> --access-token <[redacted]>

...

  "Username": "[redacted]",
  "UserAttributes": [
    {
      "Name": "sub",
      "Value": "[redacted]"
    },
    {
      "Name": "email_verified",
      "Value": "true"
    },
    {
      "Name": "name",
      "Value": "[redacted]"
    },
    {
      "Name": "email",
      "Value": "[redacted]"
    },
    {
      "Name": "custom:userid",
      "Value": "[redacted]"
    }
  ],
}
```

- The user attributes can be modified with the help of `update-user-attributes`. 
I tried changing the email and name fields but they did not work.
Then the `custom:userid` got my eye and I tried changing it, that worked.
# `@_@`

- Now I logged in to a different user, got his `custom:userid` and updated my user with the new userid.
It successfully updated the value and when I logged-in I got all the privileges of that user. 
# `*_*`

- The next thing was to get the Super Admin's `userid`, which I got from the **API data leak vulnerability** from the same scope on which I wrote [this]({% post_url 2022-05-11-leakyapi %}) article.


- Upon getting the Super Admin's `userid` and updating it on my user, I got my Privileges Escalated to Super Admins.
# \ (•◡•) /


---
### Threat & Impact
Since it was a 3-day engagement I had to sum up the engagement on successful privilege escalation. Hence I could not leverage it further to Remote Code Execution from the Admin Panel.

Anyways, that still brings the threat that it can be chained with a social engineering attack to trick Faculties, Students and other users for malicious intent, etc

The Possibilities are endless once you are a _SUPER ADMIN_.


---
### Mitigation
- Remove sensitive details from server responses, including Cognito Identity Pool Id. Server must return minimal data whenever a server request is made.
- Create a Policy in AWS to disable regular Users to Write/Update specific User Attributes.
- From AWS: [Identity-based policy examples for Amazon Cognito](https://docs.aws.amazon.com/cognito/latest/developerguide/security_iam_id-based-policy-examples.html)

----