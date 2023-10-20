# required imports
from ortools.sat.python import cp_model

import array as arr
import numpy as np
import time


def main():
    # Start timing
    time_start = time.time()

    # 0.) Define input:
    # 0.1) Define number of employees / jobs / qualifications / days / shifts per day
    # !!ATTENTION: When adjusting the numbers, also adjust the corresponding parts of the code (see following comments)
    number_employees: int = 36
    number_jobs: int = 3

    number_weekdays: int = 21
    number_weekendholidays: int = 9
    number_days = number_weekdays + number_weekendholidays

    number_shifts_per_day: int = 5
    number_total_shifts = number_days * number_shifts_per_day

    shift_names = ['day-shift',                         # shift 1
                   'standby-shift',                     # shift 2
                   'on-call-shift',                     # shift 3
                   'standby-shift (weekend/holiday)',   # shift 4
                   'on-call-shift (weekend/holiday)']   # shift 5
    job_names = ['FA', 'A1', 'A2']
    skill_level_names = ['FA', 'A1', 'A2']

    # 0.2) Define individual working hours and priorities per employee
    # For each employee, individual minimum and maximum workinghours are assigned, 'i' creates an array of integers.
    # Additionally, priorities are assigned to each employee, where 2 indicates high priority and 1 indicates low
    # priority, so that those employees with higher priority are assigned to jobs first
    # !!ATTENTION: Max./min. workinghours per employee need to be adjusted to the time period considered for scheduling
    # !!ATTENTION: Array lengths need to be adjusted to number of employees
    # Otherwise max workinghours might not be sufficient to cover the required total number of shifts
    max_shifts_per_employee = arr.array('i', [4, 0, 4, 4, 4, 2, 4, 2, 4, 2,
                                              4, 0, 4, 0, 4, 0, 0, 0, 4, 4,
                                              4, 4, 4, 4, 4, 4, 4, 4, 4, 4,
                                              4, 4, 4, 4, 4, 4])
    min_shifts_per_employee = arr.array('i', [0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
                                              0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
                                              0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
                                              0, 0, 0, 0, 0, 0])
    employee_priorities = arr.array('i', [0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
                                          0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
                                          0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
                                          0, 0, 0, 0, 0, 0])

    # 0.3) Definition of Day-Shift-Matrix: Specific shifts to be scheduled on each day
    # Each row = one shift, each column = one day
    # 1 indicates that the shift has to be scheduled, 0 indicates the shift does not have to be scheduled
    # !!ATTENTION: Matrix needs to be adjusted to days (columns) and number of shifts per day (rows)
    # Day number:
    #     1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30
    day_shift_matrix = np.array(
        [[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],  # shift 1
         [1, 1, 0, 0, 1, 1, 1, 0, 1, 0, 0, 1, 1, 1, 1, 1, 0, 0, 1, 1, 1, 1, 1, 0, 0, 1, 1, 1, 1, 1],  # shift 2
         [1, 1, 0, 0, 1, 1, 1, 0, 1, 0, 0, 1, 1, 1, 1, 1, 0, 0, 1, 1, 1, 1, 1, 0, 0, 1, 1, 1, 1, 1],  # shift 3
         [0, 0, 1, 1, 0, 0, 0, 1, 0, 1, 1, 0, 0, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0],  # shift 4
         [0, 0, 1, 1, 0, 0, 0, 1, 0, 1, 1, 0, 0, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0]   # shift 5
         ])
    # Array generation from day_shift_matrix: this generates an array that indicates which shifts need to be scheduled.
    # The array is of length days x shifts and for each day, the five shifts are indicated
    # (e.g. elements 1 - 5 correspond to shifts on day 1, element 6 - 10 correspond to the shifts on day 2)
    # ‘F’ means to flatten in column-major order (columns = days)
    # 1 indicates that the shift has to be scheduled, 0 indicates the shift does not have to be scheduled
    day_shift_array = day_shift_matrix.flatten(order='F')

    # 0.4) Definition of Shift-Job-Matrix: Jobs only need to be scheduled for certain shifts
    # Each row = one job, each column = one shift per day
    # 1 indicates that the job has to be scheduled, 0 indicates the job does not have to be scheduled
    # !!ATTENTION: Matrix needs to be adjusted to jobs (rows) and number of shifts per day (columns)
    shift_job_matrix = np.array([[1, 0, 1, 0, 1],  # job 1
                                 [1, 1, 0, 1, 0],  # job 2
                                 [1, 1, 0, 1, 0]   # job 3
                                 ])
    # Matrix extension: this generates a new matrix that repeats the previously defined shift_job_matrix
    # by the number of days
    day_shift_job_matrix = np.tile(shift_job_matrix, number_days)

    # 0.5) Definition of the working hours per shift
    # The working hours for each shift are saved as an array, where each column = one shift (of one day)
    # The array is then repeated n-times (n = number of days) and stored as a new array
    # !!ATTENTION: Array needs to be adjusted to number of shifts per day
    workinghours_shifts_1day = arr.array('i', [8, 16, 4, 24, 6])
    # Array extension: workinghours_shifts is an array that contains the workinghours of each shift to be scheduled,
    # number of elements of this array: number_jobs x number_shifts x number_days
    workinghours_shifts = np.tile(np.repeat(workinghours_shifts_1day, number_jobs), number_days)

    # 0.6) Definition of Availability matrix: availability of each employee for each day
    # Each row = one employee, each column = one day
    # 1 represents an available employee, 0 represents an absent employee who thus can not be assigned to a job
    # !!ATTENTION: Matrix needs to be adjusted to days (= number of columns)
    # !!ATTENTION: Matrix needs to be adjusted to number of employees (= number of rows)
    # Day number:
    #     1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30
    employee_availability_matrix_wholedays = np.array(
        [[1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 1
         [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],  # employee 2
         [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 1],  # employee 3
         [0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0],  # employee 4
         [1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 5
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 6
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],  # employee 7
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 8
         [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 9
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 10
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 11
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 12
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 13
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 14
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 15
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 16
         [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],  # employee 17
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 18
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 19
         [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 20
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 21
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 22
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 1, 1],  # employee 23
         [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 0, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 0, 1, 1],  # employee 24
         [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 25
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 26
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0],  # employee 27
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 28
         [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 29
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 30
         [1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 0, 0, 1, 1, 1, 1, 1, 1, 1],  # employee 31
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0],  # employee 32
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 33
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 34
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 35
         [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 0, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1]   # employee 36
         ])
    #     1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30

    # 0.7) Definition of employee qualification matrix
    # Each row = one employee, each column = one qualification assessed on a scale from 1 to 3
    # 1 represents high skill, 3 represents low skill
    # !!ATTENTION: Matrix needs to be adjusted to number of employees (= number of rows)
    # !!ATTENTION: Matrix needs to be adjusted to number of qualifications (= number of columns)
    employee_qualification_matrix = np.array([[3],  # employee 1
                                              [2],  # employee 2
                                              [1],  # employee 3
                                              [1],  # employee 4
                                              [2],  # employee 5
                                              [3],  # employee 6
                                              [2],  # employee 7
                                              [3],  # employee 8
                                              [1],  # employee 9
                                              [2],  # employee 10
                                              [1],  # employee 11
                                              [3],  # employee 12
                                              [2],  # employee 13
                                              [3],  # employee 14
                                              [1],  # employee 15
                                              [3],  # employee 16
                                              [2],  # employee 17
                                              [3],  # employee 18
                                              [3],  # employee 19
                                              [1],  # employee 20
                                              [3],  # employee 21
                                              [1],  # employee 22
                                              [2],  # employee 23
                                              [3],  # employee 24
                                              [1],  # employee 25
                                              [3],  # employee 26
                                              [1],  # employee 27
                                              [3],  # employee 28
                                              [1],  # employee 29
                                              [2],  # employee 30
                                              [1],  # employee 31
                                              [1],  # employee 32
                                              [1],  # employee 33
                                              [2],  # employee 34
                                              [1],  # employee 35
                                              [3]   # employee 36
                                              ])

    # 0.8) Definition of Job Requirements Matrix
    # Each row = one job, each column = one qualification assessed on a scale from 1 to 3
    # !!ATTENTION: Matrix needs to be adjusted to number of jobs (rows) and number of qualifications (columns)
    job_required_qualification_matrix = np.array([[1],  # job 1
                                                  [2],  # job 2
                                                  [3]   # job 3
                                                  ])

    # 0.9) Definition (Initialization) of Employee-Job Matrix
    # A matrix of the size number employees x number of jobs is generated.
    # Later on the matrix is used to calculate which job can be done by which employee based on their qualifications
    # and the required job qualifications --> see step 4.3
    employee_job_calculation_matrix = np.zeros((number_employees, number_jobs), dtype=int)

    # 0.10) Definition of the Employee_Offdays_Preference_Matrix
    # Each row = one employee, each column = one day
    # 1 indicates that the employee has no preference regarding that day,
    # 0 indicates that the employee would like to have the day off
    # !!ATTENTION: Matrix needs to be adjusted to days (= number of columns)
    # !!ATTENTION: Matrix needs to be adjusted to number of employees (= number of rows)
    # Day number:
    #     1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30
    employee_offdays_preference_matrix = np.array(
        [[1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 1
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 2
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 1, 1, 0, 0, 0, 0, 1, 1, 1, 1, 0, 1, 0],  # employee 3
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 4
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 5
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 6
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 7
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 8
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 9
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 10
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 1, 1, 1, 1, 1],  # employee 11
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 12
         [1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 0, 0, 0, 0, 1, 1, 1, 1],  # employee 13
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 14
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 15
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 16
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 17
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 18
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 19
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 1, 1, 1, 1, 1],  # employee 20
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 21
         [1, 1, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1, 0, 0, 0, 1, 1, 1, 1, 0],  # employee 22
         [1, 1, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0],  # employee 23
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 24
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 25
         [1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 26
         [1, 1, 1, 0, 1, 0, 1, 1, 1, 1, 0, 1, 0, 1, 1, 1, 1, 0, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 27
         [1, 1, 0, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 0],  # employee 28
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 29
         [1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 30
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 1, 1, 0, 1, 1, 1, 1, 1, 0],  # employee 31
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 32
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 33
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 34
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],  # employee 35
         [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1]   # employee 36
         ])
    #     1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30

    # The employee_offdays_preference_matrix is then being inverted, which means 1 is replaced by 0, 0 is replaced by 1.
    # This step is necessary to calculate preference scores and to be able to define specific constraints.
    # Note that numpy.invert inverts  bit-wise, which is why 2 is then added to each element
    employee_offdays_preference_matrix_inverted = np.add(np.invert(employee_offdays_preference_matrix), 2)

    # 0.11) Initialization of the Employee_Jobs_Preference_matrix and definition of job preferences
    # Each row = one employee, each column = one job
    employee_jobs_preference_matrix = np.zeros((number_employees, number_total_shifts, number_jobs), dtype=int)

    # The following part of the code transfers the preferred jobs of each employee into the matrix, so that
    # 1 indicates that the employee would like to work on that specific job during the specific shift and day
    # 0 indicates that the employee does not have any preference regarding the job
    # The indices of the employee, day, shift of the day, and job are defined manually and the corresponding element
    # of the preference matrix is set to 1 automatically.
    # !!ATTENTION: For each preference, the code has to be repeated
    # !!ATTENTION: Pay attention to the ranges of the numbers to be filled in (see comment behind the variables)
    # !!ATTENTION: For the correct number to be filled in refer to the lists 'shift_names' and 'job_names'
    # !!ATTENTION: Pay attention to the correlation of job_number and shift_number, as defined by the shift_job_matrix
    # !!ATTENTION: Pay attention to the correlation of day_number and shift_number, as defined by the day_shift_matrix

    # Employee Number 5
    jobpref_employee_number = 5  # ranging from 1 to number_employees
    jobpref_day_number = 6  # ranging from 1 to number_days
    jobpref_shift_number = 2  # ranging from 1 to number_shifts_per_day
    jobpref_job_number = 2  # ranging from 1 to number_jobs (1 = A2, 2 = A1, 3 = FA)
    employee_jobs_preference_matrix[(jobpref_employee_number - 1),
                                    (jobpref_day_number - 1) * number_shifts_per_day + (jobpref_shift_number - 1),
                                    (jobpref_job_number - 1)] = 1

    jobpref_employee_number = 5  # ranging from 1 to number_employees
    jobpref_day_number = 18  # ranging from 1 to number_days
    jobpref_shift_number = 4  # ranging from 1 to number_shifts_per_day
    jobpref_job_number = 2  # ranging from 1 to number_jobs (1 = A2, 2 = A1, 3 = FA)
    employee_jobs_preference_matrix[(jobpref_employee_number - 1),
                                    (jobpref_day_number - 1) * number_shifts_per_day + (jobpref_shift_number - 1),
                                    (jobpref_job_number - 1)] = 1

    jobpref_employee_number = 5  # ranging from 1 to number_employees
    jobpref_day_number = 25  # ranging from 1 to number_days
    jobpref_shift_number = 4  # ranging from 1 to number_shifts_per_day
    jobpref_job_number = 2  # ranging from 1 to number_jobs (1 = A2, 2 = A1, 3 = FA)
    employee_jobs_preference_matrix[(jobpref_employee_number - 1),
                                    (jobpref_day_number - 1) * number_shifts_per_day + (jobpref_shift_number - 1),
                                    (jobpref_job_number - 1)] = 1

    jobpref_employee_number = 5  # ranging from 1 to number_employees
    jobpref_day_number = 29  # ranging from 1 to number_days
    jobpref_shift_number = 2  # ranging from 1 to number_shifts_per_day
    jobpref_job_number = 2  # ranging from 1 to number_jobs (1 = A2, 2 = A1, 3 = FA)
    employee_jobs_preference_matrix[(jobpref_employee_number - 1),
                                    (jobpref_day_number - 1) * number_shifts_per_day + (jobpref_shift_number - 1),
                                    (jobpref_job_number - 1)] = 1

    # Employee Number 7
    jobpref_employee_number = 7  # ranging from 1 to number_employees
    jobpref_day_number = 15  # ranging from 1 to number_days
    jobpref_shift_number = 2  # ranging from 1 to number_shifts_per_day
    jobpref_job_number = 2  # ranging from 1 to number_jobs (1 = A2, 2 = A1, 3 = FA)
    employee_jobs_preference_matrix[(jobpref_employee_number - 1),
                                    (jobpref_day_number - 1) * number_shifts_per_day + (jobpref_shift_number - 1),
                                    (jobpref_job_number - 1)] = 1

    # Employee Number 9
    jobpref_employee_number = 9  # ranging from 1 to number_employees
    jobpref_day_number = 17  # ranging from 1 to number_days
    jobpref_shift_number = 5  # ranging from 1 to number_shifts_per_day
    jobpref_job_number = 1  # ranging from 1 to number_jobs (1 = A2, 2 = A1, 3 = FA)
    employee_jobs_preference_matrix[(jobpref_employee_number - 1),
                                    (jobpref_day_number - 1) * number_shifts_per_day + (jobpref_shift_number - 1),
                                    (jobpref_job_number - 1)] = 1

    # Employee Number 13
    jobpref_employee_number = 13  # ranging from 1 to number_employees
    jobpref_day_number = 2  # ranging from 1 to number_days
    jobpref_shift_number = 2  # ranging from 1 to number_shifts_per_day
    jobpref_job_number = 2  # ranging from 1 to number_jobs (1 = A2, 2 = A1, 3 = FA)
    employee_jobs_preference_matrix[(jobpref_employee_number - 1),
                                    (jobpref_day_number - 1) * number_shifts_per_day + (jobpref_shift_number - 1),
                                    (jobpref_job_number - 1)] = 1

    jobpref_employee_number = 13  # ranging from 1 to number_employees
    jobpref_day_number = 18  # ranging from 1 to number_days
    jobpref_shift_number = 4  # ranging from 1 to number_shifts_per_day
    jobpref_job_number = 2  # ranging from 1 to number_jobs (1 = A2, 2 = A1, 3 = FA)
    employee_jobs_preference_matrix[(jobpref_employee_number - 1),
                                    (jobpref_day_number - 1) * number_shifts_per_day + (jobpref_shift_number - 1),
                                    (jobpref_job_number - 1)] = 1

    jobpref_employee_number = 13  # ranging from 1 to number_employees
    jobpref_day_number = 29  # ranging from 1 to number_days
    jobpref_shift_number = 2  # ranging from 1 to number_shifts_per_day
    jobpref_job_number = 2  # ranging from 1 to number_jobs (1 = A2, 2 = A1, 3 = FA)
    employee_jobs_preference_matrix[(jobpref_employee_number - 1),
                                    (jobpref_day_number - 1) * number_shifts_per_day + (jobpref_shift_number - 1),
                                    (jobpref_job_number - 1)] = 1

    # Employee Number 15
    jobpref_employee_number = 15  # ranging from 1 to number_employees
    jobpref_day_number = 2  # ranging from 1 to number_days
    jobpref_shift_number = 3  # ranging from 1 to number_shifts_per_day
    jobpref_job_number = 1  # ranging from 1 to number_jobs (1 = A2, 2 = A1, 3 = FA)
    employee_jobs_preference_matrix[(jobpref_employee_number - 1),
                                    (jobpref_day_number - 1) * number_shifts_per_day + (jobpref_shift_number - 1),
                                    (jobpref_job_number - 1)] = 1

    jobpref_employee_number = 15  # ranging from 1 to number_employees
    jobpref_day_number = 29  # ranging from 1 to number_days
    jobpref_shift_number = 3  # ranging from 1 to number_shifts_per_day
    jobpref_job_number = 1  # ranging from 1 to number_jobs (1 = A2, 2 = A1, 3 = FA)
    employee_jobs_preference_matrix[(jobpref_employee_number - 1),
                                    (jobpref_day_number - 1) * number_shifts_per_day + (jobpref_shift_number - 1),
                                    (jobpref_job_number - 1)] = 1

    # Employee Number 19
    jobpref_employee_number = 19  # ranging from 1 to number_employees
    jobpref_day_number = 6  # ranging from 1 to number_days
    jobpref_shift_number = 2  # ranging from 1 to number_shifts_per_day
    jobpref_job_number = 3  # ranging from 1 to number_jobs (1 = A2, 2 = A1, 3 = FA)
    employee_jobs_preference_matrix[(jobpref_employee_number - 1),
                                    (jobpref_day_number - 1) * number_shifts_per_day + (jobpref_shift_number - 1),
                                    (jobpref_job_number - 1)] = 1

    jobpref_employee_number = 19  # ranging from 1 to number_employees
    jobpref_day_number = 25  # ranging from 1 to number_days
    jobpref_shift_number = 4  # ranging from 1 to number_shifts_per_day
    jobpref_job_number = 3  # ranging from 1 to number_jobs (1 = A2, 2 = A1, 3 = FA)
    employee_jobs_preference_matrix[(jobpref_employee_number - 1),
                                    (jobpref_day_number - 1) * number_shifts_per_day + (jobpref_shift_number - 1),
                                    (jobpref_job_number - 1)] = 1

    jobpref_employee_number = 19  # ranging from 1 to number_employees
    jobpref_day_number = 29  # ranging from 1 to number_days
    jobpref_shift_number = 2  # ranging from 1 to number_shifts_per_day
    jobpref_job_number = 3  # ranging from 1 to number_jobs (1 = A2, 2 = A1, 3 = FA)
    employee_jobs_preference_matrix[(jobpref_employee_number - 1),
                                    (jobpref_day_number - 1) * number_shifts_per_day + (jobpref_shift_number - 1),
                                    (jobpref_job_number - 1)] = 1

    # Employee Number 22
    jobpref_employee_number = 22  # ranging from 1 to number_employees
    jobpref_day_number = 3  # ranging from 1 to number_days
    jobpref_shift_number = 5  # ranging from 1 to number_shifts_per_day
    jobpref_job_number = 1  # ranging from 1 to number_jobs (1 = A2, 2 = A1, 3 = FA)
    employee_jobs_preference_matrix[(jobpref_employee_number - 1),
                                    (jobpref_day_number - 1) * number_shifts_per_day + (jobpref_shift_number - 1),
                                    (jobpref_job_number - 1)] = 1

    jobpref_employee_number = 22  # ranging from 1 to number_employees
    jobpref_day_number = 18  # ranging from 1 to number_days
    jobpref_shift_number = 5  # ranging from 1 to number_shifts_per_day
    jobpref_job_number = 1  # ranging from 1 to number_jobs (1 = A2, 2 = A1, 3 = FA)
    employee_jobs_preference_matrix[(jobpref_employee_number - 1),
                                    (jobpref_day_number - 1) * number_shifts_per_day + (jobpref_shift_number - 1),
                                    (jobpref_job_number - 1)] = 1

    # Employee Number 23
    jobpref_employee_number = 23  # ranging from 1 to number_employees
    jobpref_day_number = 1  # ranging from 1 to number_days
    jobpref_shift_number = 2  # ranging from 1 to number_shifts_per_day
    jobpref_job_number = 3  # ranging from 1 to number_jobs (1 = A2, 2 = A1, 3 = FA)
    employee_jobs_preference_matrix[(jobpref_employee_number - 1),
                                    (jobpref_day_number - 1) * number_shifts_per_day + (jobpref_shift_number - 1),
                                    (jobpref_job_number - 1)] = 1

    jobpref_employee_number = 23  # ranging from 1 to number_employees
    jobpref_day_number = 8  # ranging from 1 to number_days
    jobpref_shift_number = 4  # ranging from 1 to number_shifts_per_day
    jobpref_job_number = 2  # ranging from 1 to number_jobs (1 = A2, 2 = A1, 3 = FA)
    employee_jobs_preference_matrix[(jobpref_employee_number - 1),
                                    (jobpref_day_number - 1) * number_shifts_per_day + (jobpref_shift_number - 1),
                                    (jobpref_job_number - 1)] = 1

    jobpref_employee_number = 23  # ranging from 1 to number_employees
    jobpref_day_number = 10  # ranging from 1 to number_days
    jobpref_shift_number = 4  # ranging from 1 to number_shifts_per_day
    jobpref_job_number = 2  # ranging from 1 to number_jobs (1 = A2, 2 = A1, 3 = FA)
    employee_jobs_preference_matrix[(jobpref_employee_number - 1),
                                    (jobpref_day_number - 1) * number_shifts_per_day + (jobpref_shift_number - 1),
                                    (jobpref_job_number - 1)] = 1

    jobpref_employee_number = 23  # ranging from 1 to number_employees
    jobpref_day_number = 14  # ranging from 1 to number_days
    jobpref_shift_number = 2  # ranging from 1 to number_shifts_per_day
    jobpref_job_number = 2  # ranging from 1 to number_jobs (1 = A2, 2 = A1, 3 = FA)
    employee_jobs_preference_matrix[(jobpref_employee_number - 1),
                                    (jobpref_day_number - 1) * number_shifts_per_day + (jobpref_shift_number - 1),
                                    (jobpref_job_number - 1)] = 1

    # Employee Number 25
    jobpref_employee_number = 25  # ranging from 1 to number_employees
    jobpref_day_number = 16  # ranging from 1 to number_days
    jobpref_shift_number = 3  # ranging from 1 to number_shifts_per_day
    jobpref_job_number = 1  # ranging from 1 to number_jobs (1 = A2, 2 = A1, 3 = FA)
    employee_jobs_preference_matrix[(jobpref_employee_number - 1),
                                    (jobpref_day_number - 1) * number_shifts_per_day + (jobpref_shift_number - 1),
                                    (jobpref_job_number - 1)] = 1

    jobpref_employee_number = 25  # ranging from 1 to number_employees
    jobpref_day_number = 22  # ranging from 1 to number_days
    jobpref_shift_number = 3  # ranging from 1 to number_shifts_per_day
    jobpref_job_number = 1  # ranging from 1 to number_jobs (1 = A2, 2 = A1, 3 = FA)
    employee_jobs_preference_matrix[(jobpref_employee_number - 1),
                                    (jobpref_day_number - 1) * number_shifts_per_day + (jobpref_shift_number - 1),
                                    (jobpref_job_number - 1)] = 1

    jobpref_employee_number = 25  # ranging from 1 to number_employees
    jobpref_day_number = 25  # ranging from 1 to number_days
    jobpref_shift_number = 5  # ranging from 1 to number_shifts_per_day
    jobpref_job_number = 1  # ranging from 1 to number_jobs (1 = A2, 2 = A1, 3 = FA)
    employee_jobs_preference_matrix[(jobpref_employee_number - 1),
                                    (jobpref_day_number - 1) * number_shifts_per_day + (jobpref_shift_number - 1),
                                    (jobpref_job_number - 1)] = 1

    jobpref_employee_number = 25  # ranging from 1 to number_employees
    jobpref_day_number = 29  # ranging from 1 to number_days
    jobpref_shift_number = 3  # ranging from 1 to number_shifts_per_day
    jobpref_job_number = 1  # ranging from 1 to number_jobs (1 = A2, 2 = A1, 3 = FA)
    employee_jobs_preference_matrix[(jobpref_employee_number - 1),
                                    (jobpref_day_number - 1) * number_shifts_per_day + (jobpref_shift_number - 1),
                                    (jobpref_job_number - 1)] = 1

    # Employee Number 27
    jobpref_employee_number = 27  # ranging from 1 to number_employees
    jobpref_day_number = 22  # ranging from 1 to number_days
    jobpref_shift_number = 3  # ranging from 1 to number_shifts_per_day
    jobpref_job_number = 1  # ranging from 1 to number_jobs (1 = A2, 2 = A1, 3 = FA)
    employee_jobs_preference_matrix[(jobpref_employee_number - 1),
                                    (jobpref_day_number - 1) * number_shifts_per_day + (jobpref_shift_number - 1),
                                    (jobpref_job_number - 1)] = 1

    # Employee Number 28
    jobpref_employee_number = 28  # ranging from 1 to number_employees
    jobpref_day_number = 20  # ranging from 1 to number_days
    jobpref_shift_number = 2  # ranging from 1 to number_shifts_per_day
    jobpref_job_number = 3  # ranging from 1 to number_jobs (1 = A2, 2 = A1, 3 = FA)
    employee_jobs_preference_matrix[(jobpref_employee_number - 1),
                                    (jobpref_day_number - 1) * number_shifts_per_day + (jobpref_shift_number - 1),
                                    (jobpref_job_number - 1)] = 1
    # Each variable is assigned a range
    employees = range(number_employees)
    jobs = range(number_jobs)
    days = range(number_days)
    schedule = range(number_total_shifts)
    shifts_1day = range(number_shifts_per_day)

    # 1) Creation of the model
    model = cp_model.CpModel()

    # 2) Creation of decision variable
    # newBoolVar: creates a 0-1 with the given name
    # shifts[(e, s, j)] = 1 if job j is assigned to employee e on shift s, otherwise = 0
    shifts = {}
    for e in employees:
        for s in schedule:
            for j in jobs:
                shifts[(e, s, j)] = model.NewBoolVar('shift_n%id%is%i' % (e, s, j))

    # 3) Creation of General Constraints
    # 3.1) (NEW) General_Constraint: Shifts are only scheduled if they need to be scheduled on that specific day
    for e in employees:
        for s in schedule:
            for j in jobs:
                model.Add(shifts[(e, s, j)] <= day_shift_array[s])

    # 3.2) (NEW) General_Constraint: Jobs are only scheduled if they need to be scheduled during that specific shift
    for e in employees:
        for s in schedule:
            for j in jobs:
                model.Add(shifts[(e, s, j)] <= day_shift_job_matrix[j, s])

    # 3.3) General_Constraint: Each job on a shift has to be assigned to exactly 4 employees for day-shifts and
    # 1 employee for on-call- and standby-shifts
    # For each s and j the sum of all employees must be equal to 1 * day_shift_array * day_shift_job_matrix
    for s in schedule:
        for j in jobs:
            if s % 5 == 0:
                model.Add(sum(shifts[(e, s, j)] for e in employees)
                          == 4 * day_shift_array[s] * day_shift_job_matrix[j, s])
            else:
                model.Add(sum(shifts[(e, s, j)] for e in employees)
                          == 1 * day_shift_array[s] * day_shift_job_matrix[j, s])

    # 3.4) General_Constraint: Each employee works maximum on 1 job per shift
    # For each e and d the sum of all jobs must be smaller or equal to 1
    for e in employees:
        for s in schedule:
            model.Add(sum(shifts[(e, s, j)] for j in jobs) <= 1)

    # 4) Creation of Specific  Constraints
    # 4.1) Specific constraint: Limitation of shifts to min./max. values
    # Each employee works a maximum of X and a minimum of Y shifts per considered time period.
    # For each employee the sum over all jobs and days must be smaller/equal to X and be larger/equal to Y.
    # Number of shifts per employee are counted, and the sum should be smaller than the (individual) maximum
    # shifts X per employee and larger than the (individual) minimum shifts Y per employee
    for e in employees:
        num_shifts_of_employee = []
        for s in schedule:
            for j in jobs:
                num_shifts_of_employee.append(shifts[(e, s, j)])
        model.Add(sum(num_shifts_of_employee) <= max_shifts_per_employee[e])
        model.Add(sum(num_shifts_of_employee) >= min_shifts_per_employee[e])

    # 4.2) Specific constraint: Each employee can only be assigned to shifts when he/she is available
    for e in employees:
        for s in schedule:
            for j in jobs:
                model.Add(shifts[(e, s, j)] <= employee_availability_matrix_wholedays[e, (s // number_shifts_per_day)])

    # 4.3) Specific constraint: Each employee needs to have a minimum skill level to be scheduled on a specific job
    # 4.3.1) Step 1: Calculate employee_job_calculation_matrix
    for e in employees:
        for j in jobs:
            employee_job_calculation_matrix[e, j] = \
                calculateScore(e, j, employee_qualification_matrix, job_required_qualification_matrix)
    print('employee_job_calculation_matrix:')
    print(employee_job_calculation_matrix)
    print()

    # 4.3.2) Step 2: Use employee_job_calculation_matrix to define constraint on solution
    # Each time 0 occurs in the employee_job_calculation_matrix, the employee is not able to work on the respective job
    # ATTENTION!! starts counting for e and j at 0 and not at 1
    print('The employee-job restrictions are:')
    for e in employees:
        for j in jobs:
            if employee_job_calculation_matrix[e, j] == 0:
                print('Employee', e, 'can not work as', job_names[j])
                for s in schedule:
                    model.Add(shifts[(e, s, j)] == 0)
    print()

    # 4.4) Specific constraint: Prevent employees to be scheduled on overlapping shifts, and on shifts taking
    # place at the same time.
    # Note, that this constraint is specific to the shifts of the considered department and hospital.
    # The first term prevents each employee working on the first shift of the day,
    # if they have worked on the second, third, fourth, or fifth shift of the previous day.
    # The second term prevents each employee working simultaneously on the second and third shift of each day.
    # The third term prevents each employee working simultaneously on the fourth and fifth shift of each day.
    for e in employees:
        for d in days:
            if d > 0:
                for k in range(1, 5):
                    model.Add(sum(shifts[(e, 5 * d - k, j)] for j in jobs) +
                              sum(shifts[(e, 5 * d, j)] for j in jobs) <= 1)
            model.Add(sum(shifts[(e, 5 * d + 1, j)] for j in jobs) + sum(shifts[(e, 5 * d + 2, j)] for j in jobs) <= 1)
            model.Add(sum(shifts[(e, 5 * d + 3, j)] for j in jobs) + sum(shifts[(e, 5 * d + 4, j)] for j in jobs) <= 1)

    ###################################################################################################################
    # The following constraints are all new constraints derived from the 'Act on the Implementation of Measures       #
    # of Occupational Safety and Health' and need to be considered when scheduling doctors in hospitals.              #
    # Some constraints are specific to the considered department and hospital, these constraints are marked with **.. #
    ###################################################################################################################

    # 4.5**) Specific constraint: Prevent employees to be scheduled on consecutive shifts that prevent 11 hours of rest
    # time between working days
    # The terms prevent each employee working on the fourth or fifth shift of the day,
    # if they have worked on the fourth or fifth shift of the previous day.
    for e in employees:
        for d in days:
            if d > 0:
                model.Add(sum(shifts[(e, 5 * d - 2, j)] for j in jobs) +
                          sum(shifts[(e, 5 * d + 3, j)] for j in jobs) <= 1)
                model.Add(sum(shifts[(e, 5 * d - 2, j)] for j in jobs) +
                          sum(shifts[(e, 5 * d + 4, j)] for j in jobs) <= 1)
                model.Add(sum(shifts[(e, 5 * d - 1, j)] for j in jobs) +
                          sum(shifts[(e, 5 * d + 3, j)] for j in jobs) <= 1)
                model.Add(sum(shifts[(e, 5 * d - 1, j)] for j in jobs) +
                          sum(shifts[(e, 5 * d + 4, j)] for j in jobs) <= 1)

    ###################################################################################################################
    # The following constraints are derived from the collective agreements of the "Marburger Bund".                   #
    # Again, if there are e.g. specific consecutive shifts to be avoided as a result of the collective agreements     #
    # of the "Marburger Bund", these constraints will still be marked with **, as the shifts are dependent on the     #
    # considered department and hospital.                                                                             #
    ###################################################################################################################

    # 4.6**) Specific constraint: Limitation of on-call- and standby-shifts to a maximum of 4 on-call-shifts per month
    # This constraint limits the number of on-call- and standby-shifts, hence second, third, fourth and fifth
    # shift of each day to a maximum of 4 per month
    # !!ATTENTION: Limit needs to be adjusted to the time period that is considered
    # (Alternative to be used as maximum number of on-call-shifts: np.ceil(4 / (356 / 12) * number_days))
    for e in employees:
        num_on_call_shifts1_of_employee = []
        num_on_call_shifts2_of_employee = []
        num_on_call_shifts3_of_employee = []
        num_on_call_shifts4_of_employee = []
        for d in days:
            for j in jobs:
                num_on_call_shifts1_of_employee.append(shifts[(e, d * 5 + 1, j)])
                num_on_call_shifts2_of_employee.append(shifts[(e, d * 5 + 2, j)])
                num_on_call_shifts3_of_employee.append(shifts[(e, d * 5 + 3, j)])
                num_on_call_shifts4_of_employee.append(shifts[(e, d * 5 + 4, j)])
        model.Add(sum(num_on_call_shifts1_of_employee) + sum(num_on_call_shifts2_of_employee) +
                  sum(num_on_call_shifts3_of_employee) + sum(num_on_call_shifts4_of_employee) <= 5)

    # 4.7**) Specific constraint: minimum of 2 weekends off per month
    # This constraint limits the number of weekendshifts, hence fourth and fifth shift of each day
    # to a maximum of 2 per month
    # !!ATTENTION: Limit needs to be adjusted to the time period that is considered
    # (Alternative to be used as maximum number of weekendshifts: np.ceil(2 / (356 / 12) * number_days))
    for e in employees:
        num_on_call_shifts3_of_employee = []
        num_on_call_shifts4_of_employee = []
        for d in days:
            for j in jobs:
                num_on_call_shifts3_of_employee.append(shifts[(e, d * 5 + 3, j)])
                num_on_call_shifts4_of_employee.append(shifts[(e, d * 5 + 4, j)])
        model.Add(sum(num_on_call_shifts3_of_employee) + sum(num_on_call_shifts4_of_employee) <= 2)

    ###################################################################################################################
    # The following constraints are ergonomic constraints to reduce the workload of each doctor.                      #
    ###################################################################################################################

    # 4.8**) Specific constraint: Limitation of consecutive on-call- or standby-shifts to a maximum of 2 days in a row.
    # The constraint prevents each employee working on the second, third, fourth, or fifth shift (weekdayshifts) of
    # each day for more than 2 days in a row.
    for e in employees:
        for d in days:
            if d < number_days - 2:
                model.Add(sum((shifts[(e, ((5 * (d + k)) + 1), j)] + shifts[(e, ((5 * (d + k)) + 2), j)]
                              + shifts[(e, ((5 * (d + k)) + 3), j)] + shifts[(e, ((5 * (d + k)) + 4), j)])
                              for j in jobs for k in range(3)) <= 2)

    # 4.9**) Specific constraint: Limitation of on-call- or standby-shifts to a maximum of 1 in x days in a row,
    # if there are no preferences for specific jobs during these x days.
    # If there are preferences, the sum of the shifts is limited to the amount of preferences within these x days.
    # The corresponding formula for both cases is: shifts_in_x_days = 1 + (sum_of_preferences_in_3_days – 1).
    range_x_days = range(3)
    shifts_1to4 = range(1, number_shifts_per_day)
    for e in employees:
        for d in days:
            if d < number_days - 2:
                number_preferences_in_three_days = (sum(employee_jobs_preference_matrix[(e, ((5 * (d + k)) + i), j)]
                                                        for j in jobs for i in shifts_1to4 for k in range_x_days))
                if number_preferences_in_three_days >= 1:
                    variable_a = 0
                else:
                    variable_a = 1
                model.Add(sum(shifts[(e, ((5 * (d + k)) + i), j)]
                              for j in jobs for i in shifts_1to4 for k in range_x_days)
                          <= variable_a + number_preferences_in_three_days)

    # 5) Definition objective: maximize employee preferences
    # The objective is to maximize the fulfilled preferences for offdays and jobs.
    # Weights for preferences can be adjusted to adjust the solution
    offday_pref_weight = 1
    job_pref_weight = 1
    objective = sum(offday_pref_weight * (shifts[(e, s, j)] - 1) *
                    (employee_offdays_preference_matrix[e, (s // number_shifts_per_day)] - 1)
                    for e in employees for s in schedule for j in jobs) + \
                sum(job_pref_weight * shifts[(e, s, j)] * employee_jobs_preference_matrix[(e, s, j)]
                    for e in employees for s in schedule for j in jobs) + \
                sum(shifts[(e, s, j)] * employee_priorities[e]
                    for e in employees for s in schedule for j in jobs)
    model.Maximize(objective)

    # 6.) Problem solver
    # Cp.Solver(): searches for solutions
    solver = cp_model.CpSolver()
    solver.Solve(model)

    # 7.) Show solution:
    # Create array which saves number of shifts per employee and is increased by 1 each time value = 1
    number_shifts_per_employee = [0 for i in range(number_employees)]
    number_workinghours_per_employee = [0 for i in range(number_employees)]
    counter_day: int = 1

    # Print solution: Overall schedule
    for s in schedule:
        if day_shift_array[s] == 1:
            print('Day', counter_day, shift_names[s % number_shifts_per_day], s + 1)
        counter_day = divsible(s + 1, number_shifts_per_day, counter_day)
        for e in employees:
            shift_within_day = s % number_shifts_per_day
            for j in jobs:
                if solver.Value(shifts[(e, s, j)]) == 1:
                    print('Employee', e + 1, 'works as', job_names[j])
                    number_shifts_per_employee[e] = number_shifts_per_employee[e] + 1
                    number_workinghours_per_employee[e] += workinghours_shifts_1day[shift_within_day]
        print()

    # Print number of shifts per employee
    print()
    print('Overview total number of shifts per employee:')
    for e in employees:
        print('Employee', e + 1, 'works on', number_shifts_per_employee[e], 'shifts in total')

    # Print total preference score for off-days of all employees (= first part of the objective)
    total_score_off_days: int = 0
    individual_score_off_days = [0 for i in range(number_employees)]
    employee_indices_with_preferences = [e for e in employees
                                         if sum(employee_offdays_preference_matrix_inverted[e]) > 0]
    number_off_day_preferences_per_employee = np.count_nonzero(employee_offdays_preference_matrix_inverted, axis=1)
    number_employees_with_offday_preferences = np.size(employee_indices_with_preferences)
    total_number_offdays_preferences = np.sum(number_off_day_preferences_per_employee)
    average_number_offdays_preferences_per_employee = \
        total_number_offdays_preferences / number_employees_with_offday_preferences

    for e in employees:
        for d in days:
            if employee_offdays_preference_matrix[e, d] == 0 and \
                    sum(solver.Value(shifts[(e, 5 * d + s, j)]) for j in jobs for s in shifts_1day) == 0:
                total_score_off_days = total_score_off_days + 1
                individual_score_off_days[e] = individual_score_off_days[e] + 1

    print()
    print('The total number of fulfilled off-day preferences (total off-day preference score) is:',
          total_score_off_days)

    # Print individual preference score for off-days per employee
    print()
    print('Individual number of fulfilled off-day preferences (individual off-day preference score) per employee:')
    for e in employees:
        print('Employee ', e + 1, 'has an off-day preference score of: ', individual_score_off_days[e])

    # Print maximum (possible) preference score for off-days per employee
    print()
    # Array maxInRows_off_days saves the maximum (possible) preference score for off-days for each employee
    # in the employee_jobs_preference_matrix_inverted
    maxInRows_off_days = np.sum(employee_offdays_preference_matrix_inverted, axis=1)

    print('Number of submitted off-day preferences (maximum individual off-day preference score) per employee:')
    for e in employees:
        print('Employee ', e + 1, 'has a maximum possible off-day preference score of: ', int(maxInRows_off_days[e]))
    print('Note: between those employees with preferences for off-days, '
          'the average number of preferences for off-days is: ', average_number_offdays_preferences_per_employee)

    # Print percentage of fulfilled preferences for off-days per employee
    total_fulfilled_preferences_off_days = 0
    print()
    print('Individual percentage of fulfilled off-day preferences per employee:')
    for e in employees:
        if maxInRows_off_days[e] == 0:
            print('Employee ', e + 1, ': has no preferences for off-days.')
        else:
            percentage_fulfilled_off_day_preferences = individual_score_off_days[e] / maxInRows_off_days[e] * 100
            total_fulfilled_preferences_off_days += individual_score_off_days[e]
            print('Employee ', e + 1, ':', int(percentage_fulfilled_off_day_preferences), '% (',
                  individual_score_off_days[e], '/', int(maxInRows_off_days[e]), ') of preferences for off-days '
                                                                                 'were fulfilled.')
    print('Total percentage of fulfilled off-day preferences: ', total_fulfilled_preferences_off_days, '/',
          total_number_offdays_preferences, ' = ',
          int(total_fulfilled_preferences_off_days / total_number_offdays_preferences * 100), '%')

    # Print total preference score for jobs of all employees (= second part of the objective, which should be minimized)
    total_score_jobs: int = 0
    individual_score_jobs = [0 for i in range(number_employees)]
    total_fulfilled_preferences_jobs = 0
    employee_indices_with_job_preferences = [e for e in employees
                                             if np.sum(np.sum(employee_jobs_preference_matrix, axis=1), axis=1)[e] > 0]
    number_job_preferences_per_employee = np.count_nonzero(employee_jobs_preference_matrix, axis=1)
    number_employees_with_job_preferences = np.size(employee_indices_with_job_preferences)
    total_number_job_preferences = np.sum(number_job_preferences_per_employee)
    average_number_job_preferences_per_employee = total_number_job_preferences / number_employees_with_job_preferences

    for e in employees:
        for s in schedule:
            for j in jobs:
                if employee_jobs_preference_matrix[e, s, j] == 1 and \
                        solver.Value(shifts[(e, s, j)]) == employee_jobs_preference_matrix[e, s, j]:
                    total_score_jobs = total_score_jobs + 1
                    individual_score_jobs[e] = individual_score_jobs[e] + 1
    print()
    print('The total number of fulfilled job preferences (total job preference score):', total_score_jobs)

    # Print individual preference score for jobs per employee
    print()
    print('Individual number of fulfilled job preferences (individual job preference score) per employee:')
    for e in employees:
        print('Employee ', e + 1, 'has a job preference score of: ', individual_score_jobs[e])

    # Print maximum (possible) preference score for jobs per employee
    print()
    # Array maxInRows_jobs saves the maximum (possible) preference score for jobs for each employee
    # in the employee_jobs_preference_matrix
    maxInRows_jobs = np.sum(np.sum(employee_jobs_preference_matrix, axis=1), axis=1)

    print('Number of submitted job preferences (maximum job preference score) per employee:')
    for e in employees:
        print('Employee ', e + 1, 'has a maximum possible job preference score of: ', int(maxInRows_jobs[e]))
    print('Note: between those employees with preferences for jobs, the average number of preferences is: ',
          average_number_job_preferences_per_employee)

    # Print percentage of fulfilled preferences for jobs per employee
    print()
    print('Individual percentage of fulfilled job preferences per employee:')
    for e in employees:
        if maxInRows_jobs[e] == 0:
            print('Employee ', e + 1, ': has no preferences for jobs.')
        else:
            percentage_fulfilled_job_preferences = individual_score_jobs[e] / maxInRows_jobs[e] * 100
            total_fulfilled_preferences_jobs += individual_score_jobs[e]
            print('Employee ', e + 1, ':', int(percentage_fulfilled_job_preferences), '% (', individual_score_jobs[e],
                  '/', int(maxInRows_jobs[e]), ') of preferences for jobs were fulfilled.')
    print('Total percentage of fulfilled job preferences: ',
          total_fulfilled_preferences_jobs, '/', total_number_job_preferences, '=',
          int(total_fulfilled_preferences_jobs / total_number_job_preferences * 100), '%')

    # Code to determine if an optimal solutions could be found
    status = solver.Solve(model)

    print()
    print('The status is:', status)

    # Optimality of solution:
    if status == cp_model.OPTIMAL:
        print('Optimal solution found')
    elif status == cp_model.FEASIBLE:
        print('A solutions found, but may not be optimal')
    else:
        print('No solution could be found')

    # Print solution: individual schedule for each employee and each employee's skill level in brackets
    # ATTENTION: The replace function has to be adjusted if there are more skill levels to be considered
    skill_levels_employee = employee_qualification_matrix[:, 0]
    replace = {1: skill_level_names[0], 2: skill_level_names[1], 3: skill_level_names[2]}
    skill_levels_employee = [replace.get(i, i) for i in skill_levels_employee]
    skill_levels_employee = list(skill_levels_employee)
    print()
    print('Individual schedule per employee:')

    # Counter_day resets counter day to 1 at the beginning and for each employee
    # (counter day to transform shifts into days)
    for e in employees:
        print('Employee', e + 1, '(', skill_levels_employee[e], ') has the following schedule:')
        counter_day = 1
        for s in schedule:
            for j in jobs:
                if solver.Value(shifts[(e, s, j)]) == 1:
                    print('day', counter_day, 'in', shift_names[s % number_shifts_per_day], 'as', job_names[j])
            counter_day = divsible(s + 1, number_shifts_per_day, counter_day)
        print()

    print()
    print('Number of approved days off per employee:')
    number_offdays = np.count_nonzero(employee_availability_matrix_wholedays == 0, axis=1)
    for e in employees:
        print('Employee ', e + 1, 'has', int(number_offdays[e]), 'approved days off')
    print()

    # Solution: Overall scheduling matrix
    shift_employee_matrix = [[[] for s in schedule] for e in employees]
    for e in employees:
        for s in schedule:
            for j in jobs:
                if solver.Value(shifts[(e, s, j)]) == 1:
                    shift_employee_matrix[e][s].append(j + 1)

    # Print overview of fulfilled preferences
    print()
    print('Overview of total number of fulfilled preferences:')
    print('Total fulfilled off-day preferences: ', total_fulfilled_preferences_off_days, '/',
          total_number_offdays_preferences, ' = ',
          round(total_fulfilled_preferences_off_days / total_number_offdays_preferences * 100, 2), '%')
    print('Total fulfilled job preferences: ', total_fulfilled_preferences_jobs, '/',
          total_number_job_preferences, ' = ',
          round(total_fulfilled_preferences_jobs / total_number_job_preferences * 100, 2), '%')
    print()
    print()

    # Stop time and print out the time taken to calculate a solution
    time_end = time.time()
    print(f"Runtime of the program is {time_end - time_start} seconds")


def divsible(currentschedule, numbershifts, counter_day):
    if currentschedule % numbershifts == 0:
        counter_day = counter_day + 1
    return counter_day

    # Method that calculates if an employee has the required level of qualification for a job


def calculateScore(employee, job, employee_qualification_matrix, job_required_qualification_matrix):
    if employee_qualification_matrix[employee] > job_required_qualification_matrix[job]:
        return 0
    return 1


if __name__ == '__main__':
    main()
