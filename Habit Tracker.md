
```datacorejsx
// Constants and color configuration
const HABITS = [
  { id: 'reading', emoji: '📚', label: 'Pages', defaultDuration: 25, unit: 'pages' },
  { id: 'meditation', emoji: '🧘‍♂️', label: 'Meditation', defaultDuration: 20, unit: 'minutes' },
  { id: 'weightlifting', emoji: '🏋️', label: 'Weight Lifting', defaultDuration: 45, unit: 'minutes' },
  { id: 'run', emoji: '🏃‍♂️', label: 'Run', defaultDuration: 2, unit: 'miles' },
  { id: 'writing', emoji: '✍️', label: 'Writing', defaultDuration: 30, unit: 'minutes' },
  { id: 'sleep', emoji: '😴', label: 'Sleep', defaultDuration: 8, unit: 'hours' }
];

const GOALS = {
  perfectDays: {
    monthly: 20,
    yearly: 250
  }
};

const DAYS = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'];

// Utility Functions
const formatMetricValue = (value, habit) => {
  switch(habit.unit) {
    case 'minutes':
      return value === 1 ? '1 Minute' : `${value} Minutes`;
    case 'miles':
      return value === 1 ? '1 Mile' : `${value} Miles`;
    case 'pages':
      return value === 1 ? '1 Page' : `${value} Pages`;
    case 'hours':
      return value === 1 ? '1 Hour' : `${value} Hours`;
    default:
      return value;
  }
};

const calculateTrendPercentage = (current, previous) => {
  if (previous === 0) return current > 0 ? 100 : 0;
  return ((current - previous) / previous) * 100;
};

const getCompletionColorClass = (percentage) => {
  if (percentage >= 75) return 'high';
  if (percentage >= 50) return 'medium';
  return 'low';
};

// Base Components
const CircularProgress = ({ value, size, color = 'var(--interactive-accent)' }) => {
  const radius = (size - 8) / 2;
  const circumference = 2 * Math.PI * radius;
  const progress = ((100 - value) / 100) * circumference;

  return (
    <svg width={size} height={size} style={{ transform: 'rotate(-90deg)' }}>
      <circle
        cx={size / 2}
        cy={size / 2}
        r={radius}
        stroke="var(--background-modifier-border)"
        strokeWidth="4"
        fill="none"
      />
      <circle
        cx={size / 2}
        cy={size / 2}
        r={radius}
        stroke={color}
        strokeWidth="4"
        strokeDasharray={circumference}
        strokeDashoffset={progress}
        fill="none"
        style={{ transition: 'stroke-dashoffset 0.5s ease' }}
      />
    </svg>
  );
};

const TrendIndicator = ({ current, previous }) => {
  const trend = calculateTrendPercentage(current, previous);
  let indicatorClass = 'trend-indicator-neutral';
  let indicator = '→';
  
  if (trend > 0) {
    indicatorClass = 'trend-indicator-up';
    indicator = '↑';
  } else if (trend < 0) {
    indicatorClass = 'trend-indicator-down';
    indicator = '↓';
  }

  return (
    <span className={indicatorClass}>
      {indicator} {Math.abs(trend).toFixed(1)}%
    </span>
  );
};

const TimeInput = ({ 
  entry, 
  habitId, 
  editingTime, 
  setEditingTime, 
  updateHabitDuration, 
  getHabitStatus, 
  getHabitDuration 
}) => {
  const duration = getHabitDuration(entry, habitId);
  const isEditing = editingTime?.entryPath === entry.$path && editingTime?.habitId === habitId;
  
  if (!getHabitStatus(entry, habitId)) return null;
  
  if (isEditing) {
    return (
      <input
        type="number"
        defaultValue={duration}
        min="0"
        className="time-input"
        onBlur={(e) => updateHabitDuration(entry, habitId, e.target.value)}
        autoFocus
      />
    );
  }

  return (
    <span 
      onClick={() => setEditingTime({ entryPath: entry.$path, habitId })}
      className="time-display"
    >
      {formatMetricValue(duration, HABITS.find(h => h.id === habitId))}
    </span>
  );
};

const ProgressBar = ({ value, max, colorClass = 'high' }) => {
  const percentage = Math.min((value / max) * 100, 100);
  return (
    <div className="progress-bar-container">
      <div 
        className={`progress-bar-fill ${colorClass}`}
        style={{ width: `${percentage}%` }}
      />
    </div>
  );
};

const StyledCard = ({ children }) => (
  <div className="styled-card">
    {children}
  </div>
);

const ActionButton = ({ icon, label, onClick, isActive }) => (
  <button
    onClick={onClick}
    className={`action-button ${isActive ? 'active' : 'inactive'}`}
  >
    <span style={{ fontSize: '20px' }}>{icon}</span>
    {label && <span>{label}</span>}
  </button>
);

const NavigationControls = ({ selectedDate, navigateDate }) => (
  <div className="navigation-controls">
    <div className="navigation-buttons">
      <ActionButton
        icon="←"
        onClick={() => navigateDate(-1)}
        isActive={true}
      />
      <div className="navigation-date">
        {selectedDate.toFormat('MMMM dd, yyyy')}
      </div>
      <ActionButton
        icon="→"
        onClick={() => navigateDate(1)}
        isActive={true}
      />
    </div>
  </div>
);

const CalendarView = ({ 
  selectedDate, 
  sortedNotes, 
  getHabitStatus, 
  calculateCompletedHabits,
  updateHabit,
  getHabitDuration,
  editingTime,
  setEditingTime,
  updateHabitDuration
}) => {
  const dates = [];
  let currentDate = selectedDate;
  
  for (let i = 0; i < 6; i++) {
    dates.push(currentDate.minus({ days: i }));
  }

  const rows = [];
  for (let i = 0; i < dates.length; i += 3) {
    rows.push(dates.slice(i, i + 3));
  }

  const notesMap = new Map(sortedNotes.map(note => [note.$name, note]));
  const today = dc.luxon.DateTime.now().startOf('day');

  return (
    <div className="calendar-view">
      {rows.map((row, rowIndex) => (
        <div key={rowIndex} className="calendar-row">
          {row.map((date) => {
            const dateStr = date.toFormat('yyyy-MM-dd');
            const entry = notesMap.get(dateStr);
            const isSelected = date.hasSame(selectedDate, 'day');
            const isToday = date.hasSame(today, 'day');

            return (
              <CalendarDayCard
                key={dateStr}
                date={date}
                entry={entry}
                getHabitStatus={getHabitStatus}
                calculateCompletedHabits={calculateCompletedHabits}
                isSelected={isSelected}
                isToday={isToday}
                updateHabit={updateHabit}
                getHabitDuration={getHabitDuration}
                editingTime={editingTime}
                setEditingTime={setEditingTime}
                updateHabitDuration={updateHabitDuration}
              />
            );
          })}
        </div>
      ))}
    </div>
  );
};

const CalendarDayCard = ({ 
  date, 
  entry, 
  getHabitStatus, 
  calculateCompletedHabits, 
  isSelected, 
  isToday,
  updateHabit,
  getHabitDuration,
  editingTime,
  setEditingTime,
  updateHabitDuration
}) => {
  const completedCount = calculateCompletedHabits(entry);
  const completionPercentage = entry ? Math.round((completedCount / HABITS.length) * 100) : 0;
  const colorClass = getCompletionColorClass(completionPercentage);

  return (
    <div className={`calendar-day-card ${isSelected ? 'selected' : ''}`}>
      <div className="calendar-day-header">
        <span className="calendar-day-weekday">
          {DAYS[date.weekday % 7]}
        </span>
        <span className="calendar-day-date">
          {date.toFormat('MM-dd')}
        </span>
      </div>

      {entry && (
        <>
          <div className="habits-grid">
            {HABITS.map(habit => {
              const isCompleted = getHabitStatus(entry, habit.id);
              const duration = getHabitDuration(entry, habit.id);
              
              return (
                <div
                  key={habit.id}
                  onClick={() => updateHabit(entry, habit.id)}
                  className={`habit-item ${isCompleted ? 'completed' : 'incomplete'}`}
                >
                  <span className="habit-emoji">
                    {habit.emoji}
                  </span>
                  <span className={`habit-label ${isCompleted ? 'completed' : 'incomplete'}`}>
                    {habit.label}
                  </span>
                  {isCompleted && duration && (
                    <span className={`habit-duration ${isCompleted ? 'completed' : 'incomplete'}`}>
                      {formatMetricValue(duration, habit)}
                    </span>
                  )}
                </div>
              );
            })}
          </div>

          <div className="progress-section">
            <ProgressBar
              value={completedCount}
              max={HABITS.length}
              colorClass={colorClass}
            />
            <div className="progress-percentage" style={{ color: `var(--color-${colorClass === 'high' ? 'green' : colorClass === 'medium' ? 'yellow' : 'red'})` }}>
              {completionPercentage}%
            </div>
          </div>
        </>
      )}
    </div>
  );
};

const MetricCard = ({ habit, current, previous, ytdTotal }) => {
  const trend = calculateTrendPercentage(current, previous);
  const formattedTotal = formatMetricValue(current, habit);
  const formattedYTD = formatMetricValue(ytdTotal, habit);

  return (
    <div className="metric-card">
      <div className="metric-header">
        <div className="metric-icon">
          {habit.emoji}
        </div>
        <div>
          <h3 className="metric-title">{habit.label}</h3>
          <div className="metric-subtitle">
            Last 30 Days
          </div>
        </div>
      </div>

      <div className="metric-values">
        <div className="metric-value-box">
          <div className="metric-value-label">Current</div>
          <div className="metric-value-number">
            {formattedTotal}
          </div>
        </div>

        <div className="metric-value-box">
          <div className="metric-value-label">YTD</div>
          <div className="metric-value-number">
            {formattedYTD}
          </div>
        </div>
      </div>

      <div className="metric-trend">
        <span>Trend</span>
        <TrendIndicator current={current} previous={previous} />
      </div>
    </div>
  );
};

const TrendsView = ({ trends }) => {
  const monthlyProgress = (trends.currentMonth.perfectDays / GOALS.perfectDays.monthly) * 100;
  const yearlyProgress = (trends.yearToDate.perfectDays / GOALS.perfectDays.yearly) * 100;

  return (
    <div className="trends-view">
      <div className="trends-goals">
        <div className="goal-card">
          <div className="circular-progress-wrapper">
           <CircularProgress value={monthlyProgress} size={100} color="var(--interactive-accent)" />
          </div>
          <div className="goal-info">
            <h3>Monthly Goal</h3>
            <div className="goal-value">
              {trends.currentMonth.perfectDays}/{GOALS.perfectDays.monthly} Perfect Days
            </div>
            <div className="goal-progress">
              {monthlyProgress.toFixed(1)}% Complete
            </div>
          </div>
        </div>

        <div className="goal-card">
          <div className="circular-progress-wrapper">
           <CircularProgress value={yearlyProgress} size={100} color="var(--color-green)" />
          </div>
          <div className="goal-info">
            <h3>Yearly Goal</h3>
            <div className="goal-value">
              {trends.yearToDate.perfectDays}/{GOALS.perfectDays.yearly} Perfect Days
            </div>
            <div className="goal-progress">
              {yearlyProgress.toFixed(1)}% Complete
            </div>
          </div>
        </div>

        <div className="goal-card">
          <div className="circular-progress-wrapper">
           <div className="streak-icon">
            🔥
           </div>
	      </div>
          <div className="goal-info">
            <h3>Current Streak</h3>
            <div className="goal-value">
              {trends.last30Days.perfectDays} Days
            </div>
            <div className="goal-progress">
              Last 30 Days
            </div>
          </div>
        </div>
      </div>

      <div className="trends-metrics">
        {HABITS.map(habit => (
          <MetricCard
            key={habit.id}
            habit={habit}
            current={trends.last30Days.habitMetrics[habit.id].total}
            previous={trends.last30Days.habitMetrics[habit.id].previousPeriodTotal}
            ytdTotal={trends.yearToDate.habitMetrics[habit.id].total}
          />
        ))}
      </div>
    </div>
  );
};

const HistoricalView = ({ 
  sortedNotes, 
  currentPage, 
  setCurrentPage,
  updateHabit,
  getHabitStatus,
  getHabitDuration,
  editingTime,
  setEditingTime,
  updateHabitDuration,
  calculateCompletedHabits
}) => {
  const itemsPerPage = 20;
  const totalPages = Math.ceil(sortedNotes.length / itemsPerPage);
  const startIndex = currentPage * itemsPerPage;
  const displayNotes = sortedNotes.slice(startIndex, startIndex + itemsPerPage);

  return (
    <div className="historical-view">
      <h3 className="historical-title">Historical Data</h3>
      
      <div className="table-container">
        <table className="habits-table">
          <thead>
            <tr>
              <th>Date</th>
              {HABITS.map(habit => (
                <th key={habit.id} className="center">
                  <div style={{ fontSize: '1.4em' }}>{habit.emoji}</div>
                  <div>{habit.label}</div>
                </th>
              ))}
              <th className="center">Completion</th>
            </tr>
          </thead>
          <tbody>
            {displayNotes.map((entry, index) => (
              <tr key={entry.$path} className={index % 2 === 0 ? 'table-row-even' : 'table-row-odd'}>
                <td>{entry.$name}</td>
                {HABITS.map(habit => {
                  const isCompleted = getHabitStatus(entry, habit.id);
                  return (
                    <td key={habit.id} className="center">
                      <div className="habit-cell">
                        <div
                          onClick={() => updateHabit(entry, habit.id)}
                          className={`habit-toggle ${isCompleted ? 'completed' : 'incomplete'}`}
                        >
                          {isCompleted ? '✓' : '×'}
                        </div>
                        {isCompleted && (
                          <TimeInput
                            entry={entry}
                            habitId={habit.id}
                            editingTime={editingTime}
                            setEditingTime={setEditingTime}
                            updateHabitDuration={updateHabitDuration}
                            getHabitStatus={getHabitStatus}
                            getHabitDuration={getHabitDuration}
                          />
                        )}
                      </div>
                    </td>
                  );
                })}
                <td className="center">
                  {Math.round((calculateCompletedHabits(entry) / HABITS.length) * 100)}%
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>

      <div className="pagination-controls">
        <ActionButton
          icon="←"
          onClick={() => setCurrentPage(prev => Math.max(0, prev - 1))}
          isActive={currentPage !== 0}
        />
        <span className="pagination-page">
          Page {currentPage + 1} of {totalPages}
        </span>
        <ActionButton
          icon="→"
          onClick={() => setCurrentPage(prev => Math.min(totalPages - 1, prev + 1))}
          isActive={currentPage !== totalPages - 1}
        />
      </div>
    </div>
  );
};

function HabitTracker() {
  // State Management
  const [selectedDate, setSelectedDate] = dc.useState(dc.luxon.DateTime.now());
  const [activeView, setActiveView] = dc.useState(null);
  const [editingTime, setEditingTime] = dc.useState(null);
  const [currentPage, setCurrentPage] = dc.useState(0);

  // Data Queries and Utility Functions
  const dailyNotes = dc.useQuery(`
    @page 
    AND path("Notes/Daily Notes")
  `);

  const sortedNotes = dc.useMemo(() => {
    return [...dailyNotes].sort((a, b) => b.$name.localeCompare(a.$name));
  }, [dailyNotes]);

  const getNotesForPeriod = (startDate) => {
    return sortedNotes.filter(note => {
      const noteDate = dc.luxon.DateTime.fromISO(note.$name);
      return noteDate >= startDate;
    });
  };

  const last30DaysNotes = dc.useMemo(() => 
    getNotesForPeriod(selectedDate.minus({ days: 30 })), 
    [sortedNotes, selectedDate]
  );

  const yearToDateNotes = dc.useMemo(() => 
    getNotesForPeriod(selectedDate.startOf('year')),
    [sortedNotes, selectedDate]
  );

  const currentMonthNotes = dc.useMemo(() => 
    getNotesForPeriod(selectedDate.startOf('month')),
    [sortedNotes, selectedDate]
  );

  const previousMonthNotes = dc.useMemo(() => 
    sortedNotes.filter(note => {
      const noteDate = dc.luxon.DateTime.fromISO(note.$name);
      const monthAgo = selectedDate.minus({ months: 1 });
      return noteDate >= monthAgo && noteDate < selectedDate.startOf('month');
    }),
    [sortedNotes, selectedDate]
  );

  const getHabitStatus = (entry, habitId) => {
    const habits = entry?.value('habits');
    
    if (Array.isArray(habits)) {
      const habitItem = habits.find(item => item[habitId] !== undefined);
      return habitItem ? habitItem[habitId] : false;
    }
    
    return habits?.[habitId] ?? false;
  };

  const getHabitDuration = (entry, habitId) => {
    const habits = entry?.value('habits');
    
    if (Array.isArray(habits)) {
      const durationKey = `${habitId}_duration`;
      const durationItem = habits.find(item => item[durationKey] !== undefined);
      return durationItem ? durationItem[durationKey] : null;
    }
    
    return habits?.[`${habitId}_duration`] ?? null;
  };

  const calculateCompletedHabits = (entry) => {
    if (!entry) return 0;
    return HABITS.reduce((count, habit) => 
      count + (getHabitStatus(entry, habit.id) ? 1 : 0), 0);
  };

  const calculatePerfectDays = (notes) => {
    return notes.reduce((count, note) => 
      count + (calculateCompletedHabits(note) === HABITS.length ? 1 : 0), 0);
  };

  const calculateTrends = () => {
    const trends = {
      last30Days: {
        perfectDays: calculatePerfectDays(last30DaysNotes),
        habitMetrics: {}
      },
      yearToDate: {
        perfectDays: calculatePerfectDays(yearToDateNotes),
        habitMetrics: {}
      },
      currentMonth: {
        perfectDays: calculatePerfectDays(currentMonthNotes),
        progress: 0
      }
    };

    trends.currentMonth.progress = (trends.currentMonth.perfectDays / GOALS.perfectDays.monthly) * 100;

    HABITS.forEach(habit => {
      const last30Total = last30DaysNotes.reduce((sum, note) => 
        sum + (getHabitDuration(note, habit.id) || 0), 0);
      
      const ytdTotal = yearToDateNotes.reduce((sum, note) => 
        sum + (getHabitDuration(note, habit.id) || 0), 0);

      const previousMonthTotal = previousMonthNotes.reduce((sum, note) => 
        sum + (getHabitDuration(note, habit.id) || 0), 0);

      trends.last30Days.habitMetrics[habit.id] = {
        total: last30Total,
        previousPeriodTotal: previousMonthTotal
      };

      trends.yearToDate.habitMetrics[habit.id] = {
        total: ytdTotal
      };
    });

    return trends;
  };

  async function updateHabit(entry, habitId) {
    const file = app.vault.getAbstractFileByPath(entry.$path);
    await app.fileManager.processFrontMatter(file, (frontmatter) => {
      if (!frontmatter.habits) frontmatter.habits = [];
      
      if (!Array.isArray(frontmatter.habits)) {
        const oldHabits = frontmatter.habits;
        frontmatter.habits = [];
        for (const [key, value] of Object.entries(oldHabits)) {
          frontmatter.habits.push({ [key]: value });
        }
      }
      
      const existingIndex = frontmatter.habits.findIndex(item => item[habitId] !== undefined);
      const currentStatus = existingIndex !== -1 ? frontmatter.habits[existingIndex][habitId] : false;
      const newStatus = !currentStatus;
      
      if (newStatus) {
        if (existingIndex !== -1) {
          frontmatter.habits[existingIndex][habitId] = true;
        } else {
          frontmatter.habits.push({ [habitId]: true });
        }
        
        const habit = HABITS.find(h => h.id === habitId);
        const durationKey = `${habitId}_duration`;
        const durationIndex = frontmatter.habits.findIndex(item => item[durationKey] !== undefined);
        
        if (durationIndex === -1) {
          frontmatter.habits.push({ [durationKey]: habit.defaultDuration });
        }
      } else {
        if (existingIndex !== -1) {
          frontmatter.habits[existingIndex][habitId] = false;
        } else {
          frontmatter.habits.push({ [habitId]: false });
        }
      }
    });
  }

  async function updateHabitDuration(entry, habitId, duration) {
    const file = app.vault.getAbstractFileByPath(entry.$path);
    await app.fileManager.processFrontMatter(file, (frontmatter) => {
      if (!frontmatter.habits) frontmatter.habits = [];
      
      if (!Array.isArray(frontmatter.habits)) {
        const oldHabits = frontmatter.habits;
        frontmatter.habits = [];
        for (const [key, value] of Object.entries(oldHabits)) {
          frontmatter.habits.push({ [key]: value });
        }
      }
      
      const durationKey = `${habitId}_duration`;
      const existingIndex = frontmatter.habits.findIndex(item => item[durationKey] !== undefined);
      const parsedDuration = parseInt(duration) || 0;
      
      if (existingIndex !== -1) {
        frontmatter.habits[existingIndex][durationKey] = parsedDuration;
      } else {
        frontmatter.habits.push({ [durationKey]: parsedDuration });
      }
    });
    setEditingTime(null);
  }

  const navigateDate = (direction) => {
    setSelectedDate(prev => prev.plus({ days: direction }));
  };

  return (
    <div className="habit-tracker-container">
      <NavigationControls
        selectedDate={selectedDate}
        navigateDate={navigateDate}
        activeView={activeView}
        setActiveView={setActiveView}
      />
      
      <StyledCard>
        <CalendarView
          selectedDate={selectedDate}
          sortedNotes={sortedNotes.slice(0, 6)}
          getHabitStatus={getHabitStatus}
          calculateCompletedHabits={calculateCompletedHabits}
          updateHabit={updateHabit}
          getHabitDuration={getHabitDuration}
          editingTime={editingTime}
          setEditingTime={setEditingTime}
          updateHabitDuration={updateHabitDuration}
        />
        
        <div className="button-group">
          <ActionButton
            icon="📊"
            onClick={() => setActiveView(activeView === 'stats' ? null : 'stats')}
            isActive={activeView === 'stats'}
          />
          <ActionButton
            icon="📚"
            onClick={() => setActiveView(activeView === 'history' ? null : 'history')}
            isActive={activeView === 'history'}
          />
        </div>
      </StyledCard>

      {activeView === 'stats' && <TrendsView trends={calculateTrends()} />}
      {activeView === 'history' && (
        <HistoricalView
          sortedNotes={sortedNotes}
          currentPage={currentPage}
          setCurrentPage={setCurrentPage}
          updateHabit={updateHabit}
          getHabitStatus={getHabitStatus}
          getHabitDuration={getHabitDuration}
          editingTime={editingTime}
          setEditingTime={setEditingTime}
          updateHabitDuration={updateHabitDuration}
          calculateCompletedHabits={calculateCompletedHabits}
        />
      )}
    </div>
  );
}

return HabitTracker;
```
