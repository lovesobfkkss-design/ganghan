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
├── vocab/
│   ├── books/{bookId}: {name, units:[...]}
│   ├── rules/{ruleId}: {student, teacher, bookId, startRange, unitSize, overlap, overlapMode, weekdays[], startDate, maxUnit, occurrences}
│   └── events/{eventId}: {id, ruleId, student, teacher, bookId, date, rangeStart, rangeEnd, done, isRescheduled?, originalDate?}
└── settings/
    └── defaultDailyTasks: [array of default task labels]
```

### Core Functionality Modules

1. **Date Navigation System** (`index.html:406-465`)
   - `currentViewDate`: Global state variable tracking the currently viewed date
   - `changeDate(dir)`: Navigate forward/backward by days
   - `updateDateDisplay()`: Updates UI to reflect current view date
   - Most data is scoped to `currentViewDate`, except retest and vocab scheduler

2. **Listener Management** (`index.html:840-973`)
   - **Date-scoped listeners**: Attached/detached when date changes (daily, korean, hsMgmt, typing, classcard, vocabAccum)
   - **Global listeners**: Attached once at startup (retest, settings, vocab)
   - `detachListeners()`: Cleans up date-scoped Firebase listeners
   - `attachListenersForDate()`: Establishes Firebase listeners for current date
   - `attachGlobalListenersOnce()`: Sets up persistent listeners

3. **Daily Task Checklist** (`index.html:502-532`)
   - Auto-seeds from `defaultDaily` array on first load each day
   - Each task has: label, checked status, and optional note
   - `seedDefaultDaily(date)`: Creates initial tasks for a new date

4. **Subject-Specific Management**
   - **Korean Coaching** (`index.html:534-546`): Track student scores, attendance, wrong problems
   - **History/Science Management** (`index.html:548-560`): Subject-specific tracking with score and range
   - **Typing Tasks** (`index.html:562-580`): Work items with progress tracking and completion toggle

5. **Subject-Based Tutoring Sessions** (`index.html:678-746`)
   - Global scope (not date-specific) - stored in `/tutoringSessions`
   - **Supports multiple subjects**: Math and English with visual badges
   - Fields: subject, name, time, topic, range, note, date
   - **Subject filtering**: All/Math/English filter buttons
   - Categorizes sessions: Today (blue highlight), Upcoming, Past
   - `renderTutoring()`: Sorts by date and time, filters by subject, groups by category
   - `setTutoringFilter()`: Changes active filter and updates UI
   - Includes "오늘 보강" badge showing today's count
   - Color-coded subject badges: Purple for Math, Green for English

6. **Retest Management** (`index.html:708-762`)
   - Global scope (not date-specific)
   - Categorizes retests: Today (red highlight), Upcoming, Past
   - Includes vocabulary book and range fields
   - `renderRetest()`: Sorts and groups by date category

7. **Classcard & Vocab Accumulation Tests** (`index.html:764-815`)
   - Two similar roster-based systems
   - Maintains two sets: `roster` (all students) and `done` (completed)
   - `diffPending()`: Calculates students who haven't completed
   - Bulk input via comma/newline separated lists

8. **Subject-Based Tutoring Analytics** (`index.html:1579-1737`)
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

9. **Vocab Scheduler** (`index.html:1147-1570`)
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

10. **Notification System** (`index.html:831-871`)
   - Red badge displays total count of:
     - Incomplete daily tasks
     - Today's retests
     - **Today's tutoring sessions (all subjects)**
     - Incomplete typing tasks
     - Today's vocab test events
   - Notification panel shows subject labels for tutoring sessions
   - Only visible when viewing today's date
   - `checkNotifications()`: Recalculates badge after any data change
   - `updateNotificationPanel()`: Displays detailed breakdown with subject labels

## Development Guidelines

### Making Changes

**Working with Date-Scoped Data:**
- Always use `currentViewDate` variable, not `todayYMD()`
- Remember to call appropriate `render*()` function after Firebase updates
- Consider whether changes need listener reattachment via `detachListeners()` + `attachListenersForDate()`

**Working with Global Data:**
- Tutoring sessions, retest, and vocab scheduler data is NOT date-scoped in Firebase
- Global listeners remain active during date navigation
- Changes persist across date views
- Tutoring sessions are stored in `/tutoringSessions` with explicit `date` and `subject` fields for filtering
- Subject filtering is applied at render time, not in Firebase queries

**Adding New Features:**
- Inline all code in `index.html` - no external files
- Use Firebase's `push().key` for generating unique IDs
- Follow existing patterns: state object → Firebase listener → render function
- Add to `checkNotifications()` if feature should trigger badge

**Firebase Operations:**
- Use `.set()` to overwrite entire object
- Use `.update()` to patch specific fields
- Use `.remove()` or `set(null)` to delete
- Always add `.catch()` error handlers

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

### Common Modifications

**Adding a New Task Section:**
1. Define state object (e.g., `let newSection = {}`)
2. Add render function (e.g., `renderNewSection()`)
3. Add CRUD functions (add/delete/toggle)
4. Add Firebase listener in `attachListenersForDate()` or `attachGlobalListenersOnce()`
5. Add HTML section in the main card area
6. Update notification system if needed

**Modifying Date Scope:**
- To make date-scoped data global: Move from `attachListenersForDate()` to `attachGlobalListenersOnce()`
- To make global data date-scoped: Change Firebase path to include `currentViewDate`, move listener setup

**Changing Vocab Scheduler Logic:**
- Overlap calculation: `index.html:1206-1217`
- Date calculation: `vocabNextDates()` at `index.html:1041-1049`
- Range progression: Look for `nextStart` and `nextEnd` calculations
- Reschedule logic: `vocabRescheduleEvent()` at `index.html:1532-1570`

**Working with Reschedule Feature:**
- Reschedule automatically finds next date based on rule's weekdays
- Tracks original date to preserve scheduling history
- Visual indicators (orange badge) help identify rescheduled tests
- Can reschedule already-rescheduled tests (preserves original date)
- Confirmation dialog shows old and new dates in Korean format

## Important Notes

- Korean language UI - all user-facing text is in Korean
- Mobile-responsive via Tailwind's `md:` breakpoints
- No build process - direct file execution
- Firebase credentials are embedded (consider environment variables for production)
- No authentication system - shared access model
- **Subject-based tutoring system**: Unified system for Math and English with filtering and analytics
- **Multi-subject analytics**: Provides comprehensive data analysis with subject filtering for tracking student progress
- **Vocab test reschedule**: Handles student absences by postponing tests to next scheduled date automatically
- Global data (tutoring sessions, retests, vocab) persists across all date views for comprehensive tracking
- Easy to extend to additional subjects by adding options to `tutoringSubject` dropdown and updating color schemes
- Rescheduled tests maintain audit trail with original date tracking
