# Focus Score Module

This document provides a detailed explanation of the `focus_score.py` file, which calculates productivity scores based on app usage data. It tracks daily scores and provides analysis over time.

## Overview
The `FocusScore` class is the core component of this module. It calculates daily productivity scores, tracks productivity streaks, and provides weekly analysis.

## Code Explanation

### Imports
```python
import os
import csv
import json
from datetime import datetime, timedelta
import config
```
- **os**: Provides functions to interact with the operating system, such as creating directories.
- **csv**: Used for reading and writing CSV files.
- **json**: Used for parsing and generating JSON data.
- **datetime, timedelta**: Provides functions to manipulate dates and times.
- **config**: A custom module containing configuration settings like file paths and weights for productivity calculations.

### Class Definition
```python
class FocusScore:
    """
    Calculates productivity scores based on app usage data.
    Tracks daily scores and provides analysis over time.
    """
```
- **FocusScore**: A class that encapsulates all functionality related to calculating productivity scores.

### Initialization
```python
    def __init__(self):
        self.scores_file = config.FOCUS_SCORE_FILE
        
        # Create data directory if it doesn't exist
        os.makedirs(config.DATA_DIRECTORY, exist_ok=True)
        
        # Load existing scores or create new ones
        self.scores = self._load_scores()
        
        print("Focus score calculator initialized")
```
- **__init__**: The constructor method initializes the class attributes and sets up the environment for score calculation.
- **self.scores_file**: The file path for storing focus scores.
- **os.makedirs**: Creates the data directory if it doesn't exist.
- **self.scores**: Loads existing scores or initializes a new dictionary.
- **print**: Outputs a message indicating that the focus score calculator is initialized.

### Load Scores
```python
    def _load_scores(self):
        """Load focus scores from file"""
        if os.path.exists(self.scores_file):
            try:
                with open(self.scores_file, 'r') as file:
                    return json.load(file)
            except Exception as e:
                print(f"Error loading scores: {e}")
                return {}
        else:
            return {}
```
- **_load_scores**: Loads focus scores from a file if it exists.
- **os.path.exists**: Checks if the scores file exists.
- **open**: Opens the file in read mode.
- **json.load**: Loads the scores from the file.
- **try-except**: Handles exceptions during file reading.
- **print**: Outputs an error message if loading fails.

### Save Scores
```python
    def _save_scores(self):
        """Save focus scores to file"""
        try:
            with open(self.scores_file, 'w') as file:
                json.dump(self.scores, file, indent=2)
        except Exception as e:
            print(f"Error saving scores: {e}")
```
- **_save_scores**: Saves focus scores to a file.
- **open**: Opens the file in write mode.
- **json.dump**: Writes the scores to the file.
- **try-except**: Handles exceptions during file writing.
- **print**: Outputs an error message if saving fails.

### Calculate Daily Score
```python
    def calculate_daily_score(self, date=None):
        """
        Calculate focus score for a given date.
        Score is based on productive vs. unproductive time.
        """
        if date is None:
            date = datetime.now().strftime('%Y-%m-%d')
        
        # If we already calculated today's score, return it
        if date in self.scores:
            return self.scores[date]['score']
        
        # Calculate from activity log
        total_time = 0
        productive_time = 0
        unproductive_time = 0
        neutral_time = 0
        
        try:
            # Read activity data
            with open(config.ACTIVITY_LOG_FILE, 'r', newline='') as file:
                reader = csv.reader(file)
                next(reader)  # Skip header
                
                for row in reader:
                    if row[0].startswith(date):
                        duration = float(row[3])
                        is_productive = row[4]
                        
                        total_time += duration
                        
                        if is_productive == 'True':
                            productive_time += duration
                        elif is_productive == 'False':
                            unproductive_time += duration
                        else:
                            neutral_time += duration
        except Exception as e:
            print(f"Error reading activity data: {e}")
            return 0
        
        # Calculate score (0-100)
        if total_time < 60:  # Less than a minute of data
            score = 0
        else:
            # Weighted score - productive time increases score, unproductive decreases it
            weighted_productive = productive_time * config.PRODUCTIVE_TIME_WEIGHT
            weighted_unproductive = unproductive_time * config.UNPRODUCTIVE_TIME_WEIGHT
            
            if weighted_productive + weighted_unproductive > 0:
                score = (weighted_productive / (weighted_productive + weighted_unproductive)) * 100
            else:
                score = 50  # Neutral score if no data
            
            # Cap score between 0-100
            score = max(0, min(100, score))
        
        # Save score with details
        self.scores[date] = {
            'score': score,
            'total_time': total_time,
            'productive_time': productive_time,
            'unproductive_time': unproductive_time,
            'neutral_time': neutral_time
        }
        
        self._save_scores()
        return score
```
- **calculate_daily_score**: Calculates the focus score for a given date based on productive and unproductive time.
- **if date is None**: Uses today's date if no date is provided.
- **if date in self.scores**: Returns the score if already calculated.
- **open**: Opens the activity log file for reading.
- **csv.reader**: Creates a CSV reader object.
- **next(reader)**: Skips the header row.
- **for row in reader**: Iterates over each row in the log file.
- **if row[0].startswith(date)**: Filters rows by the specified date.
- **score calculation**: Calculates a weighted score based on productive and unproductive time.
- **self._save_scores()**: Saves the calculated score to the file.

### Productivity Streak
```python
    def get_streak(self):
        """Calculate the current productivity streak"""
        min_score = 50  # Minimum score to count as productive day
        
        # Sort dates in reverse order (newest first)
        dates = sorted(self.scores.keys(), reverse=True)
        
        if not dates:
            return 0
        
        streak = 0
        today = datetime.now().date()
        
        for i, date_str in enumerate(dates):
            date = datetime.strptime(date_str, '%Y-%m-%d').date()
            score = self.scores[date_str]['score']
            
            # Check if this date is part of continuous streak
            expected_date = today - timedelta(days=i)
            
            if date == expected_date and score >= min_score:
                streak += 1
            else:
                break
        
        return streak
```
- **get_streak**: Calculates the current productivity streak based on daily scores.
- **min_score**: The minimum score required to count as a productive day.
- **sorted**: Sorts dates in reverse order.
- **for i, date_str in enumerate(dates)**: Iterates over each date.
- **if date == expected_date and score >= min_score**: Checks if the date is part of a continuous streak.

### Weekly Analysis
```python
    def get_weekly_analysis(self):
        """Get analysis for the past week"""
        end_date = datetime.now().date()
        start_date = end_date - timedelta(days=6)  # 7 days including today
        
        dates = []
        scores = []
        productive_times = []
        unproductive_times = []
        
        # Get data for each day in the week
        current_date = start_date
        while current_date <= end_date:
            date_str = current_date.strftime('%Y-%m-%d')
            dates.append(date_str)
            
            # Calculate score if not already calculated
            if date_str not in self.scores:
                self.calculate_daily_score(date_str)
            
            # Add data if available
            if date_str in self.scores:
                data = self.scores[date_str]
                scores.append(data['score'])
                productive_times.append(data['productive_time'] / 3600)  # Convert to hours
                unproductive_times.append(data['unproductive_time'] / 3600)  # Convert to hours
            else:
                scores.append(0)
                productive_times.append(0)
                unproductive_times.append(0)
            
            current_date += timedelta(days=1)
        
        # Most productive day
        if scores:
            max_score_index = scores.index(max(scores))
            most_productive_day = dates[max_score_index]
        else:
            most_productive_day = None
        
        # Calculate improvement suggestions
        suggestions = self._generate_suggestions()
        
        return {
            'dates': dates,
            'scores': scores,
            'productive_times': productive_times,
            'unproductive_times': unproductive_times,
            'streak': self.get_streak(),
            'most_productive_day': most_productive_day,
            'average_score': sum(scores) / len(scores) if scores else 0,
            'suggestions': suggestions
        }
```
- **get_weekly_analysis**: Provides an analysis of productivity for the past week.
- **end_date, start_date**: Defines the date range for the analysis.
- **while current_date <= end_date**: Iterates over each day in the week.
- **if date_str not in self.scores**: Calculates the score if not already calculated.
- **scores.append, productive_times.append, unproductive_times.append**: Collects data for each day.
- **most_productive_day**: Identifies the most productive day of the week.
- **self._generate_suggestions()**: Generates improvement suggestions.

This document should help someone with basic Python knowledge understand the functionality and purpose of each part of the `focus_score.py` file.

---

With this, we have covered the explanations for all the Python files in your project. If you have any further questions or need additional explanations, feel free to ask!