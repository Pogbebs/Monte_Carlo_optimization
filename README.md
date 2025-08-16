# Sturgill Manufacturing Simulation Model

**Author:** PRAISE OGBEBOR  
**Date:** 2025-03-07  
**Case Study:** Monte Carlo Optimization for Resource Planning

## Problem Statement

Sturgill Manufacturing needs to determine the optimal number of machines (A, B, C) and employees required to meet weekly production demand for six key parts. The challenge lies in accounting for uncertainties in:

- Shop efficiency (70% ± 5%)
- Variability in production times

A Monte Carlo simulation models these uncertainties and provides resource estimates while avoiding excessive capital investment in machinery and labor costs.

## Solution Approach

Monte Carlo simulation employs 10,000 trials to model the inherent variability in the manufacturing process. By simulating thousands of potential scenarios incorporating random efficiency factors and production times, we determine the resource requirements that will meet demand with 95% confidence.

## Key Parameters

- **Weekly capacity:** 120 hours per week (3 shifts × 5 days × 8 hours)
- **Weekly demand for parts:** [42, 18, 6, 6, 6, 6]
- **Shop efficiency:** Mean = 0.7, Standard deviation = 0.05
- **Staffing rules:**
  - Machines A and B: 1 employee can operate 2 machines
  - Machine C: 1 employee per machine

## Mathematical Model

### Total Production Time
For each machine type m ∈ {A, B, C}, total time is calculated using truncated normal distributions where efficiency cannot be < 0% or > 100%.

### Machines Required
```
x_m = Total Time / (120 × Efficiency)
```

### Employees Required
- Employees for A & B: ⌈machines_A/2⌉ + ⌈machines_B/2⌉
- Employees for C: machines_C
- Total employees: employees_AB + employees_C

## Implementation

### Required Libraries
```r
library(truncnorm)
```

### Key Variables
```r
# Simulation parameters
mean_eff <- 0.7          # Mean shop efficiency
sd_eff <- 0.05           # Standard deviation of efficiency
run_time_per_week <- 120 # Weekly operating hours
nsim <- 10000            # Number of simulations
set.seed(123)            # For reproducibility
```

### Production Time Matrices

**Mean production times (hours) for parts 1-6 across machines A-C:**
```r
mean_Times <- matrix(
  c(3.5, 2.6, 8.9,
    3.4, 2.5, 8.0,
    1.8, 3.5, 12.6,
    2.4, 5.8, 12.5,
    4.2, 4.3, 28.0,
    4.0, 4.3, 28.0),
  nrow = 6, byrow = TRUE
)
```

**Standard deviations:**
```r
sd_Times <- matrix(
  c(0.15, 0.12, 0.15,
    0.15, 0.12, 0.15,
    0.10, 0.15, 0.25,
    0.15, 0.15, 0.25,
    0.15, 0.15, 0.50,
    0.15, 0.15, 0.50),
  nrow = 6, byrow = TRUE
)
```

### Simulation Logic

The simulation uses truncated normal distributions to ensure realistic constraints:

```r
for (i in 1:nsim) {
  # Simulate weekly efficiency (truncated normal)
  efficiency <- rtruncnorm(1, a = 0, b = 1, mean_eff, sd_eff)
  effective_hours <- run_time_per_week * efficiency
  
  # Calculate total production time for each machine type
  for (part in 1:6) {
    time_A <- sum(rtruncnorm(demand[part], a = 0, b = Inf, 
                            mean = mean_Times[part, 1], 
                            sd = sd_Times[part, 1]))
    # Similar calculations for machines B and C
  }
  
  # Calculate required machines (round up)
  machines_A <- ceiling(total_time_A / effective_hours)
  machines_B <- ceiling(total_time_B / effective_hours)
  machines_C <- ceiling(total_time_C / effective_hours)
  
  # Calculate total employees needed
  employees_AB <- ceiling(machines_A / 2) + ceiling(machines_B / 2)
  total_employees <- employees_AB + machines_C
}
```

## Results and Analysis

### Summary Statistics

| Resource   | Mean  | Median | 95th Percentile |
|------------|-------|--------|-----------------|
| Machines A | 3.967 | 4.000  | 4               |
| Machines B | 3.698 | 4.000  | 4               |
| Machines C | 12.52 | 12.00  | 14              |
| Employees  | 16.53 | 16.00  | 18              |

### 95% Confidence Intervals

| Resource   | Lower Bound | Upper Bound |
|------------|-------------|-------------|
| Machines A | 3.96        | 3.97        |
| Machines B | 3.69        | 3.71        |
| Machines C | 12.50       | 12.54       |
| Employees  | 16.52       | 16.55       |

### 95% Percentile Intervals

| Resource   | 2.5% | 97.5% |
|------------|------|-------|
| Machines A | 3    | 4     |
| Machines B | 3    | 4     |
| Machines C | 11   | 14    |
| Employees  | 15   | 18    |

## Sensitivity Analysis

### 1. Impact of Shop Efficiency
Analysis of resource requirements at different efficiency levels (5% risk tolerance):

| Efficiency | Machine A | Machine B | Machine C | Employees |
|------------|-----------|-----------|-----------|-----------|
| 60%        | 5         | 5         | 17        | 23        |
| 65%        | 5         | 4         | 15        | 20        |
| 70%        | 4         | 4         | 14        | 18        |
| 75%        | 4         | 4         | 13        | 17        |
| 80%        | 4         | 4         | 12        | 16        |

**Key Insight:** Improving shop efficiency by 5% could save one Machine C unit and one employee, while a drop to 65% efficiency would require additional resources.

### 2. Demand Variability
When demand for part 1 increases from 42 to 84 units (maintaining 70% efficiency):
- **Required:** 6 units of Machine A, 5 units of Machine B, and 25 employees

## Recommendations

Based on the Monte Carlo simulation results, we recommend:

1. **Install:** 4 units of Machine A, 4 units of Machine B, and 14 units of Machine C
2. **Employ:** 18 workers to operate these machines

This configuration ensures a **95% probability** of meeting production demand while accounting for variability in shop efficiency and processing times.

## Key Benefits

- Accounts for inherent process variability
- Provides statistically confident resource estimates
- Identifies efficiency improvement opportunities
- Minimizes risk of production shortfalls
- Optimizes capital investment decisions

## Future Considerations

The sensitivity analysis indicates that initiatives to improve shop efficiency could yield significant savings in resource requirements. Consider:

- Process improvement programs to increase efficiency
- Regular monitoring of actual vs. simulated performance
- Periodic model updates with new production data
