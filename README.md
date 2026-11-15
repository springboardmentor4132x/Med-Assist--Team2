# MedAssist AI - Module 1: User Management

All 5 functionalities of the module are done: registration, login with JWT, patient profile management, role-based access control (RBAC) and medical history tracking (with admin page and doctor-patient assignment).

## Tech used
- Frontend: HTML, CSS, JavaScript
- Backend: Python Flask
- Database: SQLite
- Password hashing: bcrypt
- Authentication: JWT (Flask-JWT-Extended)

## Folder structure
```
medassist-ai/
  backend/    app.py, config.py, models.py, auth.py, user_profile.py, admin.py,
              history.py, doctor.py, decorators.py, make_admin.py, requirements.txt
  frontend/   index, register, login, dashboard, profile, history, patients, admin,
              privacy, terms (.html)
    css/      style.css, auth.css, dashboard.css
    js/       common.js, register.js, login.js, dashboard.js, profile.js,
              history.js, patients.js, admin.js
```

## How to run

### 1. Backend
```
cd backend
python -m venv venv
venv\Scripts\activate          (Windows)
source venv/bin/activate       (Mac/Linux)
pip install -r requirements.txt
python app.py
```
Backend runs at http://127.0.0.1:5000 and database.db is created automatically.

### 2. Frontend
Open the `frontend` folder in VS Code, right click `index.html` and choose "Open with Live Server".
(It usually opens on http://127.0.0.1:5500)

## API endpoints

| Method | URL | Description | Token needed |
|---|---|---|---|
| POST | /api/register | create new account | No |
| POST | /api/login | login, returns token | No |
| GET | /api/me | get logged in user details | Yes |
| GET | /api/profile | view own profile | Yes |
| PUT | /api/profile | update own profile (name, age, gender, contact) | Yes |
| GET | /api/profile/<id> | view a profile (own, admin, or a doctor whose patient it is) | Yes |
| GET | /api/admin/users | list all users | Admin only |
| GET | /api/admin/stats | user count per role | Admin only |
| PUT | /api/admin/users/<id>/role | change a user's role | Admin only |
| DELETE | /api/admin/users/<id> | delete a user (also deletes his history and assignments) | Admin only |
| GET | /api/admin/assignments | list doctor-patient assignments | Admin only |
| POST | /api/admin/assignments | assign a patient to a doctor | Admin only |
| DELETE | /api/admin/assignments/<id> | remove an assignment | Admin only |
| POST | /api/history | add own medical history entry | Patient only |
| GET | /api/history | view own medical history | Patient only |
| GET | /api/doctor/patients | patients assigned to the logged in doctor | Doctor only |
| GET | /api/history/patient/<id> | history of an assigned patient | Doctor only (assigned patients) |

## Postman tests

**1. Register** - POST http://127.0.0.1:5000/api/register (Body > raw > JSON)
```json
{
  "full_name": "Rahul Kumar",
  "email": "rahul@mail.com",
  "password": "secret123",
  "age": 25,
  "gender": "Male",
  "contact": "9876543210",
  "role": "patient"
}
```
`role` is optional and can be patient, doctor or clinic (the tabs on the register page). Anything else (like admin) gives 400. Expected: 201 "Registration successful". Send the same request again, expected: 409 "Account already exists with this email".

**2. Login** - POST http://127.0.0.1:5000/api/login
```json
{ "email": "rahul@mail.com", "password": "secret123" }
```
Expected: 200 with a token. Wrong password gives 401 "Invalid email or password".

**3. Protected route** - GET http://127.0.0.1:5000/api/me
Add header `Authorization: Bearer <token from login>`. Expected: 200 with user details.
Without token: 401 "Please login to continue".

**4. Update profile** - PUT http://127.0.0.1:5000/api/profile (with token)
```json
{ "full_name": "Rahul K", "age": 31, "gender": "Male", "contact": "9000000001" }
```
Expected: 200 "Profile updated successfully". Age 500 gives 400.

**5. Admin APIs** - GET http://127.0.0.1:5000/api/admin/users
With a patient token: 403 "You do not have permission to do this". With an admin token: 200 with the users list.

**6. Change role** - PUT http://127.0.0.1:5000/api/admin/users/2/role (admin token)
```json
{ "role": "doctor" }
```

**7. Medical history** - POST http://127.0.0.1:5000/api/history (patient token)
```json
{ "condition": "Typhoid fever", "record_date": "2025-03-10", "notes": "Took antibiotics" }
```
Expected: 201. A future date or a doctor token gives 400 / 403. GET /api/history returns your own records.

**8. Assign a patient to a doctor** - POST http://127.0.0.1:5000/api/admin/assignments (admin token)
```json
{ "doctor_id": 3, "patient_id": 2 }
```
Expected: 201. Then GET /api/history/patient/2 with the doctor's token gives 200, and with another doctor's token gives 403.

## Doctor and clinic registration
The register page has Patient, Doctor and Clinic tabs. Nobody can become a doctor, clinic or admin by registering: every new account is created as a patient and the chosen type is saved in the `requested_role` column. The admin sees it in the "Requested" column of the Manage Users page and changes the role if the request is fine.

## How doctor access works
A doctor can see a patient's profile and history only after the admin assigns that patient to him (doctor_patient table). If the role of the doctor or the patient is changed, or a user is deleted, the related assignments are removed automatically.

## How to make the first admin
Nobody can register as admin, so make the first one from the terminal.
1. Register a normal account from the website.
2. In the backend folder run: `python make_admin.py youremail@gmail.com`
3. Logout and login again, the dashboard now shows "Manage Users".

## Roles
| Role | Dashboard links |
|---|---|
| patient | My Profile, Enter Symptoms*, Health Reports*, Medical History |
| doctor | My Profile, My Patients (assigned patients and their history), Health Reports* |
| clinic | My Profile, Clinic Information* |
| admin | My Profile, Manage Users |

*coming soon (these belong to the next modules: symptom analysis and prediction)

The role is checked from the database on every protected admin request (decorators.py), so a changed role works immediately.

## Common errors
- **CORS error / Failed to fetch**: backend is not running, or API_URL in js/common.js is wrong.
- **ModuleNotFoundError**: activate the virtual environment and run pip install again.
- **403 on admin page**: your account is not an admin, run make_admin.py and login again.
- **no such column: requested_role**: old database.db is used, restart the backend once (the column is added automatically) or delete database.db.
- **Token expired**: token is valid for 1 hour (change JWT_EXPIRY in config.py).
