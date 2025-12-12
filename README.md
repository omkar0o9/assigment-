/*
 * Lab 5
 * Student Name
 * COSC 1436
 * Fall 2023
 */

#include <iostream>
#include <string>
#include <iomanip>
#include <cmath>

using namespace std;

//Stop details
struct Stop
{
    int x;
    int y;
};

// Clear input buffer
void ClearInputBuffer()
{
    cin.ignore(INT_MAX, '\n');
}

// Display error message
void DisplayError(string message)
{
    cout << "ERROR: " << message << endl;
}

// Read integer with range
int ReadInt(int minValue, int maxValue)
{
    int value;
    while (true)
    {
        cin >> value;
        
        if (value >= minValue && value <= maxValue)
        {
            ClearInputBuffer();
            return value;
        }
        
        DisplayError("Value must be between " + to_string(minValue) + " and " + to_string(maxValue));
        cin.clear();
        ClearInputBuffer();
    }
}

// Read character choice
char ReadChar(string options)
{
    string input;
    getline(cin, input);
    
    if (input.length() > 0)
    {
        char choice = toupper(input[0]);
        
        for (char opt : options)
        {
            if (toupper(opt) == choice)
                return choice;
        }
    }
    
    DisplayError("Please enter one of: " + options);
    return ReadChar(options);
}

// Confirm action
bool Confirm(string message)
{
    cout << message << " (Y/N): ";
    string answer;
    getline(cin, answer);
    
    return (answer.length() > 0 && toupper(answer[0]) == 'Y');
}

// Calculate distance between points
double GetDistance(int x1, int y1, int x2, int y2)
{
    double dx = x2 - x1;
    double dy = y2 - y1;
    return sqrt(dx * dx + dy * dy);
}

// Add stop to array
bool AddStopToArray(Stop* trip[], int size, int& count, Stop* stop)
{
    if (count >= size)
        return false;
    
    for (int i = 0; i < size; i++)
    {
        if (trip[i] == nullptr)
        {
            trip[i] = stop;
            count++;
            return true;
        }
    }
    
    return false;
}

// Find stop by number (1-based)
Stop* FindStopByNumber(Stop* trip[], int count, int stopNum)
{
    if (stopNum < 1 || stopNum > count)
        return nullptr;
    
    return trip[stopNum - 1];
}

// Remove stop from array
void RemoveStopFromArray(Stop* trip[], int size, int& count, Stop* stop)
{
    // Find which index this stop is at
    int index = -1;
    for (int i = 0; i < count; i++)
    {
        if (trip[i] == stop)
        {
            index = i;
            break;
        }
    }
    
    if (index == -1)
        return;
    
    // Delete the stop
    delete trip[index];
    
    // Move everything up
    for (int i = index; i < count - 1; i++)
    {
        trip[i] = trip[i + 1];
    }
    
    trip[count - 1] = nullptr;
    count--;
}

// Clear all stops
void ClearAllStops(Stop* trip[], int size, int& count)
{
    for (int i = 0; i < count; i++)
    {
        delete trip[i];
        trip[i] = nullptr;
    }
    count = 0;
}

// View trip with distances and times
void ViewTrip(Stop* trip[], int count, int speed)
{
    if (count == 0)
    {
        cout << "No stops in the trip." << endl << endl;
        return;
    }
    
    cout << endl;
    cout << "Stop            Location    Distance (miles)      Time (minutes)" << endl;
    cout << "--------------------------------------------------------------------" << endl;
    
    double totalDistance = 0;
    double totalTime = 0;
    int prevX = 0, prevY = 0;
    
    cout << fixed << setprecision(2);
    
    for (int i = 0; i < count; i++)
    {
        Stop* currentStop = trip[i];
        
        double distance = GetDistance(prevX, prevY, currentStop->x, currentStop->y);
        double timeMinutes = (distance / speed) * 60.0;
        int roundedTime = static_cast<int>(ceil(timeMinutes));
        
        totalDistance += distance;
        totalTime += roundedTime;
        
        cout << right << setw(4) << (i + 1) << "            "
             << "(" << setw(3) << currentStop->x 
             << ", " << setw(3) << currentStop->y << ")"
             << right << setw(17) << distance 
             << setw(21) << roundedTime << endl;
        
        prevX = currentStop->x;
        prevY = currentStop->y;
    }
    
    cout << "--------------------------------------------------------------------" << endl;
    cout << right << setw(4) << count 
         << "                          "
         << right << setw(17) << totalDistance
         << setw(21) << totalTime << endl << endl;
}

// Menu function to add stop
void MenuAddStop(Stop* trip[], int maxSize, int& count)
{
    if (count >= maxSize)
    {
        DisplayError("Trip is full! Maximum " + to_string(maxSize) + " stops allowed.");
        return;
    }
    
    cout << "Adding new stop:" << endl;
    
    cout << "Enter X coordinate (-100 to 100): ";
    int x = ReadInt(-100, 100);
    
    cout << "Enter Y coordinate (-100 to 100): ";
    int y = ReadInt(-100, 100);
    
    Stop* newStop = new Stop;
    newStop->x = x;
    newStop->y = y;
    
    if (AddStopToArray(trip, maxSize, count, newStop))
        cout << "Stop added successfully!" << endl;
    else
    {
        DisplayError("Failed to add stop.");
        delete newStop;
    }
}

// Menu function to remove stop
void MenuRemoveStop(Stop* trip[], int maxSize, int& count)
{
    if (count == 0)
    {
        DisplayError("No stops to remove.");
        return;
    }
    
    cout << "Current trip has " << count << " stops." << endl;
    cout << "Enter stop number to remove (1-" << count << "): ";
    int stopNumber = ReadInt(1, count);
    
    Stop* stopToRemove = FindStopByNumber(trip, count, stopNumber);
    if (stopToRemove == nullptr)
    {
        DisplayError("Stop not found.");
        return;
    }
    
    if (Confirm("Are you sure you want to remove stop " + to_string(stopNumber) + "?"))
    {
        RemoveStopFromArray(trip, maxSize, count, stopToRemove);
        cout << "Stop " << stopNumber << " removed successfully!" << endl;
    }
    else
        cout << "Stop removal cancelled." << endl;
}

// Menu function to clear trip
void MenuClearTrip(Stop* trip[], int maxSize, int& count)
{
    if (count == 0)
    {
        cout << "Trip is already empty." << endl;
        return;
    }
    
    if (Confirm("Are you sure you want to clear the entire trip?"))
    {
        ClearAllStops(trip, maxSize, count);
        cout << "Trip cleared successfully!" << endl;
    }
    else
        cout << "Trip clearance cancelled." << endl;
}

// Show and handle main menu
bool HandleMainMenu(Stop* trip[], int maxSize, int& count, int speed)
{
    cout << endl << "MAIN MENU" << endl;
    cout << "---------" << endl;
    cout << "A) Add Stop" << endl;
    cout << "V) View Trip" << endl;
    cout << "R) Remove Stop" << endl;
    cout << "C) Clear Trip" << endl;
    cout << "Q) Quit" << endl << endl;
    
    cout << "Enter your choice: ";
    char choice = ReadChar("AVRCQ");
    
    switch (choice)
    {
        case 'A':
            MenuAddStop(trip, maxSize, count);
            break;
            
        case 'V':
            ViewTrip(trip, count, speed);
            break;
            
        case 'R':
            MenuRemoveStop(trip, maxSize, count);
            break;
            
        case 'C':
            MenuClearTrip(trip, maxSize, count);
            break;
            
        case 'Q':
            if (Confirm("Are you sure you want to quit?"))
                return false;
            break;
    }
    
    return true;
}

// Main function
int main()
{
    cout << "Trip Planner Program" << endl;
    cout << "Created by Student Name" << endl;
    cout << "COSC 1436 - Fall 2023" << endl << endl;
    
    // Get speed
    cout << "Enter your travel speed (1-60 mph): ";
    int speed = ReadInt(1, 60);
    cout << endl;
    
    // Create trip array
    const int MAX_STOPS = 100;
    Stop* trip[MAX_STOPS] = {nullptr};
    int stopCount = 0;
    
    // Main loop
    bool continueRunning = true;
    while (continueRunning)
    {
        continueRunning = HandleMainMenu(trip, MAX_STOPS, stopCount, speed);
    }
    
    // Clean up memory before exiting
    ClearAllStops(trip, MAX_STOPS, stopCount);
    
    return 0;
}
