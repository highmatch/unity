<script src="https://cdn.jsdelivr.net/npm/mermaid/dist/mermaid.min.js"></script>
<script>
    mermaid.initialize({startOnLoad:true});
    mermaid.run({
      querySelector: '.language-mermaid',
    });
</script>


# Unity API Questions

## GET /Catalog QUESTIONS 

https://app.swaggerhub.com/apis-docs/hrnx4/HRNXScreening/2.1.0#/Screening/Packages 

### C1. What does the Services array do in GET /Catalog?

When a catalog request arrives, HighMatch will return an array of `package` objects where `package.packageId` is the ID you will send for a new Order and `package.name` is what is displayed in the target ATS.

What does the `package.services` array do?

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

### C2. Error Handling - 405

Swagger shows a 405 error code under GET Catalog. Should we return this if the `clientId` parameter is invalid (or no longer a valid HighMatch customer)?

### C3. Error Handling - 503

Swagger shows a 503 error code under GET Catalog. Should we return this when we are offline for maintenance?

#### C4. Error Handling - Unable to Connect

-  In the event Joynd cannot connect to Compass, what is the retry pattern/interval? 
-  In the event Joynd cannot connect to Compass, is the end-user (the ATS user) impacted (i.e. this is synchronous)? If so, what happens?

#### C5. Error Handling - 400

It appears if we have any other project, we should return the following model:

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
-  Are there any specific codes we need to return?

## POST /Order `orderInfo` Questions

#### OI1. Questions on `orderInfo` fields that may not be relevant to HighMatch.

```json
{
  "orderInfo": {
    "requesterRefId": "string", //do we need to retain this?
    "clientId": "string", 
    "packageId": "string",
    "jobId": "string", //is this relevant to HighMatch?
    "jobTitle": "string", //is this the job to which the candidate is apply?
    "jobScoringRef": "string", //what do we do with this?
    "requesterId": "string", //what do we do with this?
    "requesterEmail": "string", //is this the recruiter? Do we need this?
    "emailInvite": true,
    "billingReference1": "string", //what do we do with this?
    "billingReference2": "string" //what do we do with this?
  }
}
```

-  `requesterRefId` 

   Is this a Joynd ID or the origin ATS user ID that initiated the Order?

-  `jobId` 

   Is this a Joynd ID or the origin ATS job ID for `jobTitle`?

-  `jobTitle` 

   Is this the job to which the candidate is applying?

-  `jobScoringRef`

   What is this for?

-  `requesterId`

   Is this a Joynd ID or the origin ATS user ID for `requesterEmail`?

-  `requesterEmail`

   Is this the email address of the origin ATS user that initiated the Order?

-  `billingReference1` and `billingReference1` 

   Are these relevant to HighMatch? How are they typically used?

#### OI2. If `orderInfo.emailInvite` is `true`, HighMatch should invite candidate to complete assessment via email?

```json
{
  "orderInfo": {
    "clientId": "string", 
    "packageId": "string",
    "emailInvite": true //if false, can HM assume the ATS is handling invitation?
  }
}
```

#### OI3. Is there a text/SMS equivalent to `orderInfo.emailInvite`? HighMatch gets best results sending text messages for invitations.

#### OI4. What is the purpose of `sendResultsToUrl`? We assumed we will POST results to *joyndApiUrl*`/Results`.

#### OI5. Is `onCompletionURL` going to be empty if redirection is unnecessary? Is this only used for candidate-initiated (portal-based) applications?

## POST /Order `subject` questions

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

