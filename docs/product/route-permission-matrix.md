# Route & Permission Matrix - Phrasongsa Timetable System

**Version:** 1.0  
**Last Updated:** December 16, 2025  
**Auth System:** Better Auth with Google OAuth

---

## 1. Role Definitions

### 1.1 Application Roles

```typescript
type AppRole = "admin" | "teacher" | "student"
```

| Role | Description | Authentication | Capabilities |
|------|-------------|----------------|--------------|
| `admin` | System administrator | Required (Google OAuth / Email) | Full CRUD on all entities, publish, export |
| `teacher` | Teaching staff | Optional (linked via email) | View own schedules, profile |
| `student` | Student user | Optional | View class schedules |
| `guest` | Unauthenticated visitor | None | Browse published schedules only |

### 1.2 Role Hierarchy

```
admin > teacher > student > guest
```

- Admin inherits all permissions
- Higher roles can access lower role routes
- Guest has most restricted access

---

## 2. Route Permission Matrix

### Legend

| Symbol | Meaning |
|--------|---------|
| ✅ | Full access |
| 👁️ | Read-only access |
| ❌ | No access |
| 🔒 | Own data only |
| ⚠️ | Conditional access |

---

### 2.1 Public Routes (No Auth Required)

| Route | Pattern | Guest | Student | Teacher | Admin | Data Entities | Notes |
|-------|---------|-------|---------|---------|-------|---------------|-------|
| Homepage | `/` | ✅ | ✅ | ✅ | ✅ | teacher, gradelevel | Teacher search, class browse |
| Teacher Schedule | `/teachers/[id]` | 👁️ | 👁️ | 👁️ | ✅ | teacher, class_schedule, timeslot, room | Public view of teacher's timetable |
| Teacher Schedule (term) | `/teachers/[id]/[term]` | 👁️ | 👁️ | 👁️ | ✅ | teacher, class_schedule, timeslot | Term-specific view |
| Class Schedule | `/classes/[gradeId]` | 👁️ | 👁️ | 👁️ | ✅ | gradelevel, class_schedule, timeslot | Public view of class timetable |
| Class Schedule (term) | `/classes/[gradeId]/[term]` | 👁️ | 👁️ | 👁️ | ✅ | gradelevel, class_schedule | Term-specific view |
| Sign In | `/signin` | ✅ | ✅ | ✅ | ✅ | User, Session | Auth page |
| Sign Up | `/signup` | ✅ | ✅ | ✅ | ✅ | User, Account | Registration |

---

### 2.2 Authentication Routes

| Route | Pattern | Guest | Student | Teacher | Admin | Backend Endpoint | Notes |
|-------|---------|-------|---------|---------|-------|-----------------|-------|
| Auth Callback | `/api/auth/*` | ✅ | ✅ | ✅ | ✅ | Better Auth handlers | OAuth callbacks, session management |
| Sign Out | `/api/auth/signout` | ❌ | ✅ | ✅ | ✅ | POST | Destroys session |

---

### 2.3 Dashboard Routes (Auth Required)

| Route | Pattern | Guest | Student | Teacher | Admin | Data Entities | Actions |
|-------|---------|-------|---------|---------|-------|---------------|---------|
| Dashboard Root | `/dashboard` | ❌ | 👁️ | 👁️ | ✅ | table_config | Semester overview |
| Dashboard (term) | `/dashboard/[term]` | ❌ | 👁️ | 👁️ | ✅ | table_config, class_schedule | Term dashboard |
| Analytics | `/dashboard/[term]/analytics` | ❌ | ❌ | ❌ | ✅ | Aggregated stats | View charts |
| Conflicts | `/dashboard/[term]/conflicts` | ❌ | ❌ | ❌ | ✅ | class_schedule | View/resolve conflicts |
| Settings | `/dashboard/[term]/settings` | ❌ | ❌ | ❌ | ✅ | table_config | Semester settings |

---

### 2.4 Management Routes (Admin Only)

| Route | Pattern | Guest | Student | Teacher | Admin | Data Entities | CRUD Operations |
|-------|---------|-------|---------|---------|-------|---------------|-----------------|
| Teacher Management | `/management/teacher` | ❌ | ❌ | ❌ | ✅ | teacher | Create, Read, Update, Delete |
| Subject Management | `/management/subject` | ❌ | ❌ | ❌ | ✅ | subject | Create, Read, Update, Delete |
| Room Management | `/management/room` | ❌ | ❌ | ❌ | ✅ | room | Create, Read, Update, Delete |
| Class Management | `/management/class` | ❌ | ❌ | ❌ | ✅ | gradelevel | Create, Read, Update, Delete |
| Program Management | `/management/program` | ❌ | ❌ | ❌ | ✅ | program, program_subject | Create, Read, Update, Delete |

---

### 2.5 Schedule Routes (Admin Only)

| Route | Pattern | Guest | Student | Teacher | Admin | Data Entities | Actions |
|-------|---------|-------|---------|---------|-------|---------------|---------|
| Schedule Config | `/schedule/[term]` | ❌ | ❌ | ❌ | ✅ | table_config, timeslot | Configure semester |
| Assignment Root | `/schedule/[term]/assign` | ❌ | ❌ | ❌ | ✅ | teacher, teachers_responsibility | Teacher listing |
| Teacher Responsibility | `/schedule/[term]/assign/teacher_responsibility` | ❌ | ❌ | ❌ | ✅ | teachers_responsibility, subject, gradelevel | Assign subjects to teachers |
| Arrange Root | `/schedule/[term]/arrange` | ❌ | ❌ | ❌ | ✅ | class_schedule | Arrangement overview |
| Teacher Arrange | `/schedule/[term]/arrange/teacher-arrange` | ❌ | ❌ | ❌ | ✅ | class_schedule, timeslot, room | Drag-drop scheduling |
| Class Arrange | `/schedule/[term]/arrange/class-arrange` | ❌ | ❌ | ❌ | ✅ | class_schedule, timeslot | View by class |
| Room Arrange | `/schedule/[term]/arrange/room-arrange` | ❌ | ❌ | ❌ | ✅ | class_schedule, room | View by room |
| Lock Management | `/schedule/[term]/lock` | ❌ | ❌ | ❌ | ✅ | timeslot | Lock/unlock timeslots |

---

### 2.6 API Routes

| Endpoint | Method | Guest | Student | Teacher | Admin | Rate Limit | Data Entities |
|----------|--------|-------|---------|---------|-------|------------|---------------|
| `/api/admin/*` | ALL | ❌ | ❌ | ❌ | ✅ | 100/min | Various |
| `/api/export/excel` | GET | ❌ | ❌ | ❌ | ✅ | 10/min | class_schedule |
| `/api/export/pdf` | GET | ❌ | ❌ | ❌ | ✅ | 10/min | class_schedule |
| `/api/schedule/*` | GET | ❌ | ❌ | ⚠️ | ✅ | 60/min | class_schedule |
| `/api/schedule/*` | POST/PUT/DELETE | ❌ | ❌ | ❌ | ✅ | 30/min | class_schedule |
| `/api/teachers` | GET | ✅ | ✅ | ✅ | ✅ | 60/min | teacher |
| `/api/classes` | GET | ✅ | ✅ | ✅ | ✅ | 60/min | gradelevel |
| `/api/subjects` | GET | ❌ | ❌ | 👁️ | ✅ | 60/min | subject |
| `/api/rooms` | GET | ❌ | ❌ | 👁️ | ✅ | 60/min | room |

---

## 3. Server Actions Permission Matrix

### 3.1 Management Actions

| Action | File | Guest | Student | Teacher | Admin | Entities Modified |
|--------|------|-------|---------|---------|-------|-------------------|
| `createTeacherAction` | `teacher.actions.ts` | ❌ | ❌ | ❌ | ✅ | teacher |
| `updateTeacherAction` | `teacher.actions.ts` | ❌ | ❌ | ❌ | ✅ | teacher |
| `deleteTeacherAction` | `teacher.actions.ts` | ❌ | ❌ | ❌ | ✅ | teacher |
| `createSubjectAction` | `subject.actions.ts` | ❌ | ❌ | ❌ | ✅ | subject |
| `updateSubjectAction` | `subject.actions.ts` | ❌ | ❌ | ❌ | ✅ | subject |
| `deleteSubjectAction` | `subject.actions.ts` | ❌ | ❌ | ❌ | ✅ | subject |
| `createRoomAction` | `room.actions.ts` | ❌ | ❌ | ❌ | ✅ | room |
| `updateRoomAction` | `room.actions.ts` | ❌ | ❌ | ❌ | ✅ | room |
| `deleteRoomAction` | `room.actions.ts` | ❌ | ❌ | ❌ | ✅ | room |
| `createGradeAction` | `grade.actions.ts` | ❌ | ❌ | ❌ | ✅ | gradelevel |
| `updateGradeAction` | `grade.actions.ts` | ❌ | ❌ | ❌ | ✅ | gradelevel |
| `deleteGradeAction` | `grade.actions.ts` | ❌ | ❌ | ❌ | ✅ | gradelevel |

### 3.2 Assignment Actions

| Action | File | Guest | Student | Teacher | Admin | Entities Modified |
|--------|------|-------|---------|---------|-------|-------------------|
| `syncAssignmentsAction` | `assign.actions.ts` | ❌ | ❌ | ❌ | ✅ | teachers_responsibility |
| `deleteAssignmentAction` | `assign.actions.ts` | ❌ | ❌ | ❌ | ✅ | teachers_responsibility |
| `getTeacherAssignmentsAction` | `assign.actions.ts` | ❌ | ❌ | 🔒 | ✅ | teachers_responsibility (read) |

### 3.3 Schedule Actions

| Action | File | Guest | Student | Teacher | Admin | Entities Modified |
|--------|------|-------|---------|---------|-------|-------------------|
| `createScheduleAction` | `schedule.actions.ts` | ❌ | ❌ | ❌ | ✅ | class_schedule |
| `updateScheduleAction` | `schedule.actions.ts` | ❌ | ❌ | ❌ | ✅ | class_schedule |
| `deleteScheduleAction` | `schedule.actions.ts` | ❌ | ❌ | ❌ | ✅ | class_schedule |
| `bulkDeleteSchedulesAction` | `schedule.actions.ts` | ❌ | ❌ | ❌ | ✅ | class_schedule |
| `moveScheduleAction` | `schedule.actions.ts` | ❌ | ❌ | ❌ | ✅ | class_schedule |

### 3.4 Timeslot Actions

| Action | File | Guest | Student | Teacher | Admin | Entities Modified |
|--------|------|-------|---------|---------|-------|-------------------|
| `lockTimeslotAction` | `timeslot.actions.ts` | ❌ | ❌ | ❌ | ✅ | timeslot |
| `unlockTimeslotAction` | `timeslot.actions.ts` | ❌ | ❌ | ❌ | ✅ | timeslot |
| `bulkLockTimeslotsAction` | `timeslot.actions.ts` | ❌ | ❌ | ❌ | ✅ | timeslot |
| `applyLockTemplateAction` | `timeslot.actions.ts` | ❌ | ❌ | ❌ | ✅ | timeslot |

### 3.5 Config Actions

| Action | File | Guest | Student | Teacher | Admin | Entities Modified |
|--------|------|-------|---------|---------|-------|-------------------|
| `createSemesterAction` | `config.actions.ts` | ❌ | ❌ | ❌ | ✅ | table_config, timeslot |
| `updateSemesterAction` | `config.actions.ts` | ❌ | ❌ | ❌ | ✅ | table_config |
| `publishSemesterAction` | `config.actions.ts` | ❌ | ❌ | ❌ | ✅ | table_config |
| `archiveSemesterAction` | `config.actions.ts` | ❌ | ❌ | ❌ | ✅ | table_config |

---

## 4. Data Entity Access Matrix

### 4.1 Read Access

| Entity | Guest | Student | Teacher | Admin | Notes |
|--------|-------|---------|---------|-------|-------|
| `teacher` | ✅ (public info) | ✅ | ✅ | ✅ | Name, department visible to all |
| `subject` | ❌ | ❌ | 👁️ | ✅ | Visible in schedules only |
| `room` | ❌ | ❌ | 👁️ | ✅ | Visible in schedules only |
| `gradelevel` | ✅ | ✅ | ✅ | ✅ | Public class listing |
| `program` | ❌ | ❌ | ❌ | ✅ | Admin curriculum setup |
| `program_subject` | ❌ | ❌ | ❌ | ✅ | Admin curriculum setup |
| `timeslot` | 👁️ (in schedule) | 👁️ | 👁️ | ✅ | Times visible in timetables |
| `class_schedule` | 👁️ (published) | 👁️ | 🔒 | ✅ | Published schedules public |
| `teachers_responsibility` | ❌ | ❌ | 🔒 | ✅ | Own assignments only for teacher |
| `table_config` | ❌ | ❌ | ❌ | ✅ | Semester configuration |
| `User` | ❌ | 🔒 | 🔒 | ✅ | Own profile only |
| `Session` | ❌ | 🔒 | 🔒 | ✅ | Own sessions only |
| `Account` | ❌ | 🔒 | 🔒 | ✅ | Own linked accounts only |

### 4.2 Write Access

| Entity | Guest | Student | Teacher | Admin | Dangerous Operations |
|--------|-------|---------|---------|-------|---------------------|
| `teacher` | ❌ | ❌ | ❌ | ✅ | Delete cascade to assignments |
| `subject` | ❌ | ❌ | ❌ | ✅ | Delete cascade to schedules |
| `room` | ❌ | ❌ | ❌ | ✅ | Delete sets NULL on schedules |
| `gradelevel` | ❌ | ❌ | ❌ | ✅ | Delete cascade to schedules |
| `program` | ❌ | ❌ | ❌ | ✅ | Delete cascade to grades |
| `program_subject` | ❌ | ❌ | ❌ | ✅ | Affects curriculum |
| `timeslot` | ❌ | ❌ | ❌ | ✅ | Lock affects scheduling |
| `class_schedule` | ❌ | ❌ | ❌ | ✅ | Core scheduling data |
| `teachers_responsibility` | ❌ | ❌ | ❌ | ✅ | Affects scheduling options |
| `table_config` | ❌ | ❌ | ❌ | ✅ | Publish affects public view |
| `User` | ❌ | 🔒 | 🔒 | ✅ | Profile updates only |

---

## 5. Middleware & Guards

### 5.1 Route Middleware

```typescript
// src/middleware.ts
export const config = {
  matcher: [
    "/dashboard/:path*",
    "/management/:path*", 
    "/schedule/:path*",
    "/api/admin/:path*",
  ],
}
```

| Middleware | Purpose | Applied To |
|------------|---------|------------|
| `authMiddleware` | Verify session exists | All protected routes |
| `roleMiddleware` | Check user role | Admin-only routes |
| `rateLimitMiddleware` | Prevent abuse | All API routes |
| `csrfMiddleware` | Prevent CSRF attacks | Mutation endpoints |

### 5.2 Server Action Guards

```typescript
// Pattern used in server actions
async function protectedAction() {
  const session = await auth.api.getSession()
  if (!session?.user) {
    throw new Error("Unauthorized")
  }
  if (session.user.role !== "admin") {
    throw new Error("Forbidden: Admin access required")
  }
  // ... action logic
}
```

---

## 6. Security Considerations

### 6.1 Rate Limiting

| Context | Limit | Window | Action on Exceed |
|---------|-------|--------|------------------|
| Production API | 50 requests | 60 seconds | 429 Too Many Requests |
| Development API | 200 requests | 60 seconds | 429 Too Many Requests |
| Auth endpoints | 10 requests | 60 seconds | 429 + CAPTCHA |
| Export endpoints | 10 requests | 60 seconds | 429 Too Many Requests |

### 6.2 Audit Logging

| Event | Logged Data | Retention |
|-------|-------------|-----------|
| Login attempt | IP, email, success/failure, timestamp | 90 days |
| Data modification | User ID, entity, before/after, timestamp | 1 year |
| Export | User ID, export type, entity IDs, timestamp | 90 days |
| Publish | User ID, semester, timestamp | Permanent |

### 6.3 Dangerous Operations

| Operation | Risk Level | Safeguards |
|-----------|------------|------------|
| Delete Teacher | High | Check for assignments, soft delete |
| Delete Subject | High | Check for schedules, cascade warning |
| Bulk Delete Schedules | High | Confirmation dialog, audit log |
| Publish Semester | Medium | Completeness check, conflicts check |
| Lock Timeslots | Low | Preview before apply |

---

## 7. Feature Flags & Access

### 7.1 Feature Flag Matrix

| Feature Flag | Guest | Student | Teacher | Admin | Config Location |
|--------------|-------|---------|---------|-------|-----------------|
| `ENABLE_ANALYTICS` | ❌ | ❌ | ❌ | ⚠️ | Vercel Edge Config |
| `ENABLE_EXPORT_PDF` | ❌ | ❌ | ❌ | ⚠️ | Vercel Edge Config |
| `ENABLE_BULK_OPERATIONS` | ❌ | ❌ | ❌ | ⚠️ | Vercel Edge Config |
| `ENABLE_ADVANCED_CONFLICTS` | ❌ | ❌ | ❌ | ⚠️ | Vercel Edge Config |

### 7.2 Environment-Based Access

| Environment | Public Routes | Auth Routes | Admin Routes | Debug Tools |
|-------------|---------------|-------------|--------------|-------------|
| Production | ✅ | ✅ | ✅ | ❌ |
| Preview | ✅ | ✅ | ✅ | ⚠️ |
| Development | ✅ | ✅ | ✅ | ✅ |

---

## 8. Route → Component → Action Mapping

### 8.1 Management Module

```
/management/teacher
├── page.tsx (Server Component)
│   ├── TeacherList (Client Component)
│   │   ├── createTeacherAction → teacher table
│   │   ├── updateTeacherAction → teacher table
│   │   └── deleteTeacherAction → teacher table
│   └── TeacherForm (Client Component)
│       └── Valibot validation

/management/subject
├── page.tsx (Server Component)
│   ├── SubjectList (Client Component)
│   │   ├── createSubjectAction → subject table
│   │   ├── updateSubjectAction → subject table
│   │   └── deleteSubjectAction → subject table
│   └── SubjectForm (Client Component)
│       └── MOE code validation

/management/room
├── page.tsx (Server Component)
│   └── RoomList (Client Component)
│       ├── createRoomAction → room table
│       ├── updateRoomAction → room table
│       └── deleteRoomAction → room table

/management/class
├── page.tsx (Server Component)
│   └── ClassList (Client Component)
│       ├── createGradeAction → gradelevel table
│       ├── updateGradeAction → gradelevel table
│       └── deleteGradeAction → gradelevel table

/management/program
├── page.tsx (Server Component)
│   └── ProgramList (Client Component)
│       ├── createProgramAction → program table
│       ├── addSubjectToProgramAction → program_subject table
│       └── removeSubjectFromProgramAction → program_subject table
```

### 8.2 Schedule Module

```
/schedule/[term]/assign
├── page.tsx (Server Component)
│   └── ShowTeacherData (Client Component)
│       └── Navigate to teacher_responsibility

/schedule/[term]/assign/teacher_responsibility
├── page.tsx (Server Component)
│   └── AssignmentEditor (Client Component)
│       ├── syncAssignmentsAction → teachers_responsibility table
│       └── deleteAssignmentAction → teachers_responsibility table

/schedule/[term]/arrange/teacher-arrange
├── page.tsx (Server Component)
│   └── TeacherArrangeGrid (Client Component)
│       ├── createScheduleAction → class_schedule table
│       ├── updateScheduleAction → class_schedule table
│       ├── deleteScheduleAction → class_schedule table
│       └── DndContext (drag-drop handling)

/schedule/[term]/lock
├── page.tsx (Server Component)
│   └── LockGrid (Client Component)
│       ├── lockTimeslotAction → timeslot.IsLocked
│       ├── unlockTimeslotAction → timeslot.IsLocked
│       └── applyLockTemplateAction → timeslot.IsLocked (bulk)
```

### 8.3 Public Module

```
/teachers/[id]/[term]
├── page.tsx (Server Component)
│   ├── Fetches teacher schedules (read-only)
│   └── TimetableGrid (Server Component)
│       └── Renders weekly schedule

/classes/[gradeId]/[term]
├── page.tsx (Server Component)
│   ├── Fetches class schedules (read-only)
│   └── TimetableGrid (Server Component)
│       └── Renders weekly schedule
```

---

## 9. Error Handling by Permission

| Error Code | Meaning | User Message (Thai) | Retry Allowed |
|------------|---------|---------------------|---------------|
| 401 | Unauthenticated | กรุณาเข้าสู่ระบบ | Yes (redirect to signin) |
| 403 | Forbidden (wrong role) | ไม่มีสิทธิ์เข้าถึง | No |
| 404 | Resource not found | ไม่พบข้อมูล | No |
| 409 | Conflict (duplicate) | ข้อมูลซ้ำ | Yes (with changes) |
| 422 | Validation error | ข้อมูลไม่ถูกต้อง | Yes (fix input) |
| 429 | Rate limited | กรุณารอสักครู่ | Yes (after cooldown) |
| 500 | Server error | เกิดข้อผิดพลาด | Yes |
