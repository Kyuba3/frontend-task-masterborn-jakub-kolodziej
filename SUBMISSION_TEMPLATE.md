# Submission: Jakub Kołodziej

## Time Spent

Total time: **around 4 hours** (approximate)

## Ticket Triage

### Tickets I Addressed

List the ticket numbers you worked on, in the order you addressed them:

1. **CFG-142**: Fixed the timestamp comparison bug so the price always updates correctly.
2. **CFG-148**: Used functional state updates to fix the broken UI when toggling packages.
3. **CFG-143**: Added the missing cleanup function to the resize event listener.
4. **CFG-147**: Switched to `encodeURIComponent` to natively handle Unicode.
5. **CFG-154**: Changed `>` to `>=` so exactly 50 items actually get the 10% discount.
6. **CFG-144**: Cleaned up the Quick Add feature (button, handler, CSS) completely.
7. **CFG-151**: Created human-readable error messages to replace cryptic numeric codes.
8. **CFG-156**: Used stable IDs instead of array indices to eliminate console warnings.
9. **CFG-152**: Added keyboard focus and `tabIndex` to color swatches.
10. **CFG-149**: Added a spinner animation during price calculations.

### Tickets I Deprioritized

List tickets you intentionally skipped and why:

| Ticket  | Reason                                                                 |
| ------- | ---------------------------------------------------------------------- |
| CFG-145 | Contradicts the Product Owner's decision to remove Quick Add (CFG-144) |
| CFG-153 | Too large scope (2-3 weeks) for a 4-hour sprint                        |
| CFG-155 | Dark mode is a nice-to-have, but requires extensive CSS refactoring      |
| CFG-157 | "Unsaved changes" dialog is standard UX but not a demo blocker         |
| CFG-146 | Minor cosmetic timezone issue                                          |
| CFG-150 | Mobile color picker alignment requires a design decision first          |

### Tickets That Need Clarification

List any tickets where you couldn't proceed due to ambiguity:

| Ticket  | Question                                                                       |
| ------- | ------------------------------------------------------------------------------ |
| CFG-154 | The context says "50+", but Sarah's Slack notes implied "a different threshold". Are the bounds 50 and 10? |

---

## Technical Write-Up

### Critical Issues Found

Describe the most important bugs you identified:

#### Issue 1: Price race condition

**Ticket(s):** CFG-142

**What was the bug?**

The app was comparing an incrementing API counter (`response.timestamp = 1, 2, 3...`) against `Date.now()`. The condition was virtually always false, so price updates were silently dropped.

**How did you find it?**

I traced the data flow from the API response back to the state update. The mismatch between the API request counter and `Date.now()` was immediately obvious.

**How did you fix it?**

I fixed the logic to compare `requestTime` against `latestRequestRef.current`, ensuring only the latest request wins.

**Why this approach?**

The simplest fix is to use the actual timestamps. I considered using the `useDebouncedPriceCalculation` hook, but it contains a stale-ref bug that would require more refactoring.

---

#### Issue 2: Broken UI state on add-on toggle

**Ticket(s):** CFG-148

**What was the bug?**

When removing dependent add-ons, the code was modifying the `selectedAddOns` array in-place with `splice()` and passing the exact same array reference back to React, which caused React to bail out of rendering.

**How did you find it?**

I traced the reproduction steps for the "Include Packaging" toggle, finding the `splice()` call. React state mutation is a common anti-pattern. While the ticket reported a hard crash, I found that this mutation mainly leaves the app in a broken, inconsistent state.

**How did you fix it?**

I replaced the `splice` mutation with an immutable `.filter()` update using a functional state updater: `setSelectedAddOns(prev => prev.filter(...))`.

**Why this approach?**

Functional state updates are React best practice for array manipulations and avoid stale closures in the `useCallback`.

---

#### Issue 3: Share URL encoding breaks with special characters

**Ticket(s):** CFG-147

**What was the bug?**

The `btoa()` encoding crashes if it encounters Unicode characters (e.g., if a new configuration option contained special characters).

**How did you find it?**

The ticket mentioned crashes on specific configurations. I checked `api.ts` and saw the raw `btoa()` call, which is known to fail on non-ASCII characters.

**How did you fix it?**

I replaced `btoa()` and `atob()` with `encodeURIComponent()` and `decodeURIComponent()`.

**Why this approach?**

It handles Unicode natively, drops deprecated APIs (like `unescape`), and makes the share URL much easier to debug than base64.

---

### Other Changes Made

Brief description of any other modifications:

- **CFG-143 (Memory leak):** Added a cleanup function to the `window` resize event listener in `useEffect`.
- **CFG-154 (Discount tiers off-by-one):** Changed `>` to `>=` so exactly 50 items trigger the 10% discount.
- **CFG-144 (Quick Add):** Removed the button, handler, and related `.quick-add` CSS styles.
- **CFG-151 (Error messages):** Created an `ERROR_MESSAGES` dictionary to display user-friendly text instead of raw `ERR_` codes.
- **CFG-156 (React keys):** Switched array mappings to use stable `choice.id` and `addOn.id` keys instead of indices.
- **CFG-152 (Accessibility):** Added `tabIndex=0` and keyboard handlers (Enter/Space) to color swatches.
- **CFG-149 (Loading indicator):** Added a CSS spinner animation next to the price value during calculation.

---

## Code Quality Notes

### Things I Noticed But Didn't Fix

List any issues you noticed but intentionally left:

| Issue   | Why I Left It                                         |
| ------- | ----------------------------------------------------- |
| 1000-line Component | Architectural refactor is out of scope for a 4-hour sprint. |
| Empty try/catch blocks | Pre-existing tech debt. Fixing requires a broad error-handling strategy. |
| `useDebouncedPriceCalculation` | Hook is unused, but contains a stale-ref bug that will cause issues if activated. |

### Potential Improvements for the Future

If you had more time, what would you improve?

1. **Add Unit Tests:** The calculations in `pricing.ts` are pure functions and desperately need test coverage before anyone tweaks the discount logic again.
2. **Break down the component:** `ProductConfigurator.tsx` is doing way too much. Splitting out the ColorPicker, AddOnList, and PricePanel would make it much easier to maintain.

---

## Questions for the Team

Questions you would ask in a real scenario:

1. Did Customer Success reach out to TechStyle about removing the Quick Add feature (CFG-144)? They specifically requested it in CFG-145, and I want to make sure they aren't surprised during the demo.
2. What is the deadline for full WCAG 2.1 AA accessibility compliance? I added basic keyboard focus (CFG-152), but full compliance will require focus trapping and aria-labels across the board.

---

## Assumptions Made

List any assumptions you made to proceed:

1. The Product Owner's decision to remove Quick Add (CFG-144) overrules the Customer Success feature request (CFG-145) since usage is very low (2%).
2. The discount tiers in `CONTEXT.md` (50+: 10%, 10-49: 5%) are the source of truth, outweighing fuzzy slack messages.

---

## Self-Assessment

### What went well?

I managed to track down the root causes logically without just guessing, especially the race condition. I addressed all 10 demo-blocking and crucial tickets within the time limit.

### What was challenging?

Scoping the work was tricky—choosing between removing Quick Add or enhancing it—but siding with the Product Owner over a 2% usage feature is usually the right call before a major demo. The state mutation crash report was also ambiguous, but fixing the mutation solved the inconsistent state.

### What would you do differently with more time?

I would have written unit tests for the pricing and utility functions first. Dealing with the off-by-one errors would be much safer and faster with a fast test runner checking edge cases.

---

## Additional Notes

Anything else you want us to know:

I used `encodeURIComponent` for the share URLs. It generates slightly longer URLs than base64 but is far standard, inherently Unicode-safe, and easier to debug.
