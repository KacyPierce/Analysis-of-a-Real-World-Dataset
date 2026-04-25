# Analysis-of-a-Real-World-Dataset--

[Final Project_Pierce_Kacy.sql](https://github.com/user-attachments/files/27092458/Final.Project_Pierce_Kacy.sql)

CREATE DATABASE belhaven_hospital_db;

-- Created the departments table first, as doctors will reference it
CREATE TABLE departments (
    department_id SERIAL PRIMARY KEY,
    department_name VARCHAR(100) UNIQUE NOT NULL,
	location TEXT NOT NULL
);

-- Created the doctors table
CREATE TABLE doctors (
    doctor_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    specialty VARCHAR(100) NOT NULL,
	email VARCHAR(100) NOT NULL,
    department_id INTEGER REFERENCES departments(department_id),
	department TEXT NOT NULL
);

-- Created the patients table
CREATE TABLE patients (
    patient_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
	dob DATE NOT NULL,
	gender VARCHAR(10) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL
);

-- Created the appointments table
CREATE TABLE appointments (
    appointment_id SERIAL PRIMARY KEY,
    patient_id INTEGER NOT NULL REFERENCES patients(patient_id),
    doctor_id INTEGER NOT NULL REFERENCES doctors(doctor_id),
    appointment_date TIMESTAMP NOT NULL,
    reason TEXT NOT NULL
);

-- Created the treatments table
CREATE TABLE treatments (
    treatment_id SERIAL PRIMARY KEY,
    appointment_id INTEGER NOT NULL,
	treatment_description TEXT NOT NULL,
    cost NUMERIC(10, 2) CHECK (cost >= 0) -- Optional cost field, but if provided must be non-negative
);

-- Created the medications table
CREATE TABLE medications (
    medication_id SERIAL PRIMARY KEY,
    treatment_id INTEGER NOT NULL REFERENCES treatments(treatment_id),
    medication_name VARCHAR(100) NOT NULL,
    dosage VARCHAR(50) CHECK (dosage <> '') -- Check constraint to ensure dosage is not empty
);

-- Created the billing table
CREATE TABLE billing (
    billing_id SERIAL PRIMARY KEY,
    appointment_id INTEGER NOT NULL,
    total_amount NUMERIC(10, 2) NOT NULL CHECK (total_amount > 0),
    paid_status BOOLEAN NOT NULL DEFAULT FALSE
);

--Testing to confirm each table in the Database.
SELECT * FROM departments;
SELECT * FROM doctors;
SELECT * FROM patients;
SELECT * FROM appointments;
SELECT * FROM treatments;
SELECT * FROM medications;
SELECT * FROM billing;

-- Import csv files
COPY departments (department_name, location)
FROM 'C:/Users/Public/Documents/departments.csv'
DELIMITER ',' CSV HEADER

COPY doctors(first_name, last_name, specialty, email, department)
FROM 'C:/Users/Public/Documents/doctors.csv'
DELIMITER ',' CSV HEADER;

COPY patients(first_name, last_name, dob, gender, email)
FROM 'C:/Users/Public/Documents/patients.csv'
DELIMITER ',' CSV HEADER;

COPY appointments(patient_id, doctor_id, appointment_date, reason)
FROM 'C:/Users/Public/Documents/appointments.csv'
DELIMITER ',' CSV HEADER;

COPY treatments(appointment_id, treatment_description)
FROM 'C:/Users/Public/Documents/treatments.csv'
DELIMITER ',' CSV HEADER;

COPY medications(treatment_id, medication_name, dosage)
FROM 'C:/Users/Public/Documents/medications.csv'
DELIMITER ',' CSV HEADER;

COPY billing(appointment_id, total_amount, paid_status)
FROM 'C:/Users/Public/Documents/billing.csv'
DELIMITER ',' CSV HEADER;


--Testing to confirm each table in the Database.
SELECT * FROM departments;
SELECT * FROM doctors;
SELECT * FROM patients;
SELECT * FROM appointments;
SELECT * FROM treatments;
SELECT * FROM medications;
SELECT * FROM billing;

--Adding a new column to patients (e.g., insurance_provider with a default value).
ALTER TABLE patients 
ADD COLUMN insurance_provider VARCHAR(100) DEFAULT 'Uninsured';

--Renaming a column in appointments (e.g., reason → visit_reason).
ALTER TABLE appointments 
RENAME COLUMN reason TO visit_reason;

--Changing a column’s data type (e.g., phone number length or billing total to NUMERIC(8,2)).
ALTER TABLE billing
ALTER COLUMN total_amount TYPE NUMERIC(8,2);

--Adding a CHECK constraint to an existing column (e.g., ensure total_amount > 0).
ALTER TABLE billing
ADD CONSTRAINT chk_total_amount CHECK (total_amount > 0);

--Dropping and recreating a table (e.g., treatments) to add a new column such as treatment_cost.
DROP TABLE treatments;

CREATE TABLE treatments (
    treatment_id SERIAL PRIMARY KEY,
	patient_id INT, treatment_name VARCHAR(100),
	treatment_cost NUMERIC(8,2)
);

--1. INNER JOIN
SELECT 
    p.first_name AS patient_first_name, 
    p.last_name AS patient_last_name,
    d.first_name AS doctor_first_name,
    d.last_name AS doctor_last_name,
    a.appointment_date
FROM 
    patients p
INNER JOIN 
    appointments a ON p.patient_id = a.patient_id
INNER JOIN 
    doctors d ON a.doctor_id = d.doctor_id;

--2. LEFT JOIN: Display all patients and their appointments, including those without any
SELECT 
    p.first_name, 
    p.last_name, 
    a.appointment_date
FROM 
    patients p
LEFT JOIN 
    appointments a ON p.patient_id = a.patient_id;

--3. RIGHT JOIN: Display all doctors and the patients they've seen, including doctors with no appointments
SELECT 
    d.first_name AS doctor_first_name, 
    d.last_name AS doctor_last_name,
    p.first_name AS patient_first_name,
    p.last_name AS patient_last_name
FROM 
    patients p
RIGHT JOIN 
    appointments a ON p.patient_id = a.patient_id
RIGHT JOIN
    doctors d ON a.doctor_id = d.doctor_id;

--4. GROUP BY with Aggregate: Count total appointments per doctor
SELECT 
    d.first_name, 
    d.last_name, 
    COUNT(a.appointment_id) AS total_appointments
FROM 
    doctors d
LEFT JOIN 
    appointments a ON d.doctor_id = a.doctor_id
GROUP BY 
    d.doctor_id, d.first_name, d.last_name;

--5. HAVING: Identify doctors with more than two appointments
SELECT 
    d.first_name, 
    d.last_name, 
    COUNT(a.appointment_id) AS total_appointments
FROM 
    doctors d
INNER JOIN 
    appointments a ON d.doctor_id = a.doctor_id
GROUP BY 
    d.doctor_id, d.first_name, d.last_name
HAVING 
    COUNT(a.appointment_id) > 2;

--6. Subquery (Single Row): Display the patient with the earliest appointment
SELECT 
    first_name, 
    last_name
FROM 
    patients
WHERE 
    patient_id = (SELECT 
                      patient_id
                  FROM 
                      appointments
                  ORDER BY 
                      appointment_date ASC
                  LIMIT 1);

--7. Subquery (Multi-row): List all patients who have been prescribed a specific medication (e.g., 'Ibuprofen')
SELECT 
    first_name, 
    last_name
FROM 
    patients
WHERE 
    patient_id IN (SELECT 
                      patient_id
                   FROM 
                      medications
                   WHERE 
                      medication_name = 'Ibuprofen');

--8. EXISTS: Find patients who have received at least one treatment
SELECT 
    first_name, 
    last_name
FROM 
    patients p
WHERE 
    EXISTS (SELECT 
                1
            FROM 
                treatments t
            WHERE 
                t.patient_id = p.patient_id);

--9. Nested Query: Display all appointments for patients younger than the average age
SELECT *
FROM appointments
WHERE patient_id IN (
    SELECT patient_id
    FROM patients
    WHERE AGE(dob) < (
        SELECT AVG(AGE(dob))
        FROM patients
    )
);

--10. Window Function (PostgreSQL-only): Rank doctors by the total number of appointments
SELECT 
    d.first_name,
    d.last_name,
    COUNT(a.appointment_id) AS total_appointments,
    RANK() OVER (ORDER BY COUNT(a.appointment_id) DESC) as doctor_rank
FROM 
    doctors d
LEFT JOIN 
    appointments a ON d.doctor_id = a.doctor_id
GROUP BY 
    d.doctor_id, d.first_name, d.last_name;

--Drop the departments table and recreate it.
DROP TABLE IF EXISTS departments CASCADE;

CREATE TABLE departments (
    department_id SERIAL PRIMARY KEY,
    department_name VARCHAR(100) NOT NULL
);

--Use DROP TABLE IF EXISTS to remove optional tables without causing errors.
DROP TABLE IF EXISTS optional_feature_table;

--Verify that dependent tables respond correctly to cascading drops.
SELECT * FROM departments;

SELECT * FROM department_name;

SELECT constraint_name, constraint_type
FROM information_schema.table_constraints
WHERE table_name = 'doctors' AND constraint_name = 'fk_department';

SELECT * FROM doctors;

--1. My database design utilizes a relational model that ensures data integrity
-- and organization, while reducing errors by moving away from spreadsheet
-- errors. It also implements normalization to eliminate data repeating, ensures
-- reference integrity by using foreign keys, and uses constraints to guarantee
-- accuracy of the data.

--2. NOT NULL ensures there are no NULL values. UNIQUE assures
-- that all values in a column are distinct. CHECK enforces
-- specific conditions when comparing text and values. DEFAULT
-- assigns a value if none is provided. PRIMARY KEY uniquely
-- identifies rows by combining UNIQUE and NOT NULL.

--3. Christian ethics sets a firm foundation for handling sensitive data.
-- Privacy and confidentiality ensures respect for patients as a person
-- and not just a number. Stewardship guarantees responsible care when
-- viewing data as an entrusted resource. Care for others encourages the
-- person behind the data to design systems that improve patient out comes
-- reduce errors. Truthfulness maintains the integrity in data is accurate,
-- honest, and consistent.
