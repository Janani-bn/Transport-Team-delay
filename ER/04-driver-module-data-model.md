# Driver Module — Data Model

Scope: entities owned or primarily used by the driver-facing flow (start/end trip, stop-reached marking, QR attendance, manual override, trip snapshot logging). `StopReachedLog` records only that a stop was marked reached (a timestamp) — no GPS/live-location fields.

## Entities

### User
| Field | Type | Notes |
|---|---|---|
| id | uuid | PK |
| name | string | |
| email | string | |
| phone | string | |

### Driver
| Field | Type | Notes |
|---|---|---|
| driver_id | uuid | PK |
| user_id | uuid | FK → User.id |
| license_no | string | |
| status | enum | Available, OnTrip |

### Bus
| Field | Type | Notes |
|---|---|---|
| bus_id | uuid | PK |
| bus_number | string | |
| status | string | |

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
| driver_id | uuid | FK → Driver.driver_id |
| date | date | |
| start_time | time | |
| end_time | time | |
| status | string | |

### StopReachedLog
| Field | Type | Notes |
|---|---|---|
| log_id | uuid | PK |
| trip_id | uuid | FK → Trip.trip_id |
| stop_id | uuid | FK → Stop.stop_id |
| reached_at | timestamp | feeds the AI agent used by students who missed their bus |

### QRAttendanceSession
| Field | Type | Notes |
|---|---|---|
| session_id | uuid | PK |
| trip_id | uuid | FK → Trip.trip_id, unique |
| qr_code | string | |
| generated_at | timestamp | |
| expires_at | timestamp | |

### ManualAttendanceOverride
| Field | Type | Notes |
|---|---|---|
| override_id | uuid | PK |
| trip_id | uuid | FK → Trip.trip_id |
| student_id | uuid | |
| marked_by_driver_id | uuid | FK → Driver.driver_id |
| reason | string | e.g. QR scan failure |
| marked_at | timestamp | |

### TripSnapshot
| Field | Type | Notes |
|---|---|---|
| snapshot_id | uuid | PK |
| trip_id | uuid | FK → Trip.trip_id, unique |
| driver_id | uuid | FK → Driver.driver_id |
| bus_id | uuid | FK → Bus.bus_id |
| route_id | uuid | FK → Route.route_id |
| student_count | int | |
| stops_completed_count | int | |
| created_at | timestamp | |

## Relationships

| From | To | Cardinality | Meaning |
|---|---|---|---|
| Driver | Trip | 1 — many | a driver runs many trips |
| Bus | Trip | 1 — many | a bus is used on many trips |
| Route | Trip | 1 — many | a route is covered by many trips |
| Trip | StopReachedLog | 1 — many | a trip produces one reached-log entry per stop |
| Stop | StopReachedLog | 1 — many | a stop accumulates reached-log entries across trips |
| Trip | QRAttendanceSession | 1 — 1 | each trip has one QR session at college |
| Trip | ManualAttendanceOverride | 1 — many | a trip can have several manual overrides |
| Driver | ManualAttendanceOverride | 1 — many | a driver can make many manual overrides |
| Trip | TripSnapshot | 1 — 1 | each trip is logged as exactly one snapshot |
| Driver | TripSnapshot | 1 — many | a driver's trips each produce a snapshot |
| Bus | TripSnapshot | 1 — many | a bus's trips each produce a snapshot |
| Route | TripSnapshot | 1 — many | a route's trips each produce a snapshot |
