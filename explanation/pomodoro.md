# Pomodoro Module

This document provides a detailed explanation of the `pomodoro.py` file, which implements a Pomodoro timer with work sessions and breaks. It supports pausing, resuming, and notifications.

## Overview
The `PomodoroTimer` class is the core component of this module. It manages the Pomodoro technique, which involves alternating work sessions and breaks to improve focus and productivity.

## Code Explanation

### Imports
```python
import time
import threading
from datetime import datetime, timedelta
from plyer import notification
import config
```
- **time**: Used for time-related functions, like getting the current time.
- **threading**: Allows the program to run multiple operations concurrently.
- **datetime, timedelta**: Provides functions to manipulate dates and times.
- **plyer.notification**: Used to send desktop notifications.
- **config**: A custom module containing configuration settings like Pomodoro durations.

### Class Definition
```python
class PomodoroTimer:
    """
    Implements a Pomodoro timer with work sessions and breaks.
    Supports pausing, resuming, and notifications.
    """
```
- **PomodoroTimer**: A class that encapsulates all functionality related to the Pomodoro timer.

### Initialization
```python
    def __init__(self):
        self.work_duration = config.POMODORO_WORK_DURATION * 60  # Convert to seconds
        self.short_break_duration = config.POMODORO_SHORT_BREAK_DURATION * 60
        self.long_break_duration = config.POMODORO_LONG_BREAK_DURATION * 60
        self.cycles_before_long_break = config.POMODORO_CYCLES_BEFORE_LONG_BREAK
        
        self.current_cycle = 0
        self.time_remaining = self.work_duration
        self.current_phase = "Ready"
        self.is_running = False
        self.is_paused = False
        self.timer_thread = None
        
        # Callback functions
        self.on_tick = None  # Called every second with time update
        self.on_phase_change = None  # Called when phase changes (work → break)
        self.on_complete = None  # Called when a full pomodoro cycle completes
        
        print("Pomodoro timer initialized")
```
- **__init__**: The constructor method initializes the class attributes and sets up the environment for the Pomodoro timer.
- **self.work_duration, self.short_break_duration, self.long_break_duration**: Store the durations for work sessions and breaks in seconds.
- **self.cycles_before_long_break**: The number of work cycles before a long break.
- **self.current_cycle, self.time_remaining, self.current_phase**: Track the current state of the timer.
- **self.is_running, self.is_paused**: Flags to control the timer's state.
- **self.timer_thread**: A thread for running the timer loop.
- **self.on_tick, self.on_phase_change, self.on_complete**: Callback functions for various events.
- **print**: Outputs a message indicating that the Pomodoro timer is initialized.

### Start Timer
```python
    def start(self):
        """Start the Pomodoro timer"""
        if self.is_running:
            print("Timer already running")
            return
        
        self.is_running = True
        self.is_paused = False
        self.current_phase = "Work"
        self.time_remaining = self.work_duration
        
        # Start timer in a separate thread
        self.timer_thread = threading.Thread(target=self._timer_loop)
        self.timer_thread.daemon = True
```
- **start**: Begins the Pomodoro timer.
- **if self.is_running**: Checks if the timer is already running.
- **self.is_running, self.is_paused, self.current_phase, self.time_remaining**: Sets the initial state for the timer.
- **threading.Thread**: Creates a new thread to run the timer loop.
- **self.timer_thread.daemon**: Sets the thread as a daemon, meaning it will exit when the main program exits.

### Pause Timer
```python
    def pause(self):
        """Pause the Pomodoro timer"""
        if self.is_running and not self.is_paused:
            self.is_paused = True
            print("Timer paused")
```
- **pause**: Pauses the Pomodoro timer.
- **if self.is_running and not self.is_paused**: Checks if the timer is running and not already paused.
- **self.is_paused**: Sets the pause flag.
- **print**: Outputs a message indicating that the timer is paused.

### Resume Timer
```python
    def resume(self):
        """Resume the Pomodoro timer"""
        if self.is_running and self.is_paused:
            self.is_paused = False
            print("Timer resumed")
```
- **resume**: Resumes the Pomodoro timer.
- **if self.is_running and self.is_paused**: Checks if the timer is running and paused.
- **self.is_paused**: Resets the pause flag.
- **print**: Outputs a message indicating that the timer is resumed.

### Stop Timer
```python
    def stop(self):
        """Stop the Pomodoro timer"""
        if self.is_running:
            self.is_running = False
            self.is_paused = False
            self.current_phase = "Ready"
            self.time_remaining = self.work_duration
            print("Timer stopped")
```
- **stop**: Stops the Pomodoro timer.
- **if self.is_running**: Checks if the timer is running.
- **self.is_running, self.is_paused, self.current_phase, self.time_remaining**: Resets the timer state.
- **print**: Outputs a message indicating that the timer is stopped.

### Timer Loop
```python
    def _timer_loop(self):
        """Main loop for the Pomodoro timer"""
        while self.is_running:
            if not self.is_paused:
                if self.time_remaining > 0:
                    self.time_remaining -= 1
                    if self.on_tick:
                        self.on_tick(self.time_remaining)
                else:
                    self._handle_phase_complete()
            time.sleep(1)
```
- **_timer_loop**: The main loop that decrements the timer and handles phase changes.
- **while self.is_running**: Runs the loop as long as the timer is active.
- **if not self.is_paused**: Checks if the timer is not paused.
- **if self.time_remaining > 0**: Decrements the time remaining if there is time left.
- **self.on_tick**: Calls the tick callback function if registered.
- **else**: Handles the completion of a phase if the time is up.
- **time.sleep(1)**: Pauses for a second before the next check.

### Handle Phase Complete
```python
    def _handle_phase_complete(self):
        """Handle the completion of a Pomodoro phase"""
        if self.current_phase == "Work":
            self.current_cycle += 1
            if self.current_cycle % self.cycles_before_long_break == 0:
                self.current_phase = "Long Break"
                self.time_remaining = self.long_break_duration
                self._notify("Long Break", "Time for a long break!")
            else:
                self.current_phase = "Short Break"
                self.time_remaining = self.short_break_duration
                self._notify("Short Break", "Time for a short break!")
        else:
            self.current_phase = "Work"
            self.time_remaining = self.work_duration
            self._notify("Work", "Time to work!")
        
        if self.on_phase_change:
            self.on_phase_change(self.current_phase)
```
- **_handle_phase_complete**: Manages the transition between work and break phases.
- **if self.current_phase == "Work"**: Checks if the current phase is a work session.
- **self.current_cycle += 1**: Increments the cycle counter.
- **if self.current_cycle % self.cycles_before_long_break == 0**: Checks if it's time for a long break.
- **self.current_phase, self.time_remaining**: Sets the phase and time for the next phase.
- **self._notify**: Sends a notification for the phase change.
- **if self.on_phase_change**: Calls the phase change callback function if registered.

### Notify
```python
    def _notify(self, title, message):
        """Send a desktop notification"""
        notification.notify(
            title=title,
            message=message,
            timeout=10
        )
```
- **_notify**: Sends a desktop notification with a title and message.
- **notification.notify**: Uses the plyer library to send a notification.

### Get Time Remaining String
```python
    def get_time_remaining_str(self):
        """Get the time remaining as a formatted string"""
        minutes = self.time_remaining // 60
        seconds = self.time_remaining % 60
        return f"{minutes:02d}:{seconds:02d}"
```
- **get_time_remaining_str**: Formats the time remaining as a string.
- **minutes, seconds**: Calculates the minutes and seconds from the time remaining.
- **return**: Returns the formatted time string.

### Get Progress
```python
    def get_progress(self):
        """Get the progress of the current phase as a percentage"""
        if self.current_phase == "Work":
            total = self.work_duration
        elif self.current_phase == "Short Break":
            total = self.short_break_duration
        elif self.current_phase == "Long Break":
            total = self.long_break_duration
        else:
            return 0
        
        return ((total - self.time_remaining) / total) * 100
```
- **get_progress**: Calculates the progress of the current phase as a percentage.
- **if self.current_phase == "Work"**: Determines the total duration based on the current phase.
- **return**: Returns the progress as a percentage.

This document should help someone with basic Python knowledge understand the functionality and purpose of each part of the `pomodoro.py` file.

---

Next, I'll proceed with creating an explanation for the `focus_score.py` file. Let me know if you want to continue with this or if there's anything specific you'd like to focus on.