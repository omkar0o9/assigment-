/*
 * Omkar Sharma
 * Lab 5 – Trip Planner
 * COSC 1436
 * Fall 2025
 */

#include <iostream>
#include <iomanip>
#include <string>
#include <cctype>
#include <cmath>
#include <climits>

struct Stop
{
    int x;
    int y;
};

// clears bad input
void ClearInput()
{
    std::cin.clear();
    std::cin.ignore(INT_MAX, '\n');
}

// story 1
void ShowHeader()
{
    std::cout << "Trip Planner Program\n";
    std::cout << "Omkar Sharma\n";
    std::cout << "COSC 1436 Fall 2025\n";
    std::cout << "------------------------\n\n";
}

void ShowError(const std::string& msg)
{
    std::cout << "ERROR: " << msg << std::endl;
}

bool AskYesNo(const std::string& msg)
{
    while (true)
    {
        std::cout << msg << " (Y/N): ";
        std::string input;
        std::getline(std::cin, input);

        if (input.empty())
            continue;

        char choice = (char)std::toupper(input[0]);

        if (choice == 'Y') return true;
        if (choice == 'N') return false;

        ShowError("Please enter Y or N");
    }
}

int GetInt(const std::string& prompt, int min, int max)
{
    while (true)
    {
        std::cout << prompt;

        int value;
        std::cin >> value;

        if (std::cin.fail())
        {
            ShowError("Invalid number");
            ClearInput();
            continue;
        }

        ClearInput();

        if (value < min || value > max)
        {
            ShowError("Number out of range");
            continue;
        }

        return value;
    }
}

// story 8
int GetSpeed()
{
    return GetInt("Enter speed (1–60 mph): ", 1, 60);
}

char ShowMenu()
{
    std::cout << "\nTrip Menu\n";
    std::cout << "---------\n";
    std::cout << "A) Add Stop\n";
    std::cout << "V) View Trip\n";
    std::cout << "D) Delete Stop\n";
    std::cout << "C) Clear Trip\n";
    std::cout << "Q) Quit\n";
    std::cout << "Choice: ";

    std::string input;
    std::getline(std::cin, input);

    if (input.empty())
        return '\0';

    return (char)std::toupper(input[0]);
}

// story 3
int GetStopCount(Stop* trip[], int size)
{
    int count = 0;

    for (int i = 0; i < size; i++)
    {
        if (trip[i] == nullptr)
            break;
        count++;
    }

    return count;
}

// story 4
bool InsertStop(Stop* trip[], int size, Stop* s)
{
    for (int i = 0; i < size; i++)
    {
        if (trip[i] == nullptr)
        {
            trip[i] = s;
            return true;
        }
    }
    return false;
}

void AddStopMenu(Stop* trip[], int size)
{
    int x = GetInt("Enter X (-100 to 100): ", -100, 100);
    int y = GetInt("Enter Y (-100 to 100): ", -100, 100);

    Stop* s = new Stop;
    s->x = x;
    s->y = y;

    if (!InsertStop(trip, size, s))
    {
        ShowError("Trip already full");
        delete s;
        return;
    }

    std::cout << "Stop added successfully\n";
}

// story 8
double CalcDistance(int x1, int y1, int x2, int y2)
{
    double dx = x2 - x1;
    double dy = y2 - y1;
    return std::sqrt(dx * dx + dy * dy);
}

// story 5 + 8
void ViewTripMenu(Stop* trip[], int size, int speed)
{
    int count = GetStopCount(trip, size);

    if (count == 0)
    {
        std::cout << "Trip has no stops\n";
        return;
    }

    std::cout << std::fixed << std::setprecision(2);

    std::cout << " #"
              << std::setw(16) << "Location"
              << std::setw(18) << "Distance"
              << std::setw(16) << "Time\n";

    std::cout << std::setw(60) << std::setfill('-') << ""
              << std::setfill(' ') << "\n";

    int lastX = 0;
    int lastY = 0;

    double totalDist = 0;
    double totalTime = 0;

    for (int i = 0; i < count; i++)
    {
        Stop* s = trip[i];

        double d = CalcDistance(lastX, lastY, s->x, s->y);
        double mins = std::ceil((d / speed) * 60);

        totalDist += d;
        totalTime += mins;

        std::string loc = "(" + std::to_string(s->x) + ", " +
                          std::to_string(s->y) + ")";

        std::cout << std::setw(4) << (i + 1)
                  << std::setw(16) << loc
                  << std::setw(18) << d
                  << std::setw(16) << mins << "\n";

        lastX = s->x;
        lastY = s->y;
    }

    std::cout << std::setw(60) << std::setfill('-') << ""
              << std::setfill(' ') << "\n";

    std::cout << std::setw(4) << count
              << std::setw(16) << ""
              << std::setw(18) << totalDist
              << std::setw(16) << totalTime << "\n";
}

// story 6
void ShiftStops(Stop* trip[], int size)
{
    int index = 0;

    for (int i = 0; i < size; i++)
    {
        if (trip[i] != nullptr)
            trip[index++] = trip[i];
    }

    for (int i = index; i < size; i++)
        trip[i] = nullptr;
}

void DeleteStopMenu(Stop* trip[], int size)
{
    int num = GetInt("Enter stop number: ", 1, size);
    int idx = num - 1;

    if (idx < 0 || idx >= size || trip[idx] == nullptr)
    {
        ShowError("Stop does not exist");
        return;
    }

    delete trip[idx];
    trip[idx] = nullptr;

    ShiftStops(trip, size);

    std::cout << "Stop removed\n";
}

// story 7
void ClearTrip(Stop* trip[], int size)
{
    for (int i = 0; i < size; i++)
    {
        delete trip[i];
        trip[i] = nullptr;
    }
}

void ClearTripMenu(Stop* trip[], int size)
{
    if (!AskYesNo("Clear entire trip"))
        return;

    ClearTrip(trip, size);
    std::cout << "Trip cleared\n";
}

// main
int main()
{
    ShowHeader();

    const int MAX_STOPS = 100;
    Stop* trip[MAX_STOPS] = { nullptr };

    int speed = GetSpeed();

    bool quit = false;
    while (!quit)
    {
        char choice = ShowMenu();

        switch (choice)
        {
            case 'A': AddStopMenu(trip, MAX_STOPS); break;
            case 'V': ViewTripMenu(trip, MAX_STOPS, speed); break;
            case 'D': DeleteStopMenu(trip, MAX_STOPS); break;
            case 'C': ClearTripMenu(trip, MAX_STOPS); break;

            case 'Q':
                if (AskYesNo("Quit program"))
                    quit = true;
                break;

            default:
                ShowError("Unknown menu option");
        }
    }

    ClearTrip(trip, MAX_STOPS);
    return 0;
}
