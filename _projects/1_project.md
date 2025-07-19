---
layout: page
title: Halo Collar Activity Recognition
description: GPS and sensor-based machine learning model for canine activity classification
# img: assets/img/halo.jpg
importance: 1
category: work
---

## Project Overview

Engineered a GPS and sensor-based machine learning model for canine activity classification using Python and scikit-learn, achieving **89% accuracy** across 8 distinct behavioral patterns. This project was developed in **Fall 2022** as part of a partnership with PAWS LLC, processing real-time sensor data from 500+ collar devices.

## Key Achievements

- **89% accuracy** in classifying 8 distinct canine behavioral patterns
- Real-time processing of sensor data from **500+ collar devices**
- Partnership with **PAWS LLC** for production deployment
- Comprehensive data pipeline for GPS and accelerometer sensor fusion

## Technical Implementation

### Machine Learning Pipeline
- **Algorithm**: Custom ensemble model using scikit-learn
- **Data Processing**: Real-time sensor fusion of GPS coordinates and accelerometer data
- **Feature Engineering**: Time-series analysis of movement patterns, GPS trajectory analysis
- **Validation**: Cross-validation with stratified sampling across different dog breeds and sizes

### Data Sources
- GPS coordinates with timestamp data
- 3-axis accelerometer readings
- Collar orientation sensors
- Environmental context data

### Behavioral Pattern Classification
The model successfully distinguishes between:
1. Walking/Running
2. Playing
3. Resting/Sleeping
4. Eating/Drinking
5. Barking
6. Scratching
7. Exploratory behavior
8. Aggressive behavior

## Impact and Applications

This project demonstrates the practical application of machine learning in IoT devices for pet monitoring and health tracking. The system provides valuable insights for pet owners and veterinarians about:

- Daily activity levels and exercise patterns
- Behavioral changes that might indicate health issues
- Sleep quality and rest periods
- Social interaction patterns with other animals

## Technologies Used

- **Python** for data processing and model development
- **scikit-learn** for machine learning algorithms
- **pandas/numpy** for data manipulation
- **GPS and accelerometer** sensor integration
- **Real-time data processing** pipelines

The successful deployment across 500+ devices validates the model's robustness and scalability for commercial IoT applications in the pet care industry.
