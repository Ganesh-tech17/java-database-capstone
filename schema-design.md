1. Introduction

This document presents the initial data structure and schema planning for the Smart Clinic Management System. The goal is to design scalable, real-world SQL and MongoDB schemas that support:

Patient & doctor registration
Appointment scheduling and tracking
Doctor availability management
Prescriptions
Notes/feedback

(Optional) chat records, payment history, documents
These schema ideas will guide backend model creation and database logic.



2. Core Entities (Real-World Breakdown)

A smart clinic typically deals with:
Patients
Personal details\
Contact info
Medical history (optional future feature)
Appointments
Doctors
Profile information
Specialization
Availability
Appointments
Prescriptions
Appointments
Patient ↔ Doctor link
Date & time
Duration
Status (booked, completed, cancelled)
Prescriptions
Created per appointment
Medications list
Notes
Doctor Availability
Working hours
Blocked/unavailable slots
Additional Optional Entities
Payments
Chat messages
Uploaded medical documents
Feedback

3. Relational Database Design (SQL)

Below is the proposed SQL schema with tables and relationships.

3.1 Patients Table

patient_id (PK)
name
email (unique)
password_hash
phone
created_at

3.2 Doctors Table

doctor_id (PK)
name
specialization
email (unique)
password_hash
contact_number
bio
created_at

3.3 Appointments Table

appointment_id (PK)
patient_id (FK → patients)
doctor_id (FK → doctors
date
start_time
end_time
status (booked/completed/cancelled)
created_at

3.4 Doctor Availability Table

availability_id (PK)
doctor_id (FK → doctors)
date
start_time
end_time
is_available (bool)

3.5 Prescriptions Table

prescription_id (PK)
appointment_id (FK → appointments)
doctor_id (FK → doctors)
patient_id (FK → patients)
notes (text)
created_at

3.6 Prescription Items Table

item_id (PK)
prescription_id (FK → prescriptions)
medicine_name
dosage
duration

3.7 Optional Tables

Payments
payment_id
appointment_id
amount
mode
status
timestamp
Chat Messages
chat_id
sender_id
receiver_id
message
timestamp
Documents
document_id
patient_id
file_path
uploaded_at

4. MongoDB Schema Design

In MongoDB, documents can embed related data. Suggested collections:

4.1 patients Collection
{
  "_id": ObjectId(),
  "name": "John Doe",
  "email": "john@example.com",
  "password_hash": "...",
  "phone": "9999999999",
  "created_at": "..."
}

4.2 doctors Collection
{
  "_id": ObjectId(),
  "name": "Dr. Smith",
  "specialization": "Cardiologist",
  "email": "drsmith@example.com",
  "contact_number": "9876543210",
  "bio": "10 years experience",
  "created_at": "...",
  "availability": [
    {
      "date": "2025-01-10",
      "start_time": "10:00",
      "end_time": "14:00",
      "is_available": true
    }
  ]
}

4.3 appointments Collection
{
  "_id": ObjectId(),
  "patient_id": ObjectId(),
  "doctor_id": ObjectId(),
  "date": "2025-01-12",
  "start_time": "12:00",
  "end_time": "13:00",
  "status": "booked",
  "created_at": "..."
}

4.4 prescriptions Collection
{
  "_id": ObjectId(),
  "appointment_id": ObjectId(),
  "doctor_id": ObjectId(),
  "patient_id": ObjectId(),
  "notes": "Drink water",
  "items": [
    {
      "medicine_name": "Paracetamol",
      "dosage": "500mg",
      "duration": "3 days"
    }
  ],
  "created_at": "..."
}

4.5 optional collections (future)

payments
chats
documents

5. Relationship Summary
SQL (Normalized)

One patient → many appointments

One doctor → many appointments

One appointment → one prescription

One prescription → many medication items

One doctor → many availability slots

MongoDB (Flexible)

Doctors embed availability

Prescriptions embed medication items

Appointments remain a separate collection for cross-referencing

6. Next Steps

Finalize SQL schema (add indexes, constraints, cascades)

Finalize MongoDB collections and embedding strategy

Use this design to build backend models (Java/Node/Python)

Create ER diagram + MongoDB diagram (optional)

This document will continue to evolve as more features are added.




MySQL Database Design

Below are recommended SQL tables, columns, types, keys, constraints and design notes. This design assumes a relational store for the core operational data (patients, doctors, appointments, admin) while keeping flexible or heavy/optional data in MongoDB.

Table: patients

id: INT, PRIMARY KEY, AUTO_INCREMENT, NOT NULL

first_name: VARCHAR(100), NOT NULL

last_name: VARCHAR(100), NULL

email: VARCHAR(255), UNIQUE, NOT NULL

password_hash: VARCHAR(255), NOT NULL

phone: VARCHAR(20), NULL

date_of_birth: DATE, NULL

gender: ENUM('male','female','other'), NULL

created_at: DATETIME, NOT NULL, DEFAULT CURRENT_TIMESTAMP

updated_at: DATETIME, NULL, ON UPDATE CURRENT_TIMESTAMP

Constraints & notes: Email should be UNIQUE. Phone format validation can be performed in application layer. When a patient is deleted, consider soft-delete (is_deleted boolean) to keep historical appointments and prescriptions; avoid hard deleting to retain medical history.

Table: doctors

id: INT, PRIMARY KEY, AUTO_INCREMENT, NOT NULL

first_name: VARCHAR(100), NOT NULL

last_name: VARCHAR(100), NULL

email: VARCHAR(255), UNIQUE, NOT NULL

password_hash: VARCHAR(255), NOT NULL

specialization: VARCHAR(150), NULL

contact_number: VARCHAR(20), NULL

bio: TEXT, NULL

created_at: DATETIME, NOT NULL, DEFAULT CURRENT_TIMESTAMP

updated_at: DATETIME, NULL, ON UPDATE CURRENT_TIMESTAMP

Constraints & notes: Email UNIQUE. Use soft-delete for doctors too if needed. Doctor availability will be stored in a separate table to allow fine-grained slots and to prevent overlapping appointment rules.

Table: appointments

id: INT, PRIMARY KEY, AUTO_INCREMENT, NOT NULL

patient_id: INT, FOREIGN KEY → patients(id) ON DELETE RESTRICT ON UPDATE CASCADE, NOT NULL

doctor_id: INT, FOREIGN KEY → doctors(id) ON DELETE RESTRICT ON UPDATE CASCADE, NOT NULL

start_time: DATETIME, NOT NULL

end_time: DATETIME, NOT NULL

status: ENUM('scheduled','checked_in','completed','cancelled'), NOT NULL DEFAULT 'scheduled'

type: ENUM('in_person','telemedicine'), NULL

reason: VARCHAR(500), NULL

created_at: DATETIME, NOT NULL, DEFAULT CURRENT_TIMESTAMP

updated_at: DATETIME, NULL, ON UPDATE CURRENT_TIMESTAMP

Constraints & notes: Add a UNIQUE index on (doctor_id,start_time,end_time) OR enforce via application logic to prevent overlapping appointments for a doctor. For stricter DB-level protection, use exclusion constraints (Postgres) or application checks in MySQL with transactions. If a patient is deleted, consider keeping appointments (soft-delete patient) or setting patient_id to NULL with ON DELETE SET NULL depending on privacy/compliance.

Table: admin

id: INT, PRIMARY KEY, AUTO_INCREMENT, NOT NULL

username: VARCHAR(100), UNIQUE, NOT NULL

email: VARCHAR(255), UNIQUE, NOT NULL

password_hash: VARCHAR(255), NOT NULL

role: ENUM('superadmin','staff','viewer'), NOT NULL DEFAULT 'staff'

created_at: DATETIME, NOT NULL, DEFAULT CURRENT_TIMESTAMP

Notes: Admin table holds platform users with elevated privileges. Use RBAC in application layer.

Table: doctor_availability (time slots)

id: INT, PRIMARY KEY, AUTO_INCREMENT, NOT NULL

doctor_id: INT, FOREIGN KEY → doctors(id) ON DELETE CASCADE, NOT NULL

date: DATE, NOT NULL

start_time: TIME, NOT NULL

end_time: TIME, NOT NULL

is_available: TINYINT(1), NOT NULL DEFAULT 1

recurrence_rule: VARCHAR(255), NULL -- optional iCal RRULE string for recurring availability

created_at: DATETIME, NOT NULL, DEFAULT CURRENT_TIMESTAMP

Notes: Keeping slots separate allows blocking specific ranges and simplifies availability calculations. Use constraints to ensure start_time < end_time in application or DB check.

Table: prescriptions

id: INT, PRIMARY KEY, AUTO_INCREMENT, NOT NULL

appointment_id: INT, FOREIGN KEY → appointments(id) ON DELETE CASCADE, NOT NULL

doctor_id: INT, FOREIGN KEY → doctors(id) ON DELETE SET NULL, NULL

patient_id: INT, FOREIGN KEY → patients(id) ON DELETE SET NULL, NULL

notes: TEXT, NULL

created_at: DATETIME, NOT NULL, DEFAULT CURRENT_TIMESTAMP

Table: prescription_items

id: INT, PRIMARY KEY, AUTO_INCREMENT

prescription_id: INT, FOREIGN KEY → prescriptions(id) ON DELETE CASCADE, NOT NULL

medicine_name: VARCHAR(255), NOT NULL

dosage: VARCHAR(100), NULL

frequency: VARCHAR(100), NULL

duration_days: INT, NULL

Notes: Prescriptions are tied to appointments for auditability. Keeping items normalized simplifies queries for analytics.

Table: payments (optional)

id: INT, PRIMARY KEY, AUTO_INCREMENT

appointment_id: INT, FOREIGN KEY → appointments(id) ON DELETE SET NULL

patient_id: INT, FOREIGN KEY → patients(id) ON DELETE SET NULL

amount: DECIMAL(10,2), NOT NULL

currency: VARCHAR(10), NOT NULL DEFAULT 'INR'

status: ENUM('pending','completed','failed','refunded'), NOT NULL DEFAULT 'pending'

method: ENUM('card','upi','cash'), NULL

transaction_ref: VARCHAR(255), NULL

created_at: DATETIME, NOT NULL, DEFAULT CURRENT_TIMESTAMP

Design decisions & deeper thinking

Soft-delete vs hard-delete: Prefer soft-delete for patients/doctors to retain historical medical records, payments, and audit trails. Implement is_deleted and deleted_at fields where needed.

Overlapping appointments: Enforce via application logic with transaction locks; MySQL lacks native exclusion constraints. Alternatively use a calendar table with atomic slot reservation.

Retention policy: Keep appointment & prescription history indefinitely unless compliance/policy requires purging; consider archiving older records.

Indexes: Index appointments(doctor_id,start_time), appointments(patient_id,start_time), doctors(email), patients(email) for fast lookups.

MongoDB Collection Design

MongoDB complements MySQL by holding flexible, evolving, or write-heavy documents: prescriptions with rich metadata, chat logs, feedback, file references, and read-optimized aggregates.

Collection: prescriptions (example)

This collection stores enriched prescription documents that may include embedded metadata, pharmacy suggestions, and scanned attachments. Use references to MySQL IDs (appointment_id, patient_id, doctor_id) to keep ownership in the relational store while allowing flexible fields here.

{
  "_id": {"$oid": "64abc1234567890abcdef0001"},
  "appointment_id": 123,
  "patient_id": 45,
  "doctor_id": 7,
  "issued_at": "2025-01-12T09:30:00Z",
  "notes": "Take with food",
  "items": [
    {
      "name": "Amoxicillin",
      "dosage": "500mg",
      "frequency": "3 times a day",
      "duration_days": 7,
      "instructions": "After meals"
    }
  ],
  "pharmacy_suggestions": [
    { "name": "City Pharmacy", "distance_meters": 350 },
    { "name": "CareMeds", "distance_meters": 800 }
  ],
  "attachments": [
    { "file_id": "s3://bucket/prescriptions/123.pdf", "mime": "application/pdf" }
  ],
  "tags": ["antibiotic","urgent"],
  "metadata": {
    "created_by": "doctor_webapp",
    "clinic_location_id": 2,
    "verified": true
  }
}

Design notes:

Store numeric relational IDs (MySQL IDs) to keep a single source of truth in MySQL. Alternatively, if a service adopts MongoDB as primary later, store full patient/doctor snapshots inside the document.

Use embedded items for quick reads. Attachments only store references (S3/Blob).

Schemaless fields like tags and metadata allow the system to evolve without migrations.
