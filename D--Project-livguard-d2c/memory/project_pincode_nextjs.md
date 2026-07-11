---
name: changeDefaultPinCode — Next.js integration analysis
description: Analysis of the changeDefaultPinCode feature in AccountController for use in the Next.js app, covering guest and logged-in behavior
type: project
originSessionId: 7d798927-63b7-4558-8ef3-21d612b00add
---
Analyzed `changeDefaultPinCode` variants in `AccountController.groovy` for Next.js integration.

**Why:** User wants to wire pincode selection into the Next.js app for both guest and logged-in users.

**How to apply:** Use `changeDefaultPinCodeWeb` as the target endpoint. Do not use `changeDefaultPinCodeApi` — it requires `ROLE_USER` and will reject guests.

## Endpoint to use
- `POST /updatepincode` (web URL mapping group → `changeDefaultPinCodeWeb`)
- Param key: `pincode` (not `pinCode`) — from `request.JSON.pincode` or query param
- Send `credentials: 'include'` so session cookie is forwarded

## Guest (not logged in)
- Pincode saved to server-side session: `session["default_pin_code"]`
- No pincode format validation — any value is accepted
- Returns 200 OK + success message

## Logged in
- User resolved via `getUser()` or `session['userMobile']` → `User.findByMobile`
- Calls `accountService.changeDefaultPinCode(user, pinCode)` in `AccountService.groovy:411`
  - Looks up location via `locationService.findLocationByPinCode`
  - Sets `user.area`, `user.defaultPinCode`, `user.pinCode` and saves to MongoDB
- Returns 200 OK on success, 406 NOT_ACCEPTABLE on failure

## Known gaps
- Guest path skips pincode validation entirely
- Guest session pincode is NOT migrated to user record on login — may need to handle this in Next.js after login
- `changeDefaultPinCodeWeb` also skips explicit `locationService.validatePincode` call (unlike `changeDefaultPinCodeApi`)

## Response shape
```json
{ "statusCode": 200, "messages": ["Pincode updated successfully"], "errors": [] }
```
