```mermaid
erDiagram
    USERS {
        int user_id PK
        string email
        string password_hash
        enum role
        boolean is_active
        timestamp created_at
    }

    PROFILES {
        int profile_id PK
        int user_id FK
        string first_name
        string last_name
        date date_of_birth
        string phone_number
        string address
        enum gender
    }

    %% -----------------------
    %% ORGANIZATIONAL
    %% -----------------------
    DEPARTMENTS {
        int dept_id PK
        string name
        string description
        string location
    }

    ROOMS {
        string room_number PK
        enum room_type
        decimal charge_per_day
        boolean is_available
        int current_doctor_id FK
    }

    HOSPITAL_EXPENSES {
        int expense_id PK
        string category
        decimal amount
        string description
        timestamp expense_date
    }

    %% -----------------------
    %% PEOPLE (roles tied to USERS)
    %% -----------------------
    DOCTORS {
        int doctor_id PK
        int user_id FK
        int dept_id FK
        string specialization
        string license_number
        decimal consultation_fee
        date joining_date
    }

    PATIENTS {
        int patient_id PK
        int user_id FK
        string blood_group
        string emergency_contact_info
        string insurance_provider
    }

    STAFF {
        int staff_id PK
        int user_id FK
        int dept_id FK
        string job_title
        decimal salary
        string shift
        date joining_date
    }

    VALID_MEDICAL_LICENSES {
        string license_number PK
        boolean is_registered
    }

    %% -----------------------
    %% CLINICAL / ENCOUNTERS
    %% -----------------------
    APPOINTMENTS {
        int appointment_id PK
        int patient_id FK
        int doctor_id FK
        datetime appointment_date
        string reason
        enum status
        timestamp created_at
    }

    MEDICAL_RECORDS {
        int record_id PK
        int appointment_id FK
        string diagnosis
        string symptoms
        string treatment_plan
        json vitals
        timestamp created_at
    }

    PRESCRIPTIONS {
        int prescription_id PK
        int record_id FK
        string notes
        timestamp issued_at
    }

    PRESCRIPTION_ITEMS {
        int item_id PK
        int prescription_id FK
        int medicine_id FK
        string dosage
        string frequency
        int duration_days
    }

    MEDICINES {
        int medicine_id PK
        string name
        string manufacturer
        decimal unit_price
        int stock_quantity
        date expiry_date
    }

    MEDICAL_TESTS {
        int test_id PK
        string test_name
        decimal cost
        int estimated_duration_minutes
        string assigned_room_number FK
    }

    PATIENT_TESTS {
        int record_id PK
        int patient_id FK
        int test_id FK
        int doctor_id FK
        string room_number FK
        enum status
        timestamp scheduled_date
        string result_summary
    }

    LAB_RESULTS {
        int result_id PK
        int record_id FK
        int test_id FK
        string result_value
        string remarks
    }

    %% -----------------------
    %% PHARMACY / ORDERS
    %% -----------------------
    PHARMACY_ORDERS {
        int order_id PK
        int patient_id FK
        decimal total_amount
        enum status
        timestamp created_at
    }

    PHARMACY_ORDER_ITEMS {
        int item_id PK
        int order_id FK
        int medicine_id FK
        int quantity
        decimal unit_price
    }

    %% -----------------------
    %% BILLING / ADMISSION / SCHEDULES
    %% -----------------------
    ADMISSIONS {
        int admission_id PK
        int patient_id FK
        string room_number FK
        timestamp admission_date
        timestamp discharge_date
        decimal total_cost
        enum status
    }

    INVOICES {
        int invoice_id PK
        /* nullable FKs: only one of these usually set (appointment / patient_test / admission / pharmacy_order) */
        int appointment_id FK
        int patient_test_record_id FK
        int admission_id FK
        int pharmacy_order_id FK
        decimal total_amount
        decimal net_amount
        enum status
        timestamp generated_at
    }

    PAYMENTS {
        int payment_id PK
        int invoice_id FK
        decimal amount
        enum payment_method
        string transaction_ref
        timestamp payment_date
    }

    SCHEDULES {
        int schedule_id PK
        int doctor_id FK
        enum day_of_week
        time start_time
        time end_time
        string room_number
    }

    DOCTOR_LEAVES {
        int leave_id PK
        int doctor_id FK
        date start_date
        date end_date
        string reason
    }

    STAFF_LEAVES {
        int leave_id PK
        int staff_id FK
        date start_date
        date end_date
        string reason
    }

    %% -----------------------
    %% RELATIONSHIPS (read top->down)
    %% -----------------------
    USERS ||--|| PROFILES : "1:1 has"
    USERS ||--o| DOCTORS : "1:0..1 is a"
    USERS ||--o| PATIENTS : "1:0..1 is a"
    USERS ||--o| STAFF : "1:0..1 is a"

    DEPARTMENTS ||--o{ DOCTORS : "1:0..* employs"
    DEPARTMENTS ||--o{ STAFF : "1:0..* employs"

    DOCTORS ||--o{ SCHEDULES : "1:0..* has"
    DOCTORS ||--o{ DOCTOR_LEAVES : "1:0..* takes"
    STAFF ||--o{ STAFF_LEAVES : "1:0..* takes"

    PATIENTS ||--o{ APPOINTMENTS : "1:0..* books"
    DOCTORS ||--o{ APPOINTMENTS : "1:0..* attends"
    APPOINTMENTS ||--o| MEDICAL_RECORDS : "1:0..1 generates"
    MEDICAL_RECORDS ||--o{ PRESCRIPTIONS : "1:0..* contains"
    PRESCRIPTIONS ||--o{ PRESCRIPTION_ITEMS : "1:1..* includes"
    MEDICINES ||--o{ PRESCRIPTION_ITEMS : "1:0..* supplied in"

    MEDICAL_TESTS ||--o{ PATIENT_TESTS : "1:0..* performed as"
    PATIENTS ||--o{ PATIENT_TESTS : "1:0..* undergoes"
    DOCTORS ||--o{ PATIENT_TESTS : "1:0..* prescribes"
    PATIENT_TESTS ||--o{ LAB_RESULTS : "1:0..* results"
    MEDICAL_RECORDS ||--o{ LAB_RESULTS : "1:0..* contains"

    PATIENTS ||--o{ ADMISSIONS : "1:0..* admitted to"
    ROOMS ||--o{ ADMISSIONS : "1:0..* houses"

    PATIENTS ||--o{ PHARMACY_ORDERS : "1:0..* orders"
    PHARMACY_ORDERS ||--o{ PHARMACY_ORDER_ITEMS : "1:1..* contains"
    MEDICINES ||--o{ PHARMACY_ORDER_ITEMS : "1:0..* included in"

    APPOINTMENTS ||--o{ INVOICES : "1:0..* billed as"
    PATIENT_TESTS ||--o{ INVOICES : "1:0..* billed as"
    ADMISSIONS ||--o{ INVOICES : "1:0..* billed as"
    PHARMACY_ORDERS ||--o{ INVOICES : "1:0..* billed as"

    INVOICES ||--o{ PAYMENTS : "1:0..* paid by"

    DOCTORS ||--|| VALID_MEDICAL_LICENSES : "1:1 has (license_number)" 
```
