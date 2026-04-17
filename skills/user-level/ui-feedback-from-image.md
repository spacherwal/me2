# UI Feedback from Image

Analyze a UI screenshot and give structured, actionable feedback across standard UX/UI dimensions.

## Usage

```
/ui-feedback-from-image <path-to-image>
```

Example:
```
/ui-feedback-from-image D:\CheckoutScreenshot.jpg
```

---

## Step 1: Load the Image

Read the image file at the path provided in the arguments using the Read tool. If no path is given, ask the user to provide one.

---

## Step 2: Identify the Screen

Before giving feedback, briefly state:
- **Screen type** (e.g. checkout, product listing, login, dashboard)
- **Device context** (desktop / mobile / unclear)
- **Primary user goal** on this screen (what is the user trying to accomplish?)

This frames all feedback that follows.

---

## Step 3: Evaluate Each Area

Go through every area below. For each one, determine: **Found / Not found / N/A**.

---

### 1. Visual Hierarchy
- Is there a clear focal point? Does the eye know where to go first?
- Are primary, secondary, and tertiary elements visually distinct?
- Is the most important action (CTA) the most visually prominent element?

### 2. Typography & Readability
- Are font sizes appropriate? Body text should be ≥14px.
- Is there sufficient contrast between text and background?
- Are headings, labels, and body text visually differentiated?
- Is line length comfortable (not too wide or too narrow)?

### 3. Color & Contrast
- Do interactive elements (buttons, links) stand out from non-interactive content?
- Is contrast sufficient for accessibility (WCAG AA: 4.5:1 for normal text)?
- Is color used consistently (e.g. same color always means "action")?
- Are error/warning/success states color-coded and distinguishable without color alone?

### 4. Spacing & Alignment
- Is there consistent padding and margin throughout?
- Are related elements grouped together with tighter spacing?
- Are elements aligned to an implied grid?
- Does anything feel cramped or overly spread out?

### 5. Navigation & Wayfinding
- Does the user know where they are in the app/flow?
- Is there a breadcrumb, progress indicator, or step counter if needed?
- Is there a clear way to go back or cancel?
- Are page/section titles present and descriptive?

### 6. Calls to Action (CTAs)
- Is the primary CTA immediately visible without scrolling (above the fold)?
- Is there only one primary CTA per section/screen?
- Are CTA labels specific and action-oriented (e.g. "Place Order" not "Submit")?
- On mobile, is the primary CTA pinned to the bottom or otherwise reachable without scrolling?

### 7. Forms & Inputs
- Do all inputs have visible labels (not just placeholder text)?
- Is the tab/focus order logical?
- Are required fields marked?
- Is there space reserved for inline error messages so they don't cause layout shifts?

### 8. Feedback & States
- Are loading, empty, error, and success states handled visually?
- Is there confirmation after a destructive or important action?
- Does the UI show progress for async operations?

### 9. Trust & Credibility
- Are security signals present where relevant (HTTPS badge, payment icons)?
- Is pricing transparent (no hidden fees revealed late)?
- Are there unnecessary internal identifiers (IDs, tokens) visible to the user?

### 10. Mobile & Touch
- Are touch targets at least 44×44px?
- Is text large enough to read without zooming?
- Does any content appear to overflow or get clipped?
- Are interactive elements reachable with one thumb on a phone?

### 11. Information Architecture
- Is content grouped logically?
- Is the most critical information at the top?
- Is secondary/supporting content clearly subordinate?
- Are there redundant elements that could be removed?

### 12. Micro-copy & Labels
- Are labels, headings, and button text clear and unambiguous?
- Is error copy helpful (tells the user what to do, not just what went wrong)?
- Is any text truncated in a way that hides important information?

---

## Output Format

For each area above:

```
### <Area Name>
**Status:** Found issue / Looks good / N/A
**Location:** Describe where on the screen (top-left, right column, below the fold, etc.)
**Issue:** One-line description of what's wrong
**Recommendation:** Specific, actionable fix
```

If an area looks good, one line is enough: `**Status:** Looks good`

---

## Final Section: Priority List

End with a ranked list of the **top 5 issues** to fix first, ordered by impact on the user's primary goal.

```
## Top 5 Issues to Fix

1. [Highest impact] — <issue> — <why it matters>
2. ...
3. ...
4. ...
5. ...
```
