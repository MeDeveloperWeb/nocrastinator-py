# Activity Tracker Module

This document provides a detailed explanation of the `activity_tracker.py` file, which is responsible for monitoring application and website usage, categorizing activities, and triggering alerts based on productivity.

## Overview
The `ActivityTracker` class is the core component of this module. It tracks user activity, logs usage time, and determines whether activities are productive or unproductive. It also manages alerts for excessive unproductive time.

## Code Explanation

### Imports
```python
import csv
import os
import time
import psutil
import win32gui
import win32process
from datetime import datetime
from plyer import notification
import threading
import config
```
- **csv**: Used for reading and writing activity logs in CSV format.
- **os**: Provides functions to interact with the operating system, such as creating directories.
- **time**: Used for time-related functions, like getting the current time.
- **psutil**: A library for retrieving information on system processes and system utilization.
- **win32gui** and **win32process**: Part of the PyWin32 library, used to interact with Windows GUI and processes.
- **datetime**: Provides functions to manipulate dates and times.
- **plyer.notification**: Used to send desktop notifications.
- **threading**: Allows the program to run multiple operations concurrently.
- **config**: A custom module containing configuration settings like app categorization and thresholds.

### Class Definition
```python
class ActivityTracker:
    """
    Tracks user activity, including applications and websites visited.
    Records usage time and categorizes activities as productive or unproductive.
    """
```
- **ActivityTracker**: A class that encapsulates all functionality related to tracking user activity.

### Initialization
```python
    def __init__(self):
        self.current_app = None
        self.current_window_title = None
        self.app_start_time = None
        
        # For unproductive time tracking
        self.unproductive_start_time = None
        self.total_unproductive_time = 0
        self.is_currently_unproductive = False
        self.alert_triggered = False
        self.last_productive_timestamp = time.time()
        
        self.on_unproductive_alert = None  # Callback for UI updates
        
        # Create data directory if it doesn't exist
        os.makedirs(config.DATA_DIRECTORY, exist_ok=True)
        
        # Create or load activity log file
        self.init_activity_log()
        
        # Thread for tracking activities
        self.tracking_thread = None
        self.is_tracking = False
        
        print("Activity tracker initialized")
```
- **__init__**: The constructor method initializes the class attributes and sets up the environment for tracking activities.
- **self.current_app**: Stores the name of the currently active application.
- **self.current_window_title**: Stores the title of the currently active window.
- **self.app_start_time**: Records the start time of the current app session.
- **self.unproductive_start_time**: Records the start time of unproductive activity.
- **self.total_unproductive_time**: Accumulates the total unproductive time.
- **self.is_currently_unproductive**: A flag indicating if the current activity is unproductive.
- **self.alert_triggered**: A flag to prevent repeated alerts for the same unproductive session.
- **self.last_productive_timestamp**: Records the last time a productive app was used.
- **self.on_unproductive_alert**: A callback function for UI updates when an alert is triggered.
- **os.makedirs**: Creates the data directory if it doesn't exist.
- **self.init_activity_log()**: Initializes the activity log file.
- **self.tracking_thread**: A thread for running the activity tracking loop.
- **self.is_tracking**: A flag to control the tracking loop.
- **print**: Outputs a message indicating that the tracker is initialized.

### Activity Log Initialization
```python
    def init_activity_log(self):
        """Initialize activity log file with headers if it doesn't exist"""
        if not os.path.exists(config.ACTIVITY_LOG_FILE):
            with open(config.ACTIVITY_LOG_FILE, 'w', newline='') as file:
                writer = csv.writer(file)
                writer.writerow([
                    'timestamp', 
                    'app_name', 
                    'window_title', 
                    'duration_seconds', 
                    'is_productive'
                ])
```
- **init_activity_log**: A method to create the activity log file with headers if it doesn't already exist.
- **os.path.exists**: Checks if the log file already exists.
- **open**: Opens the file in write mode.
- **csv.writer**: Creates a CSV writer object to write data to the file.
- **writer.writerow**: Writes the header row to the CSV file.

### Active Window Information
```python
    def get_active_window_info(self):
        """Get information about the currently active window"""
        try:
            hwnd = win32gui.GetForegroundWindow()
            _, pid = win32process.GetWindowThreadProcessId(hwnd)
            process = psutil.Process(pid)
            app_name = process.name().lower()
            window_title = win32gui.GetWindowText(hwnd)
            return app_name, window_title
        except Exception as e:
            print(f"Error getting active window: {e}")
            return None, None
```
- **get_active_window_info**: Retrieves the name and title of the currently active window.
- **win32gui.GetForegroundWindow**: Gets the handle of the current foreground window.
- **win32process.GetWindowThreadProcessId**: Retrieves the process ID of the window.
- **psutil.Process**: Creates a process object to get information about the process.
- **process.name()**: Gets the name of the process (application).
- **win32gui.GetWindowText**: Gets the title of the window.
- **try-except**: Handles exceptions that may occur during the process.
- **print**: Outputs an error message if an exception occurs.

### Productivity Check
```python
    def is_productive(self, app_name, window_title):
        """
        Determine if an app or website is productive.
        
        Returns:
        - True: Productive
        - False: Unproductive
        - None: Neutral
        """
        app_name = app_name.lower()
        
        # Check if app is in productive or unproductive lists
        if app_name in (app.lower() for app in config.PRODUCTIVE_APPS):
            return True
        elif app_name in (app.lower() for app in config.UNPRODUCTIVE_APPS):
            return False
        
        # Check website in window title
        window_title = window_title.lower()
        for website in config.PRODUCTIVE_WEBSITES:
            if website in window_title:
                return True
        
        for website in config.UNPRODUCTIVE_WEBSITES:
            if website in window_title:
                return False
        
        # Default to neutral
        return None
```
- **is_productive**: Determines if the current app or website is productive, unproductive, or neutral.
- **app_name.lower()**: Converts the app name to lowercase for comparison.
- **config.PRODUCTIVE_APPS**: A list of apps considered productive.
- **config.UNPRODUCTIVE_APPS**: A list of apps considered unproductive.
- **config.PRODUCTIVE_WEBSITES**: A list of websites considered productive.
- **config.UNPRODUCTIVE_WEBSITES**: A list of websites considered unproductive.
- **return**: Returns True, False, or None based on the productivity status.

### Tracking Control
```python
    def start_tracking(self):
        """Start tracking user activity in a separate thread"""
        if self.tracking_thread and self.tracking_thread.is_alive():
            print("Tracking already active")
            return
        
        self.is_tracking = True
        self.tracking_thread = threading.Thread(target=self._track_activity_loop)
        self.tracking_thread.daemon = True
        self.tracking_thread.start()
        print("Activity tracking started")

    def stop_tracking(self):
        """Stop tracking user activity"""
        self.is_tracking = False
        if self.tracking_thread:
            self.tracking_thread.join(timeout=1)
        print("Activity tracking stopped")
```
- **start_tracking**: Begins tracking user activity in a separate thread.
- **self.tracking_thread.is_alive()**: Checks if the tracking thread is already running.
- **threading.Thread**: Creates a new thread to run the tracking loop.
- **self.tracking_thread.daemon**: Sets the thread as a daemon, meaning it will exit when the main program exits.
- **self.tracking_thread.start()**: Starts the tracking thread.
- **stop_tracking**: Stops the tracking thread and waits for it to finish.
- **self.tracking_thread.join**: Waits for the thread to complete.

### Activity Tracking Loop
```python
    def _track_activity_loop(self):
        """Main loop for tracking activity"""
        while self.is_tracking:
            app_name, window_title = self.get_active_window_info()
            
            if app_name:
                current_time = time.time()
                
                # If app has changed
                if app_name != self.current_app or window_title != self.current_window_title:
                    # Log previous app session if it exists
                    if self.current_app and self.app_start_time:
                        duration = current_time - self.app_start_time
                        is_productive = self.is_productive(self.current_app, self.current_window_title)
                        self.log_activity(self.current_app, self.current_window_title, duration, is_productive)
                    
                    # Start tracking new app
                    self.current_app = app_name
                    self.current_window_title = window_title
                    self.app_start_time = current_time
                    
                    # Check if the new app is productive/unproductive/neutral
                    is_productive = self.is_productive(app_name, window_title)
                    
                    # Track unproductive time across multiple apps
                    if is_productive is False:  # Explicitly unproductive
                        print(f"Using unproductive app: {app_name}")
                        
                        # If this is the first unproductive app in this session
                        if not self.is_currently_unproductive:
                            self.unproductive_start_time = current_time
                            self.is_currently_unproductive = True
                            self.alert_triggered = False
                            print(f"Started tracking unproductive time at {datetime.fromtimestamp(self.unproductive_start_time).strftime('%H:%M:%S')}")
                    
                    elif is_productive is True:  # Explicitly productive
                        # Reset unproductive tracking when switching to a productive app
                        if self.is_currently_unproductive:
                            print(f"Switching to productive app: {app_name}. Unproductive session ended.")
                            elapsed_unproductive = current_time - self.unproductive_start_time
                            print(f"Unproductive time: {elapsed_unproductive:.1f} seconds")
                            
                            self.is_currently_unproductive = False
                            self.unproductive_start_time = None
                            self.total_unproductive_time = 0
                            self.alert_triggered = False
                            self.last_productive_timestamp = current_time
                
                # Check for unproductive time threshold
                if (self.is_currently_unproductive and 
                    self.unproductive_start_time and 
                    current_time - self.unproductive_start_time >= config.UNPRODUCTIVE_TIME_THRESHOLD and 
                    not self.alert_triggered):
                    self._trigger_unproductive_alert()
                    self.alert_triggered = True
                
            time.sleep(1)  # Check every second
```
- **_track_activity_loop**: The main loop that continuously checks the active window and logs activity.
- **while self.is_tracking**: Runs the loop as long as tracking is active.
- **self.get_active_window_info()**: Retrieves the current app and window title.
- **if app_name**: Proceeds if an app is detected.
- **current_time = time.time()**: Gets the current time.
- **if app_name != self.current_app**: Checks if the active app has changed.
- **self.log_activity**: Logs the previous app session.
- **self.is_productive**: Determines if the new app is productive.
- **if is_productive is False**: Handles unproductive app usage.
- **if not self.is_currently_unproductive**: Starts tracking unproductive time.
- **if is_productive is True**: Resets unproductive tracking for productive apps.
- **if self.is_currently_unproductive**: Checks if unproductive time exceeds the threshold.
- **self._trigger_unproductive_alert()**: Triggers an alert if needed.
- **time.sleep(1)**: Pauses for a second before the next check.

### Activity Logging
```python
    def log_activity(self, app_name, window_title, duration, is_productive):
        """Log app activity to CSV file"""
        try:
            timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
            
            with open(config.ACTIVITY_LOG_FILE, 'a', newline='') as file:
                writer = csv.writer(file)
                writer.writerow([
                    timestamp,
                    app_name,
                    window_title,
                    round(duration, 2),
                    str(is_productive)
                ])
        except Exception as e:
            print(f"Error logging activity: {e}")
```
- **log_activity**: Logs the details of an app session to a CSV file.
- **datetime.now().strftime**: Gets the current timestamp.
- **open**: Opens the log file in append mode.
- **csv.writer**: Creates a CSV writer object.
- **writer.writerow**: Writes a row of activity data to the file.
- **try-except**: Handles exceptions during logging.
- **print**: Outputs an error message if logging fails.

### Unproductive Alert
```python
    def _trigger_unproductive_alert(self):
        """Trigger an alert for unproductive app usage"""
        print(f"Triggering unproductive time alert")
        current_time = time.time()
        unproductive_minutes = (current_time - self.unproductive_start_time) / 60
        
        # Send desktop notification
        notification.notify(
            title="Productivity Alert",
            message=f"You've been unproductive for over {int(unproductive_minutes)} minute(s). Consider switching to a productive task.",
            timeout=10
        )
        
        # Call the UI callback if registered
        if self.on_unproductive_alert:
            self.on_unproductive_alert("unproductive time")
        
        # Mark alert as triggered to prevent repeated alerts
        self.alert_triggered = True
        print(f"Unproductive time alert triggered after {unproductive_minutes:.1f} minutes")
```
- **_trigger_unproductive_alert**: Sends a notification if unproductive time exceeds the threshold.
- **print**: Outputs a message indicating an alert is triggered.
- **notification.notify**: Sends a desktop notification.
- **if self.on_unproductive_alert**: Calls a UI update function if registered.
- **self.alert_triggered = True**: Prevents repeated alerts for the same session.

### Force Alert
```python
    def force_alert(self):
        """Force an alert for testing purposes"""
        print("Forcing productivity alert")
        app_name, _ = self.get_active_window_info()
        self._trigger_unproductive_alert()
```
- **force_alert**: Manually triggers an alert for testing.
- **print**: Outputs a message indicating a forced alert.
- **self._trigger_unproductive_alert()**: Calls the alert function.

### Daily Summary
```python
    def get_daily_summary(self, date=None):
        """
        Get a summary of activity for a specific date.
        If date is None, uses today's date.
        """
        if date is None:
            date = datetime.now().strftime('%Y-%m-%d')
        
        total_time = 0
        productive_time = 0
        unproductive_time = 0
        app_usage = {}
        
        try:
            with open(config.ACTIVITY_LOG_FILE, 'r', newline='') as file:
                reader = csv.reader(file)
                next(reader)  # Skip header
                
                for row in reader:
                    if row[0].startswith(date):
                        app_name = row[1]
                        duration = float(row[3])
                        is_productive = row[4]
                        
                        total_time += duration
                        
                        # Count productive and unproductive time
                        if is_productive == 'True':
                            productive_time += duration
                        elif is_productive == 'False':
                            unproductive_time += duration
                        
                        # Track app usage time
                        if app_name in app_usage:
                            app_usage[app_name] += duration
                        else:
                            app_usage[app_name] = duration
        except Exception as e:
            print(f"Error getting daily summary: {e}")
            return None
        
        # Sort apps by usage time
        sorted_apps = sorted(app_usage.items(), key=lambda x: x[1], reverse=True)
        
        return {
            'date': date,
            'total_time': total_time,
            'productive_time': productive_time,
            'unproductive_time': unproductive_time,
            'productive_percentage': (productive_time / total_time * 100) if total_time > 0 else 0,
            'apps': dict(sorted_apps[:10])  # Top 10 apps
        } 
```
- **get_daily_summary**: Provides a summary of activity for a given date.
- **if date is None**: Uses today's date if no date is provided.
- **total_time, productive_time, unproductive_time**: Variables to accumulate time data.
- **app_usage**: A dictionary to track usage time per app.
- **open**: Opens the log file for reading.
- **csv.reader**: Creates a CSV reader object.
- **next(reader)**: Skips the header row.
- **for row in reader**: Iterates over each row in the log file.
- **if row[0].startswith(date)**: Filters rows by the specified date.
- **sorted**: Sorts apps by usage time.
- **return**: Returns a dictionary with the summary data.

This document should help someone with basic Python knowledge understand the functionality and purpose of each part of the `activity_tracker.py` file. 