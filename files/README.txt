================================================================
 H.O.P.E - Part A Sample Files  (read me first)
================================================================

These files are the inputs for the Patient Registration application
described in your "HOPE Part A UiPath Build Guide". Build the robot by
following that guide; these files are what it reads and writes.

----------------------------------------------------------------
1. WHAT'S IN HERE  /  WHERE TO PUT IT
----------------------------------------------------------------
Drop each folder into your UiPath project folder (HOPE_HealthcareAdmin):

  Input\
    SamplePatient1.pdf              valid new patient  (happy path)
    SamplePatient2_Duplicate.pdf    same NRIC as #1    (duplicate path)
    SamplePatient3_MissingContact.pdf  Contact line removed (missing path)

  Database\
    PatientDatabase.xlsx            sheet "Patients" with the 12 headers,
                                    no data rows yet (the robot fills it)

  Scripts\
    validation.py                   Python validator (optional Part A extra)

  Output\Logs\                      empty - ProcessLog.txt is created at run
  Output\Reports\                   empty - used later in Part B

----------------------------------------------------------------
2. RECOMMENDED TEST ORDER (matches the guide's test checklist)
----------------------------------------------------------------
The duplicate check compares against the Excel database, so run #1 first.

  1) pdfFilePath = Input\SamplePatient1.pdf
        -> new row in PatientDatabase.xlsx + "SUCCESS" in ProcessLog.txt
  2) pdfFilePath = Input\SamplePatient2_Duplicate.pdf
        -> NO new row + "DUPLICATE" in the log
  3) pdfFilePath = Input\SamplePatient3_MissingContact.pdf
        -> NO new row + "MISSING" in the log
  4) pdfFilePath = a path that does not exist
        -> Catch runs + "ERROR" in the log (robot does not crash)

----------------------------------------------------------------
3. TWO THINGS TO KNOW ABOUT THE SAMPLE DATA
----------------------------------------------------------------
* NRIC numbers are checksum-VALID (S9012345I, T0123456G). This is so the
  Python validator in section 4 accepts them. They differ by one letter
  from the "S9012345A" placeholder shown in the build guide - that
  placeholder was illustrative; the regex matches either. Use these files.

* Appointment dates are set to Sep/Oct 2026 so they are in the future.
  validate_date() rejects past dates. If you test after those dates, just
  edit the date in the PDF, or set STRICT date handling aside - otherwise
  the date check will (correctly) flag them as past.

----------------------------------------------------------------
4. OPTIONAL: WIRE IN validation.py  (the ONE extra step vs the guide)
----------------------------------------------------------------
This is the single enhancement from the roadmap - it folds Python format
+ NRIC-checksum validation into Patient Registration. Everything else in
the build guide stays exactly the same.

WHERE IT GOES: inside the build guide's Step 5 "Then" branch (required
fields are present), BEFORE Step 6 (Use Excel File). So the order becomes:
   not-empty check (Step 5) -> Python validation (NEW) -> duplicate + insert.

SETUP:
  - Manage Packages -> install UiPath.Python.Activities.
  - Add a "Python Scope" activity. In its properties set your Python
    install (the folder containing python.exe) and the matching Target
    (x86/x64). Put the three activities below inside it.

ACTIVITIES (inside Python Scope):
  a) Load Python Script
        File              = "Scripts\validation.py"
        Output            = pyScript        (type: PythonObject)
  b) Invoke Python Method
        Instance          = pyScript
        Name              = "validate_patient"
        Input parameters  = {patientName, patientNRIC, patientContact,
                             patientEmail, appointmentDate}
        Output            = pyResultObj     (type: PythonObject)
  c) Get Python Object
        PythonObject      = pyResultObj
        TypeArgument      = String
        Output            = pyResult         (type: String)

THEN BRANCH ON THE RESULT:
  - If  pyResult = "VALID"  -> continue to Step 6 (Use Excel File / insert).
  - Else                    -> Append Line to ProcessLog.txt:
        timeStamp & " | INVALID | " & pyFilePath & " | " & pyResult
    and skip the insert (same idea as the MISSING path).

New variables to add: pyScript (PythonObject), pyResultObj (PythonObject),
pyResult (String).

NOTE: validation.py has STRICT_NRIC_CHECKSUM = True at the top. Set it to
False if you want a format-only NRIC check instead of the full checksum.

----------------------------------------------------------------
5. WHEN YOU SUBMIT
----------------------------------------------------------------
Zip the whole project folder (with these Input/Database/Output/Scripts
files inside) named:
  2504242E_SyedGhaziBinSyedMuhammadShafeeqAlsagoff_ProjectDeliverablesPartA.zip
Due 9 July, Thursday, 1700 hrs on TPLMS. Use .zip, not .rar.
