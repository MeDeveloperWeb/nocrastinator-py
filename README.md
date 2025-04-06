# Productivity Tracker

## Overview
The Productivity Tracker is a Python application designed to help users monitor and improve their productivity. It tracks application and website usage, categorizes tasks as productive or unproductive, and provides insights and alerts to encourage better habits.

## Features
- **Activity Tracking**: Monitors app and website usage, logging time spent on each.
- **Productivity Alerts**: Sends notifications when unproductive time exceeds a threshold.
- **Pomodoro Timer**: Implements the Pomodoro technique to boost focus.
- **Focus Score**: Calculates a daily productivity score based on usage patterns.
- **Data Analysis**: Provides insights into usage patterns and productivity trends.

## Setup Instructions
1. **Install Dependencies**: Ensure you have Python installed, then run `pip install -r requirements.txt` to install necessary packages.
2. **Configure Settings**: Edit `config.py` to customize app categorization and thresholds.
3. **Run the Application**: Execute `python src/app.py` to start the application.

## Usage
- **Dashboard**: View current activity, focus score, and Pomodoro timer.
- **Analysis**: Review weekly productivity trends and app usage statistics.
- **Settings**: Adjust app categorization and Pomodoro settings.

## Contributing
Feel free to fork the repository and submit pull requests for improvements or bug fixes.

## License
This project is licensed under the MIT License.

## Configuration

You can customize the application by editing the `src/config.py` file:

- Add or remove apps from productive/unproductive lists
- Adjust time thresholds for alerts
- Modify Pomodoro timer durations
- Customize focus score calculations

## Data Storage

All data is stored locally in the `data` directory:

- `activity_log.csv`: Log of all app usage
- `focus_scores.json`: Daily focus scores and statistics

## Requirements

- Python 3.6+
- Windows OS (for app tracking features)
- Libraries: psutil, pywin32, plyer, matplotlib, tkinter "# nocrastinator-py" 
