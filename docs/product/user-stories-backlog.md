# User Stories Backlog - Phrasongsa Timetable System

**Version:** 1.0  
**Last Updated:** December 16, 2025  
**Format:** As a {role}, I want {capability}, so that {value}

---

## Backlog Overview

| Epic | Stories | Priority | Status |
|------|---------|----------|--------|
| EP-01: Authentication & Authorization | 5 | P0 | ✅ Implemented |
| EP-02: Semester Configuration | 4 | P0 | ✅ Implemented |
| EP-03: Master Data Management | 8 | P0 | ✅ Implemented |
| EP-04: Program & Curriculum | 5 | P1 | ✅ Implemented |
| EP-05: Teacher Assignment | 4 | P0 | ✅ Implemented |
| EP-06: Schedule Arrangement | 7 | P0 | ✅ Implemented |
| EP-07: Timeslot Locking | 4 | P1 | ✅ Implemented |
| EP-08: Conflict Management | 4 | P1 | ✅ Implemented |
| EP-09: Export & Reporting | 3 | P1 | ⏳ Partial |
| EP-10: Public Schedule Viewing | 4 | P0 | ✅ Implemented |
| EP-11: Analytics Dashboard | 3 | P2 | ⏳ Partial |
| EP-12: Mobile Responsiveness | 3 | P2 | ⏳ Partial |

---

## EP-01: Authentication & Authorization

### Epic Description
Enable secure access to the system with role-based permissions to ensure only authorized users can perform administrative tasks.

### Stories

#### US-01.1: Admin Google Sign-In
**As an** administrator  
**I want to** sign in using my Google account  
**So that** I can access the admin dashboard securely without managing separate credentials

**Acceptance Criteria:**
- [ ] Google OAuth button visible on sign-in page
- [ ] Clicking button redirects to Google consent screen
- [ ] Successful auth creates session with 7-day expiry
- [ ] User email must be in approved admin list
- [ ] Failed attempts show clear error message
- [ ] Session persists across page refreshes

**Technical Notes:**
- Uses Better Auth library
- OAuth callback at `/api/auth/callback/google`
- Session stored in PostgreSQL

**Story Points:** 5  
**Priority:** P0  
**Status:** ✅ Done

---

#### US-01.2: Admin Email/Password Sign-In
**As an** administrator  
**I want to** sign in using email and password  
**So that** I have an alternative when Google OAuth is unavailable

**Acceptance Criteria:**
- [ ] Email and password fields on sign-in page
- [ ] Password must meet security requirements (8+ chars, mixed case)
- [ ] Invalid credentials show generic error (security)
- [ ] Rate limit: 5 failed attempts = 15 min lockout
- [ ] Successful login redirects to dashboard

**Story Points:** 3  
**Priority:** P1  
**Status:** ✅ Done

---

#### US-01.3: Session Management
**As an** authenticated user  
**I want** my session to be managed securely  
**So that** I don't have to re-login frequently but unauthorized access is prevented

**Acceptance Criteria:**
- [ ] Session expires after 7 days of inactivity
- [ ] Active sessions extend on user activity
- [ ] Sign-out button destroys session immediately
- [ ] Multiple sessions allowed (different devices)
- [ ] Sessions use HttpOnly, Secure, SameSite cookies

**Story Points:** 3  
**Priority:** P0  
**Status:** ✅ Done

---

#### US-01.4: Role-Based Route Protection
**As a** system  
**I want to** protect routes based on user role  
**So that** unauthorized users cannot access admin features

**Acceptance Criteria:**
- [ ] `/dashboard/*` requires authentication
- [ ] `/management/*` requires admin role
- [ ] `/schedule/*` requires admin role
- [ ] Unauthenticated users redirect to `/signin`
- [ ] Wrong role shows 403 Forbidden page
- [ ] Public routes remain accessible

**Story Points:** 5  
**Priority:** P0  
**Status:** ✅ Done

---

#### US-01.5: User Profile Display
**As an** authenticated user  
**I want to** see my profile information in the UI  
**So that** I know I'm logged in as the correct account

**Acceptance Criteria:**
- [ ] Profile picture displayed in header
- [ ] Full name shown on hover/click
- [ ] Email shown in dropdown menu
- [ ] Sign-out option in dropdown
- [ ] Graceful fallback if profile picture unavailable

**Story Points:** 2  
**Priority:** P2  
**Status:** ✅ Done

---

## EP-02: Semester Configuration

### Epic Description
Allow administrators to configure academic semesters with appropriate timeslot structures.

### Stories

#### US-02.1: Create New Semester
**As an** administrator  
**I want to** create a new semester configuration  
**So that** I can start building schedules for the upcoming term

**Acceptance Criteria:**
- [ ] Modal opens with semester configuration form
- [ ] Can select academic year (e.g., 2568)
- [ ] Can select semester (1 or 2)
- [ ] Cannot create duplicate year/semester combination
- [ ] Form validates required fields
- [ ] Success creates semester in DRAFT status

**Story Points:** 5  
**Priority:** P0  
**Status:** ✅ Done

---

#### US-02.2: Configure Timeslot Structure
**As an** administrator  
**I want to** configure period times and breaks  
**So that** the generated timeslots match our school's bell schedule

**Acceptance Criteria:**
- [ ] Can set first period start time
- [ ] Can set period duration (minutes)
- [ ] Can set break duration (minutes)
- [ ] Can set number of periods per day
- [ ] Can select active school days
- [ ] Preview shows generated timeslot grid
- [ ] Times displayed in Thai format (น.)

**Story Points:** 5  
**Priority:** P0  
**Status:** ✅ Done

---

#### US-02.3: Configure Lunch Periods
**As an** administrator  
**I want to** set different lunch periods for junior and senior levels  
**So that** the cafeteria isn't overcrowded

**Acceptance Criteria:**
- [ ] Can set junior lunch period number (e.g., 4)
- [ ] Can set senior lunch period number (e.g., 5)
- [ ] Lunch periods auto-lock on template apply
- [ ] System differentiates ม.1-3 vs ม.4-6

**Story Points:** 3  
**Priority:** P1  
**Status:** ✅ Done

---

#### US-02.4: Switch Active Semester
**As an** administrator  
**I want to** switch between semesters  
**So that** I can work on different terms

**Acceptance Criteria:**
- [ ] Semester selector in header/sidebar
- [ ] All pages update to show selected semester's data
- [ ] URL includes semester identifier (e.g., `/schedule/1-2568/...`)
- [ ] Last selected semester persists in local storage
- [ ] Can switch even if current semester is PUBLISHED

**Story Points:** 3  
**Priority:** P0  
**Status:** ✅ Done

---

## EP-03: Master Data Management

### Epic Description
Provide CRUD interfaces for managing teachers, subjects, rooms, and classes.

### Stories

#### US-03.1: Manage Teachers
**As an** administrator  
**I want to** create, view, edit, and delete teacher records  
**So that** I have an accurate teacher directory for assignments

**Acceptance Criteria:**
- [ ] List view shows all teachers with search/filter
- [ ] Can add new teacher with: prefix, first name, last name, department, email
- [ ] Can edit existing teacher details
- [ ] Can delete teacher (if no dependencies)
- [ ] Delete blocked if teacher has assignments
- [ ] Thai name prefixes supported (นาย, นาง, นางสาว)

**Story Points:** 5  
**Priority:** P0  
**Status:** ✅ Done

---

#### US-03.2: Manage Subjects
**As an** administrator  
**I want to** create, view, edit, and delete subject records  
**So that** I can define the curriculum subjects

**Acceptance Criteria:**
- [ ] List view shows all subjects grouped by learning area
- [ ] Can add new subject with: code, name, learning area, credits, hours
- [ ] Subject code validates against MOE pattern (e.g., ท21101)
- [ ] Can edit existing subject details
- [ ] Can delete subject (if no dependencies)
- [ ] 8 learning areas as per MOE standards

**Story Points:** 5  
**Priority:** P0  
**Status:** ✅ Done

---

#### US-03.3: Manage Rooms
**As an** administrator  
**I want to** create, view, edit, and delete room records  
**So that** I can assign physical spaces to schedules

**Acceptance Criteria:**
- [ ] List view shows all rooms
- [ ] Can add new room with: name/number, type, capacity, building
- [ ] Can edit existing room details
- [ ] Can delete room (sets NULL on schedules)
- [ ] Room types: ห้องเรียนทั่วไป, ห้องปฏิบัติการ, ห้องคอมพิวเตอร์, etc.

**Story Points:** 3  
**Priority:** P0  
**Status:** ✅ Done

---

#### US-03.4: Manage Classes/Grades
**As an** administrator  
**I want to** create, view, edit, and delete class records  
**So that** I can define the student groups

**Acceptance Criteria:**
- [ ] List view shows classes by year level
- [ ] Can add new class with: year (1-6), section, program
- [ ] DisplayID auto-generated (e.g., ม.1/1)
- [ ] Can edit existing class details
- [ ] Can delete class (if no schedules)
- [ ] Year uses Thai Mathayom levels (1-6)

**Story Points:** 3  
**Priority:** P0  
**Status:** ✅ Done

---

#### US-03.5: Import Teachers from CSV
**As an** administrator  
**I want to** import teacher data from a CSV file  
**So that** I can quickly populate the system

**Acceptance Criteria:**
- [ ] Upload CSV with columns: prefix, firstname, lastname, department, email
- [ ] Preview imported data before confirming
- [ ] Validate email uniqueness
- [ ] Show errors for invalid rows
- [ ] Partial import allowed (skip invalid rows)
- [ ] Progress indicator for large files

**Story Points:** 5  
**Priority:** P2  
**Status:** ⏳ Planned

---

#### US-03.6: Bulk Delete Records
**As an** administrator  
**I want to** delete multiple records at once  
**So that** I can clean up data efficiently

**Acceptance Criteria:**
- [ ] Checkbox selection on list views
- [ ] "Select all" option
- [ ] Bulk delete button appears when items selected
- [ ] Confirmation dialog shows count
- [ ] Blocked if any item has dependencies

**Story Points:** 3  
**Priority:** P2  
**Status:** ⏳ Planned

---

#### US-03.7: Search and Filter Data
**As an** administrator  
**I want to** search and filter management lists  
**So that** I can find specific records quickly

**Acceptance Criteria:**
- [ ] Search box on all management pages
- [ ] Real-time search (debounced)
- [ ] Filter by category/type where applicable
- [ ] Search matches name, code, and email
- [ ] Empty results show helpful message

**Story Points:** 3  
**Priority:** P1  
**Status:** ✅ Done

---

#### US-03.8: Data Validation Feedback
**As an** administrator  
**I want** clear validation error messages  
**So that** I can fix data entry mistakes

**Acceptance Criteria:**
- [ ] Required fields marked with asterisk
- [ ] Real-time validation on blur
- [ ] Error messages in Thai
- [ ] Specific guidance (not just "invalid")
- [ ] Form cannot submit with errors

**Story Points:** 2  
**Priority:** P1  
**Status:** ✅ Done

---

## EP-04: Program & Curriculum

### Epic Description
Enable configuration of academic programs with MOE-compliant subject structures.

### Stories

#### US-04.1: Create Academic Program
**As an** administrator  
**I want to** create academic programs for each year level  
**So that** I can define curriculum tracks (Science-Math, Arts-Lang, etc.)

**Acceptance Criteria:**
- [ ] Navigate to program management page
- [ ] Select year level (ม.1 - ม.6)
- [ ] Create program with: code, name, track, min credits
- [ ] Tracks: SCIENCE_MATH, LANGUAGE_MATH, ARTS_LANGUAGE, etc.
- [ ] Program linked to year level

**Story Points:** 5  
**Priority:** P1  
**Status:** ✅ Done

---

#### US-04.2: Add Subjects to Program
**As an** administrator  
**I want to** add subjects to a program  
**So that** I can define the curriculum requirements

**Acceptance Criteria:**
- [ ] View program detail page
- [ ] Add subject from master list
- [ ] Set category: CORE, ADDITIONAL, ACTIVITY
- [ ] Mark as mandatory or elective
- [ ] Set min/max credits for subject
- [ ] Cannot add same subject twice

**Story Points:** 3  
**Priority:** P1  
**Status:** ✅ Done

---

#### US-04.3: MOE Credit Validation
**As an** administrator  
**I want** the system to validate against MOE standards  
**So that** programs are compliant with Thai curriculum requirements

**Acceptance Criteria:**
- [ ] Lower secondary (ม.1-3): 28-32 lessons/week
- [ ] Upper secondary (ม.4-6): 30-34 lessons/week
- [ ] All 8 learning areas must be represented
- [ ] Warning shown if totals out of range
- [ ] Validation runs on save

**Story Points:** 5  
**Priority:** P1  
**Status:** ✅ Done

---

#### US-04.4: Copy Program Between Years
**As an** administrator  
**I want to** copy a program structure to another year  
**So that** I don't have to recreate similar programs

**Acceptance Criteria:**
- [ ] "Copy to..." action on program
- [ ] Select target year level
- [ ] Adjusts subject codes for new year
- [ ] Review before confirming
- [ ] Handles conflicts (duplicate programs)

**Story Points:** 3  
**Priority:** P2  
**Status:** ⏳ Planned

---

#### US-04.5: View Program Summary
**As an** administrator  
**I want to** see a summary of program credits and hours  
**So that** I can verify curriculum balance

**Acceptance Criteria:**
- [ ] Total credits displayed
- [ ] Total hours displayed
- [ ] Breakdown by category (CORE, ADDITIONAL, ACTIVITY)
- [ ] Breakdown by learning area
- [ ] Visual comparison to MOE requirements

**Story Points:** 3  
**Priority:** P2  
**Status:** ⏳ Partial

---

## EP-05: Teacher Assignment

### Epic Description
Assign teaching responsibilities to teachers for specific subjects and classes.

### Stories

#### US-05.1: View Teacher Workload
**As an** administrator  
**I want to** see each teacher's current workload  
**So that** I can balance assignments fairly

**Acceptance Criteria:**
- [ ] Teacher cards show total hours/week
- [ ] Visual indicator for overloaded teachers
- [ ] Click card to see assignment details
- [ ] Sort by name or workload
- [ ] Search by teacher name

**Story Points:** 3  
**Priority:** P0  
**Status:** ✅ Done

---

#### US-05.2: Assign Subject to Teacher
**As an** administrator  
**I want to** assign a teacher to teach a subject for a class  
**So that** the schedule system knows who teaches what

**Acceptance Criteria:**
- [ ] Select teacher from list
- [ ] Add grade/class to assignment
- [ ] Add subject(s) from class's program
- [ ] System calculates TeachHour from subject credits
- [ ] Can assign multiple subjects per class
- [ ] Save creates teachers_responsibility records

**Story Points:** 5  
**Priority:** P0  
**Status:** ✅ Done

---

#### US-05.3: Edit Teacher Assignments
**As an** administrator  
**I want to** modify existing assignments  
**So that** I can adjust when teachers change

**Acceptance Criteria:**
- [ ] View current assignments on teacher page
- [ ] Remove subject from assignment
- [ ] Add new subject to existing grade
- [ ] Add new grade with subjects
- [ ] Sync saves changes (diff-based)

**Story Points:** 3  
**Priority:** P0  
**Status:** ✅ Done

---

#### US-05.4: Assignment Empty State
**As an** administrator  
**I want** helpful guidance when no data exists  
**So that** I know what to do next

**Acceptance Criteria:**
- [ ] Empty state when no teachers exist
- [ ] Message: "เพิ่มข้อมูลครูก่อนเพื่อเริ่มมอบหมายงาน"
- [ ] Button links to teacher management
- [ ] Similar empty states for other missing prerequisites

**Story Points:** 2  
**Priority:** P1  
**Status:** ✅ Done

---

## EP-06: Schedule Arrangement

### Epic Description
Build class schedules using intuitive drag-and-drop interface with real-time conflict detection.

### Stories

#### US-06.1: Drag Subject to Timeslot
**As an** administrator  
**I want to** drag a subject from the palette and drop it on a timeslot  
**So that** I can build schedules intuitively

**Acceptance Criteria:**
- [ ] Subject palette shows assigned subjects with remaining slots
- [ ] Weekly grid shows all timeslots
- [ ] Drag subject from palette
- [ ] Drop on valid timeslot cell
- [ ] Room selection modal appears
- [ ] Schedule created on successful drop

**Story Points:** 8  
**Priority:** P0  
**Status:** ✅ Done

---

#### US-06.2: Real-Time Conflict Detection
**As an** administrator  
**I want** immediate feedback on scheduling conflicts  
**So that** I can avoid creating invalid schedules

**Acceptance Criteria:**
- [ ] Detect teacher double-booking on drop
- [ ] Detect class double-booking on drop
- [ ] Detect room double-booking on room select
- [ ] Show visual indicator (red border)
- [ ] Show error message explaining conflict
- [ ] Prevent invalid schedule creation

**Story Points:** 8  
**Priority:** P0  
**Status:** ✅ Done

---

#### US-06.3: Select Room for Schedule
**As an** administrator  
**I want to** select a room when creating a schedule  
**So that** everyone knows where classes are held

**Acceptance Criteria:**
- [ ] Room modal shows available rooms
- [ ] Unavailable rooms shown as disabled
- [ ] Search/filter rooms by name or type
- [ ] Room capacity displayed
- [ ] Selected room saved with schedule

**Story Points:** 3  
**Priority:** P0  
**Status:** ✅ Done

---

#### US-06.4: Move Existing Schedule
**As an** administrator  
**I want to** move an existing schedule to a different timeslot  
**So that** I can rearrange without deleting and recreating

**Acceptance Criteria:**
- [ ] Drag existing schedule from grid
- [ ] Drop on new timeslot
- [ ] Same conflict validation applies
- [ ] Option to keep or change room
- [ ] Atomic update (no data loss)

**Story Points:** 5  
**Priority:** P1  
**Status:** ✅ Done

---

#### US-06.5: Delete Schedule
**As an** administrator  
**I want to** delete a schedule entry  
**So that** I can clear mistakes or rebuild

**Acceptance Criteria:**
- [ ] Delete button on schedule cell
- [ ] Confirmation before delete
- [ ] Schedule removed from grid
- [ ] Palette counter incremented
- [ ] Associated conflicts cleared

**Story Points:** 2  
**Priority:** P0  
**Status:** ✅ Done

---

#### US-06.6: View by Teacher/Class/Room
**As an** administrator  
**I want to** switch between different schedule views  
**So that** I can see schedules from different perspectives

**Acceptance Criteria:**
- [ ] Teacher arrange view (default)
- [ ] Class arrange view
- [ ] Room arrange view
- [ ] Same grid layout in all views
- [ ] Selector to choose entity
- [ ] URL reflects current view

**Story Points:** 5  
**Priority:** P1  
**Status:** ✅ Done

---

#### US-06.7: Schedule Completion Tracking
**As an** administrator  
**I want to** see progress towards schedule completion  
**So that** I know how much work remains

**Acceptance Criteria:**
- [ ] Progress bar on arrange page
- [ ] Percentage complete (schedules / required)
- [ ] Completion badge when teacher fully scheduled
- [ ] Overall semester completion on dashboard
- [ ] Publish gate requires minimum completion

**Story Points:** 3  
**Priority:** P1  
**Status:** ✅ Done

---

## EP-07: Timeslot Locking

### Epic Description
Lock timeslots to prevent scheduling during assemblies, activities, and breaks.

### Stories

#### US-07.1: Toggle Timeslot Lock
**As an** administrator  
**I want to** lock individual timeslots  
**So that** no classes can be scheduled during that time

**Acceptance Criteria:**
- [ ] Lock page shows timeslot grid
- [ ] Click cell to toggle lock status
- [ ] Locked cells show lock icon
- [ ] Lock status saved immediately
- [ ] Locked slots prevent drag-drop

**Story Points:** 3  
**Priority:** P1  
**Status:** ✅ Done

---

#### US-07.2: Apply Lock Template
**As an** administrator  
**I want to** apply predefined lock templates  
**So that** I can quickly lock standard time slots

**Acceptance Criteria:**
- [ ] Template selector dropdown
- [ ] Templates: lunch breaks, assemblies, clubs
- [ ] Preview shows affected slots
- [ ] Apply button locks all matching
- [ ] Can apply multiple templates

**Story Points:** 5  
**Priority:** P1  
**Status:** ✅ Done

---

#### US-07.3: Bulk Lock Selection
**As an** administrator  
**I want to** select multiple slots and lock them together  
**So that** I can handle custom scenarios

**Acceptance Criteria:**
- [ ] Selection mode toggle
- [ ] Click to select multiple cells
- [ ] "Lock selected" button
- [ ] "Unlock selected" button
- [ ] Clear selection option

**Story Points:** 3  
**Priority:** P2  
**Status:** ✅ Done

---

#### US-07.4: Lock with Description
**As an** administrator  
**I want to** add a description to locked slots  
**So that** I remember why they're locked

**Acceptance Criteria:**
- [ ] Optional description field when locking
- [ ] Description shown on hover
- [ ] Description visible in exports
- [ ] Can edit description later
- [ ] Descriptions like "ประชุมครู", "กิจกรรมชุมนุม"

**Story Points:** 2  
**Priority:** P2  
**Status:** ⏳ Planned

---

## EP-08: Conflict Management

### Epic Description
Detect, display, and resolve scheduling conflicts to ensure valid timetables.

### Stories

#### US-08.1: View All Conflicts
**As an** administrator  
**I want to** see all current conflicts in one place  
**So that** I can systematically resolve them

**Acceptance Criteria:**
- [ ] Conflicts page shows all detected issues
- [ ] Grouped by type: teacher, class, room
- [ ] Each card shows details: who, when, what
- [ ] Click card to navigate to affected schedule
- [ ] Count badge in sidebar navigation

**Story Points:** 5  
**Priority:** P1  
**Status:** ✅ Done

---

#### US-08.2: Auto-Detect on Save
**As an** administrator  
**I want** conflicts detected automatically when I save  
**So that** I'm immediately aware of problems

**Acceptance Criteria:**
- [ ] Conflict check runs on every schedule save
- [ ] Real-time update of conflict count
- [ ] Toast notification if new conflict created
- [ ] Option to undo last action

**Story Points:** 3  
**Priority:** P1  
**Status:** ✅ Done

---

#### US-08.3: Resolve Conflict via Edit
**As an** administrator  
**I want to** resolve conflicts by editing schedules  
**So that** I can fix issues without deleting everything

**Acceptance Criteria:**
- [ ] "Fix" button on conflict card
- [ ] Navigates to affected schedule
- [ ] Conflict highlighted in grid
- [ ] Edit or delete to resolve
- [ ] Conflict removed after resolution

**Story Points:** 3  
**Priority:** P1  
**Status:** ✅ Done

---

#### US-08.4: Conflict-Free Publishing Gate
**As an** administrator  
**I want** publishing blocked when conflicts exist  
**So that** I don't publish invalid schedules

**Acceptance Criteria:**
- [ ] Publish button shows conflict count
- [ ] Warning if conflicts > 0
- [ ] Admin can override with confirmation
- [ ] Conflict warning in published state

**Story Points:** 3  
**Priority:** P1  
**Status:** ✅ Done

---

## EP-09: Export & Reporting

### Epic Description
Export schedules to downloadable formats for distribution.

### Stories

#### US-09.1: Export Teacher Schedule to Excel
**As an** administrator  
**I want to** export a teacher's schedule to Excel  
**So that** I can share it with them

**Acceptance Criteria:**
- [ ] Export button on teacher schedule view
- [ ] Downloads .xlsx file
- [ ] Weekly grid format
- [ ] Includes semester info in header
- [ ] Subject, room, class in cells

**Story Points:** 5  
**Priority:** P1  
**Status:** ✅ Done

---

#### US-09.2: Export Class Schedule to Excel
**As an** administrator  
**I want to** export a class schedule to Excel  
**So that** students can have a printable timetable

**Acceptance Criteria:**
- [ ] Export button on class schedule view
- [ ] Downloads .xlsx file
- [ ] Weekly grid format
- [ ] Subject, teacher, room in cells
- [ ] Professional styling

**Story Points:** 5  
**Priority:** P1  
**Status:** ✅ Done

---

#### US-09.3: Bulk Export All Schedules
**As an** administrator  
**I want to** export all schedules at once  
**So that** I can distribute to the entire school

**Acceptance Criteria:**
- [ ] Bulk export button on dashboard
- [ ] Select: all teachers, all classes, or both
- [ ] Multi-sheet Excel or ZIP file
- [ ] Progress indicator for large exports
- [ ] Background processing for > 50 entities

**Story Points:** 8  
**Priority:** P2  
**Status:** ⏳ Partial

---

## EP-10: Public Schedule Viewing

### Epic Description
Allow public access to published schedules for teachers, students, and parents.

### Stories

#### US-10.1: Search Teacher Schedule
**As a** visitor  
**I want to** search for a teacher's schedule  
**So that** I can find when they're available

**Acceptance Criteria:**
- [ ] Homepage has teacher search box
- [ ] Autocomplete shows matching teachers
- [ ] Select teacher navigates to schedule page
- [ ] Weekly timetable displays
- [ ] Subject, class, room visible per slot

**Story Points:** 3  
**Priority:** P0  
**Status:** ✅ Done

---

#### US-10.2: Browse Class Schedules
**As a** visitor  
**I want to** browse schedules by class  
**So that** students can find their timetable

**Acceptance Criteria:**
- [ ] Homepage has class tab/section
- [ ] Grade level tabs (ม.1 - ม.6)
- [ ] Class cards show section number
- [ ] Click card shows weekly timetable
- [ ] Subject, teacher, room visible

**Story Points:** 3  
**Priority:** P0  
**Status:** ✅ Done

---

#### US-10.3: Semester Toggle on Public View
**As a** visitor  
**I want to** switch between semesters  
**So that** I can see current or past schedules

**Acceptance Criteria:**
- [ ] Semester selector on public pages
- [ ] Only published semesters shown
- [ ] Current semester selected by default
- [ ] URL updates with semester

**Story Points:** 2  
**Priority:** P1  
**Status:** ✅ Done

---

#### US-10.4: Mobile-Friendly Public View
**As a** visitor on mobile  
**I want** schedules to display well on my phone  
**So that** I can check quickly on the go

**Acceptance Criteria:**
- [ ] Responsive layout for timetable
- [ ] Horizontal scroll for wide tables
- [ ] Touch-friendly navigation
- [ ] Fast load times on mobile
- [ ] Day-by-day view option

**Story Points:** 5  
**Priority:** P2  
**Status:** ⏳ Partial

---

## EP-11: Analytics Dashboard

### Epic Description
Provide data visualizations to help administrators understand scheduling patterns.

### Stories

#### US-11.1: Teacher Workload Chart
**As an** administrator  
**I want to** see a visualization of teacher workloads  
**So that** I can identify imbalances

**Acceptance Criteria:**
- [ ] Bar chart showing hours/week per teacher
- [ ] Sorted by workload (high to low)
- [ ] Color coding: green (normal), yellow (high), red (overloaded)
- [ ] Click bar to see teacher detail
- [ ] Filter by department

**Story Points:** 5  
**Priority:** P2  
**Status:** ⏳ Planned

---

#### US-11.2: Room Utilization Heatmap
**As an** administrator  
**I want to** see room utilization across the week  
**So that** I can optimize space usage

**Acceptance Criteria:**
- [ ] Heatmap grid: rooms × timeslots
- [ ] Color intensity shows utilization
- [ ] Hover shows booking details
- [ ] Identify underutilized rooms
- [ ] Filter by building/floor

**Story Points:** 5  
**Priority:** P2  
**Status:** ⏳ Planned

---

#### US-11.3: Schedule Completion Overview
**As an** administrator  
**I want** a dashboard overview of completion status  
**So that** I know overall progress

**Acceptance Criteria:**
- [ ] Donut chart: complete vs incomplete
- [ ] Progress bars by entity type
- [ ] List of incomplete items
- [ ] Trend over time (optional)
- [ ] Link to fix incomplete

**Story Points:** 3  
**Priority:** P2  
**Status:** ⏳ Partial

---

## EP-12: Mobile Responsiveness

### Epic Description
Ensure the system is usable on mobile devices for on-the-go access.

### Stories

#### US-12.1: Responsive Management Pages
**As an** administrator on tablet  
**I want** management pages to work on smaller screens  
**So that** I can make quick updates anywhere

**Acceptance Criteria:**
- [ ] Tables scroll horizontally on small screens
- [ ] Forms stack vertically
- [ ] Touch targets ≥ 44px
- [ ] Modal dialogs fit screen
- [ ] Navigation collapses to menu

**Story Points:** 5  
**Priority:** P2  
**Status:** ⏳ Partial

---

#### US-12.2: Mobile Schedule Viewing
**As an** administrator on mobile  
**I want** to view schedules on my phone  
**So that** I can check things quickly

**Acceptance Criteria:**
- [ ] Timetable fits narrow screen
- [ ] Can scroll/swipe through days
- [ ] Key info visible without zoom
- [ ] Day picker for mobile

**Story Points:** 5  
**Priority:** P2  
**Status:** ⏳ Partial

---

#### US-12.3: Touch-Friendly Drag-Drop
**As an** administrator on touchscreen  
**I want** drag-drop to work with touch  
**So that** I can arrange schedules on tablet

**Acceptance Criteria:**
- [ ] Long-press to initiate drag
- [ ] Visual feedback during drag
- [ ] Drop zones highlight on hover
- [ ] Cancel gesture available
- [ ] Same validation as desktop

**Story Points:** 8  
**Priority:** P3  
**Status:** ⏳ Planned

---

## Backlog Prioritization

### P0 - Must Have (Launch Blockers)
- US-01.1, US-01.3, US-01.4 (Authentication)
- US-02.1, US-02.2, US-02.4 (Semester Config)
- US-03.1, US-03.2, US-03.3, US-03.4 (Master Data)
- US-05.1, US-05.2, US-05.3 (Assignment)
- US-06.1, US-06.2, US-06.3, US-06.5 (Scheduling)
- US-10.1, US-10.2 (Public View)

### P1 - Should Have (Important)
- US-01.2 (Email/Password)
- US-02.3 (Lunch Periods)
- US-03.7, US-03.8 (Search/Validation)
- US-04.1, US-04.2, US-04.3 (Programs)
- US-05.4 (Empty State)
- US-06.4, US-06.6, US-06.7 (Schedule Views)
- US-07.1, US-07.2 (Locking)
- US-08.1, US-08.2, US-08.3, US-08.4 (Conflicts)
- US-09.1, US-09.2 (Export)
- US-10.3 (Semester Toggle)

### P2 - Nice to Have (Enhancement)
- US-01.5 (Profile Display)
- US-03.5, US-03.6 (Import/Bulk Delete)
- US-04.4, US-04.5 (Copy Program/Summary)
- US-07.3, US-07.4 (Bulk Lock/Description)
- US-09.3 (Bulk Export)
- US-10.4 (Mobile Public)
- US-11.1, US-11.2, US-11.3 (Analytics)
- US-12.1, US-12.2 (Mobile Admin)

### P3 - Future (Post-Launch)
- US-12.3 (Touch Drag-Drop)

---

## Story Status Legend

| Status | Meaning |
|--------|---------|
| ✅ Done | Implemented and tested |
| ⏳ Partial | Partially implemented |
| ⏳ Planned | Designed, not implemented |
| 🔮 Future | Backlog for future releases |
