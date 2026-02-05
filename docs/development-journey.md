# Development Journey

This document chronicles the development process of the IoT Health Monitoring Wristband project.

## Phase 1: Concept & Planning (April 2024)

### Initial Goals
- Create a wearable health monitoring device
- Integrate multiple biometric sensors
- Enable real-time cloud data visualization
- Implement predictive analytics using ML

### Hardware Selection
- ESP32/Arduino IoT board for WiFi connectivity
- MAX30102 for heart rate and SpO2 monitoring
- MPU6050 for motion tracking
- GSR sensor for stress detection

## Phase 2: Machine Learning Testing (Pre-Sensor Integration)

### Testing with Hospital Data

**Purpose**: Validate ML pipeline before collecting sensor data

**Model Link**: [https://mltools.arduino.cc/public/394857/live](https://mltools.arduino.cc/public/394857/live)

#### What We Tested
- ✅ Time series prediction on real healthcare data
- ✅ Edge Impulse training workflow
- ✅ Model optimization for edge devices
- ✅ Anomaly detection capabilities
- ✅ CSV data import and preprocessing

#### Dataset
- Real hospital patient data (CSV format)
- Multiple health parameters
- Time-stamped readings
- Various patient conditions

#### Key Learnings
1. **Data Preprocessing**: Importance of data normalization and cleaning
2. **Window Size**: Optimal window size for health predictions (2-5 seconds)
3. **Model Architecture**: LSTM performs well for time series health data
4. **Deployment**: Successfully compiled models for Arduino
5. **Accuracy**: Achieved >90% prediction accuracy with proper training data

#### Results
```
Training Accuracy: 92.3%
Validation Accuracy: 89.7%
Inference Time: 45ms
Model Size: 87KB
```

This testing phase was crucial for:
- Understanding Edge Impulse workflow
- Validating ML approach before hardware investment
- Establishing performance baselines
- Learning data requirements

## Phase 3: Arduino Simulation

### Tinkercad/Wokwi Simulation

![Arduino Simulator](../docs/images/simulator.png)

Before building physical hardware, we simulated the circuit:

**Simulation Platform**: Arduino Simulator
- Verified pin connections
- Tested I2C communication
- Validated sensor initialization
- Debugged code logic

**Simulated Components**:
- ESP32 microcontroller
- I2C sensor modules
- LED indicators
- Serial communication

**Simulation Results**:
- ✅ All sensors initialized correctly
- ✅ I2C addresses detected (0x57, 0x68)
- ✅ Data reading and averaging working
- ✅ Cloud connection logic validated

## Phase 4: Real Sensor Data Collection

### Transitioning to Physical Sensors

After validating the ML approach with hospital data, we implemented real sensor integration:

**Actual Sensor Model**: [https://mltools.arduino.cc/public/388214/latest](https://mltools.arduino.cc/public/388214/latest)

![Real Sensor Data](../docs/images/sensordata.png)

#### Hardware Assembly
- Soldered connections for stability
- Mounted sensors on breadboard
- Connected to ESP32
- Added status LEDs

#### Initial Testing
1. **Individual Sensor Tests**
   - GSR: ✅ Readings 0-1023
   - MAX30102: ✅ Heart rate detected
   - MPU6050: ✅ Motion data streaming

2. **Integration Testing**
   - All sensors working simultaneously
   - I2C bus shared successfully
   - No address conflicts

#### Data Collection Sessions
- **Session 1**: Resting state (30 minutes)
- **Session 2**: Light activity (20 minutes)
- **Session 3**: High stress simulation (15 minutes)
- **Session 4**: Sleep monitoring (overnight)

**Total Training Data**: ~5 hours of continuous monitoring

## Phase 5: Arduino IoT Cloud Integration

### Cloud Setup
- Created Thing: "WristbandHealthMonitor"
- Configured 3 variables: GSR, MAX, MPU
- Set up WiFi credentials
- Established secure connection

### Real-Time Monitoring

![Real-Time Data Dashboard](../docs/images/testrealtimedata.png)

**Dashboard Features**:
- Live GSR readings (stress levels)
- Heart rate graph (BPM over time)
- Motion activity visualization
- Historical data analysis

**Connection Stats**:
- Uptime: 99.2%
- Average latency: 120ms
- Data points collected: 180,000+

## Phase 6: Edge Impulse Model Training (Real Sensors)

### Training Pipeline
1. Exported data from Arduino IoT Cloud
2. Uploaded to Edge Impulse Studio
3. Configured impulse:
   - Window: 2000ms
   - Frequency: 10Hz
   - Features: Spectral analysis
4. Trained neural network
5. Deployed to Arduino

### Model Performance
```
Real Sensor Model Metrics:
- Training Accuracy: 94.1%
- Validation Accuracy: 91.8%
- Inference Time: 42ms
- Model Size: 92KB
- RAM Usage: 28KB
```

### Prediction Capabilities
- ✅ Activity classification (rest, active, stressed)
- ✅ 2-second ahead forecasting
- ✅ Anomaly detection (threshold: 0.5)
- ✅ Real-time inference on device

## Phase 7: Current Status

### Working Features
- ✅ Multi-sensor data collection
- ✅ Real-time cloud synchronization
- ✅ ML-based predictions
- ✅ Web dashboard visualization
- ✅ Mobile app monitoring
- ✅ Anomaly alerts

### Performance Metrics
| Metric | Value |
|--------|-------|
| Battery Life | ~10 hours |
| Sampling Rate | 10 Hz |
| Cloud Latency | 120ms avg |
| Prediction Accuracy | 91.8% |
| Uptime | 99.2% |

## Phase 8: Future Roadmap

### Short Term (Next 3 months)
- [ ] PCB design and fabrication
- [ ] 3D printed enclosure
- [ ] Battery optimization
- [ ] OTA firmware updates

### Medium Term (6 months)
- [ ] Additional sensors (temperature, ECG)
- [ ] Advanced ML models (sleep staging)
- [ ] Mobile app with offline mode
- [ ] Data export features

### Long Term (1 year)
- [ ] Medical-grade certification exploration
- [ ] Multi-user support
- [ ] Integration with health platforms
- [ ] Predictive health alerts

## Key Milestones

```
📅 April 2024      - Project initiated
📅 May 2024        - Hospital data ML testing completed
📅 June 2024       - Arduino simulation successful
📅 July 2024       - Real sensors integrated
📅 August 2024     - Cloud connection established
📅 September 2024  - ML model deployed
📅 October 2024    - Beta testing phase
📅 Current         - Stable production prototype
```

## Lessons Learned

### Technical Insights
1. **Test ML pipeline early**: Using hospital data saved weeks of development
2. **Simulate first**: Arduino simulation caught wiring errors before hardware
3. **I2C reliability**: Pull-up resistors are critical for stable communication
4. **Power management**: WiFi drains battery; optimize sleep modes
5. **Data quality**: Sensor placement affects reading accuracy significantly

### Development Best Practices
1. Version control from day one
2. Document as you build
3. Test incrementally
4. Keep credentials secure
5. Community support is invaluable

## Resources Used

### Hardware
- Arduino IoT 33 / ESP32
- Sensor modules from DFRobot
- Breadboard prototyping
- 3.7V LiPo battery

### Software
- Arduino IDE 2.x
- Edge Impulse Studio
- Arduino IoT Cloud
- Git for version control

### Learning Resources
- Arduino documentation
- Edge Impulse tutorials
- YouTube project walkthroughs
- Arduino Forum community

## Community Contributions

### Open Source
- All code released under MIT License
- Hardware schematics shared
- ML models published publicly
- Documentation freely available

### Sharing Knowledge
- Forum posts answering questions
- Blog post about development journey
- Public Edge Impulse models for learning

## Acknowledgments

Special thanks to:
- Arduino community for troubleshooting help
- Edge Impulse team for ML platform
- Open source library authors
- Beta testers for feedback

---

**Project Status**: ✅ Functional Prototype
**Next Milestone**: PCB Design
**Last Updated**: February 2026

*This project represents months of learning, testing, and iterating. From hospital data testing to real sensor integration, each phase built upon the last.*
