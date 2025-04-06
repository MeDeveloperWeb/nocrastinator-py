# App Module

This document provides a detailed explanation of the `app.py` file, which serves as the main application for the Productivity Tracker. It sets up the user interface, initializes components, and manages the main application logic.

## Overview
The `ProductivityTrackerApp` class is the core component of this module. It initializes the application, sets up the user interface, and manages interactions between different components like the activity tracker, focus score, and Pomodoro timer.

## Code Explanation

### Imports
```python
import tkinter as tk
from tkinter import ttk, messagebox
import threading
import os
import json
import sys
import time
from datetime import datetime, timedelta

# Import our modules
from activity_tracker import ActivityTracker
from focus_score import FocusScore
from pomodoro import PomodoroTimer
import config
```
- **tkinter**: A standard Python library for creating graphical user interfaces (GUIs).
- **threading**: Allows the program to run multiple operations concurrently.
- **os**: Provides functions to interact with the operating system, such as creating directories.
- **json**: Used for parsing and generating JSON data.
- **sys**: Provides access to system-specific parameters and functions.
- **time**: Used for time-related functions, like getting the current time.
- **datetime**: Provides functions to manipulate dates and times.
- **activity_tracker, focus_score, pomodoro**: Custom modules for tracking activities, calculating focus scores, and managing the Pomodoro timer.
- **config**: A custom module containing configuration settings like app categorization and thresholds.

### Class Definition
```python
class ProductivityTrackerApp:
    """Main application class for the Productivity Tracker."""
```
- **ProductivityTrackerApp**: A class that encapsulates all functionality related to the main application.

### Initialization
```python
    def __init__(self, root):
        self.root = root
        self.root.title("Productivity Tracker")
        self.root.geometry("900x700")
        self.root.minsize(800, 600)
        
        # Set up logging to file
        self.setup_logging()
        
        # Create data directory if it doesn't exist
        os.makedirs(config.DATA_DIRECTORY, exist_ok=True)
        
        # Initialize components
        self.activity_tracker = ActivityTracker()
        self.focus_score = FocusScore()
        self.pomodoro = PomodoroTimer()
        
        # Set callbacks
        self.pomodoro.on_tick = self.update_pomodoro_display
        self.pomodoro.on_phase_change = self.handle_phase_change
        self.activity_tracker.on_unproductive_alert = self.handle_unproductive_alert
        
        # Set up UI
        self.setup_ui()
        
        # Start activity tracking
        self.start_tracking()
        
        # Update UI periodically
        self.update_ui()
        
        print("Application initialized")
```
- **__init__**: The constructor method initializes the class attributes and sets up the environment for the application.
- **self.root**: The main window of the application.
- **self.setup_logging()**: Sets up logging to a file for debugging purposes.
- **os.makedirs**: Creates the data directory if it doesn't exist.
- **self.activity_tracker, self.focus_score, self.pomodoro**: Initializes the main components of the application.
- **self.pomodoro.on_tick, self.pomodoro.on_phase_change, self.activity_tracker.on_unproductive_alert**: Sets callback functions for various events.
- **self.setup_ui()**: Sets up the user interface.
- **self.start_tracking()**: Starts the activity tracking process.
- **self.update_ui()**: Updates the user interface periodically.
- **print**: Outputs a message indicating that the application is initialized.

### Logging Setup
```python
    def setup_logging(self):
        """Set up logging to a file for debugging"""
        try:
            log_file = os.path.join(config.DATA_DIRECTORY, "productivity_tracker.log")
            sys.stdout = open(log_file, "a")
            print(f"\n--- Log started at {datetime.now()} ---")
        except Exception as e:
            print(f"Error setting up logging: {e}")
```
- **setup_logging**: Configures logging to a file for debugging purposes.
- **os.path.join**: Constructs the path to the log file.
- **sys.stdout**: Redirects standard output to the log file.
- **print**: Logs the start of a new session or any errors encountered.

### User Interface Setup
```python
    def setup_ui(self):
        """Set up the main UI"""
        # Style configuration
        self.style = ttk.Style()
        self.style.configure("TNotebook", background="#f0f0f0")
        self.style.configure("Alert.TLabel", foreground="red", font=("Arial", 12, "bold"))
        self.style.configure("Header.TLabel", font=("Arial", 16, "bold"))
        self.style.configure("Subheader.TLabel", font=("Arial", 12, "bold"))
        self.style.configure("Timer.TLabel", font=("Arial", 36, "bold"))
        self.style.configure("Phase.TLabel", font=("Arial", 14))
        self.style.configure("Score.TLabel", font=("Arial", 24, "bold"))
        
        # Create notebook for tabs
        self.notebook = ttk.Notebook(self.root)
        self.notebook.pack(expand=True, fill="both", padx=10, pady=10)
        
        # Create main dashboard tab
        self.create_dashboard_tab()
        
        # Create analysis tab
        self.create_analysis_tab()
        
        # Create settings tab
        self.create_settings_tab()
        
        # Bind tab change event
        self.notebook.bind("<<NotebookTabChanged>>", self.handle_tab_change)
```
- **setup_ui**: Configures the main user interface.
- **self.style**: Configures the style for various UI elements.
- **ttk.Notebook**: Creates a tabbed interface for the application.
- **self.create_dashboard_tab, self.create_analysis_tab, self.create_settings_tab**: Creates different tabs for the application.
- **self.notebook.bind**: Binds an event handler for tab changes.

### Dashboard Tab
```python
    def create_dashboard_tab(self):
        """Create the main dashboard tab"""
        dashboard_frame = ttk.Frame(self.notebook)
        self.notebook.add(dashboard_frame, text="Dashboard")
        
        # Top section - Focus score and current activity
        top_frame = ttk.Frame(dashboard_frame)
        top_frame.pack(fill="x", pady=10)
        
        # Focus score display
        score_frame = ttk.LabelFrame(top_frame, text="Today's Focus Score")
        score_frame.pack(side="left", padx=10, fill="x", expand=True)
        
        self.score_label = ttk.Label(score_frame, text="0", style="Score.TLabel")
        self.score_label.pack(pady=5)
        
        self.streak_label = ttk.Label(score_frame, text="Current streak: 0 days")
        self.streak_label.pack(pady=5)
        
        # Current activity display
        activity_frame = ttk.LabelFrame(top_frame, text="Current Activity")
        activity_frame.pack(side="left", padx=10, fill="x", expand=True)
        
        self.activity_label = ttk.Label(activity_frame, text="None", style="Subheader.TLabel")
        self.activity_label.pack(pady=5)
        
        self.activity_type_label = ttk.Label(activity_frame, text="")
        self.activity_type_label.pack(pady=5)
        
        # Middle section - Pomodoro timer
        pomodoro_frame = ttk.LabelFrame(dashboard_frame, text="Pomodoro Timer")
        pomodoro_frame.pack(fill="x", pady=10, padx=10)
        
        # Timer display
        timer_display_frame = ttk.Frame(pomodoro_frame)
        timer_display_frame.pack(pady=10)
        
        self.timer_label = ttk.Label(timer_display_frame, text=f"{config.POMODORO_WORK_DURATION}:00", style="Timer.TLabel")
        self.timer_label.pack()
        
        self.phase_label = ttk.Label(timer_display_frame, text="Ready", style="Phase.TLabel")
        self.phase_label.pack(pady=5)
        
        # Progress bar
        self.progress_var = tk.DoubleVar(value=0)
        self.progress_bar = ttk.Progressbar(
            pomodoro_frame, 
            orient="horizontal", 
            length=400, 
            mode="determinate",
            variable=self.progress_var
        )
        self.progress_bar.pack(pady=10, padx=20, fill="x")
        
        # Buttons frame
        pomodoro_buttons_frame = ttk.Frame(pomodoro_frame)
        pomodoro_buttons_frame.pack(pady=10)
        
        self.start_button = ttk.Button(
            pomodoro_buttons_frame, 
            text="Start", 
            command=self.toggle_pomodoro,
            width=15
        )
        self.start_button.pack(side="left", padx=5)
        
        self.pause_button = ttk.Button(
            pomodoro_buttons_frame, 
            text="Pause", 
            command=self.pause_resume_pomodoro,
            width=15,
            state="disabled"
        )
        self.pause_button.pack(side="left", padx=5)
        
        self.reset_button = ttk.Button(
            pomodoro_buttons_frame, 
            text="Reset", 
            command=self.reset_pomodoro,
            width=15,
            state="disabled"
        )
        self.reset_button.pack(side="left", padx=5)
        
        # Bottom section - Alerts and productivity tips
        bottom_frame = ttk.Frame(dashboard_frame)
        bottom_frame.pack(fill="both", expand=True, pady=10)
        
        # Alerts frame
        self.alerts_frame = ttk.LabelFrame(bottom_frame, text="Productivity Alerts")
        self.alerts_frame.pack(side="left", padx=10, fill="both", expand=True)
        
        self.alert_label = ttk.Label(
            self.alerts_frame, 
            text="No alerts", 
            wraplength=300
        )
        self.alert_label.pack(pady=10)
        
        # Test button for alerts (hidden in production)
        self.test_alert_button = ttk.Button(
            self.alerts_frame,
            text="Test Alert",
            command=self.activity_tracker.force_alert
        )
        self.test_alert_button.pack(pady=5)
        
        # Tips frame
```
- **create_dashboard_tab**: Sets up the main dashboard tab, which includes sections for focus score, current activity, Pomodoro timer, and alerts.
- **ttk.Frame, ttk.LabelFrame, ttk.Label**: Used to create and organize UI elements.
- **self.score_label, self.streak_label, self.activity_label, self.activity_type_label**: Labels for displaying focus score, streak, and current activity.
- **self.timer_label, self.phase_label**: Labels for displaying the Pomodoro timer and its phase.
- **self.progress_bar**: A progress bar to show Pomodoro progress.
- **self.start_button, self.pause_button, self.reset_button**: Buttons to control the Pomodoro timer.
- **self.alert_label**: Displays productivity alerts.
- **self.test_alert_button**: A button to test alerts (hidden in production).

This document should help someone with basic Python knowledge understand the functionality and purpose of each part of the `app.py` file. 

---