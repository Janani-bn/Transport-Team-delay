# Master Data Model — College Transport Platform

Combines all four module models (Student, Faculty, Bus Head/Admin, Driver) into one schema. Location/GPS tracking is intentionally excluded — `Route` and `Stop` appear only as plain structural lookups (name, sequence), not live-tracking data.

## Entities

### Identity

**User** — id (pk), name, email (unique), phone, role (enum: student, faculty, driver, admin), password_hash, created_at

**Student** — student_id (pk), user_id (fk → User.id, unique), roll_no, class_id (fk → Class), allotted_bus_id (fk → Bus), allotted_stop_id (fk → Stop), late_count

**Faculty** — faculty_id (pk), user_id (fk → User.id, unique), department

**Driver** — driver_id (pk), user_id (fk → User.id, unique), license_no, status (enum: Available, OnTrip)

**BusHeadAdmin** — admin_id (pk), user_id (fk → User.id, unique)

### Structure & Assignments

**Class** — class_id (pk), name

**ClassFacultyAssignment** — assignment_id (pk), class_id (fk), faculty_id (fk), assigned_by_admin_id (fk → BusHeadAdmin), assigned_at

**Bus** — bus_id (pk), bus_number (unique), capacity, status

**Route** — route_id (pk), name

**Stop** — stop_id (pk), name, route_id (fk → Route), sequence_no

**BusRouteAssignment** — assignment_id (pk), bus_id (fk), route_id (fk), optimized (boolean), assigned_by_admin_id (fk → BusHeadAdmin), assigned_at

**DriverBusAssignment** — assignment_id (pk), driver_id (fk), bus_id (fk), assigned_by_admin_id (fk → BusHeadAdmin), assigned_at, status

### Trip Execution

**Trip** — trip_id (pk), bus_id (fk), route_id (fk), driver_id (fk), date, start_time, end_time, status

**StopReachedLog** — log_id (pk), trip_id (fk), stop_id (fk), reached_at

**QRAttendanceSession** — session_id (pk), trip_id (fk, unique), qr_code, generated_at, expires_at

**ManualAttendanceOverride** — override_id (pk), trip_id (fk), student_id (fk), marked_by_driver_id (fk → Driver), reason, marked_at

**TripSnapshot** — snapshot_id (pk), trip_id (fk, unique), driver_id (fk), bus_id (fk), route_id (fk), student_count, created_at

### Attendance & Compliance

**Attendance** — attendance_id (pk), student_id (fk), trip_id (fk), bus_id (fk), marked_at, method, matched_allocated_bus (boolean)

**LateMismatchLog** — log_id (pk), student_id (fk), trip_id (fk), month, late_count_snapshot, notified_approaching (boolean), notified_threshold (boolean), created_at

**ChatbotQuery** — query_id (pk), student_id (fk), query_type (enum: stop_lookup, bus_lookup, incident_report), query_text, response_text, created_at

### Reports & Escalation

**Report** — report_id (pk), student_id (fk), faculty_id (fk), trip_id (fk), driver_id (fk), description, source (enum: chatbot_agent, manual), status (enum: Pending, Declined, PollActive, Valid, Invalid, Forwarded), created_at

**Poll** — poll_id (pk), report_id (fk, unique), threshold, created_at, closes_at

**PollResponse** — response_id (pk), poll_id (fk), student_id (fk), response (enum: confirm, deny), responded_at

**ReportStatusBoard** — board_id (pk), report_id (fk, unique), status (enum: Yet to Open, Currently Open, Resolved), updated_by_admin_id (fk → BusHeadAdmin), updated_at

**ValidReport** — valid_report_id (pk), report_id (fk, unique), forwarded_at, opened_by_admin_id (fk → BusHeadAdmin), status (enum: Yet to Open, Currently Open, Resolved), resolved_at

## Relationships

### Identity
| From | To | Cardinality |
|---|---|---|
| User | Student | 1 — 1 |
| User | Faculty | 1 — 1 |
| User | Driver | 1 — 1 |
| User | BusHeadAdmin | 1 — 1 |

### Structure & Assignments
| From | To | Cardinality |
|---|---|---|
| Class | Student | 1 — many |
| Bus | Student | 1 — many (allotted bus) |
| Stop | Student | 1 — many (allotted stop) |
| Route | Stop | 1 — many |
| Class | ClassFacultyAssignment | 1 — many |
| Faculty | ClassFacultyAssignment | 1 — many |
| BusHeadAdmin | ClassFacultyAssignment | 1 — many |
| Bus | BusRouteAssignment | 1 — many |
| Route | BusRouteAssignment | 1 — many |
| BusHeadAdmin | BusRouteAssignment | 1 — many |
| Driver | DriverBusAssignment | 1 — many |
| Bus | DriverBusAssignment | 1 — many |
| BusHeadAdmin | DriverBusAssignment | 1 — many |

### Trip Execution
| From | To | Cardinality |
|---|---|---|
| Bus | Trip | 1 — many |
| Route | Trip | 1 — many |
| Driver | Trip | 1 — many |
| Trip | StopReachedLog | 1 — many |
| Stop | StopReachedLog | 1 — many |
| Trip | QRAttendanceSession | 1 — 1 |
| Trip | ManualAttendanceOverride | 1 — many |
| Student | ManualAttendanceOverride | 1 — many |
| Driver | ManualAttendanceOverride | 1 — many |
| Trip | TripSnapshot | 1 — 1 |
| Driver | TripSnapshot | 1 — many |
| Bus | TripSnapshot | 1 — many |
| Route | TripSnapshot | 1 — many |

### Attendance & Compliance
| From | To | Cardinality |
|---|---|---|
| Student | Attendance | 1 — many |
| Trip | Attendance | 1 — many |
| Bus | Attendance | 1 — many |
| Student | LateMismatchLog | 1 — many |
| Trip | LateMismatchLog | 1 — many |
| Student | ChatbotQuery | 1 — many |

### Reports & Escalation
| From | To | Cardinality |
|---|---|---|
| Student | Report | 1 — many |
| Faculty | Report | 1 — many |
| Trip | Report | 1 — many |
| Driver | Report | 1 — many |
| Report | Poll | 1 — 1 |
| Poll | PollResponse | 1 — many |
| Student | PollResponse | 1 — many |
| Report | ReportStatusBoard | 1 — 1 |
| BusHeadAdmin | ReportStatusBoard | 1 — many |
| Report | ValidReport | 1 — 1 |
| BusHeadAdmin | ValidReport | 1 — many |

## Cross-module flow summary
- A **Report** is created by a Student (via chatbot or manual filing), reviewed by Faculty (decline or poll), validated by AI + poll threshold, forwarded as a **ValidReport** to Bus Head/Admin, and its state is mirrored back to Faculty via the **ReportStatusBoard**.
- A **Trip** is the shared backbone: Bus Head/Admin sets it up (bus/route/driver assignments), the Driver executes it (stop-reached logs, QR session, snapshot), and Students are measured against it (attendance, late/mismatch tracking).
