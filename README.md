
# Obsidian Habit Tracker
[Englishi](https://github.com/ichris007/Obsidian-Habit-Tracker/) | [中文](https://github.com/ichris007/Obsidian-Habit-Tracker/blob/main/README-ZH.md)

A **habit tracker** running in the Obsidian Datacore plugin that saves daily habit records directly in the frontmatter of your daily notes, providing a visual tracking interface.

>Based on [this code](https://forum.obsidian.md/t/building-a-powerful-habit-tracker-in-obsidian-a-complete-guide/92884), the following modifications were made:
>
>- Split the JS and CSS code into separate files
>- Add dark/light theme adaptation
>- Change the habits frontmatter to array format while maintaining backward compatibility with the previous object format

---

## Features Overview

### Core Features

1. **Habit Check-in** - Click habit cards to toggle between completed/incomplete status
2. **Value Recording** - Record specific values for each habit (pages, minutes, miles, hours)
3. **Calendar View** - Display habit completion status for the last 6 days
4. **Statistics & Analytics** - View habit trends, monthly/yearly goal progress
5. **Historical Data** - Browse and edit all historical records

### Screenshots

<div align="center">
  
#### Calendar View
<img src="screenshot/Habit%20Tracker%201.jpg" alt="Calendar View showing 6 days of habits" width="600" />

#### Statistics View  
<img src="screenshot/Habit%20Tracker%202.jpg" alt="Statistics view with goals and metrics" width="600" />

#### Historical View
<img src="screenshot/Habit%20Tracker%203.jpg" alt="Historical data table with pagination" width="600" />

</div>


---

## Six Tracked Habits

| Habit | ID | Unit | Default Value |
|-------|-----|------|---------------|
| 📚 Reading | `reading` | pages | 25 pages |
| 🧘‍♂️ Meditation | `meditation` | minutes | 20 minutes |
| 🏋️ Weight Lifting | `weightlifting` | minutes | 45 minutes |
| 🏃‍♂️ Run | `run` | miles | 2 miles |
| ✍️ Writing | `writing` | minutes | 30 minutes |
| 😴 Sleep | `sleep` | hours | 8 hours |

### Goals

- **Monthly Perfect Days Goal**: 20 days (all habits completed)
- **Yearly Perfect Days Goal**: 250 days

---

## Data Storage Format

### File Location
```
Notes/Daily Notes/YYYY-MM-DD.md
```

### Frontmatter Format

```yaml
---
created: 2026-04-14
tags:
  - daily
habits:
  - reading: true
  - reading_duration: 25
  - meditation: true
  - meditation_duration: 20
  - weightlifting: true
  - weightlifting_duration: 45
  - writing: true
  - writing_duration: 30
  - run: true
  - run_duration: 2
  - sleep: true
  - sleep_duration: 8
---
```

### Data Description

- **Habit Status**: `true` means completed, `false` means incomplete
- **Value Fields**: Format is `{habitID}_duration`
- **Incomplete habits**: Can be omitted from frontmatter
- **Array format**: Habits are stored in array format (backward compatible with object format)

---

## Interface Features

### 1. Top Navigation Bar
- Displays current date
- Arrow buttons to switch dates
- Date format: `MMMM dd, yyyy` (e.g., April 14, 2026)

### 2. Calendar View (Main View)
- Shows last 6 days (today + previous 5 days)
- 3x2 grid layout
- Each card shows:
  - Day of week and date
  - 6 habit buttons (click to toggle)
  - Completion progress bar and percentage
- Selected date has highlighted border

### 3. Statistics View (📊 Button)
- **Goal Progress Cards**:
  - Monthly perfect days progress
  - Yearly perfect days progress
  - Current perfect day streak
- **Detailed Habit Statistics** (individual cards per habit):
  - Last 30 days total
  - Year-to-Date (YTD) total
  - Trend percentage (↑/↓)

### 4. Historical Data View (📚 Button)
- Table showing all daily notes
- 20 records per page with pagination
- Direct editing in table:
  - Click ✓/✗ to toggle habit status
  - Click values to modify specific amounts
- Displays daily completion percentage

### 5. Interactive Features
- **Toggle Habits**: Click habit cards to toggle completion status
- **Modify Values**: Click on completed habit values, enter new value, press Enter or blur to save
- **Auto-save**: All changes are immediately written to frontmatter

---

## Installation & Usage

### Installation Method 1: Download Habit Tracker Vault
<img src="screenshot/下载Habit%20Tracker%20Vault.jpg" alt="Calendar View showing 6 days of habits" width="600" />
After downloading the ZIP file, extract it and open it with Obsidian.

### Installation Method 2: Manual Configuration

#### Prerequisites

1. **Datacore plugin** installed and enabled
2. `Notes/Daily Notes/` folder exists

#### Steps

1. Create a `habit-tracker.css` file and paste the [CSS](https://github.com/ichris007/Obsidian-Habit-Tracker/blob/main/.obsidian/snippets/habit-tracker.css)  code into it, Put the CSS file in `.obsidian/snippets` and enable it via `Settings → Appearance → CSS snippets`
2. Create a `habit-tracker.md` file and paste the [JS](https://github.com/ichris007/Obsidian-Habit-Tracker/blob/main/Habit%20Tracker.md) code into it

````markdown
```datacorejsx
[paste js code]
```
````


### First Time Use

1. Ensure daily note files exist
2. Click habit cards to start checking in
3. Values will auto-populate with defaults
4. Click values to modify to actual amounts

---

## Customization

### Modify Habit List

Edit the `HABITS` constant:

```javascript
const HABITS = [
  { id: 'reading', emoji: '📚', label: 'Pages', defaultDuration: 25, unit: 'pages' },
  // Add new habit
  { id: 'newhabit', emoji: '🎯', label: 'New Habit', defaultDuration: 10, unit: 'minutes' }
];
```

### Modify Goals

Edit the `GOALS` constant:

```javascript
const GOALS = {
  perfectDays: {
    monthly: 20,  // Modify monthly goal
    yearly: 250   // Modify yearly goal
  }
};
```

### Modify Date Range

Change the loop count in `CalendarView` component:

```javascript
for (let i = 0; i < 6; i++) {  // 6 = number of days to display
  dates.push(currentDate.minus({ days: i }));
}
```

### Modify Items Per Page

In `HistoricalView` component:

```javascript
const itemsPerPage = 20;  // Change to desired number
```

---

## Theme Support

The code uses Obsidian's CSS variables and **automatically adapts** to dark/light themes:

- `var(--background-primary)` - Main background
- `var(--background-secondary)` - Secondary background
- `var(--text-normal)` - Normal text
- `var(--interactive-accent)` - Accent color
- `var(--color-green/red/yellow)` - Progress bar colors

When you switch Obsidian themes, the interface colors will automatically follow.

---

## Usage Tips

### Batch Edit Historical Data
Use the historical data view to quickly modify multiple days of habit records without opening each note file.

### View Yearly Trends
Check YTD data in the statistics view to understand annual habit completion.

### Perfect Day Tracking
Completing all 6 habits earns a "Perfect Day," automatically counted and displayed in goal progress.

### Value Recording
- Reading: Record actual pages read
- Exercise: Record duration (minutes)
- Running: Record mileage (miles)
- Sleep: Record hours

---

## Notes

1. **File Path**: Ensure daily notes are in `Notes/Daily Notes/` path
2. **File Naming**: Must use `YYYY-MM-DD` format
3. **Datacore Plugin**: Must be enabled and properly configured
4. **Array Format**: New data saves as array format, old data automatically compatible
5. **Performance**: Pagination provides smoother browsing with large amounts of historical data

---

## Sample Workflow

1. **Morning**: Open habit tracker, click completed habits (e.g., Sleep)
2. **Daytime**: After completing exercise, click and modify actual duration
3. **Evening**: Record reading pages, meditation duration, etc.
4. **Weekend**: Check statistics view, analyze weekly completion
5. **Month End**: Review monthly perfect days goal progress

---

## Troubleshooting

**Q: Clicking habits doesn't respond?**
A: Check if file path is correct and note files exist.

**Q: Modified values not saving?**
A: Press Enter or click elsewhere to blur the input field.

**Q: Historical data not showing?**
A: Check if daily note frontmatter format is correct.

**Q: Colors incorrect in dark mode?**
A: Ensure separate CSS file is used or code includes complete styles.

---

## License

Free to use, modify, and share.

---

**Enjoy your habit tracking journey!** 🎯
