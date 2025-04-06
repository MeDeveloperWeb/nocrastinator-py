# Productivity Tracker

A Python application that helps you track and improve your productivity by monitoring app usage, providing a Pomodoro timer, and generating personalized insights.

## Features

- **Activity Tracking**: Automatically monitors which applications and websites you're using
- **Pomodoro Timer**: Integrated timer with work and break sessions to improve focus
- **Productivity Alerts**: Notifies you when you've spent too much time on unproductive apps
- **Focus Score**: Daily productivity score based on your app usage
- **Analytics Dashboard**: Visual breakdown of your productivity trends
- **Personalized Suggestions**: Recommendations to improve your work habits

## Installation

1. Clone this repository
2. Install dependencies:
   ```
   pip install -r requirements.txt
   ```
3. Run the application:
   ```
   python src/main.py
   ```

## Configuration

You can customize the application by editing the `src/config.py` file:

- Add or remove apps from productive/unproductive lists
- Adjust time thresholds for alerts
- Modify Pomodoro timer durations
- Customize focus score calculations

## Usage

### Dashboard Tab
- View your current focus score and activity
- Use the Pomodoro timer to manage work sessions
- Receive alerts when spending too much time on distracting apps

### Analysis Tab
- See your weekly productivity overview
- Review statistics about app usage
- Get personalized suggestions for improvement

### Settings Tab
- View current app categorizations
- See Pomodoro timer settings
- Reset application data if needed

## Data Storage

All data is stored locally in the `data` directory:

- `activity_log.csv`: Log of all app usage
- `focus_scores.json`: Daily focus scores and statistics

## Requirements

- Python 3.6+
- Windows OS (for app tracking features)
- Libraries: psutil, pywin32, plyer, matplotlib, tkinter "# nocrastinator-py" 
