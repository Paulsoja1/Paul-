# Paul-
#library handling

import os
import time
from datetime import datetime

# Define the file path for storing visitor information
VISITORS_FILE = 'visitors.txt'
# Set the cooldown period in seconds (5 minutes = 300 seconds)
COOLDOWN_PERIOD = 300

class DuplicateVisitorsError(Exception):
    """A custom exception for handling visitor access errors."""
    pass

def library():
    """
    Checks if a new visitor can enter based on a cooldown and past visits,
    and logs the visit. If a visitor has already visited or if the last visit
    was within the cooldown period, it raises a custom error preventing access.
    """
    all_visitors = set()
    last_visit_time = 0
    
    # --- Error Handling for Reading File ---
    # Check if the visitors file exists and read its contents
    if os.path.exists(VISITORS_FILE):
        try:
            with open(VISITORS_FILE, 'r') as file:
                lines = [line.strip() for line in file.readlines() if line.strip()]
                for line in lines:
                    parts = line.split(',')
                    # Explicitly handle potential data format errors in the file
                    if len(parts) >= 2:
                        all_visitors.add(parts[1])
                
                if lines:
                    last_entry = lines[-1].split(',')
                    # Get the timestamp from the last valid entry
                    if len(last_entry) >= 2:
                        last_visit_time = float(last_entry[0])
                    else:
                        print("Warning: Visitor log file is corrupted. Starting with a fresh slate.")

        except (IOError, ValueError) as e:
            # Handle cases where the file cannot be read or its content is malformed
            print(f"Warning: Could not read previous visitor data due to an error: {e}")
            # Continue execution, assuming no previous valid data

    # Get the current time and visitor's name
    current_time = time.time()
    visitor_name = input("Please enter your name: ").lower().strip()

    try:
        # Check if the visitor has already visited at any point
        if visitor_name in all_visitors:
            raise DuplicateVisitorsError(f"Access denied: {visitor_name}, you have already visited the library. No back to back visit allowed.")
        
        # Check if any visitor just entered within the cooldown period
        if (current_time - last_visit_time) < COOLDOWN_PERIOD:
            remaining_cooldown = int(COOLDOWN_PERIOD - (current_time - last_visit_time))
            raise DuplicateVisitorsError(
                f"Access denied: Another visitor just entered. Please wait for {remaining_cooldown} seconds before you can visit."
            )
        else:
            # --- Error Handling for Writing to File ---
            # If the visit is allowed, append the new entry to the file
            visit_date_time_str = datetime.fromtimestamp(current_time).strftime('%Y-%m-%d %H:%M:%S')
            try:
                with open(VISITORS_FILE, 'a') as file:
                    file.write(f"{current_time},{visitor_name},{visit_date_time_str}\n")
                
                print(f"Welcome, {visitor_name}! Your visit on {visit_date_time_str} has been logged.")
            except (IOError, PermissionError) as e:
                # Handle cases where the file cannot be written to
                print(f"An error occurred while writing to the visitor log: {e}")

    except DuplicateVisitorsError as e:
        # Handle the custom exception for denied access
        print(f"Error: {e}")


if __name__ == "__main__":
    # The script now immediately prompts for the visitor's name and attempts to log the visit.
    library()
