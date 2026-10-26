# iSiri Work Planner & Attendance

A single-page web app for **iSiri — Architecture + Design** that handles staff attendance, daily task planning, project tracking and site-visit logs. Employees plan and close out their day on a Jira-style board; the principal architect gets a dashboard, an attendance calendar and per-day Excel reports.

The whole app is one file, `index.html`. There is no build step. Data is stored in Google Firebase (Firestore) so everyone sees the same live information from any device.

---

## Features

### Employees
- Sign in with name and password. **Attendance is recorded automatically** at sign-in.
- Add tasks for today or **schedule them for any future date**. Every task belongs to a project.
- Jira-style board: **To do → In progress → Completed**. Drag cards between columns or use the buttons. Tick a task to complete it and add remarks.
- **Finish day** button: tick off the day's tasks, add final remarks, optionally carry unfinished tasks to tomorrow, then log off (records the time out).
- **Team view:** open any colleague's board for any date (read-only).
- **Projects:** create projects, **edit project name/client/location/description (fix typos)**, see every task in a project (filter by person), mark a project completed or reopen it, delete a task (one confirmation) or a project (two confirmations, since everyone sees it).
- **Site visits:** log visits with date, who went and remarks, inside each project.
- **Calendar:** see holidays and headlines declared by the principal.

### Principal architect
- Dashboard: who is present, tasks completed, active projects, recent site visits, a 14-day attendance chart and upcoming holidays.
- **Attendance calendar** for every day, with names and in/out times. Past days are permanently stored.
- **Declare holidays and add headlines** to any day; every employee sees them. Edit them, clear them or **move them to another date**.
- **Edit any employee's task** (title, project, date, status, remarks), for example to fix a wrong scheduled date.
- View each employee's tasks and completion progress for today or any other day.
- **Download an Excel report for any single day** (attendance with in/out times, tasks grouped by project, project summary, site visits), or export everything.

---

## Tech stack

| Part | What it uses |
|---|---|
| App | Plain HTML, CSS and JavaScript in one file |
| Database | Firebase Firestore (live sync) |
| Connection | Firebase Anonymous Authentication (used only so the database accepts the app's requests) |
| Excel export | SheetJS (loaded from cdnjs) |
| Fonts | Google Fonts |
| Hosting | Netlify, deployed from this GitHub repository |

---

## Setup

### 1. Firebase
1. Create a project at [console.firebase.google.com](https://console.firebase.google.com).
2. **Firestore Database → Create database.** Note the **Database ID** shown in the console.
3. **Authentication → Sign-in method → enable Anonymous.**
4. **Firestore → Rules** — publish:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /studiolog/{doc} {
         allow read, write: if request.auth != null;
       }
     }
   }
   ```
5. **Project settings → Your apps → Web app** — copy the config values.

### 2. Configure the app
Near the top of `index.html`, fill in:

```js
const FIREBASE_CONFIG = { apiKey: "...", authDomain: "...", projectId: "...", storageBucket: "...", messagingSenderId: "...", appId: "..." };
const FIRESTORE_DATABASE_ID = "isiri-log";   // must match the Database ID in the Firebase console
```

While `FIREBASE_CONFIG` still holds placeholder values, the app runs in **demo mode**: it works, but data is kept only in that one browser.

### 3. Deploy (Netlify)
1. In Netlify choose **Add new site → Import an existing project → GitHub** and select this repository.
2. Leave **Build command** and **Publish directory** empty.
3. After the first deploy, add the site address (for example `your-site.netlify.app`, no `https://`) in Firebase under **Authentication → Settings → Authorized domains**. Do the same for any custom domain.

### 4. First use
Open the live site and create the **Principal Architect** account first. Only one principal account can exist. Then share the link with staff, who create **Employee** accounts themselves.

---

## Updating the app
Replace `index.html` in this repository (upload it on GitHub or `git push`). Netlify redeploys automatically on the same link. No data is lost, because it all lives in Firebase.

---

## How data is stored
All records are documents in the Firestore collection `studiolog`, named `<type>__<id>`:

| Prefix | Contents |
|---|---|
| `users__` | Accounts: name, role, salted password hash |
| `attendance__` | One per person per day: time in, time out |
| `projects__` | Project name, client, location, status |
| `tasks__` | Task, project, owner, date, status, remarks |
| `visits__` | Site visits: project, date, visitors, remarks |
| `days__` | Holiday / headline for a date |
| `_ping` | Startup check that saving works (safe to ignore) |

---

## Troubleshooting

The login page shows a coloured box when something is wrong with the cloud connection:

| Message | Fix |
|---|---|
| Amber "Demo mode" | The Firebase config in `index.html` still has placeholders. |
| `permission-denied` | Re-publish the rules above and confirm Anonymous sign-in is enabled. |
| `not-found` | `FIRESTORE_DATABASE_ID` does not match the database in the console. |
| `api-key` | The `apiKey` was copied wrongly — copy it again. |
| `unauthorized-domain` | Add the site address under Firebase → Authentication → Settings → Authorized domains. |

**Forgotten principal password:** passwords cannot be recovered. In Firestore, delete the principal's `users__…` document, then create the principal account again in the app. Tasks, projects and attendance are not affected.

**Clearing all data:** in Firestore, open the `studiolog` collection, choose the ⋮ menu, then **Delete collection**.

---

## Security notes
- Passwords are stored as salted SHA-256 hashes, never in plain text.
- The Firebase config in `index.html` is not a secret (every Firebase web app exposes it), but the rules above allow any visitor of the app to read and write the collection. Treat this as an internal tool and **do not put sensitive information in task remarks or site-visit notes**.
- Keep the repository **private** if you can, and restrict the API key to your site's address in Google Cloud Console → APIs & Services → Credentials.

## Known limitations
- Only one principal account can exist.
- There is no in-app "forgot password" flow; use the reset steps above.
- Anyone with the link can create an employee account.

---

© iSiri — Architecture + Design. Internal use only.
