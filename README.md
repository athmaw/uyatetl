# ETL Data Integration Web Application

**Author:** Athena Jamie Uy  
**Class/Schedule:** IT 3206N MW 6:00 - 8:30 PM  
**Domain:** `uyatetl.dcism.org`  
**Reference Sample:** https://etlintegexercise.dcism.org/

---

## TASK 1 - Source Data Extraction
The web application extracts data from three CSV source systems:
1. `source_A_SIS.csv` (Student Information System)
2. `source_B_LMS.csv` (Learning Management System)
3. `source_C_Accounting.csv` (Accounting System)

---

## TASK 2 - Source-to-Target Data Mapping

| Source System | Source Field | Target Field | Target Type | Transformation / Rule |
| :--- | :--- | :--- | :--- | :--- |
| **SIS** | `student_id` | `studentId` | Integer | Convert text to integer; preserve as unique identifier. |
| **SIS** | `student_name` | `studentName` | String | Trim whitespace; normalize to proper case (e.g., `'JUAN dela CRUZ'` &rarr; `'Juan Dela Cruz'`). |
| **SIS** | `program_code` | `program` | String | Trim and uppercase; map code to full program title (`BSIT`, `BSCS`, `BSIS`). |
| **SIS** | `email` | `emailAddress` | String | Trim spaces; convert to lowercase. |
| **LMS** | `course_code` | `courseCode` | String | Trim and uppercase. |
| **LMS** | `final_grade` | `finalGrade` | Number \| String | Convert numeric grades to decimal number; preserve non-numeric values such as `INC` as string status. |
| **LMS** | `completion_status` | `completionStatus` | String | Trim; normalize capitalization consistently to proper case (e.g., `'completed'` &rarr; `'Completed'`). |
| **Accounting** | `account_balance` | `outstandingBalance` | Number | Trim spaces; convert to numeric decimal. |
| **Accounting** | `currency` | `currency` | String | Trim spaces and uppercase. |

### Program Code Mapping Table:
- `BSIT` &rarr; `Bachelor of Science in Information Technology`
- `BSCS` &rarr; `Bachelor of Science in Computer Science`
- `BSIS` &rarr; `Bachelor of Science in Information Systems`

---

## TASK 3 - Transformation Logic & Design

1. **Text Trimming & Standardization**:
   - Leading and trailing spaces are trimmed across all input fields.
   - Text casing is standardized (lowercase emails, proper case names, uppercase codes and currencies).

2. **Name Proper Casing**:
   - Words are normalized so that the first letter of each word is capitalized and the remainder is lowercase (`'JUAN dela CRUZ'` &rarr; `'Juan Dela Cruz'`).

3. **Identifier Conversion**:
   - `student_id` is parsed into an integer (`Number.parseInt(idText, 10)`), retaining identifier semantics.

4. **Non-Numeric Grade (`INC`) Handling Design**:
   - In academic evaluation, `INC` signifies an administrative/academic status ("Incomplete") rather than a quantifiable numeric achievement (such as 1.0–5.0 or 0–100).
   - Forcing `INC` to a numeric value (e.g., `0`, `-1`, or `NaN`) would corrupt mathematical operations like Grade Point Average (GPA / GWA) calculations and misrepresent a student's standing as an academic failure.
   - Retaining `INC` as a string (`Number | String`) preserves semantic meaning for downstream academic evaluation processes, flags records requiring faculty resolution, and avoids invalid accounting or academic sanctions.

5. **Handling Missing LMS / Accounting Data**:
   - SIS is treated as the primary master data source (Left Outer Join).
   - If a student in SIS has no LMS entry (e.g., Student 1004), the `lms` property is set to `null` rather than dropping the student record.
   - If a student in SIS has no Accounting record (e.g., Student 1006), the `accounting` property is set to `null` while preserving the student profile.
   - Records present in LMS or Accounting but absent in SIS (e.g., Student 1005) are captured as unmatched records without corrupting master student data.

---

## TASK 4 - Integrated JSON Output

```json
[
    {
        "studentId": 1001,
        "studentName": "Juan Dela Cruz",
        "program": "Bachelor of Science in Information Technology",
        "emailAddress": "juan.dc@example.com",
        "lms": {
            "courseCode": "IT101",
            "finalGrade": 1.5,
            "completionStatus": "Completed"
        },
        "accounting": {
            "outstandingBalance": 0,
            "currency": "PHP"
        }
    },
    {
        "studentId": 1002,
        "studentName": "Maria Santos",
        "program": "Bachelor of Science in Computer Science",
        "emailAddress": "maria.santos@example.com",
        "lms": {
            "courseCode": "CS101",
            "finalGrade": 1.75,
            "completionStatus": "Completed"
        },
        "accounting": {
            "outstandingBalance": 5000,
            "currency": "PHP"
        }
    },
    {
        "studentId": 1003,
        "studentName": "Pedro Reyes",
        "program": "Bachelor of Science in Information Technology",
        "emailAddress": "pedro.reyes@example.com",
        "lms": {
            "courseCode": "IT101",
            "finalGrade": 2,
            "completionStatus": "Completed"
        },
        "accounting": {
            "outstandingBalance": 1500,
            "currency": "PHP"
        }
    },
    {
        "studentId": 1004,
        "studentName": "Ana Garcia",
        "program": "Bachelor of Science in Information Systems",
        "emailAddress": "ana.garcia@example.com",
        "lms": null,
        "accounting": {
            "outstandingBalance": 2500.5,
            "currency": "PHP"
        }
    },
    {
        "studentId": 1006,
        "studentName": "Liza Mendoza",
        "program": "Bachelor of Science in Information Technology",
        "emailAddress": "liza.mendoza@example.com",
        "lms": {
            "courseCode": "IT101",
            "finalGrade": "INC",
            "completionStatus": "In Progress"
        },
        "accounting": null
    }
]
```

