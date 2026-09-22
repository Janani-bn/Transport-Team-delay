# Faculty Module — Data Model

Scope: entities owned or primarily used by the faculty-facing flow (dashboard, reports status board, student-raised report review, polling, escalation to Bus Head/Admin).

## Entities

### User
| Field | Type | Notes |
|---|---|---|
| id | uuid | PK |
| name | string | |
| email | string | |
| phone | string | |
| role | string | |

### Faculty
| Field | Type | Notes |
|---|---|---|
| faculty_id | uuid | PK |
| user_id | uuid | FK → User.id |
| department | string | |

### Class
| Field | Type | Notes |
|---|---|---|
| class_id | uuid | PK |
| name | string | |
| faculty_id | uuid | FK → Faculty.faculty_id |

### Student
| Field | Type | Notes |
|---|---|---|
| student_id | uuid | PK |
| name | string | |
| class_id | uuid | FK → Class.class_id |

### Report
| Field | Type | Notes |
|---|---|---|
| report_id | uuid | PK |
| student_id | uuid | FK → Student.student_id |
| faculty_id | uuid | FK → Faculty.faculty_id |
| trip_id | uuid | |
| driver_id | uuid | |
| description | string | |
| source | enum | chatbot_agent, manual |
| status | enum | Pending, Declined, PollActive, Valid, Invalid, Forwarded |
| created_at | timestamp | |

### Poll
| Field | Type | Notes |
|---|---|---|
| poll_id | uuid | PK |
| report_id | uuid | FK → Report.report_id, unique |
| threshold | int | votes needed for validity |
| created_at | timestamp | |
| closes_at | timestamp | |

### PollResponse
| Field | Type | Notes |
|---|---|---|
| response_id | uuid | PK |
| poll_id | uuid | FK → Poll.poll_id |
| student_id | uuid | FK → Student.student_id |
| response | enum | confirm, deny |
| responded_at | timestamp | |

### ReportStatusBoard
| Field | Type | Notes |
|---|---|---|
| board_id | uuid | PK |
| report_id | uuid | FK → Report.report_id, unique |
| status | enum | Yet to Open, Currently Open, Resolved |
| updated_by_admin_id | uuid | written by Bus Head/Admin, read-only for faculty |
| updated_at | timestamp | |

## Relationships

| From | To | Cardinality | Meaning |
|---|---|---|---|
| Faculty | Class | 1 — many | a faculty member is assigned many classes |
| Class | Student | 1 — many | a class has many students |
| Faculty | Report | 1 — many | reports land under the faculty of the reporting student's class |
| Student | Report | 1 — many | a student can raise many reports |
| Report | Poll | 1 — 1 | each non-declined report gets one poll |
| Poll | PollResponse | 1 — many | a poll collects many student responses |
| Student | PollResponse | 1 — many | a student can respond to many polls |
| Report | ReportStatusBoard | 1 — 1 | each valid, forwarded report has one status-board entry |
