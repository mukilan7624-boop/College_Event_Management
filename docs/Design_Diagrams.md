# System Design Diagrams

## College Event Management and Participation Analysis System
**Student Register No.:** 31  
**Class:** 25IT3A&B | **Course:** 25ITT33 Software Engineering & 25ITT32 Java Programming  
**Department:** Information Technology, Velalar College of Engineering and Technology  

---

## 1. Use Case Diagram

The Use Case diagram illustrates the functional interactions between the primary actors (**Administrator**, **Student**, and **Data Analyst / Python Engine**) and the system boundaries.

```mermaid
flowchart LR
    subgraph Actors
        Admin["Administrator / Organizer"]
        Student["Student (Participant)"]
        Analyst["Python Analytics Engine"]
    end

    subgraph College Event Management System
        UC1(["Create / Update / Delete Event"])
        UC2(["View Available Events & Details"])
        UC3(["Register for Event"])
        UC4(["Validate Capacity & Prevent Duplicates"])
        UC5(["Mark Attendance (Present / Absent)"])
        UC6(["Submit Event Feedback & Rating"])
        UC7(["Generate Participation Reports"])
        UC8(["Extract Data & Visualize Analytics"])
    end

    Admin --> UC1
    Admin --> UC2
    Admin --> UC5
    Admin --> UC7

    Student --> UC2
    Student --> UC3
    Student --> UC6

    UC3 -.->|<<include>>| UC4
    UC6 -.->|<<extend>>| UC5

    Analyst --> UC8
    UC8 -.->|<<reads>>| UC7
```

---

## 2. UML Class Diagram

The Class Diagram reflects the Object-Oriented design implemented in the Java source code (`com.eventmanagement.*`), demonstrating encapsulation, inheritance, interfaces, associations, and custom exceptions.

```mermaid
classDiagram
    direction TB

    class Event {
        -int eventId
        -String eventName
        -String category
        -String department
        -String eventDate
        -String venue
        -int maxCapacity
        -int currentRegistrations
        +getEventId() int
        +getEventName() String
        +getCategory() String
        +getMaxCapacity() int
        +getCurrentRegistrations() int
        +hasAvailableSeats() boolean
        +incrementRegistration() void
    }

    class Student {
        -int studentId
        -String registerNo
        -String name
        -String department
        -String email
        -String contactNumber
        +getStudentId() int
        +getRegisterNo() String
        +getName() String
        +getDepartment() String
    }

    class Registration {
        -int registrationId
        -int eventId
        -int studentId
        -String registrationDate
        -String status
        +getRegistrationId() int
        +getEventId() int
        +getStudentId() int
        +getStatus() String
    }

    class Attendance {
        -int attendanceId
        -int registrationId
        -String status
        -String markedAt
        +getAttendanceId() int
        +getRegistrationId() int
        +getStatus() String
    }

    class Feedback {
        -int feedbackId
        -int registrationId
        -int rating
        -String comments
        -String submittedAt
        +getFeedbackId() int
        +getRating() int
        +getComments() String
    }

    class EventService {
        -EventDAO eventDAO
        -RegistrationDAO regDAO
        -FeedbackDAO feedbackDAO
        +registerStudentForEvent(int eventId, int studentId) Registration
        +markAttendance(int registrationId, String status) boolean
        +submitFeedback(int registrationId, int rating, String comments) boolean
        +getEventSummaryReport(int eventId) String
    }

    class DuplicateRegistrationException {
        +DuplicateRegistrationException(String message)
    }

    class EventFullException {
        +EventFullException(String message)
    }

    Event "1" o-- "0..*" Registration : receives
    Student "1" o-- "0..*" Registration : makes
    Registration "1" *-- "0..1" Attendance : tracks
    Registration "1" *-- "0..1" Feedback : collects
    EventService ..> Event : manages
    EventService ..> Registration : processes
    EventService ..> DuplicateRegistrationException : throws
    EventService ..> EventFullException : throws
```

---

## 3. Sequence Diagram: Student Event Registration

This sequence diagram depicts the transaction flow when a student attempts to register for an event, illustrating business validations (capacity check and duplicate check).

```mermaid
sequenceDiagram
    autonumber
    actor Student
    participant Main as Main (CLI)
    participant Service as EventService
    participant RegDAO as RegistrationDAO
    participant EventDAO as EventDAO
    participant DB as Relational Database

    Student->>Main: Request Registration(eventId, studentId)
    Main->>Service: registerStudentForEvent(eventId, studentId)
    
    Service->>EventDAO: getEventById(eventId)
    EventDAO->>DB: SELECT * FROM events WHERE event_id = ?
    DB-->>EventDAO: Event details
    EventDAO-->>Service: Event object

    alt Event has no available seats
        Service-->>Main: throw EventFullException("Capacity reached")
        Main-->>Student: Display "Event Full! Registration rejected."
    else Event has available seats
        Service->>RegDAO: isStudentRegistered(eventId, studentId)
        RegDAO->>DB: SELECT COUNT(*) FROM registrations WHERE event_id=? AND student_id=?
        DB-->>RegDAO: count
        RegDAO-->>Service: boolean exists
        
        alt Student already registered
            Service-->>Main: throw DuplicateRegistrationException("Already registered")
            Main-->>Student: Display "Duplicate Registration Not Allowed!"
        else Valid New Registration
            Service->>RegDAO: insertRegistration(eventId, studentId)
            RegDAO->>DB: INSERT INTO registrations VALUES (...)
            DB-->>RegDAO: Success (registration_id)
            Service->>EventDAO: incrementRegisteredCount(eventId)
            EventDAO->>DB: UPDATE events SET current_registrations = current_registrations + 1
            DB-->>EventDAO: OK
            Service-->>Main: Registration Object
            Main-->>Student: Display "Registration Successful! ID: REG-..."
        end
    end
```

---

## 4. Entity-Relationship (ER) Diagram

```mermaid
erDiagram
    STUDENT ||--o{ REGISTRATION : places
    EVENT ||--o{ REGISTRATION : contains
    REGISTRATION ||--o| ATTENDANCE : has
    REGISTRATION ||--o| FEEDBACK : submits

    STUDENT {
        int student_id PK
        string register_no UK
        string name
        string department
        string email UK
        string contact_number
    }

    EVENT {
        int event_id PK
        string event_name
        string category
        string department
        string event_date
        string venue
        int max_capacity
        int current_registrations
    }

    REGISTRATION {
        int registration_id PK
        int event_id FK
        int student_id FK
        datetime registration_date
        string status
    }

    ATTENDANCE {
        int attendance_id PK
        int registration_id FK,UK
        string status
        datetime marked_at
    }

    FEEDBACK {
        int feedback_id PK
        int registration_id FK,UK
        int rating
        string comments
        datetime submitted_at
    }
```
