# API Beginner Guide

This guide explains the APIs in this project in plain language. It is written for someone who is not technical yet, but wants to inspect the app and practice API calls in the browser console.

## 1. What Is An API?

Think of the frontend app as a counter in an office.

- The screen you see is the counter.
- The database is the record room.
- An API is the staff member who goes to the record room and brings data back.

When you open a page like Tasks, Projects, Telecaller, or Invoice Mail, the app asks an API:

```text
Please give me the tasks.
Please give me the projects.
Please save this form.
Please upload this file.
```

The API replies with data, usually in JSON format.

JSON looks like this:

```json
{
  "id": "123",
  "name": "Example Task",
  "status": "Started"
}
```

## 2. The Main API Files

Most API code is in:

```text
src/requests/requests.tsx
```

Login API code is in:

```text
src/pages/AuthPages/SignIn.tsx
```

Some RxPad API code is directly inside:

```text
src/pages/Telecaller.tsx
```

Attachment upload helper is in:

```text
src/utils/uploadAttachments.ts
```

## 3. The Main API Domains

The project talks to these main systems:

| System | Base URL | Simple Meaning |
|---|---|---|
| OPS | `https://ops5.indigitalit.com/api/v1` | Main operations system: tasks, projects, users, invoices, work requests |
| CRM | `https://crm5.indigitalit.com/api/v1` | CRM system: customers, purchase orders, rate requests, accounts |
| CRM custom API | `https://crm5.indigitalit.com/custom_api` | Custom CRM reports like leads and CS touch points |
| OPS custom API | `https://ops5.indigitalit.com/get_data/api` | Custom OPS reports like project comments |
| RxPad | `https://rxpad.pixika.ai/api/v1` | RxPad records and RxPad attachments |
| Google OAuth | Google login service | Used only for login |

## 4. Important Words

### Request

A request is a question sent from the app to the API.

Example:

```text
Give me 20 projects.
```

### Response

A response is the answer from the API.

Example:

```json
{
  "list": [
    { "id": "1", "name": "Project A" }
  ],
  "total": 1
}
```

### Header

A header is extra information sent with the request. In this project, headers are mostly used for permission.

Example:

```js
headers: {
  "Espo-Authorization": userCred
}
```

### GET, POST, PUT, PATCH, DELETE

These are API action types:

| Method | Meaning | Beginner Example |
|---|---|---|
| `GET` | Read data | Show me tasks |
| `POST` | Create new data | Create a new comment |
| `PUT` | Replace/update data | Update a task |
| `PATCH` | Update some fields | Change task status |
| `DELETE` | Delete data | Delete a task |

For practice, use only `GET`. `POST`, `PUT`, `PATCH`, and `DELETE` can change real data.

## 5. How Login Works

There are two login paths.

### Google Login

File:

```text
src/pages/AuthPages/SignIn.tsx
```

Flow:

1. User signs in with Google.
2. App reads the Google email.
3. App asks OPS API: "Is there an OPS user with this email?"
4. OPS returns user details.
5. App creates a credential called `userCred`.
6. App stores it in browser local storage.

The stored values are:

```text
opsUser
opsUsrCred
```

### Vendor Login

Vendor login uses username and password directly.

The app calls:

```text
GET https://ops5.indigitalit.com/api/v1/App/user
```

with this header:

```text
Espo-Authorization
```

## 6. How To Inspect APIs In Browser

Open the app in Chrome or Edge.

1. Right click on the page.
2. Click `Inspect`.
3. Open the `Network` tab.
4. Click `Fetch/XHR`.
5. Refresh the page.
6. Click any API request.

Look at these tabs:

| DevTools Tab | What You See |
|---|---|
| Headers | URL, method, request headers |
| Payload | Data sent to API, mostly for POST/PUT/PATCH |
| Preview | Easy-to-read response |
| Response | Raw JSON response |

This is the easiest way to understand what the app is doing.

## 7. Console Practice Setup

After login, open DevTools Console and paste this:

```js
const userCred = JSON.parse(localStorage.getItem("opsUsrCred"));
const opsUser = JSON.parse(localStorage.getItem("opsUser"));

console.log("Logged in user:", opsUser);
console.log("Credential exists:", Boolean(userCred));
```

If `Credential exists` is `true`, you can practice safe `GET` API calls.

## 8. Practice 1: Check Current Logged-In User

Paste this in Console:

```js
const userCred = JSON.parse(localStorage.getItem("opsUsrCred"));

fetch("https://ops5.indigitalit.com/api/v1/App/user", {
  method: "GET",
  headers: {
    "Content-Type": "application/json",
    "Espo-Authorization": userCred
  }
})
  .then(res => res.json())
  .then(data => console.log(data));
```

What this means:

- `fetch(...)` asks the API for data.
- The URL asks OPS: "Who is the current app user?"
- `Espo-Authorization` proves you are logged in.
- `console.log(data)` prints the answer.

## 9. Practice 2: Fetch First 5 Tasks

Paste this:

```js
const userCred = JSON.parse(localStorage.getItem("opsUsrCred"));

fetch("https://ops5.indigitalit.com/api/v1/Task?select=id,name,status,assignedUserName,createdAt&maxSize=5&offset=0&orderBy=createdAt&order=desc", {
  method: "GET",
  headers: {
    "Content-Type": "application/json",
    "Espo-Authorization": userCred
  }
})
  .then(res => res.json())
  .then(data => console.table(data.list));
```

What you should see:

- A table of up to 5 tasks.
- Each row may show task ID, name, status, assigned user, and created date.

Where this is used in the app:

- Dashboard home
- Manage Task pages
- Task detail modals

Main code:

```text
src/requests/requests.tsx
```

Important functions:

```text
useTasksQuery
useManageTabTasksQuery
useTaskDetailQuery
```

## 10. Practice 3: Fetch First 5 Projects

Paste this:

```js
const userCred = JSON.parse(localStorage.getItem("opsUsrCred"));

fetch("https://ops5.indigitalit.com/api/v1/CProject2?select=id,name,projectStatus,workflow,company,createdAt&maxSize=5&offset=0&orderBy=createdAt&order=desc", {
  method: "GET",
  headers: {
    "Content-Type": "application/json",
    "Espo-Authorization": userCred
  }
})
  .then(res => res.json())
  .then(data => console.table(data.list));
```

What this means:

- `CProject2` is the project entity in OPS.
- `select=...` means "only give me these columns."
- `maxSize=5` means "give me only 5 records."
- `offset=0` means "start from the beginning."

Where this is used:

- Project page
- O2D Dashboard
- Procurement report
- Production report
- Vendor management

Important hooks:

```text
useProjectsQuery
useO2DProjectsQuery
useFilteredProjectsQuery
useProductionReportProjectsQuery
```

## 11. Practice 4: Fetch Users

Paste this:

```js
const userCred = JSON.parse(localStorage.getItem("opsUsrCred"));

fetch("https://ops5.indigitalit.com/api/v1/User?userType=internal&select=id,name,userName,emailAddress&maxSize=10&offset=0&orderBy=userName&order=asc", {
  method: "GET",
  headers: {
    "Content-Type": "application/json",
    "Espo-Authorization": userCred
  }
})
  .then(res => res.json())
  .then(data => console.table(data.list));
```

Where this is used:

- Dropdowns for assigning tasks
- User lists
- Task edit forms
- Comment forms

Important hooks:

```text
useGetAllUsersQuery
useGetCsUsersQuery
useGetSalesUsersQuery
useVendorUsersQuery
```

## 12. Practice 5: Inspect Task Comments In Network

Task comments use an API key inside the app code. That key is added by the built app, so this is better practiced in the Network tab instead of manually typing it in Console.

Steps:

1. Open any task detail or comments modal in the app.
2. Open DevTools.
3. Go to `Network`.
4. Click `Fetch/XHR`.
5. Look for a request containing:

```text
/api/v1/Task/
/stream
```

You may see a URL like:

```text
https://ops5.indigitalit.com/api/v1/Task/TASK_ID/stream?filter=posts&maxSize=200&offset=0&orderBy=number&order=desc
```

What this means:

- `Task/TASK_ID` means comments for one specific task.
- `stream` means the timeline/posts area.
- `filter=posts` means only post/comment items.
- `order=desc` means newest first.

In the code, task comments use:

```text
VITE_TASK_X_API_KEY
```

Code location:

```text
src/requests/requests.tsx
```

Important hook:

```text
useTaskCommentsQuery
```

## 13. Practice 6: Fetch Invoice Mail Records

Paste this:

```js
const userCred = JSON.parse(localStorage.getItem("opsUsrCred"));

fetch("https://ops5.indigitalit.com/api/v1/CInvoiceMail?select=id,createdAt,invoiceNumber,newProjectName,invoiceQuantity,invoiceDate,formStatus,amount&maxSize=5&offset=0&orderBy=createdAt&order=desc", {
  method: "GET",
  headers: {
    "Content-Type": "application/json",
    "Espo-Authorization": userCred
  }
})
  .then(res => res.json())
  .then(data => console.table(data.list));
```

Where this is used:

- Invoice mails dashboard
- Billing form
- O2D dashboard invoice details

Important hooks:

```text
useInvoiceMailRecordsQuery
useInvoiceMailDetailQuery
useCreateInvoiceMailMutation
useUpdateInvoiceMailMutation
```

Remember:

- Query hooks read data.
- Mutation hooks create or update data.

## 14. Practice 7: Fetch Work Requests

Paste this:

```js
const userCred = JSON.parse(localStorage.getItem("opsUsrCred"));

fetch("https://ops5.indigitalit.com/api/v1/CWorkRequestForm?select=id,name,status,createdAt,requestType,companyName,assignedUserId&maxSize=5&offset=0&orderBy=createdAt&order=desc", {
  method: "GET",
  headers: {
    "Content-Type": "application/json",
    "Espo-Authorization": userCred
  }
})
  .then(res => res.json())
  .then(data => console.table(data.list));
```

Where this is used:

- Work Request form
- App sidebar count/list
- Work request list/detail screens

Important hooks:

```text
useWorkRequestsQuery
useWorkRequestDetailQuery
useCreateWorkRequestMutation
```

## 15. Practice 8: Fetch Telecaller Tickets

Paste this:

```js
const userCred = JSON.parse(localStorage.getItem("opsUsrCred"));

fetch("https://ops5.indigitalit.com/api/v1/CTicketTracker?select=id,name,createdAt,employeeCode,companyName,division,product,status&maxSize=5&offset=0&orderBy=createdAt&order=desc", {
  method: "GET",
  headers: {
    "Content-Type": "application/json",
    "Espo-Authorization": userCred
  }
})
  .then(res => res.json())
  .then(data => console.table(data.list));
```

Where this is used:

```text
src/pages/Telecaller.tsx
```

Important hooks:

```text
useTelecallerTicketsQuery
useCreateTelecallerTicketMutation
useUpdateTelecallerTicketMutation
```

## 16. How To Read A URL

Example:

```text
https://ops5.indigitalit.com/api/v1/Task?select=id,name,status&maxSize=5&offset=0
```

Break it into parts:

| Part | Meaning |
|---|---|
| `https://ops5.indigitalit.com` | Server |
| `/api/v1` | API version |
| `/Task` | Data type, here tasks |
| `?` | Start of options |
| `select=id,name,status` | Columns needed |
| `maxSize=5` | Number of records |
| `offset=0` | Starting point |

## 17. Common Entities In This Project

| Entity | Meaning |
|---|---|
| `Task` | Work/task records |
| `CProject2` | Project records |
| `User` | User records |
| `Attachment` | Uploaded files |
| `Note` | Comments/posts |
| `CInvoiceMail` | Invoice mail form records |
| `CStitchFileManagement` | Stitch/vendor invoice request data |
| `CVendorDispatch` | Vendor dispatch data |
| `CWorkRequestForm` | Work request forms |
| `CChangeRequest` | Change request forms |
| `CTicketTracker` | Telecaller tickets |
| `CVendorTracker` | Vendor management records |
| `CInventoryRequest` | Inventory requests |
| `CHiringProcess` | Hiring form submissions |
| `CExitProcess` | Exit form submissions |

## 18. How React Query Fits In

The project uses React Query.

For a beginner, understand it like this:

```text
React Query = a helper that fetches API data and remembers it for the page.
```

Example from the code:

```ts
useTasksQuery(role, userId, userCred)
```

This means:

```text
Get tasks for this role and user using the login credential.
```

Two important words:

| Word | Meaning |
|---|---|
| Query | Read data |
| Mutation | Change data |

So:

```text
useProjectsQuery = reads projects
useCreateTaskMutation = creates a task
useUpdateVendorDispatchMutation = updates vendor dispatch
```

## 19. Where APIs Are Used By Page

| Page/File | Main API Purpose |
|---|---|
| `src/pages/AuthPages/SignIn.tsx` | Login, user lookup, app user validation |
| `src/pages/Dashboard/Home.tsx` | Tasks, projects, users |
| `src/pages/Dashboard/ManageTask.tsx` | Task counts and filters |
| `src/pages/Dashboard/ManageTaskDetails.tsx` | Task list, task export, delete task |
| `src/pages/Dashboard/Project.tsx` | Project list and project filters |
| `src/pages/Dashboard/ProjectDetails.tsx` | Project details, linked tasks, stitch file records, vendor mail |
| `src/pages/Dashboard/O2DDashboard.tsx` | O2D projects, invoices, feedback, stitch/vendor dispatch data |
| `src/pages/Dashboard/InvoiceMails.tsx` | Invoice mail list and details |
| `src/pages/Forms/BillingForm.tsx` | Create invoice mail |
| `src/pages/Forms/WorkRequest.tsx` | Create work request, load therapies |
| `src/pages/Forms/ExitForm.tsx` | Create and list exit forms |
| `src/pages/Forms/HiringForm.tsx` | Create and list hiring forms |
| `src/pages/Telecaller.tsx` | Telecaller companies, divisions, projects, tickets, RxPad update |
| `src/pages/VendorManagement.tsx` | Vendor project list, create vendor records |
| `src/pages/VendorInvoiceMailForm.tsx` | Vendor invoice request forms and dispatch updates |
| `src/pages/ProcurementReport.tsx` | Vendor tracker and production status project data |
| `src/pages/ProductionReport.tsx` | Production report project data |
| `src/pages/RateRequestReport.tsx` | CRM rate requests and rate responses |
| `src/pages/Kribado.tsx` | CRM purchase orders for Kribado workflows |
| `src/pages/CsTouchPoints.tsx` | CRM CS touch points and contact activity/email history |
| `src/pages/Leads/BdmCsLeads.tsx` | CRM custom lead data |

## 20. Very Important Safety Notes

When practicing in Console:

- Use only `GET` requests.
- Do not practice `POST`, `PUT`, `PATCH`, or `DELETE` unless you are sure.
- Do not share values from `localStorage`.
- Do not share `opsUsrCred`.
- Do not share API keys from `.env`.

Simple rule:

```text
GET = safe for learning because it reads data.
POST/PUT/PATCH/DELETE = risky because it can change real data.
```

## 21. Small Practice Exercise

Try this:

1. Open the app.
2. Open DevTools.
3. Go to Network tab.
4. Refresh.
5. Click one API request.
6. Find the URL.
7. Find the method.
8. Find the response.
9. Copy only the URL path/entity name, not credentials.
10. Search that entity in `src/requests/requests.tsx`.

Example:

If Network shows:

```text
/api/v1/CProject2
```

Search:

```text
CProject2
```

You will find the code that made the request.

## 22. Beginner Mental Model

Use this mental model:

```text
Page -> API hook -> fetch call -> server -> JSON response -> table/card/form on screen
```

Example:

```text
Project page
-> useProjectsQuery
-> fetch CProject2
-> OPS server
-> project JSON
-> project table
```

That is the basic flow of almost every API in this project.

