# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a single-page web application for managing academy (학원) administrative tasks. The app is a comprehensive task management system with real-time synchronization across multiple staff members using Firebase Realtime Database. All code is contained in a single `index.html` file written in Korean.

## Architecture

### Single-Page Application Structure
- **Technology Stack**: Vanilla JavaScript, Firebase Realtime Database, Tailwind CSS (via CDN)
- **File Organization**: Monolithic single-file architecture - all HTML, CSS, and JavaScript in `index.html`
- **State Management**: Local JavaScript objects synchronized with Firebase paths
- **Real-time Sync**: Firebase listeners (`on('value')`) for automatic updates across all connected clients
- **Past Incomplete Tasks Tracking**: Automatically loads and displays incomplete tasks from past dates

### Firebase Database Schema

```
/
├── daily/{YYYY-MM-DD}/
│   └── {taskId}: {label, checked, note}
├── korean/{YYYY-MM-DD}/
│   └── {studentId}: {name, score, attendance, wrong}
├── hsMgmt/{YYYY-MM-DD}/
│   └── {studentId}: {subject, name, score, range}
├── typing/{YYYY-MM-DD}/
│   └── {taskId}: {task, progress, completed}
├── classcard/{YYYY-MM-DD}/
│   ├── roster: {studentName: true}
│   └── done: {studentName: true}
├── vocabAccum/{YYYY-MM-DD}/
│   ├── roster: {studentName: true}
│   └── done: {studentName: true}
├── tutoringSessions/ (global, not date-scoped)
│   └── {sessionId}: {id, subject, name, time, topic, range, note, date}
├── retest/ (global, not date-scoped)
│   └── {retestId}: {id, name, subject, vocabBook, vocabRange, date, time}
├── homework/ (global, not date-scoped)
│   └── {homeworkId}: {id, classId, className, subject, studentId, studentName, title, description, textbook, pageRange, assignedDate, dueDate, completed, completedAt, feedback, notificationSent, notificationSentAt}
├── classes/ (global, not date-scoped)
│   └── {classId}: {id, name, subject, teacher, status, schedule, createdAt, endedAt, autoExtracted}
├── vocab/
│   ├── books/{bookId}: {name, units:[...]}
│   ├── rules/{ruleId}: {student, teacher, bookId, startRange, unitSize, overlap, overlapMode, weekdays[], startDate, maxUnit, occurrences}
│   └── events/{eventId}: {id, ruleId, student, teacher, bookId, date, rangeStart, rangeEnd, done, isRescheduled?, originalDate?}
└── settings/
    └── defaultDailyTasks: [array of default task labels]
```

### State Variables

```javascript
// Date navigation
let currentViewDate = todayYMD();

// Settings and data
let defaultDaily = [];    // Default checklist items from settings
let dailyTasks = {};      // Current date's tasks {id: {label, checked, note}}
let pastIncompleteDailyTasks = {}; // Past incomplete tasks {date: {id: {label, checked, note}}}
let koreanCoaching = {};
let hsMgmt = {};
let typingTasks = {};     // Current date's typing tasks
let pastIncompleteTypingTasks = {}; // Past incomplete typing {date: {id: {task, progress, completed}}}
let tutoringSessions = {}; // Global tutoring sessions
let tutoringFilter = 'all'; // 'all', 'math', 'english'
let retestStudents = {};
let students = {};

// Roster-based systems
let ccState = { roster:{}, done:{} };
let vaState = { roster:{}, done:{} };

// Homework management
let homework = {};        // {id: {classId, className, subject, studentId, studentName, title, ...}}
let classes = {};         // {id: {name, subject, teacher, status, schedule, ...}}
let homeworkClassFilter = 'all';
let homeworkStatusFilter = 'all';

// Vocab scheduler
let vocabBooks = {};
let vocabRules = {};
let vocabEvents = {};
```

### Core Functionality Modules

1. **Date Navigation System** (`index.html:765-2072`)
   - `currentViewDate`: Global state variable tracking the currently viewed date
   - `changeDate(dir)`: Navigate forward/backward by days
   - `goToToday()`: Jump to today's date
   - `updateDateDisplay()`: Updates UI to reflect current view date
   - Most data is scoped to `currentViewDate`, except retest and vocab scheduler

2. **Listener Management** (`index.html:1834-2061`)
   - **Date-scoped listeners**: Attached/detached when date changes (daily, korean, hsMgmt, typing, classcard, vocabAccum)
   - **Global listeners**: Attached once at startup (retest, settings, vocab, tutoring sessions)
   - `detachListeners()`: Cleans up date-scoped Firebase listeners
   - `attachListenersForDate()`: **Async function** that loads past incomplete tasks, then establishes Firebase listeners for current date
   - `attachGlobalListenersOnce()`: Sets up persistent listeners
   - **Important**: `attachListenersForDate()` is async and awaits `loadPastIncompleteTasks()` before setting up listeners

3. **Past Incomplete Tasks System** (`index.html:1845-1896`)
   - **Purpose**: Ensures uncompleted tasks from previous dates remain visible until completed
   - `loadPastIncompleteTasks()`: Async function that scans past 30 days for incomplete items
   - Loads incomplete daily tasks (`checked: false`)
   - Loads incomplete typing tasks (`completed: false`)
   - Stores results in `pastIncompleteDailyTasks` and `pastIncompleteTypingTasks`
   - Called automatically when date changes or page loads
   - **Visual distinction**: Past items displayed with orange background (`bg-orange-50`) and original date labels
   - **Functions for past items**:
     - `togglePastTask(date, id)`: Marks past daily task as completed
     - `deletePastTask(date, id)`: Deletes past daily task
     - `togglePastTypingComplete(date, id)`: Marks past typing task as completed
     - `deletePastTyping(date, id)`: Deletes past typing task

4. **Daily Task Checklist** (`index.html:869-939`)
   - Auto-seeds from `defaultDaily` array on first load each day
   - Each task has: label, checked status, and optional note
   - `seedDefaultDaily(date)`: Creates initial tasks for a new date
   - **Past incomplete feature**: Displays uncompleted tasks from previous dates
   - `renderDailyTasks()`: Shows current date tasks + past incomplete tasks in orange section
   - Progress bar and count include both current and past incomplete items
   - Past items sorted by date (most recent first) with Korean date labels

5. **Subject-Specific Management**
   - **Korean Coaching** (`index.html:941-967`): Track student scores, attendance, wrong problems
   - **History/Science Management** (`index.html:956-967`): Subject-specific tracking with score and range
   - **Typing Tasks** (`index.html:969-1037`):
     - Work items with progress tracking and completion toggle
     - **Past incomplete feature**: Shows uncompleted typing tasks from previous dates
     - `renderTyping()`: Displays current tasks + past incomplete tasks in orange section
     - Past items include original date labels and completion buttons

6. **Subject-Based Tutoring Sessions** (`index.html:1039-1108`)
   - Global scope (not date-specific) - stored in `/tutoringSessions`
   - **Supports multiple subjects**: Math and English with visual badges
   - Fields: subject, name, time, topic, range, note, date
   - **Subject filtering**: All/Math/English filter buttons
   - Categorizes sessions: Today (blue highlight), Upcoming, Past
   - `renderTutoring()`: Sorts by date and time, filters by subject, groups by category
   - `setTutoringFilter()`: Changes active filter and updates UI
   - Includes "오늘 보강" badge showing today's count
   - Color-coded subject badges: Purple for Math, Green for English

7. **Retest Management** (`index.html:1110-1162`)
   - Global scope (not date-specific)
   - Categorizes retests: Today (red highlight), Upcoming, Past
   - Includes vocabulary book and range fields
   - `renderRetest()`: Sorts and groups by date category

8. **Classcard & Vocab Accumulation Tests** (`index.html:1164-1241`)
   - Two similar roster-based systems
   - Maintains two sets: `roster` (all students) and `done` (completed)
   - `diffPending()`: Calculates students who haven't completed
   - Bulk input via comma/newline separated lists
   - **Vocab Accumulation includes weekly statistics feature**:
     - `generateWeeklyStats()`: Analyzes vocab test performance over 7 days
     - `applyWeeklyStats()`: Adds students to roster based on test history
     - Shows student names with test counts (e.g., "홍길동: 3회")

9. **Homework Management System**
   - Global scope (not date-specific) - stored in `/homework` and `/classes`
   - **Class Management**:
     - `addClass()`: Add new class with name, subject, teacher, schedule
     - `endClass(id)`: Toggle class status between active/ended
     - `extractClassesFromStudents()`: Auto-extract class names from student data
     - `renderClasses()`: Display class list with status filtering
   - **Homework CRUD**:
     - `addHomework()`: Create homework with class, student, title, due date, etc.
     - `toggleHomeworkComplete(id)`: Mark homework as completed/incomplete
     - `addHomeworkFeedback(id)`: Add teacher feedback to homework
     - `deleteHomework(id)`: Remove homework entry
   - **Filtering & Display**:
     - `homeworkClassFilter`: Filter by class
     - `homeworkStatusFilter`: Filter by status (all/incomplete/completed/overdue)
     - `renderHomework()`: Groups by due date with visual indicators (overdue: red, today: orange)
   - **Kakao Notification Integration**:
     - `generateHomeworkMessage(id)`: Create formatted notification message
     - `openKakaoMessage(id)`: Display message in modal
     - `copyKakaoMessage()`: Copy message to clipboard for manual sending
   - **Excel Import**:
     - `importHomeworkFromExcel(event)`: Bulk import from Excel file
     - Required columns: 수업명, 학생이름, 숙제내용, 마감일
     - Optional columns: 교재, 범위
   - **Notification Badge**: Includes overdue and due-today homework in count

10. **Subject-Based Tutoring Analytics**
   - Comprehensive analytics system for tutoring sessions (Math & English)
   - **Subject filtering dropdown**: All/Math Only/English Only
   - **Three tabs**:
     - **Summary**: Total students, total sessions, 7-day sessions, today's sessions, TOP 5 students/topics
     - **Students**: Per-student statistics with subjects studied, session count, topics, and recent history
     - **Timeline**: 30-day bar chart showing daily session counts
   - `renderAnalysis()`: Updates current tab view with subject filter applied
   - `analysisSwitchTab()`: Handles tab switching
   - `renderAnalysisSummary()`, `renderAnalysisStudents()`, `renderAnalysisTimeline()`: All respect subject filter
   - Data sourced from global `tutoringSessions` object

11. **Vocab Scheduler** (`index.html:1243-1826`)
    - Complex rule-based scheduling system
    - **Books**: Define textbooks with unit counts and prefixes
    - **Rules**: Define recurring vocabulary test schedules with:
      - Student, teacher, book, start range, unit size
      - **Overlap settings**: `overlap` (number of units) + `overlapMode` ('end' or 'front')
        - 'end': 1-3 → 3-5 → 5-7 (overlaps at end)
        - 'front': 1-3 → 2-4 → 3-5 (overlaps from front)
      - Weekdays selection (0-6)
    - **Events**: Auto-generated test instances from rules
    - **Reschedule feature**: When student is absent, can postpone test to next scheduled date
      - `vocabRescheduleEvent(id)`: Moves event to next date based on rule's weekdays
      - Adds `isRescheduled` flag and `originalDate` tracking
      - Visual indicator: Orange badge showing "연기됨"
    - `vocabRegenForRule(ruleId, clearExisting)`: Generates event instances
    - `vocabNextDates()`: Calculates future test dates based on weekdays
    - `vocabNextByWeekdays()`: Finds next available date from weekdays array

12. **Notification System** (`index.html:2073-2099`)
    - Red badge displays total count of:
      - Incomplete daily tasks (current + past)
      - Today's retests
      - **Today's tutoring sessions (all subjects)**
      - Incomplete typing tasks (current + past)
      - Today's vocab test events
      - **Overdue and due-today homework**
    - **Note**: Past incomplete items are counted in notifications
    - Notification panel shows subject labels for tutoring sessions
    - Notification panel shows overdue homework with ⚠️ indicator
    - Only visible when viewing today's date
    - `checkNotifications()`: Recalculates badge after any data change
    - `updateNotificationPanel()`: Displays detailed breakdown with subject labels

## Development Guidelines

### Making Changes

**Working with Date-Scoped Data:**
- Always use `currentViewDate` variable, not `todayYMD()`
- Remember to call appropriate `render*()` function after Firebase updates
- Consider whether changes need listener reattachment via `detachListeners()` + `attachListenersForDate()`
- **Note**: `attachListenersForDate()` is async - use `await` if you need to ensure completion

**Working with Global Data:**
- Tutoring sessions, retest, and vocab scheduler data is NOT date-scoped in Firebase
- Global listeners remain active during date navigation
- Changes persist across date views
- Tutoring sessions are stored in `/tutoringSessions` with explicit `date` and `subject` fields for filtering
- Subject filtering is applied at render time, not in Firebase queries

**Working with Past Incomplete Tasks:**
- Past incomplete tasks are loaded from past 30 days
- Daily tasks: Items with `checked: false`
- Typing tasks: Items with `completed: false`
- Past items are displayed with orange highlighting and date labels
- When a past item is completed, it's removed from the past incomplete list
- `loadPastIncompleteTasks()` is called automatically on date changes

**Adding New Features:**
- Inline all code in `index.html` - no external files
- Use Firebase's `push().key` for generating unique IDs
- Follow existing patterns: state object → Firebase listener → render function
- Add to `checkNotifications()` if feature should trigger badge
- If adding date-scoped data that should persist when incomplete, add to `loadPastIncompleteTasks()`

**Firebase Operations:**
- Use `.set()` to overwrite entire object
- Use `.update()` to patch specific fields
- Use `.remove()` or `set(null)` to delete
- Always add `.catch()` error handlers
- Use `.once('value')` for one-time reads (like in `loadPastIncompleteTasks()`)
- Use `.on('value')` for real-time listeners

### Testing

**Local Testing:**
```bash
# Serve the file via any HTTP server
python -m http.server 8000
# OR
npx http-server .
```

Then navigate to `http://localhost:8000/index.html`

**Firebase Testing:**
- All data is in Firebase project: `ganghanlucy`
- Database URL: `https://ganghanlucy-default-rtdb.firebaseio.com`
- Changes are immediately visible to all connected clients
- Use different dates to test without affecting production data
- **Testing past incomplete tasks**: Create tasks on past dates with `checked: false` or `completed: false`

### Common Modifications

**Adding a New Task Section:**
1. Define state object (e.g., `let newSection = {}`)
2. Add render function (e.g., `renderNewSection()`)
3. Add CRUD functions (add/delete/toggle)
4. Add Firebase listener in `attachListenersForDate()` or `attachGlobalListenersOnce()`
5. Add HTML section in the main card area
6. Update notification system if needed
7. **Optional**: Add past incomplete tracking in `loadPastIncompleteTasks()` if applicable

**Modifying Date Scope:**
- To make date-scoped data global: Move from `attachListenersForDate()` to `attachGlobalListenersOnce()`
- To make global data date-scoped: Change Firebase path to include `currentViewDate`, move listener setup

**Adding Past Incomplete Tracking to New Features:**
1. Add state variable: `let pastIncompleteFeatureName = {}`
2. Add loading logic in `loadPastIncompleteTasks()`:
   ```javascript
   promises.push(
     db.ref('featureName/' + pastDate).once('value').then(snap => {
       if (snap.exists()) {
         const items = snap.val();
         const incomplete = {};
         for (const [id, item] of Object.entries(items)) {
           if (!item.completionField) {
             incomplete[id] = {...item, originalDate: pastDate};
           }
         }
         if (Object.keys(incomplete).length > 0) {
           pastIncompleteFeatureName[pastDate] = incomplete;
         }
       }
     }).catch(err => console.error('과거 feature 로드 실패:', pastDate, err))
   );
   ```
3. Update render function to display past incomplete items with orange styling
4. Add toggle/delete functions for past items

**Changing Vocab Scheduler Logic:**
- Overlap calculation: Look for overlap mode handling
- Date calculation: `vocabNextDates()` function
- Range progression: Look for `nextStart` and `nextEnd` calculations
- Reschedule logic: `vocabRescheduleEvent()` function

**Working with Reschedule Feature:**
- Reschedule automatically finds next date based on rule's weekdays
- Tracks original date to preserve scheduling history
- Visual indicators (orange badge) help identify rescheduled tests
- Can reschedule already-rescheduled tests (preserves original date)
- Confirmation dialog shows old and new dates in Korean format

## Important Notes

- Korean language UI - all user-facing text is in Korean
- Mobile-responsive via Tailwind's `md:` breakpoints with touch-friendly interfaces
- No build process - direct file execution
- Firebase credentials are embedded (consider environment variables for production)
- No authentication system - shared access model
- **Subject-based tutoring system**: Unified system for Math and English with filtering and analytics
- **Multi-subject analytics**: Provides comprehensive data analysis with subject filtering for tracking student progress
- **Vocab test reschedule**: Handles student absences by postponing tests to next scheduled date automatically
- **Past incomplete tasks tracking**: Ensures tasks are never forgotten - uncompleted items persist across dates
- Global data (tutoring sessions, retests, vocab) persists across all date views for comprehensive tracking
- Easy to extend to additional subjects by adding options to `tutoringSubject` dropdown and updating color schemes
- Rescheduled tests maintain audit trail with original date tracking
- Past incomplete items displayed with visual distinction (orange backgrounds) and date labels
- All async operations properly handled with `async/await` pattern

## Performance Considerations

- Past incomplete tasks scan limited to 30 days to balance comprehensiveness with performance
- Uses `Promise.all()` for parallel Firebase queries in `loadPastIncompleteTasks()`
- Firebase listeners are properly detached on date changes to prevent memory leaks
- Real-time listeners only active for current date data
- Past incomplete data loaded once per date change, not continuously monitored
