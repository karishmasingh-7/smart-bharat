# Smart Bharat - Testing Documentation

This document outlines comprehensive test cases for the Smart Bharat application, covering manual testing procedures, automated test scenarios, and edge case handling.

## Table of Contents

1. [Environment Setup](#environment-setup)
2. [Database Connection Tests](#database-connection-tests)
3. [AI Chatbot Tests (Hinglish)](#ai-chatbot-tests-hinglish)
4. [Civic Issue Reporting Tests](#civic-issue-reporting-tests)
5. [Dashboard & Tracking Tests](#dashboard--tracking-tests)
6. [Security Tests](#security-tests)
7. [Edge Case Handling](#edge-case-handling)
8. [Performance Tests](#performance-tests)

---

## Environment Setup

### Prerequisites
- Node.js v18+
- Supabase CLI (optional, for local development)
- Access to deployed Supabase instance

### Test Environment Variables
```env
NEXT_PUBLIC_SUPABASE_URL=<test-instance-url>
NEXT_PUBLIC_SUPABASE_ANON_KEY=<test-anon-key>
GEMINI_API_KEY=<test-gemini-key>  # Optional - fallback mock responses available
```

### Running the Application
```bash
npm install
npm run build
npm run dev
```

---

## Database Connection Tests

### TC-DB-01: Supabase Connection Status
**Priority:** Critical
**Type:** Manual/Automated

**Steps:**
1. Open browser DevTools Network tab
2. Navigate to Dashboard page (`/dashboard`)
3. Verify XHR requests to Supabase succeed

**Expected Result:**
- Network requests to `*.supabase.co` return 200 status
- No CORS errors in console
- Dashboard loads with data or empty state message

**Status:** [ ] Pass [ ] Fail

---

### TC-DB-02: Civic Issues Table Read Access
**Priority:** Critical
**Type:** Manual

**Steps:**
1. Navigate to Dashboard (`/dashboard`)
2. Observe issue list loading

**Expected Result:**
- Issues are fetched and displayed
- Each issue shows: ticket ID, title, category, status, date
- No authentication errors (public RLS policy allows anon read)

**Status:** [ ] Pass [ ] Fail

---

### TC-DB-03: Chat History Table Write Access
**Priority:** High
**Type:** Manual

**Steps:**
1. Navigate to AI Chat (`/chat`)
2. Send a message
3. Check Supabase dashboard for new row in `chat_history`

**Expected Result:**
- New row inserted with `session_id`, `message`, `response`
- No RLS policy violations

**Status:** [ ] Pass [ ] Fail

---

## AI Chatbot Tests (Hinglish)

### TC-AI-01: Hinglish Response Quality
**Priority:** Critical
**Type:** Manual

**Test Inputs & Expected Behaviors:**

| Input | Expected Response Characteristic |
|-------|----------------------------------|
| "Aadhaar card kaise banayein?" | Hinglish response with steps, uses Hindi/English mix |
| "What is PM-KISAN?" | Info about scheme in Hinglish |
| "Pothole report karna hai" | Guidance on using the report page |
| "Hello" | Greeting with "Namaste" in response |
| "सरकारी योजनाएं" | Response in Hindi script welcome, switches to Hinglish |

**Verification Criteria:**
- Response uses Hinglish (Hindi + English mixed)
- Includes Indian greetings ("Namaste", "Dhanyavaad")
- Polite and supportive tone
- Concise but helpful (under 500 chars ideally)

**Status:** [ ] Pass [ ] Fail

---

### TC-AI-02: Government Scheme Queries
**Priority:** High
**Type:** Manual

**Test Queries:**
1. "PM-KISAN scheme ke benefits kya hain?"
2. "Ayushman Bharat ke liye eligibility?"
3. "How to apply for PAN card online?"
4. "MGNREGA job card kaise milega?"

**Expected Response Pattern:**
- Provides general guidance (not legal advice)
- Mentions official portals where applicable
- Suggests visiting government websites for exact details

**Status:** [ ] Pass [ ] Fail

---

### TC-AI-03: Civic Issue Guidance
**Priority:** High
**Type:** Manual

**Test Queries:**
1. "How do I report a pothole?"
2. "Garbage collection issue hai"
3. "Street light nahi kaam kar rahi"

**Expected Response:**
- Directs user to `/report` page
- Explains the reporting process
- Mentions ticket tracking capability

**Status:** [ ] Pass [ ] Fail

---

### TC-AI-04: Suggested Questions Click
**Priority:** Medium
**Type:** Manual

**Steps:**
1. Navigate to AI Chat (`/chat`)
2. Click on any suggested question button
3. Verify message is sent automatically

**Expected Result:**
- Message populates input and sends
- AI response appears
- Chat state updates correctly

**Status:** [ ] Pass [ ] Fail

---

### TC-AI-05: Empty Message Prevention
**Priority:** High
**Type:** Manual

**Steps:**
1. Navigate to AI Chat
2. Leave input empty
3. Try to submit (Enter key or button click)

**Expected Result:**
- Form does not submit
- Loading state not triggered
- No network request made

**Status:** [ ] Pass [ ] Fail

---

### TC-AI-06: Gemini API Fallback (No Key)
**Priority:** Critical
**Type:** Manual

**Steps:**
1. Remove `GEMINI_API_KEY` from edge function environment
2. Send a message in chat

**Expected Result:**
- Mock response returned (from predefined Hinglish responses)
- No error shown to user
- Chat continues to function

**Status:** [ ] Pass [ ] Fail

---

## Civic Issue Reporting Tests

### TC-REP-01: Complete Issue Submission
**Priority:** Critical
**Type:** Manual

**Steps:**
1. Navigate to Report Issue (`/report`)
2. Select category: "Pothole"
3. Enter title: "Large pothole on Main Street near market"
4. Enter description: "The pothole is approximately 2 feet wide and causing traffic issues"
5. Enter location: "Main Street, Sector 15, near City Market"
6. Enter landmark: "Opposite State Bank ATM"
7. Click Submit

**Expected Result:**
- Success screen shows with ticket ID
- Ticket ID follows format `SB-YYYY-XXXXX`
- Data saved to database

**Status:** [ ] Pass [ ] Fail

---

### TC-REP-02: Image Upload Simulation
**Priority:** High
**Type:** Manual

**Steps:**
1. Navigate to Report Issue
2. Fill required fields
3. Click "Click to upload" for photo
4. Select an image file (JPEG/PNG under 10MB)

**Expected Result:**
- Image uploads via edge function
- Preview shows uploaded image
- "Remove" button appears for re-upload

**Test Files:**
- `sample.jpg` (2MB) - Should succeed
- `large.png` (15MB) - Should fail with error message
- `document.pdf` - Should be rejected (accept only images)

**Status:** [ ] Pass [ ] Fail

---

### TC-REP-03: Category-Based Priority Assignment
**Priority:** Medium
**Type:** Manual

**Test Matrix:**

| Category | Expected Priority |
|----------|------------------|
| electricity | high |
| water | high |
| pothole | medium |
| garbage | medium |
| streetlight | medium |
| drainage | medium |
| road | medium |
| public_infrastructure | medium |
| other | medium |

**Steps:**
1. Submit issues with different categories
2. Check database or dashboard for priority assignment

**Status:** [ ] Pass [ ] Fail

---

### TC-REP-04: Form Validation
**Priority:** High
**Type:** Manual

**Validation Rules:**

| Field | Rule | Error Message |
|-------|------|---------------|
| Title | min 10 chars | "Title must be at least 10 characters" |
| Description | min 20 chars | "Description must be at least 20 characters" |
| Location | min 5 chars | "Please provide a valid location" |
| Category | required | Field validation error |

**Steps:**
1. Submit form with empty/invalid fields
2. Verify error messages appear

**Status:** [ ] Pass [ ] Fail

---

### TC-REP-05: Success State & Ticket Display
**Priority:** High
**Type:** Manual

**Steps:**
1. Complete a valid submission
2. Observe success screen
3. Note the ticket ID format

**Expected Result:**
- Green success card with CheckCircle icon
- Ticket ID displayed prominently
- "Track Your Issue" button links to `/dashboard`
- "Report Another Issue" resets form

**Status:** [ ] Pass [ ] Fail

---

## Dashboard & Tracking Tests

### TC-DASH-01: Issue List Display
**Priority:** Critical
**Type:** Manual

**Steps:**
1. Navigate to Dashboard (`/dashboard`)
2. Verify all issues display in table format

**Expected Result:**
- Columns: Ticket ID, Issue, Category, Location, Status, Date
- Each row is clickable for details
- Status badges color-coded correctly

**Status:** [ ] Pass [ ] Fail

---

### TC-DASH-02: Search Functionality
**Priority:** High
**Type:** Manual

**Test Searches:**
1. Search by ticket ID (e.g., "SB-2026-")
2. Search by title keyword
3. Search by location name

**Expected Result:**
- Results filter in real-time
- No match shows empty state
- Search clears with X or delete key

**Status:** [ ] Pass [ ] Fail

---

### TC-DASH-03: Status Filter
**Priority:** High
**Type:** Manual

**Steps:**
1. Click Status filter dropdown
2. Select each status: Pending, In Progress, Resolved, Rejected
3. Verify filtered results

**Expected Result:**
- Status badges: Pending (amber), In Progress (blue), Resolved (green), Rejected (red)
- Filter combines with other active filters

**Status:** [ ] Pass [ ] Fail

---

### TC-DASH-04: Category Filter
**Priority:** High
**Type:** Manual

**Steps:**
1. Click Category filter dropdown
2. Select each category option
3. Verify filtered results with category icon

**Status:** [ ] Pass [ ] Fail

---

### TC-DASH-05: Issue Detail Modal
**Priority:** High
**Type:** Manual

**Steps:**
1. Click on any issue row
2. Verify detail modal opens
3. Check all information displayed

**Expected Content:**
- Status and priority badges
- Title and category
- Full description
- Location with landmark
- Photo (if uploaded)
- Timeline (submitted/resolved dates)
- Admin notes (if any)

**Status:** [ ] Pass [ ] Fail

---

### TC-DASH-06: Statistics Cards
**Priority:** Medium
**Type:** Manual

**Steps:**
1. Observe stat cards at top of dashboard
2. Compare with actual filtered results

**Expected:**
- Total Issues = all records count
- Pending = status='pending' count
- In Progress = status='in_progress' count
- Resolved = status='resolved' count

**Status:** [ ] Pass [ ] Fail

---

## Security Tests

### TC-SEC-01: API Key Exposure Check
**Priority:** Critical
**Type:** Manual

**Steps:**
1. Open browser DevTools
2. Navigate through all pages
3. Check Network requests and Sources
4. Search for "GEMINI_API_KEY" or "SERVICE_ROLE" in source

**Expected Result:**
- `GEMINI_API_KEY` NOT found in any client-side code
- `SUPABASE_SERVICE_ROLE_KEY` NOT found in client code
- Only `NEXT_PUBLIC_*` variables visible (these are safe)

**Status:** [ ] Pass [ ] Fail

---

### TC-SEC-02: Environment Variable Validation
**Priority:** High
**Type:** Code Review

**Verification:**
1. Check `/lib/env.ts` - validates required vars
2. Check edge functions read from `Deno.env.get()`
3. Check no hardcoded secrets in source

**Status:** [ ] Pass [ ] Fail

---

### TC-SEC-03: RLS Policy Verification
**Priority:** Critical
**Type:** Database

**Check Database:**
```sql
SELECT * FROM pg_policies WHERE tablename IN ('civic_issues', 'chat_history');
```

**Expected:**
- Both tables have RLS enabled
- Policies allow `anon` role for public access
- No overly permissive `USING (true)` on authenticated-only tables

**Status:** [ ] Pass [ ] Fail

---

### TC-SEC-04: CORS Configuration
**Priority:** High
**Type:** Manual

**Steps:**
1. Call edge functions from browser console
2. Verify CORS headers in responses

**Expected Headers:**
```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization, X-Client-Info, Apikey
```

**Status:** [ ] Pass [ ] Fail

---

## Edge Case Handling

### TC-EDGE-01: Network Failure Handling
**Priority:** High
**Type:** Manual

**Steps:**
1. Open DevTools Network tab
2. Set to "Offline" mode
3. Try submitting a chat message or issue

**Expected Result:**
- Error toast notification appears
- App doesn't crash
- Retry possible when back online

**Status:** [ ] Pass [ ] Fail

---

### TC-EDGE-02: Very Long Input Text
**Priority:** Medium
**Type:** Manual

**Steps:**
1. In chat, send message with 2000+ characters
2. In report form, enter very long title/description

**Expected Result:**
- Either truncation or validation error
- No database overflow errors
- UI doesn't break visually

**Status:** [ ] Pass [ ] Fail

---

### TC-EDGE-03: special Characters in Inputs
**Priority:** Medium
**Type:** Manual

**Test Inputs:**
- `<script>alert('xss')</script>`
- `DROP TABLE civic_issues;`
- `{{template injection}}`
- Emojis: 🚧 🛠️ 🏗️

**Expected Result:**
- No XSS execution
- No SQL injection
- Data stored safely
- Emojis display correctly

**Status:** [ ] Pass [ ] Fail

---

### TC-EDGE-04: Zero Issues State
**Priority:** Medium
**Type:** Manual

**Steps:**
1. Clear all issues from database (test only)
2. Navigate to Dashboard

**Expected Result:**
- Empty state message displayed
- "No issues found" with helpful icon
- No js errors

**Status:** [ ] Pass [ ] Fail

---

### TC-EDGE-05: Invalid File Upload Types
**Priority:** High
**Type:** Manual

**Test Files:**
- `malware.exe` - Should reject
- `script.js` - Should reject
- `data.json` - Should reject
- `image.svg` - Should reject (SVG can contain scripts)

**Expected Result:**
- Error message "Invalid file type"
- No upload attempted

**Status:** [ ] Pass [ ] Fail

---

### TC-EDGE-06: Rapid Form Submission
**Priority:** Medium
**Type:** Manual

**Steps:**
1. Fill report form
2. Rapidly click Submit button multiple times

**Expected Result:**
- Only one submission processed
- Loading state prevents double-submit
- Single ticket ID generated

**Status:** [ ] Pass [ ] Fail

---

## Performance Tests

### TC-PERF-01: Page Load Time
**Priority:** High
**Type:** Manual

**Benchmark Targets:**
| Page | Target Load Time |
|------|-----------------|
| Home | < 2 seconds |
| Chat | < 2 seconds |
| Report | < 2 seconds |
| Dashboard | < 3 seconds |

**Steps:**
1. Clear browser cache
2. Load each page with DevTools Performance tab
3. Record First Contentful Paint and Time to Interactive

**Status:** [ ] Pass [ ] Fail

---

### TC-PERF-02: Large Dataset Dashboard
**Priority:** Medium
**Type:** Manual

**Setup:**
- Insert 500+ test issues via SQL

**Steps:**
1. Navigate to Dashboard
2. Observe load time and scrolling

**Expected Result:**
- Page loads within 5 seconds
- Scrolling is smooth
- Search/filter works without lag

**Status:** [ ] Pass [ ] Fail

---

### TC-PERF-03: Image Upload Performance
**Priority:** Medium
**Type:** Manual

**Steps:**
1. Upload 5MB image
2. Measure upload time

**Target:** < 10 seconds on standard connection

**Status:** [ ] Pass [ ] Fail

---

## Test Execution Summary

### Test Run Template

```
Date: YYYY-MM-DD
Tester: [Name]
Environment: [Dev/Staging/Prod]
Browser: [Chrome/Firefox/Safari]

Results:
- Total Tests: XX
- Passed: XX
- Failed: XX
- Blocked: XX

Failed Tests:
- [TC-ID]: [Reason/Notes]
```

---

## Automated Test Scripts (Future)

### Recommended Testing Frameworks
- **Jest** + **React Testing Library** for unit tests
- **Playwright** for E2E tests
- **MSW** for API mocking

### Sample Test Structure
```typescript
// __tests__/chat.test.tsx
describe('AI Chat', () => {
  it('sends Hinglish messages correctly', async () => {
    // Test implementation
  });

  it('handles network errors gracefully', async () => {
    // Test implementation
  });
});
```

---

## Known Issues & Limitations

| Issue | Status | Mitigation |
|-------|--------|-----------|
| Gemini API can rate limit | Known | Fallback mock responses |
| Large images slow upload | Known | Client-side compress (future) |
| No real-time updates | By Design | Manual page refresh |

---

## Contact

For testing-related questions, contact the Smart Bharat development team.

**Last Updated:** July 2026
**Version:** 1.0.0
