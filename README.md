# Aeries Data Validations
List of Data validations for the Aeries SIS

## Validation Group:  ADSBully

**Key Field 1:**  Student ID                     **Link to Page 1:**  AssertiveDiscipline.aspx  
**Key Field 2:**  Sequence Number                     **Link to Page 2:**    
**Severity Level:**  E  
**Student Relate:** Yes         **User Can Refresh:** Yes  
  
**Possible Cause:**  Any discipline incident that has 48900(r) Bullying as one of the codes must also have either 48900.2 Sexual Harassment, 48900.3   Hate Violence, or 48900.4 Harassment as one of the other codes.  
  
**Possible Solution:**  Investigate the incident and select either 48900.2 Sexual Harassment, 48900.3   Hate Violence, or 48900.4 Harassment as one of the other codes.  
  

**Query Definition:**
```sql
DECLARE @yr AS VARCHAR(4) = '20' + SUBSTRING(DB_NAME(), 4, 2)

SELECT
    ADS.SCL                                           AS [SchoolCode]
   ,ADS.PID                                           AS [Student ID]
   ,ADS.SQ                                            AS [Sequence Number]
   ,ADS.SQ                                            AS [SQ?]
   ,'Incident Date: ' + CONVERT(VARCHAR, ADS.DT, 101) AS [Description]
   ,CONVERT(VARCHAR, ADS.DT, 101)                     AS [Date]
FROM ADS
WHERE '23' IN (ADS.CD, ADS.CD2, ADS.CD3, ADS.CD4, ADS.CD5)
    AND '01' NOT IN (ADS.CD, ADS.CD2, ADS.CD3, ADS.CD4, ADS.CD5)
    AND '02' NOT IN (ADS.CD, ADS.CD2, ADS.CD3, ADS.CD4, ADS.CD5)
    AND '03' NOT IN (ADS.CD, ADS.CD2, ADS.CD3, ADS.CD4, ADS.CD5)
    AND ADS.DT > '7/1/' + @yr
    AND ADS.SCL = @SchoolCode
    AND ADS.PID IN (@StudentID)
ORDER BY ADS.DT DESC  
``` 



## Validation Group:  CAROvlp

**Key Field 1:**  Student Num                     **Link to Page 1:**  CourseAttendance.aspx  
**Key Field 2:**                       **Link to Page 2:**    
**Severity Level:**  W  
**Student Relate:**  Yes         **User Can Refresh:**  Yes  
  
**Possible Cause:**  The following are errors where CAR records are overlapping for the same dates in the same period number.  
  
**Possible Solution:**  Start and end dates should not overlap between courses for the student in the same period number.  
  

**Query Definition:**
```sql
SELECT
    STU.SC                              AS [SchoolCode]
   ,STU.ID                              AS [Student ID]
   ,STU.SN                              AS [Student Num]
   ,'PD: ' + CONVERT(NVARCHAR, C1.PD)
    + '   CN: ' + C1.CN + ' ' + CONVERT(NVARCHAR, C1.DS, 1)
    + '-' + CONVERT(NVARCHAR, C1.DE, 1)
    + '   CN: ' + C2.CN + ' ' + CONVERT(NVARCHAR, C2.DS, 1)
    + '-' + CONVERT(NVARCHAR, C2.DE, 1) AS [Description]
FROM CAR AS C1
    INNER JOIN CAR AS C2
          ON  C1.SC = C2.SC
              AND C1.SN = C2.SN
              AND C1.PD = C2.PD
              AND C1.SQ > C2.SQ
              AND C1.DS <= C2.DE
              AND C2.DS <= C1.DE
    INNER JOIN STU
          ON  C1.SC = STU.SC
              AND C1.SN = STU.SN
    INNER JOIN MST
          ON  C1.SC = MST.SC
              AND C1.SE = MST.SE
              AND MST.ST NOT IN ('Y')
WHERE STU.DEL = 0
    AND C1.DEL = 0
    AND C2.DEL = 0
    AND STU.SC = @SchoolCode
    AND STU.ID IN (@StudentID)   
  ```

## Validation Group:  RET001

**Key Field 1:**  Student ID                     **Link to Page 1:**  Students.aspx  
**Key Field 2:**  Student ID2                     **Link to Page 2:**  Retentions.aspx  
**Severity Level:**  W  
**Student Relate:**  Yes         **User Can Refresh:**  Yes  
  
**Possible Cause:**  Student's **Grade**   (STU.GR) and **Next Grade**   (STU.NG) are equal but there is no retention record entered for the student.  
  
**Possible Solution:**  If the student is being retained, add a record under the **Retentions**   page. If the student is not being retained, change the Next Grade value to the grade the student will be promoted to.  
  

**Query Definition:**
```sql
DECLARE @yr AS VARCHAR(4) = '20' + SUBSTRING(DB_NAME(), 4, 2)

SELECT
    STU.SC                                         AS [SchoolCode]
   ,STU.ID                                         AS [Student ID]
   ,STU.ID                                         AS [Student ID2]
   ,STU.LN                                         AS [Last Name]
   ,STU.FN                                         AS [First Name]
   ,'Current Grade: ' + CAST(STU.GR AS VARCHAR)
    + ' -> Next Grade: ' + CAST(STU.NG AS VARCHAR) AS [Description]
FROM STU
    LEFT JOIN RET
          ON  STU.ID = RET.PID
              AND RET.DT >= '07/01/' + @yr
              AND RET.DEL = 0
WHERE STU.GR = STU.NG
    AND STU.GR <> -2
    AND STU.TG = ''
    AND STU.DEL = 0
    AND RET.PID IS NULL
    AND STU.SC = @SchoolCode
    AND STU.ID IN (@StudentID)  
  ```

  

## Validation Group:  ROL001

**Key Field 1:**  Student ID                     **Link to Page 1:**  Students.aspx  
**Key Field 2:**                       **Link to Page 2:**    
**Severity Level:**    
**Student Relate:**  Yes         **User Can Refresh:**  Yes  
  
**Possible Cause:**  The student is pre-enrolled but **Next School**   (STU.NS) field is set to another school in addition to this school. There should not be any pre-enrolled student records that are not going to this school  
  
**Possible Solution:**  If the student should not be pre-enrolled in this school, delete the student record. If the student should be pre-enrolled in this school, change the **Next School**   (STU.NS) code to this school.  
  

**Query Definition:**
```sql
SELECT
    STU.SC                                            AS [SchoolCode]
   ,STU.ID                                            AS [Student ID]
   ,STU.LN                                            AS [Last Name]
   ,STU.FN                                            AS [First Name]
   ,'Current School: ' + ISNULL(LOC.SNM, STU.SC)
    + ' -> Next School: ' + ISNULL(LOCNS.SNM, STU.NS) AS [Description]
FROM STU
    LEFT JOIN LOC
          ON  LOC.CD = STU.SC
              AND LOC.SNM <> ''
    LEFT JOIN LOC AS LOCNS
          ON  LOCNS.CD = STU.NS
              AND LOCNS.SNM <> ''
WHERE STU.TG = '*'
    AND STU.NS <> STU.SC
    AND STU.DEL = 0
    AND STU.SC = @SchoolCode
    AND STU.ID IN (@StudentID)  
  ```

  

## Validation Group:  ROL02

**Key Field 1:**  Student ID                     **Link to Page 1:**  Students.aspx  
**Key Field 2:**                       **Link to Page 2:**    
**Severity Level:**  E  
**Student Relate:**  Yes         **User Can Refresh:**  Yes  
  
**Possible Cause:**  There is a conflict in the **Next School**   (STU.NS) field between two pre-enrolled records for the same student. One school's pre-enrolled record has the student going to different next school than your site has them going to.  
  
**Possible Solution:**  If the student's Next School is not correct, change the **Next School**   (STU.NS) code to the correct school. If the other school's Next School is incorrect, reach out to them so they can update the next school for this student.  
  

**Query Definition:**
```sql
SELECT
    s1.ID                                                 AS [Student ID]
   ,s1.SC                                                 AS [SchoolCode]
   ,STRING_AGG(locSC2.SNM, ' & ')
    + '''s records say student is going to ' + locNS2.SNM AS [Description]
FROM STU AS s1
    INNER JOIN STU AS s2
          ON  s1.ID = s2.ID
              AND s1.SC <> s2.SC
              AND s1.NS <> s2.NS
    INNER JOIN LOC AS locSC2
          ON  locSC2.CD = s2.SC
              AND locSC2.TG = ''
    INNER JOIN LOC AS locNS2
          ON  locNS2.CD = s2.NS
              AND locNS2.TG = ''
WHERE s1.DEL = 0
    AND s2.DEL = 0
    AND s1.TG IN ('*', '')
    AND s2.TG IN ('*', '')
    AND s1.NG <> 13
    AND s2.NG <> 13
    AND s1.SC = @SchoolCode
    AND s1.ID IN (@StudentID)
GROUP BY s1.ID
        ,s1.SC
        ,locNS2.SNM    
```
  

## Validation Group:  SSID01

**Key Field 1:**  Student ID                     **Link to Page 1:**    
**Key Field 2:**                       **Link to Page 2:**    
**Severity Level:**  E  
**Student Relate:** Yes         **User Can Refresh:** Yes  
  
**Possible Cause:**  Student has two different SSIDs tied to the same Student ID.  
  
**Possible Solution:**  Merge the SSIDs in CALPADS and update the student records with the SSID that was kept.  
  

**Query Definition:**
```sql
SELECT
    stu.ID                                       AS [Student ID]
   ,'Linked SSIDs: ' + STRING_AGG(stu.CID, ', ') AS [Description]
FROM (SELECT DISTINCT
        STU.ID
       ,STU.CID
       ,COUNT(STU.CID) OVER (PARTITION BY STU.ID) AS totalCID
       ,COUNT(STU.ID) OVER (PARTITION BY STU.CID) AS totalID
    FROM (SELECT DISTINCT
            stu.ID
           ,stu.CID
        FROM STU
        WHERE STU.DEL = 0) AS stu
    WHERE STU.CID <> '') AS stu
WHERE stu.totalCID > stu.totalID
    AND stu.ID IN (@StudentID)
GROUP BY stu.ID  
```  

  

## Validation Group:  SSID02

**Key Field 1:**  SSID                     **Link to Page 1:**  CombineStudentRecords.aspx  
**Key Field 2:**                       **Link to Page 2:**    
**Severity Level:**  E  
**Student Relate:**  false         **User Can Refresh:** Yes  
  
**Possible Cause:**  The same SSID is linked to two different Student IDs.  
  
**Possible Solution:**  Merge the student records so only one Student ID remains.  
  

**Query Definition:**
```sql
SELECT
    p.CID                                    AS [SSID]
   ,'Duplicate IDs: ' + p.[1] + ', ' + p.[2] AS [Description]
FROM (SELECT
        CAST(stu.ID AS VARCHAR(10))                                             AS ID
       ,stu.CID
       ,ROW_NUMBER() OVER (PARTITION BY STU.CID ORDER BY STU.ID % 1000000 DESC) AS ordering
    FROM (SELECT
            STU.ID
           ,STU.CID
           ,COUNT(STU.CID) OVER (PARTITION BY STU.ID) AS totalCID
           ,COUNT(STU.ID) OVER (PARTITION BY STU.CID) AS totalID
        FROM (SELECT DISTINCT
                stu.ID
               ,stu.CID
            FROM STU
            WHERE STU.DEL = 0) AS stu
        WHERE STU.CID <> '') AS stu
    WHERE stu.totalCID < stu.totalID) t
PIVOT (MAX(t.ID) FOR t.ordering IN ([1], [2])) p
WHERE p.[1] IN (@StudentID)  
```

  

## Validation Group:  SOFF0095

**Key Field 1:**  Student ID                     **Link to Page 1:**  AssertiveDiscipline.aspx  
**Key Field 2:**  Sequence Number                     **Link to Page 2:**    
**Severity Level:**  E  
**Student Relate:** Yes         **User Can Refresh:** Yes  
  
**Possible Cause:**  Student who has an Assertive Discipline record of Dangerous Obj. Possessed/Furnish 48900(b), Firearm, Imitation 48900(m), Firearm: Possessed/Sold/Furnish 48915(c)(1), Knife: Brandished 48915(c)(2), Dangerous Object 48915(a)(1)B must have the Weapon T  
  
**Possible Solution:**   1) If the Assertive Discipline record is incorrectly populated, change it to the correct value     OR 2) If the Student Offense Code is correctly populated in the submission, ensure a correct value for Weapon Type is provided.  
  

**Query Definition:**
```sql
SELECT
    STU.SC                                            AS [SchoolCode]
   ,STU.ID                                            AS [Student ID]
   ,ADS.SQ                                            AS [Sequence Number]
   ,'Incident Date: ' + CONVERT(VARCHAR, ADS.DT, 101) AS [Description]
FROM STU
    INNER JOIN ADS
          ON  ADS.PID = STU.ID
    INNER JOIN COD
          ON  COD.TC = 'ADS'
              AND COD.FC = 'CD'
              AND ADS.CD = COD.CD
WHERE ADS.DT > '7/1/2022'
    AND ('07' IN (ADS.CD, ADS.CD2, ADS.CD3, ADS.CD4, ADS.CD5)
        OR '18' IN (ADS.CD, ADS.CD2, ADS.CD3, ADS.CD4, ADS.CD5)
        OR '25' IN (ADS.CD, ADS.CD2, ADS.CD3, ADS.CD4, ADS.CD5)
        OR '26' IN (ADS.CD, ADS.CD2, ADS.CD3, ADS.CD4, ADS.CD5)
        OR '31' IN (ADS.CD, ADS.CD2, ADS.CD3, ADS.CD4, ADS.CD5))
    AND ADS.WT = ''
    AND STU.SC = @SchoolCode
    AND STU.ID IN (@StudentID)  
  ```

  

## Validation Group:  TchStfID

**Key Field 1:**  Teacher Number                     **Link to Page 1:**  Teachers.aspx  
**Key Field 2:**                       **Link to Page 2:**    
**Severity Level:**  E  
**Student Relate:**  false         **User Can Refresh:** Yes  
  
**Possible Cause:**  Teacher record has students rostered to it but the Staff ID field is blank.  
  
**Possible Solution:**  Add the Staff ID to the teacher record by clicking on the "Change" button, then use the search glass to locate the Staff ID. If the ID is not located by name via search, contact the Personnel department to identify the Staff ID.  
  

**Query Definition:**
```sql
SELECT
DISTINCT
    TCH.SC                 AS [SchoolCode]
   ,TCH.TN                 AS [Teacher Number]
   ,TCH.TN                 AS [TN?]
   ,TCH.TF + ' ' + TCH.TLN AS [Description]
FROM TCH
    INNER JOIN MST
          ON  TCH.SC = MST.SC
              AND TCH.TN = MST.TN
              AND MST.DEL = 0
    INNER JOIN SEC
          ON  MST.SC = SEC.SC
              AND MST.SE = SEC.SE
              AND SEC.DEL = 0
WHERE TCH.ID = 0
    AND TCH.DEL = 0
    AND TCH.TN <> 0
    AND TCH.SC = @SchoolCode  
  ```

  

## Validation Group:  CERT146

**Key Field 1:**  Student ID                     **Link to Page 1:**  Programs.aspx  
**Key Field 2:**                       **Link to Page 2:**    
**Severity Level:**  E  
**Student Relate:** Yes         **User Can Refresh:** Yes  
  
**Possible Cause:**  The student must have a LIP program record 300-307 with eligibility dates that cover their enrollment if the student is an English Learner.  
  
**Possible Solution:**  Verify that the student is an English Learner and if they are verify that student has a corresponding LIP program record and that it overlaps completely with the student's enrollment.  
  

**Query Definition:**
```sql
SELECT
    STU.SC                                     AS [SchoolCode]
   ,STU.ID                                     AS [Student ID]
   ,'Student is EL without LIP Program Record' AS [Description]
FROM STU
    INNER JOIN LOC
          ON  LOC.CD = STU.SC
              AND LOC.TG = ''
    LEFT JOIN PGM
          ON  PGM.PID = STU.ID
              AND PGM.DEL = 0
              AND PGM.CD BETWEEN '300' AND '307'
              AND GETDATE() BETWEEN PGM.ESD AND ISNULL(PGM.EED, '1/1/9999')
WHERE STU.DEL = 0
    AND STU.SC = @SchoolCode
    AND STU.ID IN (@StudentID)
    AND STU.LF = 'L'
    AND STU.TG = ''
    AND PGM.CD IS NULL  
 ``` 

  

## Validation Group:  SINF0059

**Key Field 1:**  Student ID                     **Link to Page 1:**  Students.aspx  
**Key Field 2:**                       **Link to Page 2:**  Language.aspx  
**Severity Level:**  E  
**Student Relate:** Yes         **User Can Refresh:** Yes  
  
**Possible Cause:**  If Student Initial US School Enrollment Date K-12 is populated, then student age as of that date must be at least 4 years. Do not use Pre-School enter date, only enter the date for TK and above.  
  
**Possible Solution:**   Do not put the Preschool Start Date into the the Initial US School Enrollment Date K-12 field. If the date in the Initial US School Enrollment Date K-12 field is not the US School Grade K enter date, change the date to the US School Grade K enter date. If the student entered a US school in a higher grade, put that as the date into this field.     If the the Student Birth Date is incorrect, that could also cause this error. In that case, correct the student birth date.<br />  
  

**Query Definition:**
```sql
SELECT
    STU.SC                                                                 [SchoolCode]
   ,STU.ID                                                                 [Student ID]
   ,'Student Age at K-12 Enter Date: '
    + CAST(DATEDIFF(DAY, STU.BD, LAC.USS) / $365.25 AS VARCHAR) + ' years' [Description]
FROM STU
    INNER JOIN LAC
          ON  STU.ID = LAC.ID
              AND LAC.DEL = 0
WHERE STU.DEL = 0
    AND STU.TG <> 'N'
    AND LAC.USS <> ''
    -- 4 years converted to days
    AND DATEDIFF(DAY, STU.BD, LAC.USS) < 1461
    AND STU.SC = @SchoolCode
    AND STU.ID IN (@StudentID)  
  ```

  

## Validation Group:  CRSE0133

**Key Field 1:**  Section Number                     **Link to Page 1:**    
**Key Field 2:**  Course                     **Link to Page 2:**    
**Severity Level:**  E  
**Student Relate:**  false         **User Can Refresh:** Yes  
  
**Possible Cause:**  Sections for CTE courses must have the **Course Provider Code**   populated on the Master Schedule for that section.  
  
**Possible Solution:**  Set the correct Course Provider Code on the Master Schedule for that section.  
  

**Query Definition:**
```sql
SELECT
    MST.SC                       [SchoolCode]
   ,MST.SE                       [Section Number]
   ,CRS.CO + ' (' + CRS.CN + ')' [Course]
   ,''                           [Description]
   ,MST.SE                       [SE?]
FROM MST
    JOIN CRS
          ON  MST.CN = CRS.CN
WHERE MST.DEL = 0
    AND CRS.DEL = 0
    AND MST.ST NOT IN ('X', 'Z')
    AND MST.TEP = ''
    AND CRS.C3 BETWEEN 7000 AND 8999
    AND @SchoolCode = MST.SC  
  ```

  

## Validation Group:  CRSE0457

**Key Field 1:**  Section Number                     **Link to Page 1:**  MasterSchedule.aspx  
**Key Field 2:**  Course                     **Link to Page 2:**    
**Severity Level:**  E  
**Student Relate:**  false         **User Can Refresh:** Yes  
  
**Possible Cause:**   If the **Dist Lrng**   = Y, then the **Online Crs Instr. Type**     Code must be populated.  
  
**Possible Solution:**  Ensure the **Online Crs Instr. Type**   Code is populated or clear the value from the **Dist Lrng**   field.<br />  
  

**Query Definition:**
```sql
SELECT
    MST.SC                       [SchoolCode]
   ,MST.SE                       [Section Number]
   ,CRS.CO + ' (' + CRS.CN + ')' [Course]
   ,''                           [Description]
   ,MST.SE                       [SE?]
FROM MST
    JOIN CRS
          ON  MST.CN = CRS.CN
WHERE MST.DEL = 0
    AND CRS.DEL = 0
    AND MST.ST NOT IN ('X', 'Z')
    AND MST.DLI = 'Y'
    AND MST.OIT = ''
    AND @SchoolCode = MST.SC  
  ```

  

## Validation Group:  CRSE0458

**Key Field 1:**  Course                     **Link to Page 1:**  Courses.aspx  
**Key Field 2:**                       **Link to Page 2:**    
**Severity Level:**  E  
**Student Relate:**  false         **User Can Refresh:** Yes  
  
**Possible Cause:**  If Departmentalized Course Standards Grade Level Range = MID (Middle Grades 5-8), then Middle School Core Setting Indicator must equal Y or N.  
  
**Possible Solution:**  Ensure Middle School Core Setting Indicator is not equal to Not Applicable or update value for CRS-Departmentalized Course Standards Grade Level Range Code.<br />  
  

**Query Definition:**
```sql
SELECT
    CRS.CN [Course]
   ,CRS.CO [Description]
   ,CRS.CN [CN?]
FROM CRS
WHERE CRS.DEL = 0
    AND (CRS.TG = ''
        OR CRS.NTG = '')
    AND CRS.SGR = 'MID'
    AND CRS.MSC = ''  
  ```

  

## Validation Group:  IndStd001

**Key Field 1:**  Student ID                     **Link to Page 1:**  Attendance.aspx  
**Key Field 2:**                       **Link to Page 2:**  AttendanceEnr.aspx  
**Severity Level:**  E  
**Student Relate:** Yes         **User Can Refresh:** Yes  
  
**Possible Cause:**   A student cannot have "I-Independent Study" attendance codes if they do not occur during an attendance enrollment window that has a program code of "I". If a student has an independent study program code of "I" their attendance codes for those days mus  
  
**Possible Solution:**   Either clear out the attendance code for the listed days if it was entered incorrectly.<br />OR<br />Add/edit an Attendance Enrollment record for the student with an Independent Study program to cover the days that the student participated in independent study.  
  

**Query Definition:**
```sql
DECLARE @yr AS VARCHAR(4) = '20' + SUBSTRING(DB_NAME(), 4, 2)

SELECT
    STU.SC                                           [SchoolCode]
   ,STU.ID                                           [Student ID]
   ,SUBSTRING(STRING_AGG(SUBSTRING(CONVERT(
    VARCHAR(10), DAY.DT, 101), 1, 5), ', '), 1, 100) AS [Description]
FROM STU
    LEFT JOIN (ENR
              INNER JOIN DAY
                    ON  ENR.SC = DAY.SC
                        AND ENR.DEL = 0
                        AND ENR.PR = 'I'
                        AND ENR.YR = @yr
                        AND day.DEL = 0
                        AND DAY.HO NOT IN ('@', '#', '$')
                        AND day.DT <= GETDATE()
                        AND DAY.DT BETWEEN ENR.ED AND ISNULL(ENR.LD, DATEADD(DAY, 14, GETDATE())))
          ON  STU.ID = ENR.ID
              AND STU.SC = ENR.SC
    FULL OUTER JOIN ATT
          ON  STU.SC = ATT.SC
              AND STU.SN = ATT.SN
              AND ATT.DEL = 0
              AND ATT.DT = DAY.DT
              AND ATT.AL IN ('I', 'K')
WHERE STU.DEL = 0
    AND ((ATT.DT IS NULL
            AND DAY.DT IS NOT NULL)
        OR (ATT.DT IS NOT NULL
            AND DAY.DT IS NULL))
    AND STU.SC = @SchoolCode
    AND STU.ID IN (@StudentID)
GROUP BY STU.SC
        ,STU.ID  
 ``` 

  

## Validation Group:  IEP504

**Key Field 1:**  Student ID                     **Link to Page 1:**  SpecialEd.aspx  
**Key Field 2:**                       **Link to Page 2:**  Program504.aspx  
**Severity Level:**  E  
**Student Relate:** Yes         **User Can Refresh:** Yes  
  
**Possible Cause:**  Student cannot have overlapping dates for their IEP and 504 records.  
  
**Possible Solution:**  Add an end date to the either the IEP or the 504 plan so they no longer overlap  
  

**Query Definition:**
```sql
SELECT
    STU.SC                                                         AS [SchoolCode]
   ,STU.ID                                                         AS [Student ID]
   ,'SpEd: ' + ISNULL(CONVERT(VARCHAR(10), CSE.ED, 101), '')
    + '-' + ISNULL(CONVERT(VARCHAR(10), CSE.XD, 101), '(current)')
    + ' 504: ' + ISNULL(CONVERT(VARCHAR(10), FOF.SD, 101), '')
    + '-' + ISNULL(CONVERT(VARCHAR(10), FOF.ED, 101), '(current)') AS [Description]
FROM STU
    INNER JOIN CSE
          ON  STU.ID = CSE.ID
              AND CSE.PT IN ('1', '100')
              AND CSE.DEL = 0
    INNER JOIN FOF
          ON  CSE.ID = FOF.ID
              AND (FOF.SD BETWEEN CSE.ED AND ISNULL(CSE.XD, '1/1/9999')
                  OR ISNULL(FOF.ED, GETDATE()) BETWEEN CSE.ED AND ISNULL(CSE.XD, '1/1/9999'))
WHERE STU.DEL = 0
    AND STU.SC = @SchoolCode
    AND STU.ID IN (@StudentID)  
  ```

  

## Validation Group:  MstEld

**Key Field 1:**  Section                     **Link to Page 1:**  MasterSchedule.aspx  
**Key Field 2:**                       **Link to Page 2:**    
**Severity Level:**    
**Student Relate:**  false         **User Can Refresh:** Yes  
  
**Possible Cause:**  A section assigned to an ELD course does not have the "Ed Svc" field tagged correctly.  
  
**Possible Solution:**  ELD course sections should be tagged as "2-Designated ELD" if the section is an ELD course section.  
  

**Query Definition:**
```sql
SELECT
    MST.SC
   ,MST.SE                                       AS Section
   ,MST.SE                                       AS [SE?]
   ,'Section: ' + CAST(MST.SE AS VARCHAR)
    + ' Course: ' + CRS.DE + ' (' + MST.CN + ')' AS Description
FROM MST
    INNER JOIN CRS
          ON  MST.CN = CRS.CN
WHERE MST.DEL = 0
    AND CRS.C3 = '9104'
    AND MST.ESR <> '2'
    AND MST.SC = @SchoolCode  
  ```

  

## Validation Group:  SELA001

**Key Field 1:**  Student ID                     **Link to Page 1:**  Language.aspx  
**Key Field 2:**                       **Link to Page 2:**    
**Severity Level:**  E  
**Student Relate:** Yes         **User Can Refresh:** Yes  
  
**Possible Cause:**  CALPADS has a language fluency code that the Aeries SELA extract cannot update because the transition is invalid. An EO language fluency in Aeries cannot overwrite a RFEP, IFEP, or EL status in CALPADS.  
  
**Possible Solution:**   Update the language fluency code to a status that is a valid language fluency status.  
  

**Query Definition:**
```sql
;WITH base AS
  (SELECT
          STU.SC
         ,STU.ID
         ,STU.CID
         ,XRF.CD AS LF
         ,LAC.EAC
         ,LAC.EAD
         ,LAC.RD1
         ,LAC.SD
         ,LAC.IFD
         ,LAC.AES
      FROM STU
          INNER JOIN LOC
                ON  LOC.CD = STU.SC
                    AND LOC.TG = ''
          INNER JOIN LAC
                ON  STU.ID = LAC.ID
                    AND LAC.DEL = 0
          INNER JOIN XRF
                ON  XRF.DEL = 0
                    AND XRF.TC = 'CLPD'
                    AND XRF.SC = 0
                    AND XRF.TC1 = 'STU'
                    AND XRF.FC1 = 'LF'
                    AND STU.LF = XRF.CD1
      WHERE STU.DEL = 0
          AND LAC.DEL = 0
          AND RTRIM(STU.CID) <> ''
          AND STU.TG IN ('I', '', ' '))
SELECT
    base.SC
   ,base.ID                                           AS [Student ID]
   ,'CALPADS has ' + base.EAC + ' we have ' + base.LF AS Description
FROM base
WHERE base.LF = 'EO'
    AND base.EAC IN ('RFEP', 'IFEP', 'EL')
    AND base.SC = @SchoolCode
    AND base.ID IN (@StudentID)  
  ```

  

## Validation Group:  SELA002

**Key Field 1:**  Student ID                     **Link to Page 1:**  Language.aspx  
**Key Field 2:**                       **Link to Page 2:**    
**Severity Level:**  E  
**Student Relate:** Yes         **User Can Refresh:** Yes  
  
**Possible Cause:**  We have not updated our TBD status to the CALPADS language fluency status.  
  
**Possible Solution:**  Update the language fluency in Aeries to the CALPADS language fluency status.  
  

**Query Definition:**
```sql
;WITH base AS
  (SELECT
          STU.SC
         ,STU.ID
         ,STU.CID
         ,XRF.CD AS LF
         ,LAC.EAC
         ,LAC.EAD
         ,LAC.RD1
         ,LAC.SD
         ,LAC.IFD
         ,LAC.AES
      FROM STU
          INNER JOIN LOC
                ON  LOC.CD = STU.SC
                    AND LOC.TG = ''
          INNER JOIN LAC
                ON  STU.ID = LAC.ID
                    AND LAC.DEL = 0
          INNER JOIN XRF
                ON  XRF.DEL = 0
                    AND XRF.TC = 'CLPD'
                    AND XRF.SC = 0
                    AND XRF.TC1 = 'STU'
                    AND XRF.FC1 = 'LF'
                    AND STU.LF = XRF.CD1
      WHERE STU.DEL = 0
          AND LAC.DEL = 0
          AND RTRIM(STU.CID) <> ''
          AND STU.TG IN ('I', '', ' '))
SELECT
    base.SC
   ,base.ID                                           AS [Student ID]
   ,'CALPADS has ' + base.EAC + ' we have ' + base.LF AS Description
FROM base
WHERE base.LF = 'TBD'
    AND base.EAC IN ('RFEP', 'IFEP', 'EL', 'EO')
    AND base.SC = @SchoolCode
    AND base.ID IN (@StudentID)  
  ```

  

## Validation Group:  SELA003

**Key Field 1:**  Student ID                     **Link to Page 1:**  Language.aspx  
**Key Field 2:**                       **Link to Page 2:**    
**Severity Level:**  E  
**Student Relate:** Yes         **User Can Refresh:** Yes  
  
**Possible Cause:**  Student was determined to be EL in CALPADS but we have them as RFEP.  
  
**Possible Solution:**  Make sure student was correctly marked as RFEP, otherwise change them to EL status to get tested.  
  

**Query Definition:**
```sql
;WITH base AS
  (SELECT
          STU.SC
         ,STU.ID
         ,STU.CID
         ,XRF.CD AS LF
         ,LAC.EAC
         ,LAC.EAD
         ,LAC.RD1
         ,LAC.SD
         ,LAC.IFD
         ,LAC.AES
      FROM STU
          INNER JOIN LOC
                ON  LOC.CD = STU.SC
                    AND LOC.TG = ''
          INNER JOIN LAC
                ON  STU.ID = LAC.ID
                    AND LAC.DEL = 0
          INNER JOIN XRF
                ON  XRF.DEL = 0
                    AND XRF.TC = 'CLPD'
                    AND XRF.SC = 0
                    AND XRF.TC1 = 'STU'
                    AND XRF.FC1 = 'LF'
                    AND STU.LF = XRF.CD1
      WHERE STU.DEL = 0
          AND LAC.DEL = 0
          AND RTRIM(STU.CID) <> ''
          AND STU.TG IN ('I', '', ' '))
SELECT
    base.SC
   ,base.ID                                           AS [Student ID]
   ,'CALPADS has ' + base.EAC + ' we have ' + base.LF AS Description
FROM base
WHERE base.LF = 'RFEP'
    AND base.EAC IN ('EL')
    AND ISNULL(base.RD1, '1/1/9999') <= ISNULL(base.EAD, '1/1/9999')
    AND base.SC = @SchoolCode
    AND base.ID IN (@StudentID)  
  ```

  

## Validation Group:  SELA004

**Key Field 1:**  Student ID                     **Link to Page 1:**  Language.aspx  
**Key Field 2:**                       **Link to Page 2:**    
**Severity Level:**  E  
**Student Relate:** Yes         **User Can Refresh:** Yes  
  
**Possible Cause:**  The CALPADS ELAS date does not match the Aeries [EL Start Date] or [RFEP Date] or [IFEP Date] or [ADEL Date].  
  
**Possible Solution:**  Set the correct Aeries [EL Start Date] or [RFEP Date] or [IFEP Date] or [ADEL Date] to match the CALPADS ELAS date or update the CALPADS ELAS date to match Aeries.  
  

**Query Definition:**
```sql
;WITH base AS
  (SELECT
          STU.SC
         ,STU.ID
         ,STU.CID
         ,XRF.CD AS LF
         ,LAC.EAC
         ,LAC.EAD
         ,LAC.RD1
         ,LAC.SD
         ,LAC.IFD
         ,LAC.AES
      FROM STU
          INNER JOIN LOC
                ON  LOC.CD = STU.SC
                    AND LOC.TG = ''
          INNER JOIN LAC
                ON  STU.ID = LAC.ID
                    AND LAC.DEL = 0
          INNER JOIN XRF
                ON  XRF.DEL = 0
                    AND XRF.TC = 'CLPD'
                    AND XRF.SC = 0
                    AND XRF.TC1 = 'STU'
                    AND XRF.FC1 = 'LF'
                    AND STU.LF = XRF.CD1
      WHERE STU.DEL = 0
          AND LAC.DEL = 0
          AND RTRIM(STU.CID) <> ''
          AND STU.TG IN ('I', '', ' '))
SELECT
    base.SC
   ,base.ID                                                                AS [Student ID]
   ,'ELAS Date in CALPADS is ' + CONVERT(VARCHAR, base.EAD, 101)
    + ' Aeries has ' + CONVERT(VARCHAR, ISNULL(base.RD1,
    ISNULL(base.SD, ISNULL(base.IFD, ISNULL(base.AES, '1/1/1900')))), 101) AS Description
FROM base
WHERE base.LF IN ('EO', 'TBD')
    AND ISNULL(base.EAD, '1/1/1900')
    <> ISNULL(base.RD1, ISNULL(base.SD, ISNULL(base.IFD, ISNULL(base.AES, ISNULL(base.EAD, '1/1/1900')))))
    AND base.SC = @SchoolCode
    AND base.ID IN (@StudentID)  
  ```

  

## Validation Group:  DupSTU

**Key Field 1:**  Student ID                     **Link to Page 1:**  Students.aspx  
**Key Field 2:**  Duplicate ID                     **Link to Page 2:**  DistrictInfo.aspx  
**Severity Level:**  W  
**Student Relate:** Yes         **User Can Refresh:** Yes  
  
**Possible Cause:**  The listed students have the same First Name, Last Name, and Birthdate but a different Student ID which indicates this student has been incorrectly duplicated in the database. You can use the District Student Lookup search with the first name and last nam  
  
**Possible Solution:**  If you just created this student record, delete it and use the District Student Lookup page to search by name and copy the old record and update it with new information. If you have caught this error after the student has already been active for a while you need to contact the district Aeries admin to combine the student records into one ID.  
  

**Query Definition:**
```sql
SELECT DISTINCT
    S1.SC                 AS [SchoolCode]
   ,S1.ID                 AS [Student ID]
   ,S2.ID                 AS [Duplicate ID]
   ,LOC.NM + ' has (' + CAST(S2.ID AS VARCHAR) + ') '
    + S2.FN + ' ' + S2.LN AS [Description]
FROM STU AS S1
    ,STU AS s2
         LEFT JOIN LOC
               ON  S2.SC = LOC.CD
WHERE S1.DEL = 0
    AND S2.DEL = 0
    AND S1.LN = S2.LN
    AND S1.FN = S2.FN
    AND S1.BD = S2.BD
    AND S1.ID <> S2.ID
    AND S1.GR <> -2
    AND S2.GR <> -2
    AND S1.SC = @SchoolCode
    AND S1.ID IN (@StudentID)  
```  

  

## Validation Group:  SELA005

**Key Field 1:**  Student ID                     **Link to Page 1:**  Language.aspx  
**Key Field 2:**                       **Link to Page 2:**    
**Severity Level:**  W  
**Student Relate:** Yes         **User Can Refresh:** Yes  
  
**Possible Cause:**   The RFEP date is in a previous school year and won't be extracted by Aeries in the SELA file.  
  
**Possible Solution:**  Manually add the Reclassified record into CALPADS  
  

**Query Definition:**
```sql
DECLARE @yr AS VARCHAR(4) = '20' + SUBSTRING(DB_NAME(), 4, 2)

;WITH base AS
  (SELECT
          STU.SC
         ,STU.ID
         ,STU.CID
         ,XRF.CD AS LF
         ,LAC.EAC
         ,LAC.EAD
         ,LAC.RD1
         ,LAC.SD
         ,LAC.IFD
         ,LAC.AES
      FROM STU
          INNER JOIN LOC
                ON  LOC.CD = STU.SC
                    AND LOC.TG = ''
          INNER JOIN LAC
                ON  STU.ID = LAC.ID
                    AND LAC.DEL = 0
          INNER JOIN XRF
                ON  XRF.DEL = 0
                    AND XRF.TC = 'CLPD'
                    AND XRF.SC = 0
                    AND XRF.TC1 = 'STU'
                    AND XRF.FC1 = 'LF'
                    AND STU.LF = XRF.CD1
      WHERE STU.DEL = 0
          AND LAC.DEL = 0
          AND RTRIM(STU.CID) <> ''
          AND STU.TG IN ('I', '', ' '))
SELECT
    base.SC
   ,base.ID                           AS [Student ID]
   ,'Aeries ignores previous year reclassification date '
    + CONVERT(VARCHAR, base.RD1, 101) AS Description
FROM base
WHERE base.LF = 'RFEP'
    AND base.EAC IN ('EL')
    AND ISNULL(base.RD1, '1/1/9999') <= '7/1/' + @yr
    AND base.SC = @SchoolCode
    AND base.ID IN (@StudentID)  
  ```

  

## Validation Group:  ConRF

**Key Field 1:**  Student ID                     **Link to Page 1:**  EmergencyContacts.aspx  
**Key Field 2:**                       **Link to Page 2:**    
**Severity Level:**  E  
**Student Relate:** Yes         **User Can Refresh:** Yes  
  
**Possible Cause:**  Student has Parent Poral account that is linked to their ID, but the Contact record is red flagged indicating the contact should not have a parent portal account. A red flagged record is a visual reminder for a contact that is a problem or significant con  
  
**Possible Solution:**   1. Remove the red flag if the record is not for a problematic or dangerous contact.     2. If the contact is problematic or dangerous but has Educational Rights over the student, tag the contact as an Educational Rights holder.   3. If the contact is problematic or dangerous and shouldn't have a parent portal account, remove the parent portal account or contact the tech department to do it for you.  
  

**Query Definition:**
```sql
SELECT DISTINCT
    STU.ID                                                             AS [Student ID]
   ,CON.FN + ' ' + CON.LN + ' (' + PWA.EM
    + ') has a parent portal account, but this contact is red flagged' AS [Description]
FROM PWA
    INNER JOIN PWS
          ON  PWA.AID = PWS.AID
              AND PWS.DEL = 0
    INNER JOIN STU
          ON  PWS.ID = STU.ID
              AND STU.DEL = 0
              AND STU.TG IN ('', ' ')
    INNER JOIN CON
          ON  PWA.EM = CON.EM
              AND STU.ID = CON.PID
              AND CON.RF = 1
              AND CON.ERH <> 'Y'
              AND CON.DEL = 0
WHERE PWA.DEL = 0
    AND PWA.TY = 'P'
    AND (PWA.CDT IS NOT NULL
        OR PWA.LDT IS NOT NULL)
    AND PWA.XD IS NULL
    AND STU.SC = @SchoolCode
    AND STU.ID IN (@StudentID)  
 ``` 

  

## Validation Group:  UnlinkPP

**Key Field 1:**  Student ID                     **Link to Page 1:**  EmergencyContacts.aspx  
**Key Field 2:**                       **Link to Page 2:**    
**Severity Level:**  W  
**Student Relate:** Yes         **User Can Refresh:** Yes  
  
**Possible Cause:**  Student is linked to a Parent Poral account, but that account's email address is not on any Contact record for that student indicating that the parent portal account might not be a legitimate account.  
  
**Possible Solution:**   1. If it is a legitimate contact for the student add their email to the existing contact in the email field   2. If there is already a different email listed for that contact and they do not wish to update that email address, at the bottom of the contact record there are fields titled "Additional Contact 1, 2, 3, 4" where you can enter an additional email address.   2. Add a contact record with the listed email address if this is a new legitimate contact for the student   3. Contact the tech department to delete the parent portal account if that email should not be associated with the student record  
  

**Query Definition:**
```sql
SELECT DISTINCT
    STU.ID                                                                         AS [Student ID]
   ,PWA.EM + ' is linked to this student, but does not match an existing Contact.' AS [Description]
FROM PWA
    INNER JOIN PWS
          ON  PWA.AID = PWS.AID
              AND PWS.DEL = 0
    INNER JOIN STU
          ON  PWS.ID = STU.ID
              AND STU.DEL = 0
              AND STU.TG IN ('', ' ')
    LEFT JOIN CON
          ON  (PWA.EM = CON.EM
                  OR PWA.EM = TRIM(CON.CN1)
                  OR PWA.EM = TRIM(CON.CN2)
                  OR PWA.EM = TRIM(CON.CN3)
                  OR PWA.EM = TRIM(CON.CN4))
              AND STU.ID = CON.PID
              AND CON.DEL = 0
WHERE PWA.DEL = 0
    AND PWA.TY = 'P'
    AND (PWA.CDT IS NOT NULL
        OR PWA.LDT IS NOT NULL)
    AND PWA.XD IS NULL
    AND CON.PID IS NULL
    AND STU.PEM <> PWA.EM
    AND PWA.LPR > DATEADD(MONTH, -12, GETDATE())
    AND STU.SC = @SchoolCode
    AND STU.ID IN (@StudentID)  
```  

  

## Validation Group:  RFEPBlank

**Key Field 1:**  Student ID                     **Link to Page 1:**  Language.aspx  
**Key Field 2:**                       **Link to Page 2:**    
**Severity Level:**  W  
**Student Relate:** Yes         **User Can Refresh:** Yes  
  
**Possible Cause:**  Student's language fluency is set to Re-designated (RFEP) but student doesn't have re-designation date.  
  
**Possible Solution:**  Add redesignation date to student record or change language fluency to correct value.  
  

**Query Definition:**
```sql
SELECT
    STU.ID                        AS [Student ID]
   ,'Redesignation date is blank' AS [Description]
FROM STU
    INNER JOIN LAC
          ON  STU.ID = LAC.ID
              AND LAC.DEL = 0
WHERE STU.DEL = 0
    AND LAC.DEL = 0
    AND STU.LF = 'R'
    AND LAC.RD1 IS NULL
    AND RTRIM(STU.CID) <> ''
    AND STU.TG IN ('I', '', ' ')
    AND STU.SC = @SchoolCode
    AND STU.ID IN (@StudentID)  
  ```

  

## Validation Group:  AddrBlnk

**Key Field 1:**  Student ID                     **Link to Page 1:**  Students.aspx  
**Key Field 2:**                       **Link to Page 2:**    
**Severity Level:**  E  
**Student Relate:** Yes         **User Can Refresh:** Yes  
  
**Possible Cause:**  Parts of the student residence address are blank.  
  
**Possible Solution:**  Fill in a valid address for the student.  
  

**Query Definition:**
```sql
SELECT
    STU.ID                             AS [Student ID]
   ,'Student Address has blank fields' AS [Description]
FROM STU
WHERE STU.DEL = 0
    AND STU.TG IN ('I', '', ' ')
    AND '' IN (STU.AD, STU.CY, STU.ZC, STU.ST)
    AND STU.SC = @SchoolCode
    AND STU.ID IN (@StudentID)  
  ```

  

## Validation Group:  AdsCD

**Key Field 1:**  Student ID                     **Link to Page 1:**  AssertiveDiscipline.aspx  
**Key Field 2:**  Sequence Number                     **Link to Page 2:**    
**Severity Level:**  E  
**Student Relate:** Yes         **User Can Refresh:** Yes  
  
**Possible Cause:**  Incident must have the specific violation student made.  
  
**Possible Solution:**  Add violation for the incident or delete incident if it was incorrectly entered.  
  

**Query Definition:**
```sql
SELECT
    STU.SC                              [SchoolCode]
   ,STU.ID                              [Student ID]
   ,ADS.SQ                              [Sequence Number]
   ,ADS.SQ                              [SQ?]
   ,'Incident on ' + CONVERT(VARCHAR, ADS.DT, 101)
    + ' does not have a violation code' AS [Description]
FROM STU
    INNER JOIN ADS
          ON  ADS.PID = STU.ID
              AND ADS.DEL = 0
    LEFT JOIN DSP
          ON  ADS.PID = DSP.PID
              AND ADS.SQ = DSP.SQ
              AND DSP.DEL = 0
WHERE STU.DEL = 0
    AND STU.TG IN ('I', '', ' ')
    AND ADS.CD = ''
    AND ADS.DT >= DATEADD(MONTH, -10, GETDATE())
    AND STU.SC = @SchoolCode
    AND STU.ID IN (@StudentID)  
  ```

  

## Validation Group:  AdsDsp

**Key Field 1:**  Student ID                     **Link to Page 1:**  AssertiveDiscipline.aspx  
**Key Field 2:**  Sequence Number                     **Link to Page 2:**    
**Severity Level:**  W  
**Student Relate:** Yes         **User Can Refresh:** Yes  
  
**Possible Cause:**  Incident must have an associated disposition.  
  
**Possible Solution:**  Add a disposition for the incident or delete incident if it was incorrectly entered.  
  

**Query Definition:**
```sql
SELECT
    STU.SC                           [SchoolCode]
   ,STU.ID                           [Student ID]
   ,ADS.SQ                           [Sequence Number]
   ,ADS.SQ                           [SQ?]
   ,'Incident on ' + CONVERT(VARCHAR, ADS.DT, 101)
    + ' does not have a disposition' AS [Description]
FROM STU
    INNER JOIN ADS
          ON  ADS.PID = STU.ID
              AND ADS.DEL = 0
    LEFT JOIN DSP
          ON  ADS.PID = DSP.PID
              AND ADS.SQ = DSP.SQ
              AND DSP.DEL = 0
WHERE STU.DEL = 0
    AND STU.TG IN ('I', '', ' ')
    AND DSP.DS IS NULL
    AND ADS.DT >= DATEADD(MONTH, -10, GETDATE())
    AND STU.SC = @SchoolCode
    AND STU.ID IN (@StudentID)  
  ```

  

## Validation Group:  USSEnter

**Key Field 1:**  Student ID                     **Link to Page 1:**  Students.aspx  
**Key Field 2:**                       **Link to Page 2:**    
**Severity Level:**  E  
**Student Relate:** Yes         **User Can Refresh:** Yes  
  
**Possible Cause:**  The **US School K-12**   field must have the date that the student entered a K-12 grade level since they were born outside the United States.  
  
**Possible Solution:**  Enter the date that the student entered a K-12 grade at a US school.  
  

**Query Definition:**
```sql
SELECT
    STU.ID                                      [Student ID]
   ,'Student born in ' + ISNULL(COD.DE, STU.BCU)
    + ' but does not have US School Enter Date' [Description]
FROM STU
    LEFT JOIN LAC
          ON  STU.ID = LAC.ID
              AND LAC.DEL = 0
    LEFT JOIN COD
          ON  COD.TC = 'STU'
              AND COD.FC = 'BCU'
              AND COD.CD = STU.BCU
WHERE STU.DEL = 0
    AND STU.BCU NOT IN ('US', 'PR', 'UU', '')
    AND STU.TG IN ('', ' ')
    AND LAC.USS IS NULL
    AND STU.SC = @SchoolCode
    AND STU.ID IN (@StudentID)  
  ```

  

## Validation Group:  HomeDWUY

**Key Field 1:**  Student ID                     **Link to Page 1:**  Programs.aspx  
**Key Field 2:**                       **Link to Page 2:**    
**Severity Level:**  E  
**Student Relate:** Yes         **User Can Refresh:** Yes  
  
**Possible Cause:**  Student with an active Homeless record is missing Dwelling type and/or Unaccomp Youth indicator.  
  
**Possible Solution:**  Enter values for Dwelling type and Unaccomp Youth indicator.  
  

**Query Definition:**
```sql
SELECT
    STU.SC                                        [SchoolCode]
   ,STU.ID                                        [Student ID]
   ,'Missing dwelling type and/or Unaccomp Youth' AS [Description]
FROM STU
    INNER JOIN PGM
          ON  PGM.PID = STU.ID
              AND PGM.DEL = 0
              AND PGM.CD = '191'
              AND ISNULL(PGM.ESD, '1/1/9999') <= GETDATE()
              AND ISNULL(PGM.EED, '1/1/9999') >= GETDATE()
WHERE STU.DEL = 0
    AND '' IN (PGM.HDT, PGM.UY)
 ```

  

## Validation Group:  HomSIB

**Key Field 1:**  Student ID                     **Link to Page 1:**  Programs.aspx  
**Key Field 2:**                       **Link to Page 2:**    
**Severity Level:**  W  
**Student Relate:** Yes         **User Can Refresh:** Yes  
  
**Possible Cause:**  Listed student does not have an open homeless program record (PGM) but their family key (STU.FK) links them to their sibling who does have an open homeless program record. If a sibling qualifies as homeless so should the other siblings that are under the  
  
**Possible Solution:**   1. Add a homeless program record to the sibling missing the homeless program record.   2. OR If the students are incorrectly tied to the same family key while not being siblings, generate a new family key for the student with the incorrect family key.<br />  
  

**Query Definition:**
```sql
SELECT
    STU.SC                         AS [SchoolCode]
   ,STU.ID                         AS [Student ID]
   ,homeless.ID                    AS [Sibling ID]
   ,'Non-homeless student with homeless sibling ID: '
    + CAST(homeless.ID AS VARCHAR) AS [Description]
FROM STU
    INNER JOIN (SELECT DISTINCT
                  STU.ID
                 ,STU.FK
                 ,ROW_NUMBER() OVER (PARTITION BY STU.FK ORDER BY STU.ID) rower
              FROM STU
                  INNER JOIN PGM
                        ON  PGM.PID = STU.ID
                            AND PGM.DEL = 0
                            AND PGM.CD = '191'
                            AND ISNULL(PGM.ESD, '1/1/9999') <= GETDATE()
                            AND ISNULL(PGM.EED, '1/1/9999') >= GETDATE()
              WHERE STU.DEL = 0
                  AND STU.TG = ''
                  AND STU.FK <> 0) AS homeless
          ON  homeless.FK = STU.FK
              AND homeless.rower = 1
    LEFT JOIN PGM
          ON  PGM.PID = STU.ID
              AND PGM.DEL = 0
              AND PGM.CD = '191'
              AND ISNULL(PGM.ESD, '1/1/9999') <= GETDATE()
              AND ISNULL(PGM.EED, '1/1/9999') >= GETDATE()
WHERE STU.DEL = 0
    AND STU.TG = ''
    AND PGM.PID IS NULL
    AND STU.SC = @SchoolCode
    AND STU.ID IN (@StudentID)  
  ```

  

## Validation Group:  ValOld

**Key Field 1:**  SchoolCode                     **Link to Page 1:**  DataValidationResults.aspx  
**Key Field 2:**                       **Link to Page 2:**    
**Severity Level:**  W  
**Student Relate:**  false         **User Can Refresh:** Yes  
  
**Possible Cause:**  Data validations have not been fixed in over 30 days.  
  
**Possible Solution:**   1. If the specific data validation error is no longer relevant, disable it   2. If the data validation is relevant, contact the school site and offer help in fixing it.  
  

**Query Definition:**
```sql
SELECT
    DVR.SCL                              AS [SchoolCode]
   ,CAST(COUNT(*) AS VARCHAR)
    + ' old errors for [' + DVD.DE + ']' AS [Description]
FROM DVD
    INNER JOIN DVR
          ON  DVD.VID = DVR.VID
              AND DVR.DEL = 0
    INNER JOIN DVG
          ON  DVD.GID = DVG.GID
              AND DVG.DEL = 0
WHERE DVD.DS = 0 --not disabled
    AND DVD.LC > 0 --last run count
    AND DVR.DR IS NULL --date resolved
    AND DVR.AG > 30 --age of error
    AND GETDATE() < ISNULL(DVD.ED, '1/1/9999') --end date of validation
    AND DVR.SCL = @SchoolCode
GROUP BY DVD.DE
        ,DVR.SCL  
 ``` 

  

## Validation Group:  SusDays

**Key Field 1:**  Student ID                     **Link to Page 1:**  AssertiveDiscipline.aspx  
**Key Field 2:**                       **Link to Page 2:**    
**Severity Level:**  E  
**Student Relate:** Yes         **User Can Refresh:** Yes  
  
**Possible Cause:**  When a suspension disposition is entered the number of days the suspension lasted needs to be filled in as well.  
  
**Possible Solution:**  Enter the number of days the student was suspended in the **Days**   filed on the disposition record.  
  

**Query Definition:**
```sql
SELECT
    STU.SC                                        [SchoolCode]
   ,STU.ID                                        [Student ID]
   ,ADS.SQ                                        [Sequence Number]
   ,ADS.SQ                                        [SQ?]
   ,'Suspension on ' + CONVERT(VARCHAR, ADS.DT, 101)
    + ' needs to have Disposition Days filled in' AS [Description]
FROM STU
    INNER JOIN ADS
          ON  ADS.PID = STU.ID
              AND ADS.DEL = 0
    INNER JOIN DSP
          ON  ADS.PID = DSP.PID
              AND ADS.SQ = DSP.SQ
              AND DSP.DEL = 0
WHERE STU.DEL = 0
    AND STU.TG IN ('I', '', ' ')
    AND DSP.DY = 0
    AND DSP.DS IN ('SUS', 'IHS', 'TSUS')
    AND ADS.DT >= DATEADD(MONTH, -10, GETDATE())
    AND STU.SC = @SchoolCode
    AND STU.ID IN (@StudentID)  
```  

  

## Validation Group:  HisCarSE

**Key Field 1:**  Student ID                     **Link to Page 1:**  StudentTranscripts.aspx  
**Key Field 2:**                       **Link to Page 2:**  CourseAttendance.aspx  
**Severity Level:**  E  
**Student Relate:** Yes         **User Can Refresh:** Yes  
  
**Possible Cause:**  The student transcript for the student in the current year has a course and section listed that do not match any course and section that the student took this year. This is most likely caused by a course being renamed and reused after students were transferred.
  
**Possible Solution:**  For every student and course listed, edit the transcript record so that the section listed for that course on the transcript matches the section that the student actually took.  
  

**Query Definition:**
```sql
DECLARE @yr AS VARCHAR(2) = SUBSTRING(DB_NAME(), 4, 2)

SELECT DISTINCT
    STU.ID                    AS [Student ID]
   ,STU.SC                    AS [SchoolCode]
   ,'HIS Course: ' + HIS.CN
    + ' & Section: ' + CAST(HIS.SE AS VARCHAR)
    + '. Closest match CAR Section: '
    + CAST(CAR.SE AS VARCHAR) AS [Description]
FROM HIS
    INNER JOIN MST
          ON  HIS.SE = MST.SE
              AND MST.SC = HIS.ST
              AND MST.DEL = 0
    INNER JOIN STU
          ON  HIS.PID = STU.ID
              AND HIS.ST = STU.SC
              AND STU.DEL = 0
    INNER JOIN CAR
          ON  STU.SC = CAR.SC
              AND STU.SN = CAR.SN
              AND CAR.DEL = 0
              AND CAR.CN = HIS.CN
              AND HIS.SE <> CAR.SE
    LEFT JOIN CAR AS car2
          ON  STU.SC = car2.SC
              AND STU.SN = car2.SN
              AND car2.DEL = 0
              AND car2.CN = HIS.CN
              AND HIS.SE = car2.SE
WHERE HIS.DEL = 0
    AND HIS.YR = @yr
    AND HIS.CN <> MST.CN
    AND car2.SE IS NULL
    AND STU.SC = @SchoolCode
    AND STU.ID IN (@StudentID)  
  ```

  

## Validation Group:  STUFK

**Key Field 1:**  Student ID                     **Link to Page 1:**  Students.aspx  
**Key Field 2:**                       **Link to Page 2:**    
**Severity Level:**  W  
**Student Relate:** Yes         **User Can Refresh:** Yes  
  
**Possible Cause:**  The student's family key is 0 so there might be siblings in the district that are not linked to them.  
  
**Possible Solution:**  Use the **Sibling Lookup**   functionality to search for any siblings that might also be enrolled in the district. If no other siblings are enrolled, create a new **Family Key**   for the student.  
  

**Query Definition:**
```sql
SELECT
    STU.SC            [SchoolCode]
   ,STU.ID            [Student ID]
   ,'Family Key is 0' [Description]
FROM STU
WHERE STU.DEL = 0
    AND STU.TG NOT IN ('N')
    AND STU.TG = ''
    AND STU.FK = 0
    AND STU.SC = @SchoolCode
    AND STU.ID IN (@StudentID)  
```

## Validation Group:  AdsDT01

**Key Field 1:**  Student ID                     **Link to Page 1:**  AssertiveDiscipline.aspx  
**Key Field 2:**  Incident ID                     **Link to Page 2:**  DisciplineIncidents.aspx  
**Severity Level:**  E  
**Student Relate:** Yes         **User Can Refresh:** Yes  
  
**Possible Cause:**  Multiple students are linked to the same incident, but not all students have the same date for the same Incident ID.  
  
**Possible Solution:**  Go to the **Discipline Incidents**   page (link to the page is below) and search for the **Incident ID**   listed in the error (uncheck the date checkbox). Determine the correct date for the incident and edit all the incorrect incidents to have the correct date.  

**Query Definition:**
```sql
SELECT
    STU.SC                                 AS [SchoolCode]
   ,STU.ID                                 AS [Student ID]
   ,ADS.IID                                AS [Incident ID]
   ,ADS.SQ                                 AS [SQ?]
   ,'"Other Violators" have the date as ('
    + CONVERT(VARCHAR, ads2.DT, 101) + ')' AS [Description]
FROM STU
    INNER JOIN ADS
          ON  ADS.PID = STU.ID
              AND ADS.DEL = 0
    INNER JOIN (SELECT DISTINCT
                  ADS.IID
                 ,ADS.DT
              FROM ADS
              WHERE ADS.DEL = 0
                  AND ADS.IID <> 0) AS ads2
          ON  ADS.DT <> ads2.DT
              AND ADS.IID = ads2.IID
WHERE STU.DEL = 0
    AND STU.TG IN ('I', '', ' ')
    AND ADS.DT >= DATEADD(MONTH, -10, GETDATE())
    AND STU.SC = @SchoolCode
    AND STU.ID IN (@StudentID)
ORDER BY ADS.IID, STU.SC, ADS.DT
```
