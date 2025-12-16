# Use Cases - Phrasongsa Timetable System

**Version:** 1.0  
**Last Updated:** December 16, 2025  
**Format:** Cockburn "Fully Dressed" Style

---

## Use Case Index

| ID | Name | Actor | Level |
|----|------|-------|-------|
| UC-01 | Authenticate Administrator | Admin | User Goal |
| UC-02 | Create Semester Configuration | Admin | User Goal |
| UC-03 | Manage Teacher Records | Admin | User Goal |
| UC-04 | Manage Subject Records | Admin | User Goal |
| UC-05 | Manage Room Records | Admin | User Goal |
| UC-06 | Manage Grade/Class Records | Admin | User Goal |
| UC-07 | Assign Teaching Responsibilities | Admin | User Goal |
| UC-08 | Arrange Class Schedule (Drag-Drop) | Admin | User Goal |
| UC-09 | Lock Timeslots | Admin | User Goal |
| UC-10 | Detect Schedule Conflicts | System/Admin | User Goal |
| UC-11 | Resolve Schedule Conflicts | Admin | User Goal |
| UC-12 | Export Schedule to Excel | Admin | User Goal |
| UC-13 | Publish Semester Schedule | Admin | User Goal |
| UC-14 | View Teacher Schedule (Public) | Public User | User Goal |
| UC-15 | View Class Schedule (Public) | Public User | User Goal |
| UC-16 | Manage Program Curriculum | Admin | User Goal |
| UC-17 | Configure Subject for Program | Admin | Subfunction |
| UC-18 | Apply Lock Template | Admin | Subfunction |
| UC-19 | View Analytics Dashboard | Admin | User Goal |
| UC-20 | Bulk Delete Schedules | Admin | User Goal |

---

## UC-01: Authenticate Administrator

### Header
- **ID:** UC-01
- **Name:** Authenticate Administrator
- **Scope:** Phrasongsa Timetable System
- **Level:** User Goal
- **Primary Actor:** Administrator

### Stakeholders & Interests
- **Administrator:** Wants quick, secure access to system without repeated logins
- **Security Officer:** Needs audit trail of access attempts
- **System:** Must prevent unauthorized access to protected routes

### Preconditions
- User has Google account
- User's email is registered as admin in the system
- Google OAuth is properly configured

### Success Guarantee (Postconditions)
- User session is created and stored
- User is redirected to protected dashboard
- Login event is logged for audit

### Main Success Scenario
1. Admin navigates to any protected route (e.g., `/dashboard`)
2. System detects unauthenticated state
3. System redirects to `/signin` page
4. Admin clicks "Sign in with Google" button
5. System redirects to Google OAuth consent screen
6. Admin grants permission
7. Google returns OAuth tokens to callback URL
8. System validates tokens and checks user role
9. System creates session with 7-day expiry
10. System redirects to originally requested URL or dashboard
11. Admin sees personalized dashboard with their name

### Extensions (Alternate Flows)

**3a.** Admin is already authenticated:
- 3a1. System skips to step 10

**6a.** Admin denies Google permission:
- 6a1. Google redirects back with error
- 6a2. System shows "Permission denied" message
- 6a3. System offers retry option

**8a.** User email not found in admin list:
- 8a1. System shows "Access Denied: Not authorized"
- 8a2. System logs unauthorized access attempt
- 8a3. Admin must contact super-admin to request access

**8b.** Google tokens expired or invalid:
- 8b1. System shows "Authentication failed"
- 8b2. System offers retry option

### Special Requirements
- OAuth flow must complete within 60 seconds
- Rate limit: 50 auth attempts per hour per IP
- Session must be HttpOnly, Secure, SameSite=Lax

### Technology & Data Variations
- Google OAuth 2.0 via Better Auth library
- Session stored in PostgreSQL via Prisma
- Redis cache for session validation (optional)

### Frequency
- ~50 times/day during active scheduling periods
- ~10 times/day during maintenance periods

---

## UC-02: Create Semester Configuration

### Header
- **ID:** UC-02
- **Name:** Create Semester Configuration
- **Scope:** Phrasongsa Timetable System
- **Level:** User Goal
- **Primary Actor:** Administrator

### Stakeholders & Interests
- **Administrator:** Wants to quickly set up new semester with correct timeslots
- **Teachers:** Need predictable period times
- **MOE:** Requires minimum instructional hours per week

### Preconditions
- Admin is authenticated
- Semester does not already exist for same year+term

### Success Guarantee (Postconditions)
- `table_config` record created with status DRAFT
- Exactly 40 `timeslot` records generated (8 periods × 5 days)
- System context updated to new semester

### Main Success Scenario
1. Admin clicks "สร้างภาคเรียนใหม่" from dashboard
2. System opens configuration modal
3. Admin enters academic year (e.g., 2568)
4. Admin selects semester (1 or 2)
5. Admin configures school days (defaults to Mon-Fri checked)
6. Admin sets first period start time (default: 08:30)
7. Admin sets period duration in minutes (default: 50)
8. Admin sets break duration in minutes (default: 10)
9. Admin sets number of periods per day (default: 8)
10. Admin sets junior lunch period number (default: 4)
11. Admin sets senior lunch period number (default: 5)
12. System shows timeslot preview grid
13. Admin reviews and clicks "ตั้งค่าตารางเรียน"
14. System validates configuration
15. System creates `table_config` record
16. System generates `timeslot` records with calculated times
17. System redirects to new semester dashboard
18. Admin sees empty schedule grid ready for arrangement

### Extensions (Alternate Flows)

**4a.** Semester already exists:
- 4a1. System shows "ภาคเรียนนี้มีอยู่แล้ว"
- 4a2. Admin chooses different year/semester or edits existing

**12a.** Timeslot times overlap incorrectly:
- 12a1. System highlights conflicts in preview
- 12a2. Admin adjusts duration or break times
- 12a3. Return to step 12

**14a.** Validation fails (invalid times):
- 14a1. System shows specific error messages
- 14a2. Admin corrects input
- 14a3. Return to step 12

### Special Requirements
- Period times must not overlap
- Total periods × days must be reasonable (≤ 100)
- Times stored as ISO 8601 datetime strings

### Frequency
- 2 times/year (one per semester)

---

## UC-03: Manage Teacher Records

### Header
- **ID:** UC-03
- **Name:** Manage Teacher Records
- **Scope:** Phrasongsa Timetable System
- **Level:** User Goal
- **Primary Actor:** Administrator

### Stakeholders & Interests
- **Administrator:** Wants accurate teacher directory
- **Teachers:** Need correct name display and contact info
- **Schedule System:** Needs unique TeacherID for assignments

### Preconditions
- Admin is authenticated
- Semester is selected

### Success Guarantee (Postconditions)
- Teacher record created/updated/deleted successfully
- All referencing records updated or blocked

### Main Success Scenario (Create)
1. Admin navigates to `/management/teacher`
2. System displays teacher list (paginated)
3. Admin clicks "เพิ่มข้อมูลครู"
4. System opens create form
5. Admin enters Prefix (นาย/นาง/นางสาว)
6. Admin enters First Name
7. Admin enters Last Name
8. Admin selects Department/Learning Area
9. Admin enters Email (optional)
10. Admin clicks Submit
11. System validates uniqueness of email
12. System generates TeacherID
13. System creates `teacher` record
14. System refreshes list with new teacher
15. Admin sees success toast

### Extensions (Alternate Flows)

**3a.** Admin wants to edit existing teacher:
- 3a1. Admin clicks edit button on teacher row
- 3a2. System opens edit form with current values
- 3a3. Admin modifies fields
- 3a4. Admin clicks Save
- 3a5. System validates and updates record
- 3a6. Return to main flow step 14

**3b.** Admin wants to delete teacher:
- 3b1. Admin clicks delete button on teacher row
- 3b2. System checks for existing assignments
- 3b3a. If no assignments: show confirm dialog
- 3b3b. If has assignments: show warning "มีการมอบหมายงานสอน"
- 3b4. Admin confirms deletion
- 3b5. System soft-deletes record
- 3b6. Return to main flow step 14

**11a.** Email already exists:
- 11a1. System shows "อีเมลนี้ถูกใช้งานแล้ว"
- 11a2. Admin enters different email
- 11a3. Return to step 10

### Special Requirements
- Names must support Thai characters
- Email uniqueness is case-insensitive

### Frequency
- ~100 teachers managed per year
- ~10 changes per semester

---

## UC-04: Manage Subject Records

### Header
- **ID:** UC-04
- **Name:** Manage Subject Records
- **Scope:** Phrasongsa Timetable System
- **Level:** User Goal
- **Primary Actor:** Administrator

### Stakeholders & Interests
- **Administrator:** Wants accurate subject catalog
- **MOE:** Requires specific subject codes per curriculum
- **Program Manager:** Needs subjects linked to programs

### Preconditions
- Admin is authenticated

### Success Guarantee (Postconditions)
- Subject record created/updated/deleted
- Subject code follows MOE pattern (e.g., ท21101)

### Main Success Scenario (Create)
1. Admin navigates to `/management/subject`
2. System displays subject list grouped by learning area
3. Admin clicks "เพิ่มรายวิชา"
4. System opens create form
5. Admin enters Subject Code (e.g., ท21101)
6. System validates MOE code pattern
7. Admin enters Subject Name (e.g., ภาษาไทย พื้นฐาน)
8. Admin selects Learning Area (8 standard areas)
9. Admin enters Credit value
10. Admin enters Hours per week
11. Admin clicks Submit
12. System creates `subject` record
13. System refreshes list
14. Admin sees success toast

### Extensions (Alternate Flows)

**6a.** Invalid subject code format:
- 6a1. System shows pattern hint (e.g., "รูปแบบ: X00000")
- 6a2. Admin corrects code
- 6a3. Return to step 6

**6b.** Subject code already exists:
- 6b1. System shows "รหัสวิชานี้มีอยู่แล้ว"
- 6b2. Admin enters different code
- 6b3. Return to step 6

### Special Requirements
- Subject codes follow Thai MOE patterns
- 8 Learning Areas: ท, ค, ว, ส, พ, ศ, ง, อ

### Frequency
- ~50 subjects created per year

---

## UC-05: Manage Room Records

### Header
- **ID:** UC-05
- **Name:** Manage Room Records
- **Scope:** Phrasongsa Timetable System
- **Level:** User Goal
- **Primary Actor:** Administrator

### Stakeholders & Interests
- **Administrator:** Wants accurate room inventory
- **Schedule System:** Needs rooms for assignment
- **Teachers:** Need to know where to teach

### Preconditions
- Admin is authenticated

### Success Guarantee (Postconditions)
- Room record created/updated/deleted

### Main Success Scenario (Create)
1. Admin navigates to `/management/room`
2. System displays room list
3. Admin clicks "เพิ่มห้องเรียน"
4. System opens create form
5. Admin enters Room Name/Number (e.g., 101)
6. Admin enters Room Type (e.g., ห้องเรียนทั่วไป)
7. Admin enters Capacity (optional)
8. Admin enters Building (optional)
9. Admin clicks Submit
10. System creates `room` record
11. System refreshes list
12. Admin sees success toast

### Extensions (Alternate Flows)

**5a.** Room name already exists:
- 5a1. System shows "ห้องนี้มีอยู่แล้ว"
- 5a2. Admin enters different name

### Frequency
- ~50 rooms created initially
- ~5 changes per year

---

## UC-06: Manage Grade/Class Records

### Header
- **ID:** UC-06
- **Name:** Manage Grade/Class Records
- **Scope:** Phrasongsa Timetable System
- **Level:** User Goal
- **Primary Actor:** Administrator

### Stakeholders & Interests
- **Administrator:** Wants accurate class structure
- **Students:** Need to find their class schedule
- **MOE:** Requires standard grade levels (ม.1-ม.6)

### Preconditions
- Admin is authenticated
- Program exists for the year level

### Success Guarantee (Postconditions)
- Grade record created with DisplayID (e.g., ม.1/1)
- Grade linked to correct Program

### Main Success Scenario (Create)
1. Admin navigates to `/management/class`
2. System displays class list by year level
3. Admin clicks "เพิ่มห้องเรียน"
4. System opens create form
5. Admin selects Year Level (1-6, representing ม.1-ม.6)
6. Admin enters Section Number (e.g., 1, 2, 3)
7. Admin selects Program (e.g., วิทย์-คณิต)
8. System auto-generates DisplayID (e.g., ม.1/1)
9. Admin clicks Submit
10. System creates `gradelevel` record
11. System refreshes list
12. Admin sees success toast

### Extensions (Alternate Flows)

**8a.** DisplayID already exists:
- 8a1. System shows "ห้องเรียนนี้มีอยู่แล้ว"
- 8a2. Admin adjusts year or section

### Special Requirements
- DisplayID format: ม.{year}/{section}
- Year must be 1-6 (Thai Mathayom levels)

### Frequency
- ~24 classes created per semester (4 per year level × 6 levels)

---

## UC-07: Assign Teaching Responsibilities

### Header
- **ID:** UC-07
- **Name:** Assign Teaching Responsibilities
- **Scope:** Phrasongsa Timetable System
- **Level:** User Goal
- **Primary Actor:** Administrator

### Stakeholders & Interests
- **Administrator:** Wants efficient assignment workflow
- **Teachers:** Need balanced workload
- **Students:** Need qualified teachers for all subjects

### Preconditions
- Admin is authenticated
- Semester is selected
- Teachers, subjects, and grades exist

### Success Guarantee (Postconditions)
- `teachers_responsibility` records created
- Teacher workload updated

### Main Success Scenario
1. Admin navigates to `/schedule/[term]/assign`
2. System displays teacher cards with current workload
3. Admin clicks on teacher card
4. System navigates to `/schedule/[term]/assign/teacher_responsibility?TeacherID=X`
5. System shows teacher's current assignments
6. Admin clicks "เพิ่มห้องเรียน"
7. Admin selects grade from dropdown
8. System adds grade row to assignment list
9. Admin clicks "เพิ่มวิชา" on grade row
10. System shows subjects available in grade's program
11. Admin selects subject
12. System calculates TeachHour from subject credits
13. System adds subject to grade row
14. Admin repeats steps 9-13 for additional subjects
15. Admin repeats steps 6-14 for additional grades
16. Admin clicks "บันทึก"
17. System calls `syncAssignmentsAction`
18. System creates/updates `teachers_responsibility` records
19. System redirects back to teacher list
20. Admin sees updated workload on teacher card

### Extensions (Alternate Flows)

**2a.** No teachers exist:
- 2a1. System shows empty state
- 2a2. System shows "เพิ่มข้อมูลครูก่อน"
- 2a3. Admin clicks link to teacher management
- 2a4. Return to UC-03

**10a.** No subjects in program:
- 10a1. System shows "ไม่มีวิชาในหลักสูตร"
- 10a2. Admin must configure program first
- 10a3. Return to UC-16

**17a.** Teacher exceeds recommended hours:
- 17a1. System shows warning (non-blocking)
- 17a2. Admin acknowledges warning
- 17a3. Continue to step 18

### Special Requirements
- Max recommended hours: 20/week (warning only)
- Sync action handles create/delete diff

### Frequency
- ~100 teachers assigned per semester

---

## UC-08: Arrange Class Schedule (Drag-Drop)

### Header
- **ID:** UC-08
- **Name:** Arrange Class Schedule (Drag-Drop)
- **Scope:** Phrasongsa Timetable System
- **Level:** User Goal
- **Primary Actor:** Administrator

### Stakeholders & Interests
- **Administrator:** Wants intuitive schedule building
- **Teachers:** Need conflict-free schedules
- **Students:** Need complete timetables
- **Rooms:** Must not be double-booked

### Preconditions
- Admin is authenticated
- Semester is selected
- Teacher has assigned responsibilities
- Timeslots exist

### Success Guarantee (Postconditions)
- `class_schedule` record created
- No conflicts exist for created schedule
- Subject placement counter decremented

### Main Success Scenario
1. Admin navigates to `/schedule/[term]/arrange/teacher-arrange`
2. System displays teacher selector
3. Admin selects teacher from dropdown
4. System fetches teacher's responsibilities
5. System displays subject palette (left panel) with required slots
6. System displays weekly grid (center) with timeslots
7. Admin drags subject from palette
8. Admin drops on target timeslot cell
9. System shows room selection modal
10. Admin selects available room
11. System validates:
    - Teacher not double-booked
    - Class not double-booked
    - Room not double-booked
    - Timeslot not locked
12. System creates `class_schedule` record
13. System updates grid cell with subject info
14. System decrements palette counter
15. Admin repeats steps 7-14 until all subjects placed
16. System shows completion badge when all slots filled

### Extensions (Alternate Flows)

**4a.** Teacher has no responsibilities:
- 4a1. System shows "ครูยังไม่มีการมอบหมายงาน"
- 4a2. Admin clicks link to assignment page
- 4a3. Return to UC-07

**8a.** Drop on locked timeslot:
- 8a1. System shows lock icon and rejects drop
- 8a2. Admin drags to different slot

**10a.** No rooms available:
- 10a1. Modal shows "ไม่มีห้องว่าง"
- 10a2. Admin cancels or clears conflicting schedule

**11a.** Teacher conflict detected:
- 11a1. System shows red border on cell
- 11a2. System shows "ครูมีสอนแล้วในคาบนี้"
- 11a3. Admin drags to different slot

**11b.** Class conflict detected:
- 11b1. System shows red border on cell
- 11b2. System shows "ห้องเรียนมีวิชาแล้วในคาบนี้"
- 11b3. Admin drags to different slot

**11c.** Room conflict detected:
- 11c1. System shows "ห้องนี้ไม่ว่าง"
- 11c2. Admin selects different room

### Special Requirements
- Drag-drop via @dnd-kit library
- Real-time conflict detection
- Optimistic UI updates

### Frequency
- ~1000 schedule placements per semester

---

## UC-09: Lock Timeslots

### Header
- **ID:** UC-09
- **Name:** Lock Timeslots
- **Scope:** Phrasongsa Timetable System
- **Level:** User Goal
- **Primary Actor:** Administrator

### Stakeholders & Interests
- **Administrator:** Wants to block slots for non-teaching activities
- **Teachers:** Need to know unavailable times
- **School:** Requires time for assemblies, activities

### Preconditions
- Admin is authenticated
- Semester is selected
- Timeslots exist

### Success Guarantee (Postconditions)
- Selected timeslots marked as locked
- Locked slots prevent schedule placement

### Main Success Scenario
1. Admin navigates to `/schedule/[term]/lock`
2. System displays timeslot grid with current lock status
3. Admin clicks individual cells to toggle lock status
4. System updates `timeslot.IsLocked` in real-time
5. System shows lock icon on locked cells
6. Admin repeats for all desired slots
7. Admin clicks "บันทึก" (if batch mode)
8. System persists all changes

### Extensions (Alternate Flows)

**3a.** Admin wants to use template:
- 3a1. Admin clicks "ใช้เทมเพลต"
- 3a2. System shows template list
- 3a3. Admin selects template (e.g., "พักกลางวัน ม.ต้น")
- 3a4. System calculates matching timeslots
- 3a5. System shows preview of affected slots
- 3a6. Admin confirms
- 3a7. System applies locks to matching slots
- 3a8. Return to step 5

**3a4a.** No slots match template criteria:
- 3a4a1. System shows "ไม่พบคาบเรียนที่ตรงกับเกณฑ์"
- 3a4a2. Admin adjusts template parameters or chooses different template

**3b.** Admin wants batch selection:
- 3b1. Admin enters selection mode
- 3b2. Admin clicks multiple cells
- 3b3. Admin clicks "ล็อก" or "ปลดล็อก"
- 3b4. System applies to all selected
- 3b5. Return to step 5

### Special Requirements
- Templates defined in code with time-based matching
- Lock status stored on `timeslot` record

### Frequency
- ~5 template applications per semester
- ~20 manual lock toggles per semester

---

## UC-10: Detect Schedule Conflicts

### Header
- **ID:** UC-10
- **Name:** Detect Schedule Conflicts
- **Scope:** Phrasongsa Timetable System
- **Level:** User Goal
- **Primary Actor:** System (automated) / Administrator (manual trigger)

### Stakeholders & Interests
- **Administrator:** Wants proactive conflict notification
- **Teachers:** Must not be double-booked
- **Students:** Must have consistent schedules
- **Quality Assurance:** Needs validation before publish

### Preconditions
- Semester is selected
- Schedules exist

### Success Guarantee (Postconditions)
- All conflicts identified and categorized
- Conflict report available to admin

### Main Success Scenario
1. System runs conflict detection (triggered by schedule change or manual)
2. System queries for teacher double-bookings:
   ```sql
   SELECT * FROM class_schedule
   WHERE TimeslotID = X AND TeacherID = Y
   GROUP BY TimeslotID, TeacherID
   HAVING COUNT(*) > 1
   ```
3. System queries for class double-bookings
4. System queries for room double-bookings
5. System queries for unassigned slots (no teacher or room)
6. System aggregates results by conflict type
7. System stores conflict report
8. Admin navigates to `/dashboard/[term]/conflicts`
9. System displays conflict cards
10. Each card shows: type, affected entity, timeslot, conflicting items

### Extensions (Alternate Flows)

**7a.** No conflicts found:
- 7a1. System shows "ไม่พบข้อขัดแย้ง" badge
- 7a2. Publish gate is green

**9a.** Many conflicts exist (> 50):
- 9a1. System paginates results
- 9a2. System shows summary counts by type

### Special Requirements
- Detection runs on every schedule save
- Batch detection available for full validation

### Frequency
- Automatic: on every schedule save
- Manual: ~10 times per semester

---

## UC-11: Resolve Schedule Conflicts

### Header
- **ID:** UC-11
- **Name:** Resolve Schedule Conflicts
- **Scope:** Phrasongsa Timetable System
- **Level:** User Goal
- **Primary Actor:** Administrator

### Stakeholders & Interests
- **Administrator:** Wants clear resolution workflow
- **Teachers:** Need conflict-free schedules
- **System:** Must reach zero conflicts for publish

### Preconditions
- Conflicts exist in the system
- Admin is authenticated

### Success Guarantee (Postconditions)
- Conflict resolved (deleted or rescheduled)
- Conflict count decremented

### Main Success Scenario
1. Admin views conflict on conflicts page
2. Admin clicks conflict card
3. System navigates to affected schedule view
4. Admin sees conflicting entries highlighted
5. Admin chooses resolution:
   - Delete one schedule entry
   - Move one entry to different timeslot
6. System validates new state
7. System updates records
8. System re-runs conflict detection
9. Admin returns to conflicts page
10. Conflict no longer appears

### Extensions (Alternate Flows)

**5a.** Admin deletes one conflicting schedule:
- 5a1. Admin clicks delete on one entry
- 5a2. System confirms deletion
- 5a3. Jump to step 7

**5b.** Admin moves one schedule:
- 5b1. Admin drags entry to new timeslot
- 5b2. System validates new placement
- 5b3. Jump to step 7

**6a.** Move creates new conflict:
- 6a1. System shows new conflict error
- 6a2. Admin chooses different slot
- 6a3. Return to step 5b

### Frequency
- ~20 conflicts resolved per semester

---

## UC-12: Export Schedule to Excel

### Header
- **ID:** UC-12
- **Name:** Export Schedule to Excel
- **Scope:** Phrasongsa Timetable System
- **Level:** User Goal
- **Primary Actor:** Administrator

### Stakeholders & Interests
- **Administrator:** Wants distributable schedule documents
- **Teachers:** Need personal schedule printouts
- **School Office:** Needs master schedule file

### Preconditions
- Schedules exist
- Admin is authenticated

### Success Guarantee (Postconditions)
- Excel file downloaded to user device
- File contains accurate schedule data

### Main Success Scenario
1. Admin navigates to teacher or class schedule view
2. Admin clicks "ส่งออก Excel" button
3. System determines export scope (single entity)
4. System fetches schedule data
5. System generates Excel workbook via ExcelJS:
   - Header with semester info
   - Weekly grid layout
   - Subject, room, teacher columns
   - Color coding by subject type
6. System triggers browser download
7. File downloads to user device
8. Admin opens file in Excel/Google Sheets

### Extensions (Alternate Flows)

**3a.** Admin wants bulk export:
- 3a1. Admin selects multiple teachers/classes
- 3a2. System generates multi-sheet workbook
- 3a3. Each sheet = one entity
- 3a4. Jump to step 6

**4a.** No schedules exist:
- 4a1. System disables export button
- 4a2. System shows "ไม่มีตารางสอน"

### Special Requirements
- File format: .xlsx
- Max file size: 10MB
- Compatible with Excel 2016+

### Frequency
- ~50 exports per semester (end of scheduling period)

---

## UC-13: Publish Semester Schedule

### Header
- **ID:** UC-13
- **Name:** Publish Semester Schedule
- **Scope:** Phrasongsa Timetable System
- **Level:** User Goal
- **Primary Actor:** Administrator

### Stakeholders & Interests
- **Administrator:** Wants to make schedule publicly available
- **Teachers:** Need to see finalized schedules
- **Students:** Need to plan their time
- **Parents:** Want to know school schedule

### Preconditions
- Admin is authenticated
- Semester completeness ≥ 30%
- Zero critical conflicts (or override)

### Success Guarantee (Postconditions)
- Semester status = PUBLISHED
- Public routes return this semester's data
- Draft caches cleared

### Main Success Scenario
1. Admin navigates to semester dashboard
2. System displays completeness percentage
3. System displays conflict count
4. Admin verifies completeness ≥ 30%
5. Admin verifies zero conflicts
6. Admin clicks "เผยแพร่"
7. System shows confirmation dialog
8. Admin confirms action
9. System updates `table_config.Status` to PUBLISHED
10. System clears draft caches
11. System shows success toast
12. Public routes now serve this semester's schedules

### Extensions (Alternate Flows)

**4a.** Completeness < 30%:
- 4a1. Publish button is disabled
- 4a2. System shows "ต้องมีข้อมูลอย่างน้อย 30%"
- 4a3. Admin must create more schedules

**5a.** Conflicts exist:
- 5a1. System shows conflict count badge
- 5a2. Publish button shows warning state
- 5a3. Admin clicks publish anyway
- 5a4. System shows "มีข้อขัดแย้ง X รายการ ต้องการดำเนินการต่อหรือไม่"
- 5a5. Admin can confirm or cancel

**8a.** Admin cancels:
- 8a1. Dialog closes
- 8a2. No changes made

### Status Progression
- DRAFT → PUBLISHED → LOCKED → ARCHIVED

### Special Requirements
- Only one semester can be PUBLISHED at a time
- Previous semester auto-archives on new publish

### Frequency
- 2 times/year

---

## UC-14: View Teacher Schedule (Public)

### Header
- **ID:** UC-14
- **Name:** View Teacher Schedule (Public)
- **Scope:** Phrasongsa Timetable System
- **Level:** User Goal
- **Primary Actor:** Public User

### Stakeholders & Interests
- **Public User:** Wants to find teacher's teaching times
- **Teacher:** Wants schedule accessible to students/parents
- **School:** Wants transparent scheduling

### Preconditions
- Published semester exists
- Teacher has schedules

### Success Guarantee (Postconditions)
- User sees teacher's weekly timetable

### Main Success Scenario
1. User navigates to homepage (/)
2. System displays teacher search box
3. User types teacher name
4. System shows autocomplete suggestions
5. User selects teacher
6. System navigates to `/teachers/[id]/[term]`
7. System fetches teacher's schedules
8. System displays weekly grid:
   - Day columns (Mon-Fri)
   - Period rows (1-8)
   - Each cell shows: subject, class, room
9. User views schedule

### Extensions (Alternate Flows)

**4a.** No matching teachers:
- 4a1. Autocomplete shows "ไม่พบครู"

**7a.** Teacher has no schedules:
- 7a1. Grid shows empty state
- 7a2. System shows "ไม่มีตารางสอน"

### Special Requirements
- No authentication required
- Page is SEO-friendly

### Frequency
- ~500 views/day during semester

---

## UC-15: View Class Schedule (Public)

### Header
- **ID:** UC-15
- **Name:** View Class Schedule (Public)
- **Scope:** Phrasongsa Timetable System
- **Level:** User Goal
- **Primary Actor:** Public User

### Stakeholders & Interests
- **Public User:** Wants to see class timetable
- **Students:** Need to plan daily schedule
- **Parents:** Want to know child's schedule

### Preconditions
- Published semester exists
- Class has schedules

### Success Guarantee (Postconditions)
- User sees class's weekly timetable

### Main Success Scenario
1. User navigates to homepage (/)
2. User clicks "ชั้นเรียน" tab
3. System displays grade level tabs (ม.1 - ม.6)
4. User clicks year level tab
5. System displays class cards for that level
6. User clicks class card (e.g., ม.1/1)
7. System navigates to `/classes/[gradeId]/[term]`
8. System fetches class schedules
9. System displays weekly grid:
   - Day columns (Mon-Fri)
   - Period rows (1-8)
   - Each cell shows: subject, teacher, room
10. User views schedule

### Extensions (Alternate Flows)

**5a.** No classes for year level:
- 5a1. System shows "ไม่มีข้อมูลห้องเรียน"

**8a.** Class has no schedules:
- 8a1. Grid shows empty state

### Frequency
- ~1000 views/day during semester

---

## UC-16: Manage Program Curriculum

### Header
- **ID:** UC-16
- **Name:** Manage Program Curriculum
- **Scope:** Phrasongsa Timetable System
- **Level:** User Goal
- **Primary Actor:** Administrator

### Stakeholders & Interests
- **Administrator:** Wants to define curriculum structure
- **MOE:** Requires compliant program definitions
- **Teachers:** Need to know what subjects to teach

### Preconditions
- Admin is authenticated
- Subjects exist in master list

### Success Guarantee (Postconditions)
- Program created with subject mappings
- MOE credit requirements validated

### Main Success Scenario
1. Admin navigates to `/management/program`
2. System displays year level tabs (ม.1 - ม.6)
3. Admin selects year level
4. System displays programs for that level
5. Admin clicks "เพิ่ม" to create new program
6. Admin enters Program Code (e.g., M1-SCI)
7. Admin enters Program Name (e.g., หลักสูตรวิทย์-คณิต)
8. Admin selects Track (SCIENCE_MATH, LANGUAGE_MATH, etc.)
9. Admin enters Minimum Total Credits
10. Admin clicks "บันทึก"
11. System creates `program` record
12. Admin clicks program to configure subjects
13. Admin adds subjects via UC-17
14. System validates total credits against MOE standards

### Extensions (Alternate Flows)

**6a.** Program code exists:
- 6a1. System shows "รหัสหลักสูตรนี้มีอยู่แล้ว"

**14a.** Credits below MOE minimum:
- 14a1. System shows warning "หน่วยกิตต่ำกว่าเกณฑ์ MOE"
- 14a2. Admin adds more subjects

### Frequency
- ~6 programs created initially
- ~2 changes per year

---

## UC-17: Configure Subject for Program

### Header
- **ID:** UC-17
- **Name:** Configure Subject for Program
- **Scope:** Phrasongsa Timetable System
- **Level:** Subfunction
- **Primary Actor:** Administrator

### Stakeholders & Interests
- **Administrator:** Wants accurate curriculum setup
- **MOE:** Requires correct subject categorization

### Preconditions
- Program exists
- Subjects exist in master list

### Success Guarantee (Postconditions)
- Subject linked to program with category and credits

### Main Success Scenario
1. Admin views program detail page
2. Admin clicks "เพิ่มรายวิชา"
3. System shows subject selection modal
4. Admin searches/selects subject
5. Admin selects Category (CORE/ADDITIONAL/ACTIVITY)
6. Admin marks Mandatory (yes/no)
7. Admin sets Min Credits
8. Admin sets Max Credits
9. Admin clicks "บันทึก"
10. System creates `program_subject` mapping
11. System recalculates program totals

### Frequency
- ~50 subject mappings per program

---

## UC-18: Apply Lock Template

### Header
- **ID:** UC-18
- **Name:** Apply Lock Template
- **Scope:** Phrasongsa Timetable System
- **Level:** Subfunction
- **Primary Actor:** Administrator

### Stakeholders & Interests
- **Administrator:** Wants quick bulk locking
- **School:** Has standard activity times

### Preconditions
- Timeslots exist
- Templates defined in system

### Success Guarantee (Postconditions)
- Matching timeslots locked per template criteria

### Main Success Scenario
1. Admin clicks "ใช้เทมเพลต" on lock page
2. System shows available templates
3. Admin selects template (e.g., "พักกลางวัน ม.ต้น")
4. System reads template criteria:
   - Target time range (e.g., 10:40 - 11:30)
   - Target days (e.g., Mon-Fri)
5. System queries matching timeslots
6. System shows preview with count
7. Admin confirms
8. System sets `IsLocked = true` on matching slots
9. Grid updates with lock icons

### Templates Available
| Name | Criteria | Typical Slots |
|------|----------|---------------|
| พักกลางวัน (ม.ต้น) | 10:40-11:30, Mon-Fri | 5 |
| พักกลางวัน (ม.ปลาย) | 10:55-11:45, Mon-Fri | 5 |
| กิจกรรมเข้าแถว | 08:00-08:30, Mon-Fri | 5 |
| กิจกรรมชุมนุม | 14:00-16:00, Wed | 2 |
| ประชุมระดับชั้น | 14:00-16:00, Fri | 2 |

### Frequency
- ~5 template applications per semester

---

## UC-19: View Analytics Dashboard

### Header
- **ID:** UC-19
- **Name:** View Analytics Dashboard
- **Scope:** Phrasongsa Timetable System
- **Level:** User Goal
- **Primary Actor:** Administrator

### Stakeholders & Interests
- **Administrator:** Wants scheduling insights
- **Principal:** Needs resource utilization data
- **Planners:** Need workload distribution visibility

### Preconditions
- Admin is authenticated
- Semester has schedule data

### Success Guarantee (Postconditions)
- Analytics visualizations displayed

### Main Success Scenario
1. Admin navigates to `/dashboard/[term]/analytics`
2. System fetches aggregated data
3. System renders charts:
   - Teacher workload bar chart
   - Room utilization heatmap
   - Subject coverage donut chart
   - Conflict trend line chart
4. Admin interacts with filters
5. Charts update based on filters
6. Admin drills down by clicking chart elements

### Visualizations
- **Workload**: Hours/week per teacher (Recharts BarChart)
- **Utilization**: Slots used per room per day (Recharts Heatmap)
- **Coverage**: % subjects scheduled by category (Recharts PieChart)
- **Conflicts**: Trend over time (Recharts LineChart)

### Frequency
- ~20 views per semester

---

## UC-20: Bulk Delete Schedules

### Header
- **ID:** UC-20
- **Name:** Bulk Delete Schedules
- **Scope:** Phrasongsa Timetable System
- **Level:** User Goal
- **Primary Actor:** Administrator

### Stakeholders & Interests
- **Administrator:** Wants to clear schedules for rebuild
- **System:** Needs clean state for fresh scheduling

### Preconditions
- Admin is authenticated
- Schedules exist

### Success Guarantee (Postconditions)
- Selected schedules deleted
- Related conflicts removed

### Main Success Scenario
1. Admin navigates to schedule management
2. Admin enters selection mode
3. Admin selects multiple schedule entries
4. Admin clicks "ลบ" (bulk delete)
5. System shows confirmation with count
6. Admin confirms
7. System deletes selected `class_schedule` records
8. System re-runs conflict detection
9. Grid refreshes
10. Admin sees success toast

### Extensions (Alternate Flows)

**6a.** Admin cancels:
- 6a1. No changes made

**7a.** Delete fails (DB error):
- 7a1. System shows error toast
- 7a2. Transaction rolls back

### Special Requirements
- Transactional delete for consistency
- Audit log of deleted records

### Frequency
- ~5 bulk deletes per semester (for major schedule rebuilds)
