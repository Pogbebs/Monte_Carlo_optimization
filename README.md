# Optimization and Simulation
## PRAISE OGBEBOR
2025-03-07
## CASESTUDY 3: Sturgill Manufacturing Simulation Model using Monte Carlo methods

### Problem Statement
Sturgill Manufacturing needs to determine the optimal number of machines (A, B, C) and employees required to meet weekly production demand for six key parts. The challenge lies in accounting for uncertainties in shop efficiency (70% ± 5%) and variability in production times. A Monte Carlo simulation was used to model these uncertainties and provide resource estimates while avoiding excessive capital investment in machinery and labor costs.

### Solution Approach
 Monte Carlo simulation is employed to model the inherent variability in the manufacturing process. By simulating thousands of potential scenarios incorporating random efficiency factors and production times, we determine the resource requirements that will meet demand with a high level of confidence (95th percentile).

### Method: Monte Carlo simulation with 10,000 trials.
#### Key Uncertainties:

Shop efficiency: Truncated normal distribution (0 ? efficiency ? 1).
Production times: Truncated normal distributions (0 ? efficiency ? 1)).
Decision Variables:
1. Number of Machine A units.
2. Number of Machine B units.
3. Number of Machine C units.
4. Total employees.

### Key Parameters:
Weekly capacity: 120 hours per week (3 shifts × 5 days × 8 hours)
Weekly demand for parts: [42,18,6,6,6,6]
Mean production times per part (hours) for Machines A, B, C (see matrices in code).
Shop efficiency: mean = 0.7 and standard deviation = 0.05.
Staffing rules:
Machines A and B: 1 employee can operate 2 machines
Machine C: 1 employee per machine
# 

### Mathematical Model
Total Production Time
For each machine type m?{A,B,C}, total time is:
where Ntruncnorm is a truncated normal distribution (lower bound = 0 and upper bound = 1).

### Machines Required
xm =
Total Time ?,Effective Hours=120×Efficiency
Employees Required



### Implementation

Load necessary libraries
library(truncnorm)
Warning: package 'truncnorm' was built under R version 4.4.2
Mean shop efficiency
mean_eff <- 0.7  
Standard deviation of efficiency
sd_eff <- 0.05 
run_time_per_week = 120

number of simulation
nsim <- 10000

Set seed for reproducibility
set.seed(123)

Production time matrices (rows = parts 1-6, cols = machines A-C)
Mean 
mean_Times <- matrix(
  c(3.5, 2.6, 8.9,
    3.4, 2.5, 8.0,
    1.8, 3.5, 12.6,
    2.4, 5.8, 12.5,
    4.2, 4.3, 28.0,
    4.0, 4.3, 28.0),
  nrow = 6, byrow = TRUE
)

Standard deviation
sd_Times <- matrix(
  c(0.15, 0.12, 0.15,
    0.15, 0.12, 0.15,
    0.10, 0.15, 0.25,
    0.15, 0.15, 0.25,
    0.15, 0.15, 0.50,
    0.15, 0.15, 0.50),
  nrow = 6, byrow = TRUE
)
Demand for parts 1- 6
demand <- c(84, 18, 6, 6, 6, 6)
demand <- c(42, 18, 6, 6, 6, 6)  

Initialize results for Machine A, Machine B, Machine C, and Employees
results <- data.frame(
  Machines_A = integer(nsim),
  Machines_B = integer(nsim),
  Machines_C = integer(nsim),
  Employees = integer(nsim)
)


A truncated normal distribution was used to ensure realistic constraints in the simulation
Efficiency cannot be < 0% or >100% hence lower bound = 0% and upper bound = 100%
for (i in 1:nsim) {
  // Simulate weekly efficiency (truncated normal)
  efficiency <- rtruncnorm(1, a = 0, b = 1, mean_eff, sd_eff)
  effective_hours <- run_time_per_week * efficiency
  
  // Initialize total production time for each machine 
  total_time_A <- 0
  total_time_B <- 0
  total_time_C <- 0
  
 // Simulate production times (truncated to avoid negative values)
   for (part in 1:6) {
time_A <- sum(rtruncnorm(demand[part], a = 0, b = Inf, mean = mean_Times[part, 1], sd = sd_Times[part, 1]))
time_B <- sum(rtruncnorm(demand[part], a = 0, b = Inf, mean = mean_Times[part, 2], sd = sd_Times[part, 2]))
time_C <- sum(rtruncnorm(demand[part], a = 0, b = Inf, mean = mean_Times[part, 3], sd = sd_Times[part, 3]))

total_time_A <- total_time_A + time_A
total_time_B <- total_time_B + time_B
total_time_C <- total_time_C + time_C
  }
  
   #### Calculate required machines (round up)
  machines_A <- ceiling(total_time_A / effective_hours)
  machines_B <- ceiling(total_time_B / effective_hours)
  machines_C <- ceiling(total_time_C / effective_hours)
  
  #### Calculate employee working on the machine
  Since one employee can work both machine A and machine B and only one can work on machine C.
  employees_AB <- ceiling(machines_A / 2) + ceiling(machines_B / 2)
  employees_C <- machines_C
  total_employees <- employees_AB + employees_C
  
  ### Store results
  results[i, ] <- c(machines_A, machines_B, machines_C, total_employees)
}

Results<- ceiling(colMeans(results))

### Summary Statistics
// cat("=========================== Results ==================================\n")
=========================== Results ==================================
cat("Machines A:\n")
### Machines A:
print(summary(results$Machines_A))
### Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    3.000   4.000   4.000   3.967   4.000   5.000
cat("\nMachines B:\n")
## 
### Machines B:
print(summary(results$Machines_B))
####    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    3.000   3.000   4.000   3.698   4.000   5.000
cat("\nMachines C:\n")
 
### Machines C:
print(summary(results$Machines_C))
####    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    10.00   12.00   12.00   12.52   13.00   17.00
cat("\nEmployees:\n")
## 
### Employees:
print(summary(results$Employees))

    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    14.00   16.00   16.00   16.53   17.00   23.00
#### 95th Percentiles
cat("\n95th Percentiles:\n")
 
#### 95th Percentiles:
cat("Machines A:", quantile(results$Machines_A, 0.95), "\n")
#### Machines A: 4
cat("Machines B:", quantile(results$Machines_B, 0.95), "\n")
#### Machines B: 4
cat("Machines C:", quantile(results$Machines_C, 0.95), "\n")
#### Machines C: 14
cat("Employees:", quantile(results$Employees, 0.95), "\n")
#### Employees: 18
#### Confidence Interval for the Mean
confidence_intervals <- lapply(results, function(x) {
  mean_val <- mean(x)
  sd_val <- sd(x)
  n <- length(x)
  se <- sd_val / sqrt(n)
  z <- qnorm(0.975) # 1.96
  ci_lower <- mean_val - z * se
  ci_upper <- mean_val + z * se
  return(round(c(CI_Lower = ci_lower, CI_Upper = ci_upper), 2))
})

#### Percentile Interval
percentile_intervals <- lapply(results, function(x) {
  quantile(x, probs = c(0.025, 0.975))
})

#### Print Results
cat("=== 95% Confidence Intervals (Mean) ===\n")
=== 95% Confidence Intervals (Mean) === 
print(confidence_intervals)

#### $Machines_A
    CI_Lower CI_Upper 
     3.96     3.97 

#### $Machines_B
    CI_Lower CI_Upper 
    3.69     3.71 

#### $Machines_C
    CI_Lower CI_Upper 
    12.50    12.54 
## 
### $Employees
    CI_Lower CI_Upper 
    16.52    16.55
cat("\n=== 95% Percentile Intervals (Outcomes) ===\n")
 === 95% Percentile Intervals (Outcomes) ===
print(percentile_intervals)

#### $Machines_A
    2.5% 97.5% 
    3     4 
## 
#### $Machines_B
    2.5% 97.5% 
    3     4 
## 
#### $Machines_C
    2.5% 97.5% 
    11    14 
## 
#### $Employees
    2.5% 97.5% 
    15    18
## Results and Analysis
#### Key Statistics from Simulation
    #---------------------------------------
    # Resource   |  Mean  | 95th Percentile
    #--------------------------------------
    # Machines A |  3.967 | 4
    # Machines B |  3.68  | 4
    # Machine  C |  12.52 | 14
    # Employees  |  16.53 | 18
## Sensitivity Analysis
#### 1. Impact of Shop Efficiency
 We analyzed how changes in shop efficiency affect resource requirements at a 5% risk tolerance
 
    #-----------------------------------------------------------
    # Efficiency | Machine A  | Machine B | Machine C | Employees
    #-------------------------------------------------------------
    #   60%      |      5     |    5      |     17    |    23
    #   65%      |      5     |    4      |     15    |    20
    #   70%      |      4     |    4      |     14    |    18
    #   75%      |      4     |    4      |     13    |    17
    #   80%      |      4     |    4      |     12    |    16

This analysis shows that improving shop efficiency by 5% could save the company one Machine C unit and one employee, while a drop to 65% efficiency would require additional resources. This highlights the significant impact of operational efficiency on capital requirements.

#### 2. Demand Variability
Also, when the demand of 42 was changed to 84 ie (84, 18, 6, 6, 6, 6) at 70% efficiency, it will require 6 units of Machine A, 5 units of Machine B and 25 employee to operate this machines.

## Conclusion
The Monte Carlo simulation model accounts for the inherent variability in Sturgill Manufacturing's production process and provides resource requirements with statistical confidence. We recommend:
1. At 70% given in the problem, Installing 4 units of Machines A, 4 units Machines B, and 14 units of Machines C
2. Employing 18 workers to operate these machines.
These would ensure a 95% probability of meeting production demand while accounting for the variability in shop efficiency and processing times. The sensitivity analysis indicates that initiatives to improve shop efficiency could yield significant savings in resource requirements to meet the demand.

