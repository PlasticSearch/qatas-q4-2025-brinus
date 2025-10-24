# MobilityCorp Enhanced Logical Data Model

## Overview
This document extends the existing MobilityCorp data model to include battery health monitoring and vehicle prioritization capabilities for electric bikes and scooters.

## Core Entities (Existing)

### Users
- user_id (PK)
- username
- email
- phone_number
- registration_date
- subscription_type
- payment_method
- address
- emergency_contact

### Vehicles
- vehicle_id (PK)
- vehicle_type (bike/scooter)
- model
- manufacturer
- purchase_date
- current_location_lat
- current_location_lng
- status (available/in_use/maintenance)
- battery_level
- last_maintenance_date

### Trips
- trip_id (PK)
- user_id (FK)
- vehicle_id (FK)
- start_time
- end_time
- start_location_lat
- start_location_lng
- end_location_lat
- end_location_lng
- distance_km
- duration_minutes
- cost
- trip_status

### Maintenance_Records
- maintenance_id (PK)
- vehicle_id (FK)
- maintenance_type
- maintenance_date
- description
- cost
- technician_id
- next_maintenance_due

## New Battery Health & Prioritization Entities

### Battery_Health_Score
- battery_id (PK)
- vehicle_id (FK)
- health_score (DECIMAL 5,2) -- 0.00 to 100.00%
- capacity_retention (DECIMAL 5,2) -- Current capacity vs original
- charge_efficiency (DECIMAL 5,2) -- How well battery accepts charge
- discharge_efficiency (DECIMAL 5,2) -- Energy delivery consistency
- voltage_stability (DECIMAL 5,2) -- Voltage drop patterns
- last_updated (TIMESTAMP)
- degradation_rate (DECIMAL 5,2) -- % degradation per month
- health_status (ENUM: excellent, good, fair, poor, critical)

### Battery_Performance_Metrics
- metric_id (PK)
- battery_id (FK)
- measurement_timestamp (TIMESTAMP)
- temperature_avg (DECIMAL 5,2) -- Average temperature in Celsius
- temperature_max (DECIMAL 5,2) -- Peak temperature
- humidity_exposure (DECIMAL 5,2) -- Humidity level exposure
- vibration_level (DECIMAL 5,2) -- Vibration intensity
- charge_cycles (INTEGER) -- Total charge/discharge cycles
- calendar_age_days (INTEGER) -- Days since battery manufacture
- internal_resistance (DECIMAL 8,4) -- Internal resistance in ohms
- charge_time_minutes (INTEGER) -- Time to full charge
- discharge_time_minutes (INTEGER) -- Time to empty

### Battery_Prioritization_Queue
- priority_id (PK)
- vehicle_id (FK)
- battery_id (FK)
- priority_score (DECIMAL 8,2) -- Calculated priority score
- urgency_level (ENUM: critical, high, medium, low)
- location_demand_factor (DECIMAL 3,2) -- 1.00 to 3.00
- time_factor (DECIMAL 3,2) -- 1.00 to 2.00
- health_impact_factor (DECIMAL 3,2) -- 1.00 to 4.00
- battery_urgency_factor (DECIMAL 3,2) -- 1.00 to 4.00
- created_timestamp (TIMESTAMP)
- last_updated (TIMESTAMP)
- status (ENUM: pending, in_progress, completed, cancelled)

### Battery_Swap_Operations
- swap_id (PK)
- vehicle_id (FK)
- old_battery_id (FK)
- new_battery_id (FK)
- technician_id (FK)
- swap_timestamp (TIMESTAMP)
- location_lat (DECIMAL 10,8)
- location_lng (DECIMAL 11,8)
- swap_duration_minutes (INTEGER)
- notes (TEXT)
- cost (DECIMAL 8,2)

### Charging_Stations
- station_id (PK)
- station_name
- location_lat (DECIMAL 10,8)
- location_lng (DECIMAL 11,8)
- capacity (INTEGER) -- Number of charging slots
- available_slots (INTEGER)
- power_output_kw (DECIMAL 5,2)
- station_type (ENUM: fast, standard, portable)
- status (ENUM: operational, maintenance, offline)
- last_maintenance_date (DATE)

### Battery_Inventory
- inventory_id (PK)
- battery_serial_number (UNIQUE)
- battery_type (ENUM: bike_battery, scooter_battery)
- manufacturer
- model
- manufacture_date (DATE)
- initial_capacity_wh (INTEGER)
- current_capacity_wh (INTEGER)
- status (ENUM: new, in_use, charging, maintenance, retired)
- location_station_id (FK) -- Where battery is currently located
- purchase_cost (DECIMAL 8,2)
- warranty_expiry_date (DATE)

## Enhanced Vehicle Entity (Updated)

### Vehicles (Enhanced)
- vehicle_id (PK)
- vehicle_type (bike/scooter)
- model
- manufacturer
- purchase_date
- current_location_lat
- current_location_lng
- status (available/in_use/maintenance)
- battery_level (DECIMAL 5,2) -- Current charge percentage
- battery_id (FK) -- Link to current battery
- last_maintenance_date
- total_distance_km (DECIMAL 10,2)
- total_trips (INTEGER)
- average_trip_distance (DECIMAL 6,2)
- last_battery_swap_date (DATE)

## Relationships

### Primary Relationships
- Users → Trips (1:Many)
- Vehicles → Trips (1:Many)
- Vehicles → Maintenance_Records (1:Many)
- Vehicles → Battery_Health_Score (1:Many)
- Battery_Health_Score → Battery_Performance_Metrics (1:Many)
- Vehicles → Battery_Prioritization_Queue (1:Many)
- Battery_Inventory → Battery_Swap_Operations (1:Many)
- Charging_Stations → Battery_Inventory (1:Many)

## Business Rules

### Battery Health Monitoring
1. Battery health scores are calculated daily based on performance metrics
2. Batteries with health scores below 70% are flagged for replacement
3. Critical health scores (<50%) trigger immediate battery swap priority
4. Performance metrics are collected every 15 minutes during vehicle operation

### Prioritization Algorithm
```
Priority Score = (Battery Urgency × 4) + (Health Impact × 3) + (Location Demand × 2) + (Time Factor × 1)

Where:
- Battery Urgency: 1-4 (4 = <10% charge, 3 = 10-25%, 2 = 25-50%, 1 = >50%)
- Health Impact: 1-4 (4 = <50% health, 3 = 50-70%, 2 = 70-85%, 1 = >85%)
- Location Demand: 1-3 (3 = high traffic, 2 = medium, 1 = low)
- Time Factor: 1-2 (2 = peak hours, 1 = off-peak)
```

### Battery Swap Operations
1. Vehicles with priority scores >15 are scheduled for immediate battery swap
2. Battery swaps are batched by geographic proximity to optimize technician routes
3. All battery swaps are logged with performance metrics for analysis
4. Charged batteries are pre-positioned at high-demand locations during peak hours

## Data Quality & Monitoring

### Key Performance Indicators (KPIs)
- Average battery health score across fleet
- Battery swap response time (target: <30 minutes for critical priority)
- Battery utilization efficiency
- Cost per battery swap operation
- Customer satisfaction with vehicle availability

### Data Validation Rules
- Battery health scores must be between 0.00 and 100.00
- Priority scores must be positive numbers
- All timestamps must be in UTC
- Location coordinates must be valid latitude/longitude values
- Battery capacity values must be positive integers

## Future Enhancements
- Machine learning models for predictive battery failure
- Integration with weather data for battery performance optimization
- Real-time fleet optimization algorithms
- Customer notification system for battery swap operations
- Integration with third-party charging networks
