# Student Module — Data Model

Scope: entities owned or primarily used by the student-facing flow (proximity notification, boarding, attendance, late/mismatch tracking, chatbot, incident reporting). `Class`, `Bus`, `Route`, `Stop`, and `Trip` appear here only as structural references (no location/GPS fields).

## Entities

### User
| Field | Type | Notes |
|---|---|---|
| id | uuid | PK |
| name | string | |
| email | string | unique |
| phone | string | |
| role | string | |
| password_hash | string | |
| created_at | timestamp | |

### Student
| Field | Type | Notes |
|---|---|---|
| student_id | uuid | PK |
| user_id | uuid | FK → User.id, unique |
| roll_no | string | |
| class_id | uuid | FK → Class.class_id |
| allotted_bus_id | uuid | FK → Bus.bus_id |
| allotted_stop_id | uuid | FK → Stop.stop_id |
| late_count | int | resets monthly |
| created_at | timestamp | |

### Class
| Field | Type | Notes |
|---|---|---|
| class_id | uuid | PK |
| name | string | |

### Bus
| Field | Type | Notes |
|---|---|---|
| bus_id | uuid | PK |
| bus_number | string | |

### Route
| Field | Type | Notes |
|---|---|---|
| route_id | uuid | PK |
| name | string | |

### Stop
| Field | Type | Notes |
|---|---|---|
| stop_id | uuid | PK |
| name | string | |
| route_id | uuid | FK → Route.route_id |
| sequence_no | int | |

### Trip
| Field | Type | Notes |
|---|---|---|
| trip_id | uuid | PK |
| bus_id | uuid | FK → Bus.bus_id |
| route_id | uuid | FK → Route.route_id |
| driver_id | uuid | |
| date | date | |
| start_time | time | |
| end_time | time | |
| status | string | |

### Attendance
| Field | Type | Notes |
|---|---|---|
| attendance_id | uuid | PK |
| student_id | uuid | FK → Student.student_id |
| trip_id | uuid | FK → Trip.trip_id |
| bus_id | uuid | FK → Bus.bus_id |
| marked_at | timestamp | |
| method | string | e.g. QR, manual |
| matched_allocated_bus | boolean | false if boarded a different bus than allotted |

### LateMismatchLog
| Field | Type | Notes |
|---|---|---|
| log_id | uuid | PK |
| student_id | uuid | FK → Student.student_id |
| trip_id | uuid | FK → Trip.trip_id |
| month | string | for monthly reset cycle |
| late_count_snapshot | int | |
| notified_approaching | boolean | e.g. count reaches 2 of 3 |
| notified_threshold | boolean | e.g. count reaches 3 |
| created_at | timestamp | |

### ChatbotQuery
| Field | Type | Notes |
|---|---|---|
| query_id | uuid | PK |
| student_id | uuid | FK → Student.student_id |
| query_type | enum | stop_lookup, bus_lookup, incident_report |
| query_text | string | |
| response_text | string | |
| created_at | timestamp | |

### IncidentReport
| Field | Type | Notes |
|---|---|---|
| report_id | uuid | PK |
| student_id | uuid | FK → Student.student_id |
| driver_id | uuid | |
| trip_id | uuid | FK → Trip.trip_id |
| description | string | |
| drafted_by_agent | boolean | true if chatbot drafted it |
| status | string | |
| created_at | timestamp | |

## Relationships

| From | To | Cardinality | Meaning |
|---|---|---|---|
| User | Student | 1 — 1 | a user account backs one student profile |
| Class | Student | 1 — many | a class has many students |
| Bus | Student | 1 — many | a bus is the allotted bus for many students |
| Stop | Student | 1 — many | a stop is the allotted stop for many students |
| Route | Stop | 1 — many | a route contains many stops |
| Bus | Trip | 1 — many | a bus runs many trips |
| Route | Trip | 1 — many | a route is used by many trips |
| Student | Attendance | 1 — many | a student has many attendance records |
| Trip | Attendance | 1 — many | a trip produces many attendance records |
| Student | LateMismatchLog | 1 — many | a student can accrue many log entries |
| Trip | LateMismatchLog | 1 — many | a trip can trigger many log entries |
| Student | ChatbotQuery | 1 — many | a student can send many queries |
| Student | IncidentReport | 1 — many | a student can file many reports |
| Trip | IncidentReport | 1 — many | a report is tied to the trip it concerns |
