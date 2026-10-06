

# Task Tracker – Fix Notes

## 1. Summary

I reviewed the task search API and React task-loading flow and focused on correctness, reliability, and invalid-input handling within the assignment timebox.

### Fixes made

**1. SQL filtering bug – `TaskRepository.java`**
The search query had an `AND`/`OR` precedence issue. Without parentheses, the `archived` and `status` filters were not applied consistently to both title and description matches. I grouped the title/description conditions with parentheses so the logic is:

`archived = false AND (title matches OR description matches) AND status matches`.

**2. Artificial request delay – `TaskController.java`**
The API contained a query-length-based `Thread.sleep()`. It unnecessarily blocked the request thread and increased response latency. I removed the artificial delay because it had no business purpose.

**3. Pagination validation – `TaskController.java`**
Invalid values such as `page=0`, negative pages, `pageSize=0`, or excessively large page sizes could produce invalid ranges or excessive requests. I now require `page >= 1` and `pageSize` between 1 and 100.

**4. Invalid status handling – `TaskController.java`**
`TaskStatus.valueOf()` could throw an exception for an unknown status and result in a server error. I now catch the invalid value and return `400 Bad Request`.

**5. React async request handling – `useTasks.js`**
Older search requests could finish after newer requests and overwrite the latest results. I added `AbortController`. I also clear previous errors and ensure loading is settled after successful or failed requests.

## 2. Not Changed / Remaining Risks

Database pagination is still performed in Java after loading all matching rows. A production implementation should use database-level pagination and a count query. Search debouncing and resetting the page when filters change are also possible UX improvements.

## 3. Testing

I manually tested search, status filtering, pagination validation, invalid status, and frontend request behavior.

## 4. AI Usage

I used AI as a debugging and review assistant to reason about SQL operator precedence, API validation, and frontend asynchronous request handling. I verified the suggested issues against the actual code and tested the resulting behavior locally.

### One important correction

Your assignment says **`NOTES.md`**, not a new README. So your project should look roughly like:

```text
task-tracker/
│
├── backend/
├── frontend/
├── db/
├── handwritten/
├── README.md          ← original assignment README
└── NOTES.md           ← YOUR new file
```

### Also, your handwritten notes should cover the bugs

For each of the 5 fixes, write:

```text
1. Location
2. Bug
3. How I discovered it
4. Root cause
5. Fix
6. Why I chose this fix
```

For example, for SQL:

```text
Bug:
GET /api/tasks?status=OPEN returned DONE/IN_PROGRESS tasks.

Root cause:
AND has higher precedence than OR, so the SQL conditions
were grouped incorrectly.

Fix:
Added parentheses around title OR description.

Why:
This ensures archived and status filters apply to every
search result.
```

