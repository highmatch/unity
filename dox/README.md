<script src="https://cdn.jsdelivr.net/npm/mermaid/dist/mermaid.min.js"></script>
<script>
    mermaid.initialize({startOnLoad:true});
    mermaid.run({
      querySelector: '.language-mermaid',
    });
</script>


# Client

#### Target ATS

The *first* HighMatch/Joynd client uses **Bullhorn**.

The *second* client is currently using Greenhouse and plans to move to ADP in late 2024. They may want to start on Greenhouse, then migrate to ADP. We are waiting to hear their plans.

We plan to get the first client online and stable before starting deployment of the second client. We do not want greenfield coding problems to impact two clients if possible.

# Dev Environment Criteria

**Dev Client ID:** `dev-highmatch`

**Inbound HighMatch Dev URLs:**

-  Catalog: GET https://highmatch.ngrok.io/joynd/catalog

-  Order: POST https://highmatch.ngrok.io/joynd/order

-----

**Inbound HighMatch QA/Staging URLs (for final testing prior to production deploment)**

-  Catalog: GET https://api.compass.qa-highmatch.com/joynd/catalog
-  Catalog: POST https://api.compass.qa-highmatch.com/joynd/order

-----

**Inbound HighMatch Production URLs**

-  Catalog: GET https://api.compass.highmatch.com/joynd/catalog
-  Catalog: POST https://api.compass.highmatch.com/joynd/order

# Unity API Questions

## ~C#: GET /Catalog QUESTIONS 

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

### ~C1. What does the Services array do in GET /Catalog?

When a catalog request arrives, HighMatch will return an array of `package` objects where `package.packageId` is the ID you will send for a new Order and `package.name` is what is displayed in the target ATS.

What does the `package.services` array do?

### ~C2. Error Handling - 405

Swagger shows a 405 error code under GET Catalog. Should we return this if the `clientId` parameter is invalid (or no longer a valid HighMatch customer)? How does Joynd respond if a 405 is returned?

### ~C3. Error Handling - 503

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
-  Are there any specific `errorCode` value(s) we need to return?

## ~OI#: POST /Order `orderInfo` Questions

[https://app.swaggerhub.com/apis-docs/hrnx4/HRNXScreening/2.1.0#/Screening/Order](https://app.swaggerhub.com/apis-docs/hrnx4/HRNXScreening/2.1.0#/Screening/Order)

#### ~OI1. Questions on `orderInfo` fields that may not be relevant to HighMatch.

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

If not, then we will send a text if `orderInfo.emailInvite` is true and `subject.phone` is populated.

#### ~OI4. What is the purpose of `sendResultsToUrl`? We assumed we will POST results to *joyndApiUrl*`/Results`.

#### OI5. Is `onCompletionURL` going to be empty if redirection is unnecessary? Is this only used for candidate-initiated (portal-based) applications?

## ~S#: POST /Order `subject` questions

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

## ~R#: POST /Results Questions

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

#### ~R3: What is `requesterRefId`? Is that the same `requesterRefId` from the prior POST Order payload?

#### ~R4: Is `score` a floating point value typically from 0 to 100?

For example, we might score something as a percentage, so the value is 0 to 1 and formatted as a percentage.

#### ~R5: What ATS vendors can use the `scoreArray` feature (which we assume is a list of sub-scale values providing supporting evidence to the primary `score` value)?

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

## ~OQ#: Other Questions

#### ~OQ1. The first client expressed interest in deduplicating results. Alexander said there may be some capability here. If so, what is it and how does it affect our implementation effort?

For context, the client inquired where there could be a policy in place that does something similar to the following:

1.  Assume the deduplication policy is that Person A can only take assessment Z once every 90 days.
2.  Person A is assigned Z on Jan 1. Call this Z.A.1.
    -  Z has not been assigned to A in 90 days, so Order is performed.
3.  Person A is assigned Z again on Feb 1 (less than 90 days later).  Call this Z.A.2.
    -  Z was assigned to Z.A.1 within the prior 90 days, so the Order is denied *or* the results for Z.A.1 are assocated to Z.A.2 (probably a better outcome).
4.  Person A is assigned Z again on June 1 (180 days after Z.A.1). Call this Z.A.3.
    -  Z was assigned to Z.A.1 more than 90 days ago, so the Z.A.3 Order is processed.
    -  Z is now assigned to Z.A.1 and Z.A.3.



