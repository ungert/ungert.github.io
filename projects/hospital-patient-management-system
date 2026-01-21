---
layout: project
type: project
image: img/hospital.jpeg
title: "Hospital Patient Management System"
date: 2025
published: true
labels:
  - C++
summary: "For a class I was required to write a program for a Hospital Paitent Management System"
---


This program implements a menu-driven hospital patient management system in C++ using a fixed-size array of structs. It allows users to add new patients, search for patients by ID, and view all stored patient records. Each patient record includes an ID, name, age, illness, and gender, with strict validation to ensure IDs are unique and positive, ages are within a valid range, and genders match allowed values. The program uses helper functions to safely handle user input and prevent crashes from invalid entries. The system runs continuously until the user chooses to quit from the menu.
Here is some code that illustrates the program:

```cpp

while (true) {
    cout << "\n1. Add Patient\n2. Search Patient\n3. View All Patients\n4. Quit\n";
    cout << "Enter your choice (1-4): ";
    choice = getIntInput();

    switch (choice) {
        case 1: addPatient(); break;
        case 2: searchPatient(); break;
        case 3: viewAllPatients(); break;
        case 4: return 0;
        default:
            cout << "Invalid choice. Please try again.\n";
    }
}
```
