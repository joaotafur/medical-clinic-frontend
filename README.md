# Lambda Medical Clinic: Frontend

This is the website for **Lambda Medical Clinic**, the COSC 3380 Team 9 database project. It has a public site (doctors, locations, services, booking) and a signed in portal for patients, doctors, nurses, receptionists, lab technicians and admins.

It talks to the API in the backend repository (`medical-clinic-backend`), which holds the database, the schema and the installation steps for the database. Read that README first.

## What's in the submission

| File or folder | What it is |
| --- | --- |
| `medical-clinic-frontend/` | This repository: the React website |
| `medical-clinic-backend/` | The API server, database schema, migrations and tests |
| `lambda_medical_clinic_dump.sql` | SQL dump of the populated PostgreSQL 17 database |
| Project document | What the application does: data, user roles, triggers and reports |

## Stack

- **React 19 + TypeScript**, built with **Vite**
- **React Router 7** for pages
- **Tailwind CSS 4** for styling (theme colors and shadows are in `src/index.css`)
- **axios** for API calls, through the shared instance in `src/api.ts`

## Installing and running it locally

1. Install [Node.js](https://nodejs.org/) 22 or newer.
2. Load the database and start the backend first, following the backend README. It runs at http://localhost:4000.
3. In this folder, install the packages and start the website:

```bash
npm install
```

```bash
npm run dev
```

4. Open http://localhost:5173 and sign in at `/login` with any test account's email and password.

The backend only accepts requests from `http://localhost:5173`, so keep the website on that port.

| Script | What it does |
| --- | --- |
| `npm run dev` | Start the development server at http://localhost:5173 |
| `npm run build` | Type-check (`tsc -b`) and build to `dist/` |
| `npm run preview` | Serve the built site locally |
| `npm run lint` | Run ESLint |

### Backend connection

`src/api.ts` picks the backend address:

- **Local development**: `http://localhost:4000`, or `VITE_API_URL` if you set it in a `.env` file (git ignores it).
- **Hosted (Vercel)**: the same site. `vercel.json` forwards `/api/*` to the hosted backend. Don't set `VITE_API_URL` in Vercel.

Every request sends the login cookie (`withCredentials: true`).

## Roles and pages

Everyone signs in on one page, `/login`, with an email and password. The email decides the account and the role.

| Role | Main pages |
| --- | --- |
| Public visitor | Home, Find a Doctor, doctor profiles, Locations, Services, About, Contact, booking (sign up or sign in to finish) |
| Patient | Dashboard, Appointments (book with the primary doctor preselected, Upcoming, History and Cancelled tabs, cancel), Medical Records (After Visit Summary PDF, prescriptions, lab results), Billing (pay by card, invoice PDF), Forms, Profile (primary doctor, contacts, insurance) |
| Doctor | Dashboard, Appointments, visit page (summary, diagnoses, prescriptions, lab orders), Patients and full charts, Referrals (send with a reason, approve or deny as the primary doctor), Lab Orders, Schedule |
| Nurse | Dashboard, Patient Appointments, intake (vitals, nursing notes, allergies, home medications; editable only on the visit day at the nurse's office after check-in), Supplies |
| Receptionist | Dashboard, Appointments (book, check in, check out, No Show, cancel), Patients (register, insurance, forms, portal logins), Billing (payments, refund requests) |
| Lab technician | Dashboard, Lab Orders (start a test, enter results) |
| Admin | Dashboard with yearly PDF report, Employees, Offices, Specialties, Schedules and shifts, Patients, Billing (invoices, refunds, voids, fees, no-show fee), Reports (3 data reports) |

Every signed in role also has a **My Profile** page with a password change. `src/context/AuthContext.tsx` keeps the signed in user, and `src/layouts/RoleLayout.tsx` keeps each role inside its own pages.

## Referrals

1. A doctor who has seen the patient writes a referral to a specialist with a **reason**.
2. It goes to the **patient's primary doctor**, who approves it or denies it with a reason. If the primary doctor wrote it, it's approved right away.
3. Once approved, the patient or the front desk can book that specialist. Specialists can't be booked without one.

The patient only sees each referral's status, not the reasons.

## Admin reports

The admin **Reports** page (`/admin/reports`) has 3 reports, each with its own page:

| Report | Question it answers |
| --- | --- |
| Missed Appointments | Where are no-shows and cancellations coming from, and which patients should we follow up with? |
| Unpaid Balances | Who owes the clinic money, how long has it been unpaid, and is it patients or insurance that's behind? |
| Schedule Utilization | How full is each doctor's schedule compared to the time slots they offer? It also has a week calendar per doctor and an all-doctors calendar. |

Every report has a time period filter (weekly, monthly, quarterly, yearly or a custom range), filters for its topic (office, doctor, specialty, patient and more), tables you can sort by clicking a column header, a search box, and **Download PDF**. Filters are saved in the page link, so a filtered report can be bookmarked.

## Project layout

```text
src/
  main.tsx             app entry point
  App.tsx              every route
  api.ts               shared axios instance and apiErrorMessage()
  types.ts             shared data shapes
  index.css            Tailwind import, theme colors, shadows
  context/             signed in user (AuthContext)
  layouts/             public layout and the role layout with the sidebar
  components/          shared UI: form fields, pop-ups, tables, sidebar, PDF reports
    admin/             admin page frame, status badges, report period picker, sortable table, week navigation
    public/            public site pieces (header, footer, booking)
  pages/
    public/            public site and booking flow
    auth/              sign in
    patient/ doctor/ nurse/ receptionist/ lab/ admin/
    shared/            staff profile page
    staff/             nurse and receptionist dashboard switch
  services/            older shared API helpers (admin, booking, public)
  utils/               IDs, clinic time, print reports, calendars, vitals flags, small helpers
public/                favicon
vercel.json            hosting settings (forwards /api to the backend)
```

## PDF reports

Invoices, After Visit Summaries, the admin yearly report and the 3 admin reports print through the browser ("Save as PDF"):

- `components/PrintReport.tsx` is the report layout.
- `utils/printReport.ts` opens the print window and names the file.

## Deployment

The website is hosted on **Vercel** and builds from the `main` branch. `vercel.json` sends `/api/*` to the backend and every other path to `index.html`, so page refreshes work.
