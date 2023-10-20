# doctor-scheduling
Code related to the paper "Human-centered algorithmic physician scheduling in hospitals"

About the study
The study intends to detect potential for improvement in scheduling doctors in hospitals.
For this purpose, an algorithm is developed utilizing Constraint Programming, to schedule doctors in the
surgical department and Munich's hospital rechts der Isar (MRI).
The primary goal of the algorithm is to maximize the fulfilment of doctor preferences, i.e. preferences
regarding off-days and specific jobs on specific shifts.

Functionality of the code
The steps necessary to assign doctors in the surgical department are described in detail in the source
code via comments. For any related questions feel free to contact Natalie Wagner (natalie.wagner@tum.de)
or Charlotte Haid (charlotte.haid@tum.de).
The code consists of eight steps that are as follows:

1. Input data definition
2. Model creation via cp_model.CpModel()
3. Decision variable creation via model.NewBoolVar()
4. General constraints generation via model.Add()
5. Specific constraints generation via model.Add()
6. Objective formulation and maximization via model.Maximize()
7. Problem solver application via solver = cp_model.CpSolver() and solver.Solve(model)
8. Solution (output) printing

To schedule doctors of the surgical department at MRI for any chosen month, the following steps are
necessary:

1. Define minimum and maximum number of shifts per doctor or assign priorities to each doctor by defining
them in the employee_priorities, whereby  higher priorities indicate doctors to be assigned to
more shifts than others.
Scale the following variables, arrays, and matrices to the number of doctors to be scheduled:

number_employees
_max_shifts_per_employee and min_shifts_per_employee _
employee_availability_matrix_wholedays
qualification_matrix
employee_offdays_preference_matrix


2. Scale the following variables, arrays, and matrices to the number of days of the considered month:

number_weekdays
number_weekendholidays
day_shift_matrix
employee_availability_matrix_wholedays
employee_offdays_preference_matrix


3. Define the entries of the day-shift-matrix acccording to the types of days of the considered month,
i.e. the distribution of weekdays, weekends, and public holidays within the month.
Enter the data (Note: to input data more easily, especially into big matrices, it is suggested to add
extra lines of comment in the code, containing indices of days and doctors).

- minimum and maximum number of shifts per doctor into max_shifts_per_employee and
min_shifts_per_employee

- doctor availabilities into employee_availability_matrix_wholedays

- doctor qualifications into employee_qualification_matrix

- doctor off-day preferences into employee_offdays_preference_matrix

- doctor job preferences into employee_jobs_preference_matrix




The output of the code is as follows:

1. Employee-job-calculation matrix and job restrictions
2. Overall schedule
3. Number of each shift per employee
4. Number of working hours per employee
5. Total off-day preference score (number of total fulfilled preferences for off-days = first part of the
objective)
6. Individual off-day preference score per employee
7. Maximum individual off-day preference per employee (= number of submitted off-day preferences)
8. Individual and total percentage of fulfilled off-day preferences
9. Total job preference score (number of total fulfilled preferences for jobs = second part of the
objective)
10. Individual job preference score per employee
11. Maximum individual job preference score per employee (= number of submitted job preferences)
12. Individual and total percentage of fulfilled job preferences
13. Individual schedule of each employee
14. Overview of total number and percentage of fulfilled off-day and job preferences
