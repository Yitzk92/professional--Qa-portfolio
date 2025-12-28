# Bug Analysis – Silent Data Loss Due to Session Timeout on Edit Profile Form

## Summary
When a user edits their profile and the session expires in the background, submitting the form results in a silent failure.  
No error message is displayed, no redirect occurs, and none of the user’s data is saved.  
This leads to **data loss** and extremely poor user experience.

---

## Environment
- Platform: Web
- Page: /profile/edit
- Browser: Chrome 131 / Edge latest
- OS: Windows 11 / macOS Sonoma
- Build: Production – v1.12.3

---

## Preconditions
- User is logged in
- Session timeout configured to ~30 minutes

---

## Steps to Reproduce
1. Log in with a valid user account  
2. Navigate to: Profile → Edit Profile  
3. Leave the browser idle for **30+ minutes** until the session expires  
4. Modify any field in the form  
5. Click **Save**

---

## Expected Result
- System should detect expired session and:
  - Redirect user to Login  
  **OR**
  - Show clear inline error:
    “Your session has expired. Please log in again.”

And importantly:
- No silent data loss
- Clear communication to the user

---

## Actual Result
- Save button appears to work
- Loading spinner briefly appears
- No error shown
- User remains on page
- Data is NOT saved
- After refresh, all edits are lost

---

## Severity / Impact
**Severity:** High  
**Impact:**
- Affects all authenticated users
- Data loss risk
- Damage to user trust
- High probability in long workflows

**Business Risk**
- Support tickets increase
- Perception that “the system is broken”
- Potential churn

---

## Technical Observations / Logs
- API `PUT /api/profile`
- Response: **401 Unauthorized**
- Frontend behavior:
  - No global handler for 401 in this flow
  - Error swallowed
  - UI not updated
  - No redirect triggered

**Hypothesis – Root Cause**
- Session expiration not managed centrally
- 401 handling inconsistent across app
- No UX fallback when authentication token invalid

---

## Suggested Fix Direction
### Backend / Auth
- Ensure consistent 401 behavior on expired session

### Frontend
1️⃣ Add **global HTTP interceptor** to handle `401`
- Clear auth token
- Redirect to login
- Preserve user intent if possible

2️⃣ For Edit Profile page specifically:
- Show clear error banner / toast
- Disable Save after 401 detected

3️⃣ Optional UX Improvement
- Warn user **2–3 minutes before timeout**
- Allow session refresh without data loss

---

## Regression Scope
Areas at risk:
- All authenticated user flows
- Forms across platform
- Multi-tab sessions
- Mobile web behavior

---

## Recommended Test Coverage
### Manual
- Submit after timeout
- Refresh behavior
- Multi-tab logout consistency

### Automation
- E2E test mocking expired session state
- Integration testing for global 401 handler

---

## Conclusion
This is a high-severity usability and trust issue.  
Fixing it significantly improves platform reliability perception and prevents silent data loss.
