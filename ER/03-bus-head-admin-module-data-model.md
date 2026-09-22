# Bus Head / Admin Module — Data Model

Scope: entities owned or primarily used by the Bus Head/Admin flow (class-faculty assignment, bus-route assignment, driver-bus assignment, valid-reports handling).

## Entities

### User
| Field | Type | Notes |
|---|---|---|
| id | uuid | PK |
| name | string | |
| email | string | |
| phone | string | |
| role | string | |

### BusHeadAdmin
| Field | Type | Notes |
|---|---|---|
| admin_id | uuid | PK |
| user_id | uuid | FK → User.id |

### Faculty
| Field | Type | Notes |
|---|---|---|
| faculty_id | uuid | PK |
| name | string | |

### Class
| Field | Type | Notes |
|---|---|---|
| class_id | uuid | PK |
| name | string | |

### ClassFacultyAssignment
| Field | Type | Notes |
|---|---|---|
| assignment_id | uuid | PK |
| class_id | uuid | FK → Class.class_id |
| faculty_id | uuid | FK → Faculty.faculty_id |
| assigned_by_admin_id | uuid | FK → BusHeadAdmin.admin_id |
| assigned_at | timestamp | |

### Bus
| Field | Type | Notes |
|---|---|---|
| bus_id | uuid | PK |
| bus_number | string | |
| capacity | int | |
| status | string | |

### Route
| Field | Type | Notes |
|---|---|---|
| route_id | uuid | PK |
| name | string | |

### Driver
| Field | Type | Notes |
|---|---|---|
| driver_id | uuid | PK |
| user_id | uuid | FK → User.id |
| license_no | string | |
| status | enum | Available, OnTrip |

### BusRouteAssignment
| Field | Type | Notes |
|---|---|---|
| assignment_id | uuid | PK |
| bus_id | uuid | FK → Bus.bus_id |
| route_id | uuid | FK → Route.route_id |
| trip_id | string | |
| optimized | boolean | set via route optimization |
| assigned_by_admin_id | uuid | FK → BusHeadAdmin.admin_id |
| assigned_at | timestamp | |

### DriverBusAssignment
| Field | Type | Notes |
|---|---|---|
| assignment_id | uuid | PK |
| driver_id | uuid | FK → Driver.driver_id |
| bus_id | uuid | FK → Bus.bus_id |
| assigned_by_admin_id | uuid | FK → BusHeadAdmin.admin_id |
| assigned_at | timestamp | |
| status | string | |

### Report
| Field | Type | Notes |
|---|---|---|
| report_id | uuid | PK |
| description | string | |
| status | string | |

### ValidReport
| Field | Type | Notes |
|---|---|---|
| valid_report_id | uuid | PK |
| report_id | uuid | FK → Report.report_id, unique |
| forwarded_at | timestamp | forwarded by the AI agent |
| opened_by_admin_id | uuid | FK → BusHeadAdmin.admin_id |
| status | enum | Yet to Open, Currently Open, Resolved |
| resolved_at | timestamp | nullable |

## Relationships

| From | To | Cardinality | Meaning |
|---|---|---|---|
| BusHeadAdmin | ClassFacultyAssignment | 1 — many | an admin makes many class-faculty assignments |
| Class | ClassFacultyAssignment | 1 — many | a class can have assignment history |
| Faculty | ClassFacultyAssignment | 1 — many | a faculty can be assigned to many classes |
| BusHeadAdmin | BusRouteAssignment | 1 — many | an admin makes many bus-route assignments |
| Bus | BusRouteAssignment | 1 — many | a bus can be assigned to many routes over time |
| Route | BusRouteAssignment | 1 — many | a route can have many bus assignments over time |
| BusHeadAdmin | DriverBusAssignment | 1 — many | an admin makes many driver-bus assignments |
| Driver | DriverBusAssignment | 1 — many | a driver can be assigned to many buses over time |
| Bus | DriverBusAssignment | 1 — many | a bus can have many driver assignments over time |
| BusHeadAdmin | ValidReport | 1 — many | an admin opens/resolves many valid reports |
| Report | ValidReport | 1 — 1 | a report becomes exactly one valid-report record once forwarded |
