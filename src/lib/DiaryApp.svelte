<script>
    import { format, startOfWeek, addDays, isToday, addWeeks, subWeeks } from 'date-fns';
    import { hu } from 'date-fns/locale';
    import { onMount } from 'svelte';
  
    let selectedDate = new Date();
    let currentWeek = new Date();
    let entries = {};
  
    onMount(() => {
      const savedEntries = localStorage.getItem('diaryEntries');
      if (savedEntries) {
        entries = JSON.parse(savedEntries);
      }
    });
  
    $: {
      localStorage.setItem('diaryEntries', JSON.stringify(entries));
    }
  
    $: weekDays = generateWeekDays();
    $: currentWeekLabel = generateWeekLabel();
  
    function generateWeekDays() {
      const weekStart = startOfWeek(selectedDate, { locale: hu });
      return Array.from({ length: 7 }, (_, index) => {
        const date = addDays(weekStart, index);
        return {
          date,
          formattedDate: format(date, 'yyyy-MM-dd'),
          dayName: format(date, 'EEEE', { locale: hu }),
          dayNumber: format(date, 'd')
        };
      });
    }
  
    function generateWeekLabel() {
      const weekStart = startOfWeek(selectedDate, { locale: hu });
      const weekEnd = addDays(weekStart, 6);
      return `${format(weekStart, 'yyyy. MMMM d.', { locale: hu })} - ${format(weekEnd, 'yyyy. MMMM d.', { locale: hu })}`;
    }
  
    function handleEntryChange(date, event) {
      const content = event.target.value;
      entries = {
        ...entries,
        [date]: content
      };
    }
  
    function previousWeek() {
  currentWeek = subWeeks(currentWeek, 1);
  selectedDate = startOfWeek(currentWeek, { locale: hu });
  weekDays = generateWeekDays();
}

function nextWeek() {
  currentWeek = addWeeks(currentWeek, 1);
  selectedDate = startOfWeek(currentWeek, { locale: hu });
  weekDays = generateWeekDays();
}


  
    function goToCurrentWeek() {
        currentWeek = new Date();
      selectedDate = new Date();
    }
  </script>
  
  <div class="max-w-4xl mx-auto p-4">
    <div class="card">
      <h1 class="text-2xl font-bold mb-4">Személyes Napló</h1>
      
      <!-- Hét választó -->
      <div class="flex items-center justify-between mb-4">
        <button class="nav-button" on:click={previousWeek}>
          ← Előző hét
        </button>
        <button class="nav-button" on:click={nextWeek}>
            Következő hét →
          </button>
        <div class="flex gap-2 items-center">
          <span class="font-medium">{currentWeekLabel}</span>
          <button class="current-week-button" on:click={goToCurrentWeek}>
            Mai hét
          </button>
        </div>
        
      </div>
      
      <!-- Kalendárium -->
      <div class="calendar-container mb-6">
        {#each weekDays as { date, formattedDate, dayName, dayNumber }}
          <div
            class="day-card"
            class:today={isToday(date)}
            class:selected={formattedDate === format(selectedDate, 'yyyy-MM-dd')}
            on:click={() => selectedDate = date}
          >
            <div class="day-name">{dayName}</div>
            <div class="day-number">{dayNumber}</div>
          </div>
        {/each}
      </div>
  
      <!-- Naplóbejegyzés -->
      <div>
        <h3 class="text-lg font-semibold mb-2">
          Bejegyzés: {format(selectedDate, 'yyyy. MMMM d., EEEE', { locale: hu })}
        </h3>
        <textarea
          class="w-full min-h-[200px] p-2 border rounded"
          placeholder="Írd ide a mai nap történéseit..."
          value={entries[format(selectedDate, 'yyyy-MM-dd')] || ''}
          on:input={(e) => handleEntryChange(
            format(selectedDate, 'yyyy-MM-dd'),
            e
          )}
        />
      </div>
    </div>
  </div>
  
  <style>
    .card {
    background-color: white;
    padding: 1.5rem;
    border-radius: 0.5rem;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  }

  .calendar-container {
    display: flex;
    justify-content: space-between;
    gap: 0.5rem;
    padding: 0.5rem;
    background-color: #f8fafc;
    border-radius: 0.5rem;
  }

  .day-card {
    flex: 1;
    min-width: 0;
    padding: 0.75rem 0.5rem;
    background-color: white;
    border: 1px solid #e2e8f0;
    border-radius: 0.5rem;
    cursor: pointer;
    transition: all 0.2s;
  }

  .day-card:hover {
    background-color: #e0f2fe; /* Világoskék */
  transform: translateY(-3px) scale(1.03);
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
  }

  .day-name {
    font-size: 0.875rem;
    color: #64748b;
    margin-bottom: 0.25rem;
    text-align: center;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .day-number {
    font-size: 1.25rem;
    font-weight: 600;
    text-align: center;
  }

  .today {
    background-color: #fef08a; /* Világossárga */
  border-color: #facc15;
  border-width: 2px;
  font-weight: bold;
  }

  .selected {
    border-color: #57097c;
    border-width: 2px;
    background-color: #f6efff;
  }

  .nav-button {
    padding: 0.5rem 1rem;
    background-color: #a2a0cb;
    color: white;
    border-radius: 0.25rem;
    font-weight: 500;
    transition: all 0.3s ease;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
  }

  .nav-button:hover {
    background-color: #e5e7eb;
  }

  .nav-button, .current-week-button {
  color: #1e3a8a; /* Sötétkék */
}

  .current-week-button {
    padding: 0.25rem 0.75rem;
    background-color: #b9dcc7;
    border-radius: 0.25rem;
    font-size: 0.875rem;
    transition: background-color 0.2s;
  }

  .current-week-button:hover {
    background-color: #dbe7fd;
  }

  textarea {
    min-width: 600;
    font-family: inherit;
    resize: vertical;
    border-radius: 0.25rem;
  }
  </style>