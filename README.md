# School Ventilation and COVID

This GitHub repo contains data and methods for our 2021 story, [TKTK](<>). In the story, we used publicly available data on ventilation and COVID cases in New York City schools to examine whether case rates were higher in buildings that rely on windows for fresh air (as opposed to HVAC). Here, we'll show you how we did it.

## Methodology

The analysis consisted of the following steps:
1\. We gathered the ventilation reports for Department of Education buildings in NYC.
2\. We gathered information on those buildings (and the schools within them) from NYC Open Data.
3\. For each of these schools, we collected COVID case and enrollment data from the NYS Department of Health.
4\. We identified school buildings that relied mostly on windows for ventilation, as well as buildings in the same areas of the city that also have HVAC. (Experts [told](https://gothamist.com/news/nyc-officials-say-school-windows-can-always-offer-solid-ventilation-independent-scientists-disagree) WNYC/Gothamist that windows are not as reliable a source of fresh air as properly functioning HVAC.)
5\. We compared COVID case rates between the two groups.

The sections below go into more detail about the data collection and analysis.

### Ventilation data and school data

I got a list of DOE building codes in each borough from the following lists:

-   [Bronx](https://www.schools.nyc.gov/about-us/reports/lead-based-paint/bronx)
-   [Brooklyn](https://www.schools.nyc.gov/about-us/reports/lead-based-paint/brooklyn)
-   [Manhattan](https://www.schools.nyc.gov/about-us/reports/lead-based-paint/manhattan)
-   [Queens](https://www.schools.nyc.gov/about-us/reports/lead-based-paint/queens)
-   [Staten Island](https://www.schools.nyc.gov/about-us/reports/lead-based-paint/staten-island)

Then, for each building code in this combined list, I downloaded the corresponding ventilation assessment report. (Example [here](https://www.nycenet.edu/roomassessment?code=R080).) About 1,280 of the buildings on the list had ventilation assessment reports available. I put all the ventilation assessment reports into one dataset and filtered it to only include student classrooms. (Some of the buildings lack student classrooms, so this brought us to about 1,250 buildings.) For each building, I tallied up the number of classrooms marked as “Operational,” “Repair in Progress” and “Does Not Exist.” (Definitions of these terms can be found [here](https://www.schools.nyc.gov/about-us/reports/building-ventilation-status).)

On October 29th, 2021, we found 58,464 classrooms across 1,253 buildings. 98% were “Operational,” 1.9% were “Repair in Progress” and .01% were “Does Not Exist.”

To identify classrooms that rely on windows for ventilation, I filtered the dataset by the following column values:
RoomStatus: Operational or Repair in Progress (based on feedback from DOE that some classrooms marked as “repair in progress” can still be used for instruction, we included them in our analysis)
Windows: Yes
SupplyFan: Doesn’t Exist or Not Operational
ExhaustFan: Doesn’t Exist or Not Operational
UnitVentilator: Doesn’t Exist or Not Operational

I created a new column, windowsOnly, and marked it “yes” for classrooms that met all the above criteria. Then, for each school, I tallied up the number of classrooms that had a “yes” in the windowsOnly column.

Overall, we found 4,857 classrooms marked “Operational” or “Repair in Progress” that relied only on windows for ventilation—about 8% of student classrooms with these statuses.

If 90% or more of a school’s classrooms relied only on windows for ventilation, we classified the school as “low ventilation.” If 10% or less of a school’s classrooms relied only on windows for ventilation (meaning 90% or more have access to supply fans, exhaust fans or HVAC), we classified the school as “other ventilation.”

To fill in other information about the schools, I downloaded the [2019 - 2020 School Locations](https://data.cityofnewyork.us/Education/2019-2020-School-Locations/wg9x-4ke6) dataset, the most recent listing of NYC school locations available on NYC Open Data. I filtered the school locations dataset to include only open schools, then joined it with the ventilation dataset on the column Primary_building_code.

### Case data

For each Basic Education Data System (BEDS) code corresponding to an open school in the 2019 - 2020 School Locations dataset, I downloaded its [New York State COVID-19 Report Card](https://schoolcovidreportcard.health.ny.gov/#/home) as of October 29th, 2021. I pulled the following fields from the underlying JSON:
studentEnrolled
teacherEnrolled (only charter schools distinguish between teachers and staff so this is 0 for district schools)
staffEnrolled
allTimeCounts.positiveStudents
allTimeCounts.positiveTeachers (see note about teachers above)
allTimeCounts.positiveStaff
allTimeCounts.positiveTotal
I added these counts to the school locations dataset. Then, in the school locations dataset, I summed student and staff/teacher cases as well as enrollment by Primary_building_code to get each building’s total case and enrollment counts. I joined these totals with the ventilation dataset by building code.

We filtered out school buildings that weren’t in the 2019 - 2020 school locations list and schools that didn’t have case data available (about 60 of each).

### Zip code analysis

Grouping[^1]: The “other ventilation” school buildings were filtered based on whether their zip codes matched those found in the “low ventilation” group. Doing so provided 61 school buildings in the low ventilation group and 296 school buildings in the other ventilation group spread across 39 unique zip codes.

Case rates: Through October 29th, the low-ventilation buildings recorded 23% more COVID-19 cases per student and 29% more cases per staff than other-ventilation buildings. Our analysis shows the rates as cases per 10,000 students or cases per 10,000 staff.

Building age: The low-ventilation school buildings have an average opening date (1936) that’s about 24 years older than the other-ventilation buildings in the same zip code. But 19 of the low-ventilation schools are relatively young, having opened since 1990.

Classroom Stats: The low-ventilation buildings had fewer total classrooms on average than other-ventilation buildings in the same zip code (32 versus 50), a lower average student enrollment (464 versus 808), and a lower average staff size (76 versus 116). But low-ventilation schools had a similar number of students per classroom (17 versus 16) and staff per classroom (3 versus 2) as other-ventilation, suggesting that population density within buildings did not play a role.   



## Files included

### allData.csv

This is a list of all the DOE buildings for which ventilation data was available. It excludes buildings that don't have classrooms in them. It does include buildings that lack case data and have no corresponding entries in the 2019 - 2020 School Locations list, so it will need some cleaning and filtering before it's ready to use.

### windowsOnlyBuildings.csv

This is a subset of allData.csv including all the low-ventilation buildings — in other words, all the buildings that meet the following criteria:

-   School and case data are available for the building.
-   90% or more of the classrooms in the building rely only on windows for built-in ventilation.

### otherVentilationSameZipBuildings.csv

This is a subset of allData.csv including all the high-ventilation buildings in the same zipcodes as low-ventilation buildings.

## Data dictionary

All the files listed above have the same columns:

-   **Row Labels**: Building code, used to find ventilation reports.
-   **WindowsOnly Number**: Number of "Operational" or "Repair in Progress" classrooms relying only on windows for ventilation.
-   **WindowsOnly Percent**: Proportion of "Operational" or "Repair in Progress" classrooms relying only on windows for ventilation.
-   **OtherVentilation Number**: Number of "Operational" or "Repair in Progress" classrooms that have working supply fans, exhaust fans or unit ventilators.
-   **OtherVentilation Percent**: Proportion of "Operational" or "Repair in Progress" classrooms that have working supply fans, exhaust fans or unit ventilators.
-   **TotalClassrooms (not inc DNE)**: Total number of "Operational" or "Repair in Progress" classrooms.
-   **Does Not Exist Number**: Number of "Does Not Exist" classrooms (not used for instruction).
-   **Does Not Exist Percent** Proportion of "Does Not Exist" classrooms (not used for instruction).
-   **Operational Number**
-   **Operational Percent**
-   **Repair In Progress Number**
-   **Repair in Progress Percent**
-   **Does Not Exist or Repair in Progress Number**: Sum of "Does Not Exist Number" and "Repair in Progress Number."
-   **Does Not Exist or Repair in Progress Percent**
-   **Total Number Classrooms**: Total classrooms, including "Does Not Exist."
-   **Address**: Building address from 2019-2020 School Locations list.
-   **Latitude**: Latitude from 2019-2020 School Locations list.
-   **Longitude**: Longitude from 2019-2020 School Locations list.
-   **OpenCage Address**: Another address option, from geocoder OpenCage.
-   **OpenCage Zip Code**: Another zip code option, from geocoder OpenCage.
-   **OpeningDate**: Opening date of the oldest school in each building (may not be the same as construction date) from 2019-2020 School Locations.
-   **SchoolType**: Grade range of one school in the building (not including co-located schools), from 2019-2020 School Locations.
-   **NTANumber**: Neighborhood Tabulation Area of the building, from School Locations
-   **NTAName**: Ibid.
-   **JoinedSchoolListLocList**: Comma-separated list of all school names associated with the building in 2019-2020 School Locations.
-   **Census_tract**: Census tract of the building, from 2019-2020 School Locations.
-   **numSchools**: Number of schools associated with the building in 2019-2020 School Locations.
-   **studentEnrollment**: Estimated student enrollment in the building, from the New York State COVID-19 Report Cards of the schools in the building.
-   **studentCases**: Cumulative number of student cases from the 2021-2022 school year, from the New York State COVID-19 Report Cards of the schools in the building.
-   **staffAndTeacherEnrollment**: Estimated staff and teacher enrollment in the building, from the New York State COVID-19 Report Cards of the schools in the building. [^2]
-   **staffAndTeacherCases**: Cumulative number of staff and teacher cases from the 2021-2022 school year, from the New York State COVID-19 Report Cards of the schools in the building.
-   **totalCases**: Sum of student and staff/teacher cases.
-   **totalSchoolPop**: Sum of student and staff/teacher enrollment.
-   **studentsPerClassroom**: studentEnrollment divided by TotalClassrooms (not inc DNE).
-   **staffPerClassroom**: staffAndTeacherEnrollment divided by TotalClassrooms (not inc DNE).
-   **totalSchoolPopPerClassroom**: Sum of studentEnrollment and staffAndTeacherEnrollment divided by TotalClassrooms (not inc DNE).
-   **casePerStudent**: studentCases divided by studentEnrollment.
-   **casesPerStaff**: staffAndTeacherCases divided by staffAndTeacherEnrollment.
-   **totalCasesPerPop**: totalCases divided by totalSchoolPop.
-   **caseRatePer10,000Student**: casePerStudent multiplied by 10K.
-   **caseRatePer10,000Staff**: casesPerStaff multiplied by 10K.
-   **totalCaseRatePer10,000Pop**: totalCasesPerPop multiplied by 10K.

[^1]: We found similar trends when the school buildings were grouped by neighborhood designation and census tract.
[^2]: Charter schools distinguish teachers and staff in the Report Card, while district schools lump them together.

## Questions? Comments? Concerns?

If you want to replicate this analysis or do something similar and need advice or more details, [drop us a line](jjeffrey-wilensky@nypublicradio.org).
