#include <string>
#include <fstream>
#include <iomanip>
#include <limits>
#include <algorithm>
#include <vector>
#include <cstring>
#include <ctime>
using namespace std;

// ====================================================================
// SECTION 1: STRUCTURES & UNIONS (50 lines)
// ====================================================================
/**
 * STUDENT ASSIGNMENT QUESTIONS:
 * 1. Why do we use structures instead of individual variables?
 * 2. What is the advantage of using a union for contact information?
 * 3. How does using Subject structure help in managing multiple subjects?
 */

struct Date {
    int day, month, year;
};

// Union demonstrates memory efficiency by sharing space for contact info
union ContactInfo {
    char email[50];
    char phone[15];
};

struct Subject {
    string name;
    int code;
    float maxMarks;
    float weightage;
};

struct Class {
    int classId;
    string className;
    string department;
    int year;
    Subject subjects[10];
    int subjectCount;
};

struct Student {
    int id;
    string name;
    Date dob;
    ContactInfo contact;
    int contactType; // 0=email, 1=phone
    int classId;
    float marks[10];
    float average;
    float gpa;
    char grade;
};

// ====================================================================
// SECTION 2: GLOBAL VARIABLES & CONSTANTS (20 lines)
// ====================================================================
const int MAX_STUDENTS = 1000;
const int MAX_CLASSES = 20;
Student students[MAX_STUDENTS];
Class classes[MAX_CLASSES];
int studentCount = 0;
int classCount = 0;
const string DATA_FILE = "students.dat";
const string CLASS_FILE = "classes.dat";
const string BACKUP_DIR = "backups/";

// ====================================================================
// SECTION 3: FILE HANDLING FUNCTIONS (200+ lines)
// ====================================================================
/**
 * STUDENT ASSIGNMENT:
 * 1. Compare binary vs text file I/O. Which is better for this system?
 * 2. How would you implement CSV export functionality?
 * 3. What are the security considerations when saving student data?
 */

// Save all students to binary file
void saveStudentsToFile() {
    ofstream file(DATA_FILE, ios::binary);
    if (!file) {
        cout << "Error: Cannot open file for writing!\n";
        return;
    }
    
    file.write(reinterpret_cast<char*>(&studentCount), sizeof(studentCount));
    for (int i = 0; i < studentCount; i++) {
        file.write(reinterpret_cast<char*>(&students[i]), sizeof(Student));
    }
    
    file.close();
    cout << "✓ Student data saved successfully! (" << studentCount << " records)\n";
}

// Load students from binary file
void loadStudentsFromFile() {
    ifstream file(DATA_FILE, ios::binary);
    if (!file) {
        cout << "ℹ No existing student data found. Starting fresh.\n";
        return;
    }
    
    file.read(reinterpret_cast<char*>(&studentCount), sizeof(studentCount));
    if (studentCount > MAX_STUDENTS) {
        cout << "⚠ Warning: File contains more students than maximum limit!\n";
        studentCount = MAX_STUDENTS;
    }
    
    for (int i = 0; i < studentCount; i++) {
        file.read(reinterpret_cast<char*>(&students[i]), sizeof(Student));
    }
    
    file.close();
    cout << "✓ Student data loaded successfully! (" << studentCount << " students)\n";
}

// Save class information to file
void saveClassesToFile() {
    ofstream file(CLASS_FILE, ios::binary);
    if (!file) {
        cout << "Error: Cannot open class file for writing!\n";
        return;
    }
    
    file.write(reinterpret_cast<char*>(&classCount), sizeof(classCount));
    for (int i = 0; i < classCount; i++) {
        file.write(reinterpret_cast<char*>(&classes[i]), sizeof(Class));
    }
    
    file.close();
    cout << "✓ Class data saved successfully! (" << classCount << " classes)\n";
}

// Load class information from file
void loadClassesFromFile() {
    ifstream file(CLASS_FILE, ios::binary);
    if (!file) {
        // Create default classes if file doesn't exist
        cout << "ℹ Creating default classes...\n";
        classCount = 3;
        
        // Computer Science class
        Subject csSubjects[4] = {
            {"Programming", 101, 100, 3.0},
            {"Data Structures", 102, 100, 4.0},
            {"Database Systems", 103, 100, 3.5},
            {"Operating Systems", 104, 100, 3.0}
        };
        
        Subject businessSubjects[4] = {
            {"Economics", 201, 100, 3.0},
            {"Accounting", 202, 100, 3.5},
            {"Marketing", 203, 100, 3.0},
            {"Finance", 204, 100, 4.0}
        };
        
        Subject physicsSubjects[4] = {
            {"Mechanics", 301, 100, 4.0},
            {"Electronics", 302, 100, 3.5},
            {"Quantum Physics", 303, 100, 4.0},
            {"Thermodynamics", 304, 100, 3.5}
        };
        
        // Initialize classes properly
        classes[0].classId = 101;
        classes[0].className = "Computer Science";
        classes[0].department = "Engineering";
        classes[0].year = 2024;
        classes[0].subjectCount = 4;
        for(int i = 0; i < 4; i++) classes[0].subjects[i] = csSubjects[i];
        
        classes[1].classId = 102;
        classes[1].className = "Business Admin";
        classes[1].department = "Commerce";
        classes[1].year = 2024;
        classes[1].subjectCount = 4;
        for(int i = 0; i < 4; i++) classes[1].subjects[i] = businessSubjects[i];
        
        classes[2].classId = 103;
        classes[2].className = "Physics";
        classes[2].department = "Science";
        classes[2].year = 2024;
        classes[2].subjectCount = 4;
        for(int i = 0; i < 4; i++) classes[2].subjects[i] = physicsSubjects[i];
        
        saveClassesToFile();
        return;
    }
    
    file.read(reinterpret_cast<char*>(&classCount), sizeof(classCount));
    for (int i = 0; i < classCount; i++) {
        file.read(reinterpret_cast<char*>(&classes[i]), sizeof(Class));
    }
    
    file.close();
    cout << "✓ Class data loaded successfully! (" << classCount << " classes)\n";
}

// Create backup of all data
void backupData() {
    // Create backup directory if it doesn't exist
    #ifdef _WIN32
        system(("mkdir " + BACKUP_DIR + " 2>nul").c_str());
    #else
        system(("mkdir -p " + BACKUP_DIR).c_str());
    #endif
    
    string timestamp = to_string(time(nullptr));
    string backupFile = BACKUP_DIR + "backup_" + timestamp + ".dat";
    
    ofstream file(backupFile, ios::binary);
    if (!file) {
        cout << "Error: Cannot create backup file!\n";
        return;
    }
    
    // Save metadata
    file.write(reinterpret_cast<char*>(&studentCount), sizeof(studentCount));
    file.write(reinterpret_cast<char*>(&classCount), sizeof(classCount));
    
    // Save all student data
    for (int i = 0; i < studentCount; i++) {
        file.write(reinterpret_cast<char*>(&students[i]), sizeof(Student));
    }
    
    // Save all class data
    for (int i = 0; i < classCount; i++) {
        file.write(reinterpret_cast<char*>(&classes[i]), sizeof(Class));
    }
    
    file.close();
    cout << "✓ Backup created: " << backupFile << "\n";
}

// Export data to CSV format
void exportToCSV() {
    ofstream file("students_export.csv");
    if (!file) {
        cout << "Error: Cannot create CSV file!\n";
        return;
    }
    
    // Write CSV header
    file << "ID,Name,Class,Average,GPA,Grade,Email/Phone\n";
    
    // Write student data
    for (int i = 0; i < studentCount; i++) {
        file << students[i].id << ","
             << "\"" << students[i].name << "\",";
        
        // Find class name
        string className = "Unknown";
        for (int j = 0; j < classCount; j++) {
            if (classes[j].classId == students[i].classId) {
                className = classes[j].className;
                break;
            }
        }
        
        file << "\"" << className << "\","
             << fixed << setprecision(2) << students[i].average << ","
             << students[i].gpa << ","
             << students[i].grade << ",";
        
        if (students[i].contactType == 0) {
            file << students[i].contact.email;
        } else {
            file << students[i].contact.phone;
        }
        
        file << "\n";
    }
    
    file.close();
    cout << "✓ Data exported to students_export.csv successfully!\n";
}

// ====================================================================
// SECTION 4: MULTIPLE CLASSES & SUBJECTS FUNCTIONS (300+ lines)
// ====================================================================
/**
 * STUDENT ASSIGNMENT:
 * 1. How would you modify this to support unlimited classes/subjects?
 * 2. What data structure would be better than arrays for dynamic growth?
 * 3. How can we add prerequisite checking for subjects?
 */

// Add a new class with subjects
void addClass() {
    if (classCount >= MAX_CLASSES) {
        cout << "❌ Error: Maximum number of classes reached (" << MAX_CLASSES << ")!\n";
        return;
    }
    
    Class newClass;
    cout << "\n=== ADD NEW CLASS ===\n";
    
    // Get class ID with validation
    while (true) {
        cout << "Enter Class ID (100-999): ";
        cin >> newClass.classId;
        
        if (cin.fail() || newClass.classId < 100 || newClass.classId > 999) {
            cin.clear();
            cin.ignore(numeric_limits<streamsize>::max(), '\n');
            cout << "Invalid ID! Must be between 100-999.\n";
            continue;
        }
        
        // Check for duplicate class ID
        bool duplicate = false;
        for (int i = 0; i < classCount; i++) {
            if (classes[i].classId == newClass.classId) {
                cout << "Class ID already exists! Choose a different ID.\n";
                duplicate = true;
                break;
            }
        }
        
        if (!duplicate) break;
    }
    
    cin.ignore();
    cout << "Enter Class Name: ";
    getline(cin, newClass.className);
    
    cout << "Enter Department: ";
    getline(cin, newClass.department);
    
    cout << "Enter Academic Year (2000-2100): ";
    cin >> newClass.year;
    
    // Validate number of subjects
    cout << "Enter number of subjects (1-10): ";
    cin >> newClass.subjectCount;
    
    while (newClass.subjectCount < 1 || newClass.subjectCount > 10) {
        cin.clear();
        cin.ignore(numeric_limits<streamsize>::max(), '\n');
        cout << "Invalid number! Enter 1-10: ";
        cin >> newClass.subjectCount;
    }
    
    // Input subjects
    cout << "\n=== ENTER SUBJECT DETAILS ===\n";
    for (int i = 0; i < newClass.subjectCount; i++) {
        cout << "\nSubject " << (i + 1) << ":\n";
        cin.ignore();
        
        cout << "  Subject Name: ";
        getline(cin, newClass.subjects[i].name);
        
        cout << "  Subject Code (1-999): ";
        cin >> newClass.subjects[i].code;
        
        cout << "  Maximum Marks (1-200): ";
        cin >> newClass.subjects[i].maxMarks;
        
        cout << "  Weightage (0.5-5.0): ";
        cin >> newClass.subjects[i].weightage;
    }
    
    // Add class to array
    classes[classCount++] = newClass;
    saveClassesToFile();
    cout << "\n✓ Class '" << newClass.className << "' added successfully!\n";
}

// Display all classes
void displayAllClasses() {
    if (classCount == 0) {
        cout << "\nNo classes available. Add classes first.\n";
        return;
    }
    
    cout << "\n" << string(80, '=') << "\n";
    cout << "                         ALL CLASSES\n";
    cout << string(80, '=') << "\n";
    cout << left << setw(10) << "ID"
         << setw(25) << "Class Name"
         << setw(20) << "Department"
         << setw(10) << "Year"
         << setw(15) << "Subjects"
         << "\n";
    cout << string(80, '-') << "\n";
    
    for (int i = 0; i < classCount; i++) {
        cout << left << setw(10) << classes[i].classId
             << setw(25) << (classes[i].className.length() > 24 ? 
                            classes[i].className.substr(0, 24) + "." : classes[i].className)
             << setw(20) << (classes[i].department.length() > 19 ? 
                            classes[i].department.substr(0, 19) + "." : classes[i].department)
             << setw(10) << classes[i].year
             << setw(15) << classes[i].subjectCount
             << "\n";
    }
    cout << string(80, '=') << "\n";
}

// Display detailed information about a specific class
void displayClassDetails(int classId) {
    int index = -1;
    for (int i = 0; i < classCount; i++) {
        if (classes[i].classId == classId) {
            index = i;
            break;
        }
    }
    
    if (index == -1) {
        cout << "❌ Class not found!\n";
        return;
    }
    
    Class& c = classes[index];
    
    cout << "\n" << string(60, '=') << "\n";
    cout << "        CLASS DETAILS: " << c.className << "\n";
    cout << string(60, '=') << "\n";
    cout << "Class ID     : " << c.classId << "\n";
    cout << "Class Name   : " << c.className << "\n";
    cout << "Department   : " << c.department << "\n";
    cout << "Academic Year: " << c.year << "\n";
    cout << "Total Subjects: " << c.subjectCount << "\n";
    
    cout << "\n--- SUBJECTS ---\n";
    cout << left << setw(25) << "Subject Name"
         << setw(15) << "Code"
         << setw(15) << "Max Marks"
         << setw(12) << "Weightage"
         << "\n";
    cout << string(67, '-') << "\n";
    
    for (int i = 0; i < c.subjectCount; i++) {
        cout << left << setw(25) << (c.subjects[i].name.length() > 24 ? 
                                   c.subjects[i].name.substr(0, 24) + "." : c.subjects[i].name)
             << setw(15) << c.subjects[i].code
             << setw(15) << c.subjects[i].maxMarks
             << setw(12) << fixed << setprecision(2) << c.subjects[i].weightage
             << "\n";
    }
    
    // Count students in this class
    int studentInClass = 0;
    for (int i = 0; i < studentCount; i++) {
        if (students[i].classId == classId) {
            studentInClass++;
        }
    }
    
    cout << "\nStudents enrolled: " << studentInClass << "\n";
    cout << string(60, '=') << "\n";
}

// Find class by ID
int findClassById(int classId) {
    for (int i = 0; i < classCount; i++) {
        if (classes[i].classId == classId) {
            return i;
        }
    }
    return -1;
}

// Edit an existing class
void editClass() {
    int classId;
    cout << "Enter Class ID to edit: ";
    cin >> classId;
    
    int index = findClassById(classId);
    if (index == -1) {
        cout << "❌ Class not found!\n";
        return;
    }
    
    Class& c = classes[index];
    cout << "\nEditing Class: " << c.className << "\n";
    
    int choice;
    cout << "What do you want to edit?\n";
    cout << "1. Class Name\n";
    cout << "2. Department\n";
    cout << "3. Academic Year\n";
    cout << "4. Subjects\n";
    cout << "5. All Information\n";
    cout << "Enter choice: ";
    cin >> choice;
    
    switch (choice) {
        case 1:
            cin.ignore();
            cout << "Current Name: " << c.className << "\n";
            cout << "Enter new Name: ";
            getline(cin, c.className);
            break;
            
        case 2:
            cin.ignore();
            cout << "Current Department: " << c.department << "\n";
            cout << "Enter new Department: ";
            getline(cin, c.department);
            break;
            
        case 3:
            cout << "Current Year: " << c.year << "\n";
            cout << "Enter new Year: ";
            cin >> c.year;
            break;
            
        case 4:
            cout << "Enter new number of subjects (1-10): ";
            cin >> c.subjectCount;
            cin.ignore();
            
            for (int i = 0; i < c.subjectCount; i++) {
                cout << "\nSubject " << (i + 1) << ":\n";
                cout << "  Subject Name: ";
                getline(cin, c.subjects[i].name);
                
                cout << "  Subject Code: ";
                cin >> c.subjects[i].code;
                
                cout << "  Maximum Marks: ";
                cin >> c.subjects[i].maxMarks;
                
                cout << "  Weightage: ";
                cin >> c.subjects[i].weightage;
                cin.ignore();
            }
            break;
            
        case 5:
            cin.ignore();
            cout << "Enter new Class Name: ";
            getline(cin, c.className);
            
            cout << "Enter new Department: ";
            getline(cin, c.department);
            
            cout << "Enter new Academic Year: ";
            cin >> c.year;
            
            cout << "Enter new number of subjects (1-10): ";
            cin >> c.subjectCount;
            cin.ignore();
            
            for (int i = 0; i < c.subjectCount; i++) {
                cout << "\nSubject " << (i + 1) << ":\n";
                cout << "  Subject Name: ";
                getline(cin, c.subjects[i].name);
                
                cout << "  Subject Code: ";
                cin >> c.subjects[i].code;
                
                cout << "  Maximum Marks: ";
                cin >> c.subjects[i].maxMarks;
                
                cout << "  Weightage: ";
                cin >> c.subjects[i].weightage;
                cin.ignore();
            }
            break;
            
        default:
            cout << "Invalid choice!\n";
            return;
    }
    
    saveClassesToFile();
    cout << "\n✓ Class updated successfully!\n";
}

// Delete a class
void deleteClass() {
    int classId;
    cout << "Enter Class ID to delete: ";
    cin >> classId;
    
    int index = findClassById(classId);
    if (index == -1) {
        cout << "❌ Class not found!\n";
        return;
    }
    
    // Check if any students are enrolled in this class
    for (int i = 0; i < studentCount; i++) {
        if (students[i].classId == classId) {
            cout << "❌ Cannot delete class! " 
                 << "There are students enrolled in this class.\n";
            cout << "First move or delete those students.\n";
            return;
        }
    }
    
    cout << "Are you sure you want to delete class '" 
         << classes[index].className << "'? (y/n): ";
    char confirm;
    cin >> confirm;
    
    if (confirm == 'y' || confirm == 'Y') {
        // Shift all classes after the deleted one
        for (int i = index; i < classCount - 1; i++) {
            classes[i] = classes[i + 1];
        }
        classCount--;
        
        saveClassesToFile();
        cout << "✓ Class deleted successfully!\n";
    } else {
        cout << "Deletion cancelled.\n";
    }
}

// ====================================================================
// SECTION 5: SEARCH AND UPDATE FUNCTIONALITY (200+ lines)
// ====================================================================
/**
 * STUDENT ASSIGNMENT:
 * 1. Implement a binary search algorithm for searching by ID
 * 2. Add case-insensitive search for names
 * 3. How would you implement search by date of birth range?
 */

// Search student by ID
void searchById() {
    int id;
    cout << "Enter Student ID to search: ";
    cin >> id;
    
    bool found = false;
    for (int i = 0; i < studentCount; i++) {
        if (students[i].id == id) {
            cout << "\n✓ Student Found!\n";
            cout << string(50, '-') << "\n";
            cout << "ID          : " << students[i].id << "\n";
            cout << "Name        : " << students[i].name << "\n";
            
            // Find class name
            string className = "Unknown";
            for (int j = 0; j < classCount; j++) {
                if (classes[j].classId == students[i].classId) {
                    className = classes[j].className;
                    break;
                }
            }
            
            cout << "Class       : " << className << "\n";
            cout << "Date of Birth: " << students[i].dob.day << "/" 
                 << students[i].dob.month << "/" << students[i].dob.year << "\n";
            cout << "Average     : " << fixed << setprecision(2) << students[i].average << "%\n";
            cout << "GPA         : " << fixed << setprecision(2) << students[i].gpa << "\n";
            cout << "Grade       : " << students[i].grade << "\n";
            
            if (students[i].contactType == 0) {
                cout << "Email       : " << students[i].contact.email << "\n";
            } else {
                cout << "Phone       : " << students[i].contact.phone << "\n";
            }
            
            cout << string(50, '-') << "\n";
            found = true;
            break;
        }
    }
    
    if (!found) {
        cout << "❌ Student with ID " << id << " not found!\n";
    }
}

// Search student by name (partial match)
void searchByName() {
    string name;
    cout << "Enter student name to search: ";
    cin.ignore();
    getline(cin, name);
    
    bool found = false;
    cout << "\nSearch Results for '" << name << "':\n";
    cout << string(80, '-') << "\n";
    
    for (int i = 0; i < studentCount; i++) {
        // Case-insensitive partial match
        string studentName = students[i].name;
        string searchName = name;
        transform(studentName.begin(), studentName.end(), studentName.begin(), ::tolower);
        transform(searchName.begin(), searchName.end(), searchName.begin(), ::tolower);
        
        if (studentName.find(searchName) != string::npos) {
            cout << "ID: " << students[i].id 
                 << " | Name: " << students[i].name 
                 << " | Class: ";
            
            // Find class name
            for (int j = 0; j < classCount; j++) {
                if (classes[j].classId == students[i].classId) {
                    cout << classes[j].className;
                    break;
                }
            }
            
            cout << " | Average: " << fixed << setprecision(2) << students[i].average << "%"
                 << " | Grade: " << students[i].grade << "\n";
            found = true;
        }
    }
    
    if (!found) {
        cout << "No students found with name containing '" << name << "'\n";
    }
    cout << string(80, '-') << "\n";
}

// Search students by grade
void searchByGrade() {
    char grade;
    cout << "Enter grade to search (A/B/C/D/F): ";
    cin >> grade;
    grade = toupper(grade);
    
    if (grade != 'A' && grade != 'B' && grade != 'C' && grade != 'D' && grade != 'F') {
        cout << "Invalid grade! Please enter A, B, C, D, or F.\n";
        return;
    }
    
    bool found = false;
    cout << "\nStudents with Grade '" << grade << "':\n";
    cout << string(70, '-') << "\n";
    
    for (int i = 0; i < studentCount; i++) {
        if (students[i].grade == grade) {
            cout << "ID: " << students[i].id 
                 << " | Name: " << students[i].name 
                 << " | Average: " << fixed << setprecision(2) << students[i].average << "%"
                 << " | GPA: " << fixed << setprecision(2) << students[i].gpa << "\n";
            found = true;
        }
    }
    
    if (!found) {
        cout << "No students found with grade '" << grade << "'\n";
    }
    cout << string(70, '-') << "\n";
}

// Search students by class
void searchByClass() {
    int classId;
    cout << "Enter Class ID to search: ";
    cin >> classId;
    
    int classIndex = findClassById(classId);
    if (classIndex == -1) {
        cout << "❌ Class not found!\n";
        return;
    }
    
    bool found = false;
    cout << "\nStudents in Class '" << classes[classIndex].className << "':\n";
    cout << string(80, '-') << "\n";
    
    for (int i = 0; i < studentCount; i++) {
        if (students[i].classId == classId) {
            cout << "ID: " << students[i].id 
                 << " | Name: " << students[i].name 
                 << " | Average: " << fixed << setprecision(2) << students[i].average << "%"
                 << " | Grade: " << students[i].grade << "\n";
            found = true;
        }
    }
    
    if (!found) {
        cout << "No students found in this class.\n";
    } else {
        cout << "\nTotal students in class: ";
        int count = 0;
        for (int i = 0; i < studentCount; i++) {
            if (students[i].classId == classId) count++;
        }
        cout << count << "\n";
    }
    cout << string(80, '-') << "\n";
}

// Advanced search with multiple criteria
void advancedSearch() {
    cout << "\n=== ADVANCED SEARCH ===\n";
    cout << "Search by:\n";
    cout << "1. ID Range\n";
    cout << "2. Average Range\n";
    cout << "3. GPA Range\n";
    cout << "4. Date of Birth Range\n";
    cout << "Enter choice: ";
    
    int choice;
    cin >> choice;
    
    switch (choice) {
        case 1: {
            int minId, maxId;
            cout << "Enter minimum ID: ";
            cin >> minId;
            cout << "Enter maximum ID: ";
            cin >> maxId;
            
            bool found = false;
            for (int i = 0; i < studentCount; i++) {
                if (students[i].id >= minId && students[i].id <= maxId) {
                    if (!found) {
                        cout << "\nStudents with ID between " << minId << " and " << maxId << ":\n";
                        cout << string(80, '-') << "\n";
                        found = true;
                    }
                    cout << "ID: " << students[i].id 
                         << " | Name: " << students[i].name 
                         << " | Average: " << fixed << setprecision(2) << students[i].average << "%\n";
                }
            }
            if (!found) cout << "No students found in this ID range.\n";
            break;
        }
        
        case 2: {
            float minAvg, maxAvg;
            cout << "Enter minimum average: ";
            cin >> minAvg;
            cout << "Enter maximum average: ";
            cin >> maxAvg;
            
            bool found = false;
            for (int i = 0; i < studentCount; i++) {
                if (students[i].average >= minAvg && students[i].average <= maxAvg) {
                    if (!found) {
                        cout << "\nStudents with average between " << minAvg << " and " << maxAvg << ":\n";
                        cout << string(80, '-') << "\n";
                        found = true;
                    }
                    cout << "ID: " << students[i].id 
                         << " | Name: " << students[i].name 
                         << " | Average: " << fixed << setprecision(2) << students[i].average << "%\n";
                }
            }
            if (!found) cout << "No students found in this average range.\n";
            break;
        }
        
        case 3: {
            float minGpa, maxGpa;
            cout << "Enter minimum GPA: ";
            cin >> minGpa;
            cout << "Enter maximum GPA: ";
            cin >> maxGpa;
            
            bool found = false;
            for (int i = 0; i < studentCount; i++) {
                if (students[i].gpa >= minGpa && students[i].gpa <= maxGpa) {
                    if (!found) {
                        cout << "\nStudents with GPA between " << minGpa << " and " << maxGpa << ":\n";
                        cout << string(80, '-') << "\n";
                        found = true;
                    }
                    cout << "ID: " << students[i].id 
                         << " | Name: " << students[i].name 
                         << " | GPA: " << fixed << setprecision(2) << students[i].gpa << "\n";
                }
            }
            if (!found) cout << "No students found in this GPA range.\n";
            break;
        }
        
        case 4: {
            int year;
            cout << "Enter birth year: ";
            cin >> year;
            
            bool found = false;
            for (int i = 0; i < studentCount; i++) {
                if (students[i].dob.year == year) {
                    if (!found) {
                        cout << "\nStudents born in " << year << ":\n";
                        cout << string(80, '-') << "\n";
                        found = true;
                    }
                    cout << "ID: " << students[i].id 
                         << " | Name: " << students[i].name 
                         << " | DOB: " << students[i].dob.day << "/" 
                         << students[i].dob.month << "/" << students[i].dob.year << "\n";
                }
            }
            if (!found) cout << "No students found born in " << year << "\n";
            break;
        }
        
        default:
            cout << "Invalid choice!\n";
    }
}

// ====================================================================
// SECTION 6: DELETE & SORT OPERATIONS (200+ lines)
// ====================================================================
/**
 * STUDENT ASSIGNMENT:
 * 1. Implement bubble sort as an alternative to quick sort
 * 2. Add undo functionality for delete operations
 * 3. How would you implement pagination for sorted results?
 */

// Delete a student by ID
void deleteStudent() {
    int id;
    cout << "Enter Student ID to delete: ";
    cin >> id;
    
    int index = -1;
    for (int i = 0; i < studentCount; i++) {
        if (students[i].id == id) {
            index = i;
            break;
        }
    }
    
    if (index == -1) {
        cout << "❌ Student not found!\n";
        return;
    }
    
    cout << "\nStudent to be deleted:\n";
    cout << "ID: " << students[index].id << "\n";
    cout << "Name: " << students[index].name << "\n";
    cout << "Class: ";
    
    // Find class name
    for (int i = 0; i < classCount; i++) {
        if (classes[i].classId == students[index].classId) {
            cout << classes[i].className << "\n";
            break;
        }
    }
    
    cout << "Average: " << fixed << setprecision(2) << students[index].average << "%\n";
    cout << "\nAre you sure you want to delete this student? (y/n): ";
    
    char confirm;
    cin >> confirm;
    
    if (confirm == 'y' || confirm == 'Y') {
        // Backup the student data before deletion
        Student backup = students[index];
        
        // Shift all students after the deleted one
        for (int i = index; i < studentCount - 1; i++) {
            students[i] = students[i + 1];
        }
        studentCount--;
        
        saveStudentsToFile();
        cout << "✓ Student '" << backup.name << "' deleted successfully!\n";
        
        // Ask if user wants to save the deleted record to a separate file
        cout << "Save deleted record to archive? (y/n): ";
        cin >> confirm;
        
        if (confirm == 'y' || confirm == 'Y') {
            ofstream archive("deleted_students.dat", ios::binary | ios::app);
            if (archive) {
                archive.write(reinterpret_cast<char*>(&backup), sizeof(Student));
                archive.close();
                cout << "✓ Record archived.\n";
            }
        }
    } else {
        cout << "Deletion cancelled.\n";
    }
}

// Delete multiple students by criteria
void deleteMultipleStudents() {
    cout << "\n=== DELETE MULTIPLE STUDENTS ===\n";
    cout << "Delete students by:\n";
    cout << "1. Class\n";
    cout << "2. Grade\n";
    cout << "3. Average below threshold\n";
    cout << "Enter choice: ";
    
    int choice;
    cin >> choice;
    
    vector<int> toDelete;
    
    switch (choice) {
        case 1: {
            int classId;
            cout << "Enter Class ID: ";
            cin >> classId;
            
            for (int i = 0; i < studentCount; i++) {
                if (students[i].classId == classId) {
                    toDelete.push_back(i);
                }
            }
            break;
        }
            
        case 2: {
            char grade;
            cout << "Enter Grade (A/B/C/D/F): ";
            cin >> grade;
            grade = toupper(grade);
            
            for (int i = 0; i < studentCount; i++) {
                if (students[i].grade == grade) {
                    toDelete.push_back(i);
                }
            }
            break;
        }
            
        case 3: {
            float threshold;
            cout << "Enter average threshold (students below this will be deleted): ";
            cin >> threshold;
            
            for (int i = 0; i < studentCount; i++) {
                if (students[i].average < threshold) {
                    toDelete.push_back(i);
                }
            }
            break;
        }
            
        default:
            cout << "Invalid choice!\n";
            return;
    }
    
    if (toDelete.empty()) {
        cout << "No students match the criteria.\n";
        return;
    }
    
    cout << "\nFound " << toDelete.size() << " students to delete:\n";
    for (int idx : toDelete) {
        cout << "  - " << students[idx].name << " (ID: " << students[idx].id 
             << ", Avg: " << fixed << setprecision(2) << students[idx].average << "%)\n";
    }
    
    cout << "\nAre you sure you want to delete these students? (y/n): ";
    char confirm;
    cin >> confirm;
    
    if (confirm == 'y' || confirm == 'Y') {
        // Sort indices in descending order for safe deletion
        sort(toDelete.rbegin(), toDelete.rend());
        
        int deletedCount = 0;
        for (int idx : toDelete) {
            // Shift students
            for (int i = idx; i < studentCount - 1; i++) {
                students[i] = students[i + 1];
            }
            studentCount--;
            deletedCount++;
        }
        
        saveStudentsToFile();
        cout << "✓ " << deletedCount << " students deleted successfully!\n";
    } else {
        cout << "Deletion cancelled.\n";
    }
}

// Bubble sort implementation (for educational purposes)
void bubbleSortStudents(int sortType, bool ascending) {
    for (int i = 0; i < studentCount - 1; i++) {
        for (int j = 0; j < studentCount - i - 1; j++) {
            bool shouldSwap = false;
            
            switch (sortType) {
                case 1: // Sort by ID
                    if (ascending) {
                        shouldSwap = students[j].id > students[j + 1].id;
                    } else {
                        shouldSwap = students[j].id < students[j + 1].id;
                    }
                    break;
                    
                case 2: // Sort by name
                    if (ascending) {
                        shouldSwap = students[j].name > students[j + 1].name;
                    } else {
                        shouldSwap = students[j].name < students[j + 1].name;
                    }
                    break;
                    
                case 3: // Sort by average
                    if (ascending) {
                        shouldSwap = students[j].average > students[j + 1].average;
                    } else {
                        shouldSwap = students[j].average < students[j + 1].average;
                    }
                    break;
                    
                case 4: // Sort by GPA
                    if (ascending) {
                        shouldSwap = students[j].gpa > students[j + 1].gpa;
                    } else {
                        shouldSwap = students[j].gpa < students[j + 1].gpa;
                    }
                    break;
            }
            
            if (shouldSwap) {
                swap(students[j], students[j + 1]);
            }
        }
    }
}

// Merge sort helper function
void merge(Student arr[], int left, int mid, int right, int sortType, bool ascending) {
    int n1 = mid - left + 1;
    int n2 = right - mid;
    
    Student* L = new Student[n1];
    Student* R = new Student[n2];
    
    for (int i = 0; i < n1; i++)
        L[i] = arr[left + i];
    for (int j = 0; j < n2; j++)
        R[j] = arr[mid + 1 + j];
    
    int i = 0, j = 0, k = left;
    
    while (i < n1 && j < n2) {
        bool condition;
        
        switch (sortType) {
            case 1: // ID
                if (ascending) condition = L[i].id <= R[j].id;
                else condition = L[i].id >= R[j].id;
                break;
            case 2: // Name
                if (ascending) condition = L[i].name <= R[j].name;
                else condition = L[i].name >= R[j].name;
                break;
            case 3: // Average
                if (ascending) condition = L[i].average <= R[j].average;
                else condition = L[i].average >= R[j].average;
                break;
            case 4: // GPA
                if (ascending) condition = L[i].gpa <= R[j].gpa;
                else condition = L[i].gpa >= R[j].gpa;
                break;
            default:
                condition = L[i].id <= R[j].id;
        }
        
        if (condition) {
            arr[k] = L[i];
            i++;
        } else {
            arr[k] = R[j];
            j++;
        }
        k++;
    }
    
    while (i < n1) {
        arr[k] = L[i];
        i++;
        k++;
    }
    
    while (j < n2) {
        arr[k] = R[j];
        j++;
        k++;
    }
    
    delete[] L;
    delete[] R;
}

// Merge sort main function
void mergeSort(Student arr[], int left, int right, int sortType, bool ascending) {
    if (left < right) {
        int mid = left + (right - left) / 2;
        
        mergeSort(arr, left, mid, sortType, ascending);
        mergeSort(arr, mid + 1, right, sortType, ascending);
        merge(arr, left, mid, right, sortType, ascending);
    }
}

// Quick sort partition function
int partition(Student arr[], int low, int high, int sortType, bool ascending) {
    Student pivot = arr[high];
    int i = low - 1;
    
    for (int j = low; j < high; j++) {
        bool condition = false;
        
        switch (sortType) {
            case 1: // ID
                if (ascending) condition = arr[j].id < pivot.id;
                else condition = arr[j].id > pivot.id;
                break;
            case 2: // Name
                if (ascending) condition = arr[j].name < pivot.name;
                else condition = arr[j].name > pivot.name;
                break;
            case 3: // Average
                if (ascending) condition = arr[j].average < pivot.average;
                else condition = arr[j].average > pivot.average;
                break;
            case 4: // GPA
                if (ascending) condition = arr[j].gpa < pivot.gpa;
                else condition = arr[j].gpa > pivot.gpa;
                break;
        }
        
        if (condition) {
            i++;
            swap(arr[i], arr[j]);
        }
    }
    
    swap(arr[i + 1], arr[high]);
    return i + 1;
}

// Quick sort main function
void quickSort(Student arr[], int low, int high, int sortType, bool ascending) {
    if (low < high) {
        int pi = partition(arr, low, high, sortType, ascending);
        quickSort(arr, low, pi - 1, sortType, ascending);
        quickSort(arr, pi + 1, high, sortType, ascending);
    }
}

// Sort students with different algorithms
void sortStudents() {
    cout << "\n=== SORT STUDENTS ===\n";
    cout << "Sort by:\n";
    cout << "1. Student ID\n";
    cout << "2. Name\n";
    cout << "3. Average Marks\n";
    cout << "4. GPA\n";
    cout << "Enter choice: ";
    
    int sortBy;
    cin >> sortBy;
    
    if (sortBy < 1 || sortBy > 4) {
        cout << "Invalid choice!\n";
        return;
    }
    
    cout << "Sort order:\n";
    cout << "1. Ascending (A-Z, Low to High)\n";
    cout << "2. Descending (Z-A, High to Low)\n";
    cout << "Enter choice: ";
    
    int order;
    cin >> order;
    
    if (order != 1 && order != 2) {
        cout << "Invalid choice!\n";
        return;
    }
    
    bool ascending = (order == 1);
    
    cout << "\nSelect sorting algorithm:\n";
    cout << "1. Quick Sort (Fastest)\n";
    cout << "2. Bubble Sort (Educational)\n";
    cout << "3. Merge Sort (Stable)\n";
    cout << "Enter choice: ";
    
    int algo;
    cin >> algo;
    
    cout << "\nSorting... ";
    
    switch (algo) {
        case 1:
            quickSort(students, 0, studentCount - 1, sortBy, ascending);
            cout << "using Quick Sort\n";
            break;
            
        case 2:
            bubbleSortStudents(sortBy, ascending);
            cout << "using Bubble Sort\n";
            break;
            
        case 3:
            mergeSort(students, 0, studentCount - 1, sortBy, ascending);
            cout << "using Merge Sort\n";
            break;
            
        default:
            cout << "Invalid algorithm choice! Using Bubble Sort.\n";
            bubbleSortStudents(sortBy, ascending);
            break;
    }
    
    cout << "✓ Students sorted successfully!\n";
}

// Display sorted students with pagination
void displaySortedStudents() {
    if (studentCount == 0) {
        cout << "No students to display.\n";
        return;
    }
    
    int pageSize = 10;
    int totalPages = (studentCount + pageSize - 1) / pageSize;
    int currentPage = 1;
    
    while (true) {
        cout << "\nPage " << currentPage << " of " << totalPages << "\n";
        cout << string(100, '-') << "\n";
        cout << left << setw(10) << "ID" 
             << setw(25) << "Name" 
             << setw(20) << "Class" 
             << setw(12) << "Average" 
             << setw(10) << "GPA" 
             << setw(8) << "Grade" 
             << "\n";
        cout << string(100, '-') << "\n";
        
        int start = (currentPage - 1) * pageSize;
        int end = min(start + pageSize, studentCount);
        
        for (int i = start; i < end; i++) {
            // Find class name
            string className = "Unknown";
            for (int j = 0; j < classCount; j++) {
                if (classes[j].classId == students[i].classId) {
                    className = classes[j].className;
                    break;
                }
            }
            
            cout << left << setw(10) << students[i].id
                 << setw(25) << (students[i].name.length() > 24 ? 
                                students[i].name.substr(0, 24) + "." : students[i].name)
                 << setw(20) << (className.length() > 19 ? 
                                className.substr(0, 19) + "." : className)
                 << setw(12) << fixed << setprecision(2) << students[i].average
                 << setw(10) << fixed << setprecision(2) << students[i].gpa
                 << setw(8) << students[i].grade
                 << "\n";
        }
        
        cout << string(100, '-') << "\n";
        
        if (totalPages > 1) {
            cout << "\nNavigation: (N)ext page, (P)revious page, (F)irst page, (L)ast page, (E)xit\n";
            cout << "Enter choice: ";
            
            char nav;
            cin >> nav;
            nav = tolower(nav);
            
            switch (nav) {
                case 'n':
                    if (currentPage < totalPages) currentPage++;
                    else cout << "Already on last page!\n";
                    break;
                case 'p':
                    if (currentPage > 1) currentPage--;
                    else cout << "Already on first page!\n";
                    break;
                case 'f':
                    currentPage = 1;
                    break;
                case 'l':
                    currentPage = totalPages;
                    break;
                case 'e':
                    return;
                default:
                    cout << "Invalid choice!\n";
            }
        } else {
            cout << "\nPress Enter to continue...";
            cin.ignore();
            cin.get();
            break;
        }
    }
}

// ====================================================================
// SECTION 7: HELPER FUNCTIONS (150 lines)
// ====================================================================
/**
 * STUDENT ASSIGNMENT:
 * 1. Add more comprehensive date validation
 * 2. Implement email format validation
 * 3. Create a function to calculate age from date of birth
 */

// Calculate grade from average
char calculateGrade(float average) {
    if (average >= 90) return 'A';
    if (average >= 80) return 'B';
    if (average >= 70) return 'C';
    if (average >= 60) return 'D';
    return 'F';
}

// Calculate GPA from marks and subject weightages
float calculateGPA(float marks[], Subject subjects[], int count) {
    if (count == 0) return 0.0;
    
    float totalPoints = 0;
    float totalWeightage = 0;
    
    for (int i = 0; i < count; i++) {
        float percentage = (marks[i] / subjects[i].maxMarks) * 100;
        float gradePoints;
        
        if (percentage >= 90) gradePoints = 4.0;
        else if (percentage >= 80) gradePoints = 3.0;
        else if (percentage >= 70) gradePoints = 2.0;
        else if (percentage >= 60) gradePoints = 1.0;
        else gradePoints = 0.0;
        
        totalPoints += gradePoints * subjects[i].weightage;
        totalWeightage += subjects[i].weightage;
    }
    
    return totalWeightage > 0 ? totalPoints / totalWeightage : 0.0;
}

// Get valid integer input
int getValidInt(string prompt, int min, int max) {
    int value;
    while (true) {
        cout << prompt;
        cin >> value;
        
        if (cin.fail() || value < min || value > max) {
            cin.clear();
            cin.ignore(numeric_limits<streamsize>::max(), '\n');
            cout << "Invalid input! Enter a value between " << min << " and " << max << ".\n";
        } else {
            cin.ignore(numeric_limits<streamsize>::max(), '\n');
            return value;
        }
    }
}

// Get valid float input
float getValidFloat(string prompt, float min, float max) {
    float value;
    while (true) {
        if (!prompt.empty()) cout << prompt;
        cin >> value;
        
        if (cin.fail() || value < min || value > max) {
            cin.clear();
            cin.ignore(numeric_limits<streamsize>::max(), '\n');
            cout << "Invalid input! Enter a value between " << min << " and " << max << ".\n";
        } else {
            cin.ignore(numeric_limits<streamsize>::max(), '\n');
            return value;
        }
    }
}

// Get valid date
Date getValidDate() {
    Date d;
    
    d.year = getValidInt("Year (1900-2024): ", 1900, 2024);
    d.month = getValidInt("Month (1-12): ", 1, 12);
    
    // Determine days in month
    int maxDays;
    if (d.month == 2) {
        // Simple leap year check
        bool isLeap = (d.year % 4 == 0 && d.year % 100 != 0) || (d.year % 400 == 0);
        maxDays = isLeap ? 29 : 28;
    } else if (d.month == 4 || d.month == 6 || d.month == 9 || d.month == 11) {
        maxDays = 30;
    } else {
        maxDays = 31;
    }
    
    d.day = getValidInt("Day (1-" + to_string(maxDays) + "): ", 1, maxDays);
    
    return d;
}

// Get valid contact information
void getValidContact(Student& s) {
    cout << "\nContact Information:\n";
    cout << "1. Email\n";
    cout << "2. Phone\n";
    
    s.contactType = getValidInt("Choose contact type (1-2): ", 1, 2) - 1; // Convert to 0-based
    
    cin.ignore(numeric_limits<streamsize>::max(), '\n');
    if (s.contactType == 0) { // Email
        while (true) {
            cout << "Enter Email: ";
            cin.getline(s.contact.email, 50);
            
            // Basic email validation
            string email = s.contact.email;
            if (email.find('@') != string::npos && email.find('.') != string::npos) {
                break;
            } else {
                cout << "Invalid email format! Please include '@' and '.'\n";
            }
        }
    } else { // Phone
        cout << "Enter Phone: ";
        cin.getline(s.contact.phone, 15);
    }
}

// Calculate age from date of birth
int calculateAge(Date dob) {
    time_t now = time(0);
    tm* currentTime = localtime(&now);
    
    int currentYear = currentTime->tm_year + 1900;
    int currentMonth = currentTime->tm_mon + 1;
    int currentDay = currentTime->tm_mday;
    
    int age = currentYear - dob.year;
    
    if (currentMonth < dob.month || (currentMonth == dob.month && currentDay < dob.day)) {
        age--;
    }
    
    return age;
}

// ====================================================================
// SECTION 8: MENU-DRIVEN INTERFACE (300+ lines)
// ====================================================================
/**
 * STUDENT ASSIGNMENT:
 * 1. Add a login system with different user roles
 * 2. Implement a settings menu for system configuration
 * 3. Add color to the console output for better UX
 */

// Display main menu
void displayMainMenu() {
    cout << "\n" << string(70, '=') << "\n";
    cout << "           STUDENT MANAGEMENT SYSTEM v2.0\n";
    cout << string(70, '=') << "\n";
    cout << "1.  Add New Student\n";
    cout << "2.  Display All Students\n";
    cout << "3.  Search Students\n";
    cout << "4.  Update Student Information\n";
    cout << "5.  Delete Student\n";
    cout << "6.  Sort Students\n";
    cout << "7.  Display Sorted Students\n";
    cout << "8.  Manage Classes\n";
    cout << "9.  View Statistics\n";
    cout << "10. File Operations\n";
    cout << "11. System Utilities\n";
    cout << "0.  Exit Program\n";
    cout << string(70, '=') << "\n";
    cout << "Students: " << studentCount << " | Classes: " << classCount << "\n";
    cout << string(70, '=') << "\n";
    cout << "Enter your choice (0-11): ";
}

// Search menu
void searchMenu() {
    int choice;
    do {
        cout << "\n=== SEARCH MENU ===\n";
        cout << "1. Search by Student ID\n";
        cout << "2. Search by Name\n";
        cout << "3. Search by Grade\n";
        cout << "4. Search by Class\n";
        cout << "5. Advanced Search\n";
        cout << "6. Return to Main Menu\n";
        cout << "Enter choice: ";
        cin >> choice;
        
        switch (choice) {
            case 1:
                searchById();
                break;
            case 2:
                searchByName();
                break;
            case 3:
                searchByGrade();
                break;
            case 4:
                searchByClass();
                break;
            case 5:
                advancedSearch();
                break;
            case 6:
                cout << "Returning to main menu...\n";
                break;
            default:
                cout << "Invalid choice! Please try again.\n";
        }
        
        if (choice != 6) {
            cout << "\nPress Enter to continue...";
            cin.ignore();
            cin.get();
        }
    } while (choice != 6);
}

// Class management menu
void classManagementMenu() {
    int choice;
    do {
        cout << "\n=== CLASS MANAGEMENT ===\n";
        cout << "1. Add New Class\n";
        cout << "2. Display All Classes\n";
        cout << "3. View Class Details\n";
        cout << "4. Edit Class\n";
        cout << "5. Delete Class\n";
        cout << "6. Return to Main Menu\n";
        cout << "Enter choice: ";
        cin >> choice;
        
        switch (choice) {
            case 1:
                addClass();
                break;
            case 2:
                displayAllClasses();
                break;
            case 3: {
                int classId;
                cout << "Enter Class ID: ";
                cin >> classId;
                displayClassDetails(classId);
                break;
            }
            case 4:
                editClass();
                break;
            case 5:
                deleteClass();
                break;
            case 6:
                cout << "Returning to main menu...\n";
                break;
            default:
                cout << "Invalid choice!\n";
        }
        
        if (choice != 6) {
            cout << "\nPress Enter to continue...";
            cin.ignore();
            cin.get();
        }
    } while (choice != 6);
}

// File operations menu
void fileOperationsMenu() {
    int choice;
    do {
        cout << "\n=== FILE OPERATIONS ===\n";
        cout << "1. Save Data\n";
        cout << "2. Create Backup\n";
        cout << "3. Export to CSV\n";
        cout << "4. Load from Backup\n";
        cout << "5. Return to Main Menu\n";
        cout << "Enter choice: ";
        cin >> choice;
        
        switch (choice) {
            case 1:
                saveStudentsToFile();
                saveClassesToFile();
                break;
            case 2:
                backupData();
                break;
            case 3:
                exportToCSV();
                break;
            case 4:
                cout << "This feature is under development.\n";
                break;
            case 5:
                cout << "Returning to main menu...\n";
                break;
            default:
                cout << "Invalid choice!\n";
        }
        
        if (choice != 5) {
            cout << "\nPress Enter to continue...";
            cin.ignore();
            cin.get();
        }
    } while (choice != 5);
}

// Statistics menu
void statisticsMenu() {
    if (studentCount == 0) {
        cout << "No data available for statistics.\n";
        return;
    }
    
    cout << "\n=== SYSTEM STATISTICS ===\n";
    cout << string(50, '=') << "\n";
    
    // Basic statistics
    cout << "Total Students: " << studentCount << "\n";
    cout << "Total Classes: " << classCount << "\n";
    
    // Grade distribution
    int gradeA = 0, gradeB = 0, gradeC = 0, gradeD = 0, gradeF = 0;
    float totalAverage = 0, totalGPA = 0;
    
    for (int i = 0; i < studentCount; i++) {
        totalAverage += students[i].average;
        totalGPA += students[i].gpa;
        
        switch (students[i].grade) {
            case 'A': gradeA++; break;
            case 'B': gradeB++; break;
            case 'C': gradeC++; break;
            case 'D': gradeD++; break;
            case 'F': gradeF++; break;
        }
    }
    
    cout << "\n--- Academic Performance ---\n";
    cout << "Overall Average: " << fixed << setprecision(2) 
         << totalAverage / studentCount << "%\n";
    cout << "Overall GPA: " << fixed << setprecision(2) 
         << totalGPA / studentCount << "\n";
    
    cout << "\n--- Grade Distribution ---\n";
    cout << "A: " << gradeA << " (" << fixed << setprecision(1) 
         << (gradeA * 100.0 / studentCount) << "%)\n";
    cout << "B: " << gradeB << " (" << fixed << setprecision(1) 
         << (gradeB * 100.0 / studentCount) << "%)\n";
    cout << "C: " << gradeC << " (" << fixed << setprecision(1) 
         << (gradeC * 100.0 / studentCount) << "%)\n";
    cout << "D: " << gradeD << " (" << fixed << setprecision(1) 
         << (gradeD * 100.0 / studentCount) << "%)\n";
    cout << "F: " << gradeF << " (" << fixed << setprecision(1) 
         << (gradeF * 100.0 / studentCount) << "%)\n";
    
    // Class-wise statistics
    cout << "\n--- Class-wise Distribution ---\n";
    for (int i = 0; i < classCount; i++) {
        int count = 0;
        float classAvg = 0;
        
        for (int j = 0; j < studentCount; j++) {
            if (students[j].classId == classes[i].classId) {
                count++;
                classAvg += students[j].average;
            }
        }
        
        if (count > 0) {
            cout << classes[i].className << ": " << count << " students, Avg: "
                 << fixed << setprecision(2) << (classAvg / count) << "%\n";
        }
    }
    
    cout << string(50, '=') << "\n";
}

// System utilities menu
void systemUtilitiesMenu() {
    int choice;
    do {
        cout << "\n=== SYSTEM UTILITIES ===\n";
        cout << "1. Clear All Data\n";
        cout << "2. Rebuild Indexes\n";
        cout << "3. System Information\n";
        cout << "4. Return to Main Menu\n";
        cout << "Enter choice: ";
        cin >> choice;
        
        switch (choice) {
            case 1:
                cout << "WARNING: This will delete all student data!\n";
                cout << "Are you sure? (y/n): ";
                char confirm;
                cin >> confirm;
                if (confirm == 'y' || confirm == 'Y') {
                    studentCount = 0;
                    saveStudentsToFile();
                    cout << "All student data cleared.\n";
                }
                break;
            case 2:
                cout << "Index rebuilding feature is under development.\n";
                break;
            case 3:
                cout << "\n=== SYSTEM INFORMATION ===\n";
                cout << "Maximum Students: " << MAX_STUDENTS << "\n";
                cout << "Maximum Classes: " << MAX_CLASSES << "\n";
                cout << "Data File: " << DATA_FILE << "\n";
                cout << "Classes File: " << CLASS_FILE << "\n";
                cout << "Students in Memory: " << studentCount << "\n";
                cout << "Classes in Memory: " << classCount << "\n";
                cout << "Memory Used: ~" 
                     << (studentCount * sizeof(Student) + classCount * sizeof(Class)) / 1024 
                     << " KB\n";
                break;
            case 4:
                cout << "Returning to main menu...\n";
                break;
            default:
                cout << "Invalid choice!\n";
        }
        
        if (choice != 4) {
            cout << "\nPress Enter to continue...";
            cin.ignore();
            cin.get();
        }
    } while (choice != 4);
}

// Add new student function (simplified for menu)
void addNewStudent() {
    if (studentCount >= MAX_STUDENTS) {
        cout << "❌ Maximum number of students reached!\n";
        return;
    }
    
    Student newStudent;
    
    cout << "\n=== ADD NEW STUDENT ===\n";
    
    // Get student ID
    newStudent.id = getValidInt("Enter Student ID (1000-9999): ", 1000, 9999);
    
    // Check for duplicate ID
    for (int i = 0; i < studentCount; i++) {
        if (students[i].id == newStudent.id) {
            cout << "❌ Student ID already exists!\n";
            return;
        }
    }
    
    cin.ignore(numeric_limits<streamsize>::max(), '\n');
    cout << "Enter Name: ";
    getline(cin, newStudent.name);
    
    cout << "\nDate of Birth:\n";
    newStudent.dob = getValidDate();
    
    getValidContact(newStudent);
    
    // Select class
    displayAllClasses();
    newStudent.classId = getValidInt("Enter Class ID: ", 100, 999);
    
    int classIndex = findClassById(newStudent.classId);
    if (classIndex == -1) {
        cout << "❌ Class not found! Student not added.\n";
        return;
    }
    
    Class& selectedClass = classes[classIndex];
    
    // Input marks
    cout << "\nEnter marks for " << selectedClass.subjectCount << " subjects:\n";
    float total = 0;
    for (int i = 0; i < selectedClass.subjectCount; i++) {
        cout << selectedClass.subjects[i].name << " (Max: " 
             << selectedClass.subjects[i].maxMarks << "): ";
        newStudent.marks[i] = getValidFloat("", 0, selectedClass.subjects[i].maxMarks);
        total += newStudent.marks[i];
    }
    
    // Calculate academic information
    newStudent.average = total / selectedClass.subjectCount;
    newStudent.gpa = calculateGPA(newStudent.marks, selectedClass.subjects, selectedClass.subjectCount);
    newStudent.grade = calculateGrade(newStudent.average);
    
    // Add to array
    students[studentCount++] = newStudent;
    
    // Calculate age
    int age = calculateAge(newStudent.dob);
    
    cout << "\n✓ Student added successfully!\n";
    cout << "  Name: " << newStudent.name << "\n";
    cout << "  Age: " << age << " years\n";
    cout << "  Class: " << selectedClass.className << "\n";
    cout << "  Average: " << fixed << setprecision(2) << newStudent.average << "%\n";
    cout << "  GPA: " << fixed << setprecision(2) << newStudent.gpa << "\n";
    cout << "  Grade: " << newStudent.grade << "\n";
}

// Display all students (simplified)
void displayAllStudentsSimple() {
    if (studentCount == 0) {
        cout << "No students to display.\n";
        return;
    }
    
    cout << "\n" << string(120, '=') << "\n";
    cout << "                           ALL STUDENTS\n";
    cout << string(120, '=') << "\n";
    cout << left << setw(10) << "ID"
         << setw(25) << "Name"
         << setw(20) << "Class"
         << setw(12) << "DOB"
         << setw(10) << "Age"
         << setw(12) << "Average"
         << setw(10) << "GPA"
         << setw(8) << "Grade"
         << "\n";
    cout << string(120, '-') << "\n";
    
    for (int i = 0; i < studentCount; i++) {
        // Find class name
        string className = "Unknown";
        for (int j = 0; j < classCount; j++) {
            if (classes[j].classId == students[i].classId) {
                className = classes[j].className;
                break;
            }
        }
        
        // Calculate age
        int age = calculateAge(students[i].dob);
        
        // Format date
        string dob = to_string(students[i].dob.day) + "/" + 
                     to_string(students[i].dob.month) + "/" + 
                     to_string(students[i].dob.year);
        
        cout << left << setw(10) << students[i].id
             << setw(25) << (students[i].name.length() > 24 ? 
                            students[i].name.substr(0, 24) + "." : students[i].name)
             << setw(20) << (className.length() > 19 ? 
                            className.substr(0, 19) + "." : className)
             << setw(12) << dob
             << setw(10) << age
             << setw(12) << fixed << setprecision(2) << students[i].average
             << setw(10) << fixed << setprecision(2) << students[i].gpa
             << setw(8) << students[i].grade
             << "\n";
    }
    
    cout << string(120, '=') << "\n";
    cout << "Total: " << studentCount << " students\n";
}

// Update student information
void updateStudentInfo() {
    int id;
    cout << "Enter Student ID to update: ";
    cin >> id;
    
    int index = -1;
    for (int i = 0; i < studentCount; i++) {
        if (students[i].id == id) {
            index = i;
            break;
        }
    }
    
    if (index == -1) {
        cout << "❌ Student not found!\n";
        return;
    }
    
    Student& s = students[index];
    
    cout << "\nUpdating: " << s.name << " (ID: " << s.id << ")\n";
    
    int choice;
    cout << "What would you like to update?\n";
    cout << "1. Personal Information\n";
    cout << "2. Academic Information\n";
    cout << "3. Contact Information\n";
    cout << "4. Everything\n";
    cout << "Enter choice: ";
    cin >> choice;
    
    cin.ignore(numeric_limits<streamsize>::max(), '\n');
    
    int classIndex = findClassById(s.classId);
    
    switch (choice) {
        case 1: {
            cout << "Current Name: " << s.name << "\n";
            cout << "Enter new Name: ";
            getline(cin, s.name);
            
            cout << "\nDate of Birth (current: " 
                 << s.dob.day << "/" << s.dob.month << "/" << s.dob.year << ")\n";
            s.dob = getValidDate();
            break;
        }
            
        case 2: {
            if (classIndex != -1) {
                Class& c = classes[classIndex];
                cout << "Enter new marks:\n";
                float total = 0;
                for (int i = 0; i < c.subjectCount; i++) {
                    cout << c.subjects[i].name << " (current: " << s.marks[i] << "): ";
                    s.marks[i] = getValidFloat("", 0, c.subjects[i].maxMarks);
                    total += s.marks[i];
                }
                
                s.average = total / c.subjectCount;
                s.gpa = calculateGPA(s.marks, c.subjects, c.subjectCount);
                s.grade = calculateGrade(s.average);
            }
            break;
        }
            
        case 3:
            getValidContact(s);
            break;
            
        case 4:
            cout << "Enter new Name: ";
            getline(cin, s.name);
            
            cout << "\nDate of Birth:\n";
            s.dob = getValidDate();
            
            getValidContact(s);
            
            if (classIndex != -1) {
                Class& c = classes[classIndex];
                cout << "Enter new marks:\n";
                float total = 0;
                for (int i = 0; i < c.subjectCount; i++) {
                    cout << c.subjects[i].name << ": ";
                    s.marks[i] = getValidFloat("", 0, c.subjects[i].maxMarks);
                    total += s.marks[i];
                }
                
                s.average = total / c.subjectCount;
                s.gpa = calculateGPA(s.marks, c.subjects, c.subjectCount);
                s.grade = calculateGrade(s.average);
            }
            break;
            
        default:
            cout << "Invalid choice!\n";
            return;
    }
    
    saveStudentsToFile();
    cout << "\n✓ Student information updated successfully!\n";
}

// ====================================================================
// SECTION 9: MAIN FUNCTION & PROGRAM ENTRY POINT (50 lines)
// ====================================================================
int main() {
    cout << "\n" << string(60, '*') << "\n";
    cout << "    WELCOME TO STUDENT MANAGEMENT SYSTEM\n";
    cout << "            Developed for C++ Course\n";
    cout << string(60, '*') << "\n";
    
    // Load data
    cout << "\nLoading system data...\n";
    loadClassesFromFile();
    loadStudentsFromFile();
    
    int choice;
    
    // Main program loop
    do {
        displayMainMenu();
        cin >> choice;
        
        switch (choice) {
            case 1:
                addNewStudent();
                break;
            case 2:
                displayAllStudentsSimple();
                break;
            case 3:
                searchMenu();
                break;
            case 4:
                updateStudentInfo();
                break;
            case 5:
                deleteStudent();
                break;
            case 6:
                sortStudents();
                break;
            case 7:
                displaySortedStudents();
                break;
            case 8:
                classManagementMenu();
                break;
            case 9:
                statisticsMenu();
                break;
            case 10:
                fileOperationsMenu();
                break;
            case 11:
                systemUtilitiesMenu();
                break;
            case 0:
                cout << "\nSaving all data before exit...\n";
                saveStudentsToFile();
                saveClassesToFile();
                cout << "Thank you for using Student Management System!\n";
                cout << "Goodbye!\n";
                break;
            default:
                cout << "Invalid choice! Please enter 0-11.\n";
                cin.clear();
                cin.ignore(numeric_limits<streamsize>::max(), '\n');
        }
        
        if (choice != 0) {
            cout << "\nPress Enter to continue...";
            cin.ignore();
            cin.get();
        }
        
    } while (choice != 0);
    
    return 0;
}
