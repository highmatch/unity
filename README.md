# HighMatch / Joynd Integration

## Client ATS

The *first* HighMatch/Joynd client uses **Bullhorn**. 

The *second* client is currently using **Greenhouse** and plans to move to **ADP** in late 2024. They may want to start on Greenhouse, then migrate to ADP. We are waiting to hear their plans.

HighMatch plans to integrate the first client and achieve stability *before* starting deployment of the second client (though planning for the second can start earlier). We do not want greenfield coding problems from the initial coding effort to impact two clients (if possible).

## Dev Environment Criteria

**Dev Client ID:** `dev-highmatch`

**Inbound HighMatch Dev URLs:**

-  Catalog: GET https://highmatch.ngrok.io/joynd/catalog

-  Order: POST https://highmatch.ngrok.io/joynd/order

**Basic Authentication Values:**

-  Provided separately (this document is semi-public)

-----

**Inbound HighMatch QA/Staging URLs **

*Not initially used. This will be for final testing prior to production deploment.*

-  Catalog: GET https://api.compass.qa-highmatch.com/joynd/catalog
-  Catalog: POST https://api.compass.qa-highmatch.com/joynd/order

**Basic Authentication Values:**

-  Provided separately (this document is semi-public)

-----

**Inbound HighMatch Production URLs**

*Not available until the QA/Staging environment is certified as operational.*

-  Catalog: GET https://api.compass.highmatch.com/joynd/catalog
-  Catalog: POST https://api.compass.highmatch.com/joynd/order

**Basic Authentication Values:**

-  Provided separately (this document is semi-public)

## Unity API Documentation

[https://app.swaggerhub.com/apis-docs/hrnx4/HRNXScreening/2.1.0](https://app.swaggerhub.com/apis-docs/hrnx4/HRNXScreening/2.1.0)

## Unity API Questions

### ~C#: GET /Catalog QUESTIONS 

[https://app.swaggerhub.com/apis-docs/hrnx4/HRNXScreening/2.1.0#/Screening/Packages](https://app.swaggerhub.com/apis-docs/hrnx4/HRNXScreening/2.1.0#/Screening/Packages )

```json
{
  "Catalog": [
    {
      "package": {
        "packageId": "string",
        "name": "string",
        "services": [
          {
            "serviceId": "string",
            "serviceName": "string"
          }
        ]
      }
    }
  ]
}
```

#### ~C1. What does the Services array do in GET /Catalog?

When a catalog request arrives, HighMatch will return an array of `package` objects where `package.packageId` is the ID you will send for a new Order and `package.name` is what is displayed in the target ATS.

What does the `package.services` array do?

#### ~C2. Error Handling - 405

Swagger shows a 405 error code under GET Catalog. Should we return this if the `clientId` parameter is invalid (or no longer a valid HighMatch customer)? How does Joynd respond if a 405 is returned?

#### ~C3. Error Handling - 503

Swagger shows a 503 error code under GET Catalog. Should we return this when we are offline for maintenance? How does Joynd (and the source ATS) respond if a 503 is returned?

#### ~C4. Error Handling - Unable to Connect

-  In the event Joynd cannot connect to Compass, what is the retry pattern/interval? 
-  In the event Joynd cannot connect to Compass, is the end-user (the ATS user) impacted (i.e. is this synchronous)? If so, what happens?

#### ~C5. Error Handling - 400

It appears if we have any other problem, we should return the following model:

```json
{
  "errorCode": "string",
  "errorMessage": "string",
  "errorDescription": "string",
  "errorSuggestedAction": "string"
}
```

-  What happens to this information?
-  Does it cancel a retry callback? 
-  Are there any specific `errorCode` value(s) we need to return?

### ~OI#: POST /Order `orderInfo` Questions

[https://app.swaggerhub.com/apis-docs/hrnx4/HRNXScreening/2.1.0#/Screening/Order](https://app.swaggerhub.com/apis-docs/hrnx4/HRNXScreening/2.1.0#/Screening/Order)

##### ~OI1. Questions on `orderInfo` fields that may not be relevant to HighMatch.

```json
{
  "orderInfo": {
    "requesterRefId": "string", // ?
    "clientId": "string", 
    "packageId": "string",
    "jobId": "string", // ?
    "jobTitle": "string", // ?
    "jobScoringRef": "string", // ?
    "requesterId": "string", // ?
    "requesterEmail": "string", // ?
    "emailInvite": true,
    "billingReference1": "string", // ?
    "billingReference2": "string" // ?
  }
}
```

-  `orderInfo.requesterRefId` 

   Is this a Joynd ID or the origin ATS user ID that initiated the Order? How is it different from `orderInfo.requesterId`?

-  `orderInfo.jobId` 

   Is this a Joynd ID or the origin ATS job ID for `jobTitle`?

-  `orderInfo.jobTitle` 

   Is this the job to which the candidate is applying? Is it always included in the payload?

-  `jobScoringRef`

   What is this for?

-  `requesterId`

   Is this a Joynd ID or the origin ATS user ID for `requesterEmail`?

-  `requesterEmail`

   Is this the email address of the origin ATS user that initiated the Order? Is it always included in the payload?

-  `billingReference1` and `billingReference1` 

   Are these relevant to HighMatch? How are they typically used by other Joynd subscribers?

#### ~OI2. If `orderInfo.emailInvite` is `true`, HighMatch should invite the candidate to complete the assessment via email?

```json
{
  "orderInfo": {
    "clientId": "string", 
    "packageId": "string",
    "emailInvite": true //if false, can HM assume the ATS is handling invitation?
  }
}
```

#### ~OI3. Is there a text/SMS equivalent to `orderInfo.emailInvite`? HighMatch gets best results sending text messages for invitations. 

If not, then we will send a text if `orderInfo.emailInvite` is true and `subject.phone` is populated.

#### ~OI4. What is the purpose of `sendResultsToUrl`? We assumed we will POST results to *joyndApiUrl*`/Results`.

#### OI5. Is `onCompletionURL` going to be empty if redirection is unnecessary? Is this only used for candidate-initiated (portal-based) applications?

### ~S#: POST /Order `subject` questions

```json
 "subject": {
    "id": "string",
    "firstName": "string",
    "lastName": "string",
    "middleNames": "string",
    "nationalId": "string",
    "dateOfBirth": "2024-04-18",
    "phone": "string",
    "addressLine1": "string",
    "addressLine2": "string",
    "city": "string",
    "state": "string",
    "postalCode": "string",
    "country": "string",
    "email": "string",
    "positionHistory": [
      {
        "position": "string",
        "companyName": "string",
        "companyPhone": "string",
        "companyAddressLine1": "string",
        "companyAddressLine2": "string",
        "companyCity": "string",
        "companyState": "string",
        "companyPostalCode": "string",
        "companyCountry": "string",
        "startDate": "2024-04-18",
        "endDate": "2024-04-18",
        "currentEmployerFlag": true,
        "reasonForLeaving": "string",
        "permissionToContactFlag": true,
        "payEndAmount": "string",
        "payEndCurrency": "string",
        "payEndType": "hourly",
        "supervisorName": "string",
        "supervisorTitle": "string",
        "supervisorEmail": "string",
        "supervisorPhone": "string"
      }
    ],
    "educationHistory": [
      {
        "schoolName": "string",
        "schoolAddressLine1": "string",
        "schoolAddressLine2": "string",
        "schoolCity": "string",
        "schoolState": "string",
        "schoolPostalCode": "string",
        "schoolCountry": "string",
        "schoolStart": "2024-04-18",
        "schoolEnd": "2024-04-18",
        "degreeName": "string",
        "degreeType": "string",
        "degreeMajor": "string",
        "degreeDate": "2024-04-18",
        "graduatedFlag": true
      }
    ],
    "driversLicense": {
      "dlNumber": "string",
      "dlState": "string",
      "dlExpiry": "2024-04-18"
    },
    "licenses": [
      {
        "licenseName": "string",
        "licenseAgency": "string",
        "licenseState": "string",
        "licenseNumber": "string",
        "licenseExpiry": "2024-04-18"
      }
    ],
    "references": [
      {
        "referenceName": "string",
        "referenceEmail": "string",
        "referencePhone": "string",
        "referenceRelationship": "string",
        "referenceType": "string"
      }
    ],
    "customFields": [
      {
        "name": "string",
        "value": "string"
      }
    ]
  }
```

#### ~S1. Does `subject.id` always identify the same candidate record in the source ATS?

In other words, if Candidate Jeff is in Greenhouse as ID GH2345 and Candidate Jeff takes two assessments - A1 and A2, will the `subject.id` value for A1 be GH2345 and `subject.id` value for A2 also be GH2345?

#### ~S2. Are the following always included when available in the source ATS or is it client/implementation specfic?

-  `subject.positionHistory` array
-  `subject.educationHistory` array
-  `subject.references` array

#### ~S3. What are typical usecases for the `subject.customFields` key/value pair array? Do we need to plan for any specific logic with this array?

#### ~S4. Will `subject.email` always be populated?

We use that value for invitations and completion reminders.

#### ~S5. When returning a 200, what does Joynd expect in `orderAccessUrl`?

```json
{
  "providerOrderId": "string",
  "orderAccessURL": "string" //what is this?
}
```

#### ~S6. Same questions as C2 to C5 for 400, 405, and 503 error response codes.

Is the behavior the same for this method as the GET Catalog method?

### ~R#: POST /Results Questions

[https://app.swaggerhub.com/apis-docs/hrnx4/HRNXScreening/2.1.0#/Screening/results](https://app.swaggerhub.com/apis-docs/hrnx4/HRNXScreening/2.1.0#/Screening/results)

```json
{
  "requesterRefId": "string",
  "clientId": "string",
  "providerOrderId": "string",
  "status": "string",
  "orderComplete": true,
  "result": "string",
  "score": 0,
  "scoreArray": [
    {
      "label": "string",
      "value": "string"
    }
  ],
  "reportURL": "string",
  "reportURLs": [
    "string"
  ]
}
```



#### ~R1: To post a status update only (not completed results), is this the appropriate JSON payload?

```json
{
  "requesterRefId": "string",
  "clientId": "string",
  "providerOrderId": "string",
  "status": "Started, Resumed, etc.",
  "orderComplete": false
}
```

#### ~R2: What happens when posting a `orderComplete: true`?

We assume the results are posted and perhaps the workflow state is advanced. What is the typical behavior in a target ATS?

#### ~R3: What is `requesterRefId`? Is that the same `requesterRefId` from the prior POST Order payload?

#### ~R4: Is `score` a floating point value typically from 0 to 100?

For example, we might score something as a percentage, so the value is 0 to 1 and formatted as a percentage. In that case, we would need to transform it to a 0 to 100 value. We assume this is a floating point value, so an 83.3 would not be relayed to the ATS as an 83. 

#### ~R5: What ATS vendors can use the `scoreArray` feature (which we assume is a list of sub-scale values providing supporting evidence to the primary `score` value)?

```json
{ 
  ...
  "scoreArray": [
    {
      "label": "string",
      "value": "string"
    }
  ]
}
```



Our initial clients are using Bullhorn, Greenhouse, and ADP.

#### ~R6: There is a `reportUrl` scalar value and a `reportUrls` array. Do we put the primary URL in both entities or just `reportUrl`?

```json
{
  ...
  "reportURL": "string",
  "reportURLs": [
    "string"
  ]
}
```

#### ~R7: If HighMatch receives any response other than a 200 from POST /Order, we will retry N seconds later up to M times when N is an exponential backoff value computed via the attempt number. Is there a suggested approach for attempting the transaction again?

### ~OQ#: Other Questions

#### ~OQ1. The first client expressed interest in deduplicating results. Alexander relayed to Kelly there may be some capability here. If so, what is it and how does it affect our implementation effort?

For context, the client inquired whether a policy could be developed that does something similar to the following:

1.  Assume the deduplication policy is that Person A can only take assessment Z once every 90 days.
2.  Person A originates in the source ATS on Jan 1. At this point, Person A has never been assigned an assessment.
3.  Person A is assigned Z on Jan 1. This Order is *Z.A.1*.
    -  Z has *not* been assigned to Person A in 90 days, so Order is performed.
4.  Person A is assigned Z again on Feb 1 (less than 90 days later).  This Order is *Z.A.2*.
    -  Person A was assigned to *Z.A.1* within the prior 90 days, so the Order is denied *or* the results for *Z.A.1* are assocated to *Z.A.2* (probably a better outcome since it avoids an error condition in the source ATS).
5.  Person A is assigned Z again on June 1 (180 days after *Z.A.1*). This Order is *Z.A.3*.
    -  Person A was assigned to *Z.A.1* more than 90 days ago, so the *Z.A.3* Order is processed.
    -  Person A is now assigned to *Z.A.1* and *Z.A.3*.

#### OQ2. Are the same basic authentication parameters Joynd uses when calling HighMatch used when HighMatch calls Joynd?

## Example Flow 1

This flow is included to validate that HighMatch understands exactly how to work with the Joynd API and JSON paylods. It includes:

1.  Catalog request
2.  Create new candidate (subject), assign an assessment, and send an invitation.
3.  Update assessment status on start/resume.
4.  Post completed results.

### E1.1 Joynd Requests Catalog

HighMatch API receives an inbound call to `GET /joynd/catalog/highmatch-dev` where `highmatch-dev` is the `clientId` value.

HighMatch finds 3 assessments related to `clientId=highmatch-dev` and returns the following JSON with a `200` response code.

```json
{
  "Catalog": [
    {
      "package": {
        "packageId": "hm-per-asmt",
        "name": "Personality",
        "services": []
      }
    },
    {
      "package": {
        "packageId": "hm-cog-asmt",
        "name": "Cognitive",
        "services": []
      }
    },
    {
      "package": {
        "packageId": "hm-math-asmt",
        "name": "Math Fundamentals",
        "services": []
      }
    },
    ,
    {
      "package": {
        "packageId": "hm-chem-asmt",
        "name": "Chemistry Fundamentals",
        "services": []
      }
    }
  ]
}
```

### E1.2 Joynd Posts an Order for Math Fundamentals Assessment

HighMatch API receives an inbound call to `POST /joynd/order` for a recruiter-initiated assessment order (not candidate-initiated via a portal application). 

The candidate is:

-  ATS ID: C31415
-  Jeff Adkisson
-  Jeff@HighMatch.com
-  444-555-1234

The job is:

-  ATS ID: J2718
-  Jr. Software Engineer

The recruiter is:

-  ATS ID: R1618
-  Frank Herbert
-  Frank@SpacingGuild.com

```json
{
  "orderInfo": {
    "requesterRefId": "???",
    "clientId": "highmatch-dev",
    "packageId": "hm-math-asmt",
    "jobId": "J2718",
    "jobTitle": "Jr. Software Engineer",
    "jobScoringRef": "???",
    "requesterId": "R1618",
    "requesterEmail": "Frank@SpacingGuild.com",
    "emailInvite": true,
    "billingReference1": "???",
    "billingReference2": "???"
  },
  "sendResultsToURL": "???",
  "onCompletionURL": "", //assuming empty on recruiter-initiated
  "subject": {
    "id": "C31415",
    "firstName": "Jeff",
    "lastName": "Adkisson",
    "middleNames": "",
    "nationalId": "???",
    "dateOfBirth": "1770-12-16",
    "phone": "444-555-1234",
    "addressLine1": "1313 Mockingbird Lane",
    "addressLine2": "",
    "city": "Mockingbird Heights",
    "state": "Los Angeles",
    "postalCode": "90210",
    "country": "USA",
    "email": "Jeff@HighMatch.com",
    "positionHistory": [],
    "educationHistory": [],
    "driversLicense": {},
    "licenses": [],
    "references": [],
    "customFields": []
  }
}
```

*??? = unclear what that field does/contains.*

HighMatch processes the inbound JSON.

1.  `subject.id: C31415` does not map to an existing HighMatch candidate in the HighMatch account related to `highmatch-dev`, so HighMatch creates an assessment participant named `Jeff Adkisson` related to `subject.id: C31415`.
2.  The `highmatch-dev` HighMatch account, contains an assessment whose ID matches `packageId: hm-math-asmt`. HighMatch assigns the assessment to `subject.id: C31415` and assigns the order ID to `C31415.hm-math-asmt.1`.
3.  HighMatch synchronously returns a `200` success to the awaiting Joynd HTTP caller with the following payload:

```json
{
  "providerOrderId": "C31415.hm-math-asmt.1",
  "orderAccessURL": "???"
}	
```

*??? = unclear what that field does/contains.*

4.  `emailInvite` is `true`, so HighMatch asynchronously sends an assessment email invitation to `Jeff@HighMatch.com` for provider order ID  `C31415.hm-math-asmt.1`.  HighMatch also schedules numerous completion reminder emails to be sent to provider order ID `C31415.hm-math-asmt.1`.

### E1.3 HighMatch Updates Status for an Order for Math Fundamentals Assessment

Jeff Adkisson (`providerOrderId: C31415.hm-math-asmt.1` ) receives a Math Fundamentals assessment email invitation sent by HighMatch.

Jeff clicks the link and starts the assessment.

HighMatch asynchronously calls Joynd to update the candidate's status in the target ATS to note that Jeff started the Math Fundamentals assessment.

HighMatch calls `POST joyndApiUrl/Results` with the following JSON payload:

```json
{
  "requesterRefId": "???",
  "clientId": "highmatch-dev",
  "providerOrderId": "C31415.hm-math-asmt.1", //Jeff Adkisson, Math Fundamentals
  "status": "Started Assessment",
  "orderComplete": false
}
```

*??? = unclear what that field does/contains.*

Jeff finishes half of the assessment, closes his browser, then takes a long nap with  his dog. After his nap, he watches some Tik Tok videos, eats some cold oatmeal, and worries about finishing the assessment he was assigned.

Jeff does some self-affirmation exercises, then nervously resumes the assessment. HighMatch calls `POST joyndApiUrl/Results` with the following JSON payload:

```json
{
  "requesterRefId": "???",
  "clientId": "highmatch-dev",
  "providerOrderId": "C31415.hm-math-asmt.1", //Jeff Adkisson, Math Fundamentals
  "status": "Resumed Assessment",
  "orderComplete": false
}
```

*??? = unclear what that field does/contains.*

### E1.4 HighMatch Posts Completed Results for an Order for Math Fundamentals Assessment

Jeff Adkisson (`providerOrderId: C31415.hm-math-asmt.1` ) completed his Math Fundamentals assessment. Jeff feels a sense of accomplishment and is confident he aced the assessment. His dog gives him a couple of congratulatory licks, then shoves a slobbery ball into his hand. 

-  Result: Needs Improvement
-  Score: 60th Percentile (0.60)
-  Sub Scores:
   -  Addition: 80, Acceptable
   -  Subtraction: 70, Acceptable
   -  Multiplication: 40, Needs Improvment
   -  Division: 12, Embarrassing

While Jeff throws the ball to the dog, HighMatch calls `POST joyndApiUrl/Results` with the following JSON payload: 

```json
{
  "requesterRefId": "???",
  "clientId": "highmatch-dev",
  "providerOrderId": "C31415.hm-math-asmt.1", //Jeff Adkisson, Math Fundamentals
  "status": "Complete",
  "orderComplete": true
  "result": "Needs Improvement",
  "score": 60,
  "scoreArray": [
    {
      "label": "Addition",
      "value": "80th percentile, Acceptable"
    },
		{
      "label": "Subtraction",
      "value": "70th percentile, Acceptable"
    },
		{
      "label": "Multiplication",
      "value": "40th percentile, Needs Improvement"
    },
		{
      "label": "Division",
      "value": "12th percentile, Embarrasing"
    }
  ],
  "reportURL": "https://reports.highmatch.com/Z12345678",
  "reportURLs": []
}
```

*??? = unclear what that field does/contains.*

In the event the call from HighMatch to Joynd fails, HighMatch will queue the message and reattempt delivery up to 7 days. If all delivery attempts fail, HighMatch will mark the message as Dead Letter, which will alert the administrator that a manual intervention may be necessary to restart processing.


## Example Flow 2

This flow is included to validate that HighMatch understands exactly how to work with the Joynd API and JSON paylods. It includes:

1.  Create assessment for existing candidate (subject)

This flow continues the example from Example Flow 1.

### E2.1 Joynd Posts an Order for Chemistry Fundamentals Assessment for Previously Integrated ATS Candidate

HighMatch API receives an inbound call to `POST /joynd/order` for a recruiter-initiated assessment order (not candidate-initiated via a portal application). 

The candidate is:

-  ATS ID: C31415
-  Jeff Adkisson
-  Jeff@HighMatch.com
-  444-555-1234

The job is:

-  ATS ID: J60221408
-  Chemical Engineer

The recruiter is:

-  ATS ID: R8314
-  Dimitri Mendeleev
-  Dimitri@SpacingGuild.com

```json
{
  "orderInfo": {
    "requesterRefId": "???",
    "clientId": "highmatch-dev",
    "packageId": "hm-chem-asmt",
    "jobId": "J60221408",
    "jobTitle": "Chemical Engineer",
    "jobScoringRef": "???",
    "requesterId": "R8314",
    "requesterEmail": "Dimitri@SpacingGuild.com",
    "emailInvite": true,
    "billingReference1": "???",
    "billingReference2": "???"
  },
  "sendResultsToURL": "???",
  "onCompletionURL": "", //assuming empty on recruiter-initiated
  "subject": {
    "id": "C31415",
    "firstName": "Jeffrey",
    "lastName": "Adkisson",
    "middleNames": "W.",
    "nationalId": "???",
    "dateOfBirth": "1770-12-16",
    "phone": "444-555-1234",
    "addressLine1": "1313 Mockingbird Lane",
    "addressLine2": "",
    "city": "Mockingbird Heights",
    "state": "Los Angeles",
    "postalCode": "90210",
    "country": "USA",
    "email": "Jeffrey@HighMatch.com",
    "positionHistory": [],
    "educationHistory": [],
    "driversLicense": {},
    "licenses": [],
    "references": [],
    "customFields": []
  }
}
```

*??? = unclear what that field does/contains.*

HighMatch processes the inbound JSON.

1.  `subject.id: C31415` maps to an existing HighMatch candidate in the HighMatch account related to `highmatch-dev`, so HighMatch updates the assessment participant related to `subject.id: C31415`. The participant's name previously was Jeff Adkisson. The inbound JSON has a different name, so the participant's name is updated to Jeffrey W. Adkisson and his email address is updated from Jeff@HighMatch.com to Jeff@HighMatch.com
2.  The `highmatch-dev` HighMatch account, contains an assessment whose ID matches `packageId: hm-chem-asmt`. HighMatch assigns the assessment to `subject.id: C31415` and assigns the order ID to `C31415.hm-chem-asmt.1`.
3.  HighMatch synchronously returns a `200` success to the awaiting Joynd HTTP caller with the following payload:

```json
{
  "providerOrderId": "C31415.hm-chem-asmt.1",
  "orderAccessURL": "???"
}	
```

*??? = unclear what that field does/contains.*

4.  `emailInvite` is `true`, so HighMatch asynchronously sends an assessment email invitation to `Jeffrey@HighMatch.com` for provider order ID  `C31415.hm-math-asmt.1`.  HighMatch also schedules numerous completion reminder emails to be sent to provider order ID `C31415.hm-chem-asmt.1`.
