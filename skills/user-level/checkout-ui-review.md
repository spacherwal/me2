# Checkout UI Review Skill

Review the checkout screen UI code and give structured feedback on UX, correctness, and mobile behaviour.

## Usage

```
/checkout-ui-review
```

Run from any e-commerce project with a checkout screen.

---

## What to Review

Read all files under `src/components/checkout/` and `src/app/checkout/` before giving feedback.

Evaluate each of the following areas and flag issues found:

---

### 1. Internal data exposed to the user
- Look for any IDs (cart ID, order ID, user ID, session tokens) rendered in the UI
- These are internal identifiers — users have no use for them and they look unprofessional
- Flag and recommend removing them

### 2. Progress / step indicator
- Check if there is a visible step indicator (e.g. Address → Payment → Confirm)
- Users need to know where they are in the flow and how many steps remain
- Flag if missing

### 3. Section step numbering
- Check if major sections (address, payment) have visual step numbers
- Numbered indicators make the flow order unambiguous
- Flag if missing

### 4. Mobile — CTA button placement
- Check if the Place Order / Proceed button is pinned to the bottom on mobile
- It should NOT be buried inside a scrollable section
- Flag if the only CTA is inside a card that requires scrolling to reach

### 5. Hardcoded delivery / shipping values
- Check if shipping/delivery charge is hardcoded as "Free" or any fixed value
- It should come from the API response (e.g. `cartData.deliveryCharges`)
- Flag any hardcoded charge assumptions

### 6. Dead / unconnected buttons
- Scan for buttons with no `onClick`, pointing to `#`, or with console.log placeholders
- Common culprits: "Add New Address", "Edit", "Apply Coupon", footer links
- Flag each one and note whether to wire up or remove

### 7. Payment method pre-selection sync
- Check if auto-selected payment methods (on load) notify the parent state via callback
- A visually selected option that isn't reflected in parent state causes silent "please select payment method" errors
- Flag if `onSelectPaymentMethod` (or equivalent) is only called on user click, not on auto-select

### 8. Error loop risk on broken images
- Check `onError` handlers on `<img>` tags
- If `onError` sets `src` to a file that also doesn't exist, it creates an infinite request loop
- Flag if fallback image path points to a non-existent file in `public/`

### 9. Form / input error message layout
- Check if error messages use `position: absolute` below inputs
- If content below can overlap, flag it — prefer normal document flow for errors

---

## Output Format

List findings grouped by area. For each issue:
- **Status**: Found / Not found
- **Location**: file:line
- **Issue**: one-line description
- **Recommendation**: what to fix

End with a **Priority** section ranking the top 3 issues to fix first.
