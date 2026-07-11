---
name: LSQ Activity Creation Pattern
description: Standard pattern to follow when creating new LeadSquared activity models and service methods
type: feedback
---

Every new LSQ activity must include these two fields in the model and populate them in the service — no exceptions.

**Model:** Always add `activityDate` (Date) and `createdByEmailId` (String) before the `@CreatedDate` field:
```java
private Date activityDate;
private String createdByEmailId;
@CreatedDate
private Date dateCreated;
```

**Service impl:** Always populate them before the repository save using the established constants:
```java
String activityDateString = (String) activity.get(LeadSquaredApiConstants.FIELD_CREATED_ON);
Date activityDate = null;
if (activityDateString != null && !activityDateString.trim().isEmpty()) {
    activityDate = dateFormat.parse(activityDateString);
}
lsqXxxActivity.setActivityDate(activityDate);
lsqXxxActivity.setCreatedByEmailId((String) activity.get(LeadSquaredApiConstants.FIELD_CREATED_BY_EMAIL_ADDRESS));
```

**Why:** These fields were retroactively added to all existing activities (Razorpay, PaymentLink, ThreeSixtyAll, CollectVerifyDocument, SaleVerification, LoanProcessing). Going forward they are part of the standard activity template.

**How to apply:** Any time a new LSQ activity model + service method is created, include these fields and their population logic from the start — do not wait to be asked.
