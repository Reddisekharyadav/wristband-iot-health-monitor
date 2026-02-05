# Edge Impulse ML Integration Guide

Complete guide to integrating Edge Impulse machine learning models for time series prediction and anomaly detection.

## What is Edge Impulse?

Edge Impulse is a development platform for machine learning on edge devices. It enables:
- Time series forecasting
- Anomaly detection
- Pattern recognition
- Real-time inference on microcontrollers

**Project Model**: https://mltools.arduino.cc/public/388214/latest

## Prerequisites

- Edge Impulse account (free at https://studio.edgeimpulse.com/)
- Edge Impulse CLI (optional, for advanced features)
- Arduino project with sensor data collection
- Python 3.7+ (for data export scripts)

## Part 1: Data Collection

### Method 1: Arduino IoT Cloud Export

1. **Download Historical Data**
   - Go to Arduino IoT Cloud dashboard
   - Select your Thing
   - Navigate to **"Data"** tab
   - Export CSV file with timestamp + sensor values

2. **Data Format**
   ```csv
   timestamp,GSR,MAX,MPU
   1618747200000,485,72,1024
   1618747201000,490,73,1020
   ...
   ```

### Method 2: Direct Serial Collection

Create a data logging sketch:

```cpp
void loop() {
  // Read sensors
  float gsrValue = readGSR();
  float heartRate = readHeartRate();
  float motion = readMotion();
  
  // Print in CSV format
  Serial.print(millis());
  Serial.print(",");
  Serial.print(gsrValue);
  Serial.print(",");
  Serial.print(heartRate);
  Serial.print(",");
  Serial.println(motion);
  
  delay(100); // 10Hz sampling
}
```

Save Serial Monitor output as `sensor_data.csv`

### Method 3: Edge Impulse Data Forwarder

Install Edge Impulse CLI:
```bash
npm install -g edge-impulse-cli
```

Connect and stream data:
```bash
edge-impulse-data-forwarder
```

Follow prompts to:
- Login to Edge Impulse account
- Select project
- Stream live sensor data

## Part 2: Edge Impulse Studio Setup

### Create New Project

1. Go to https://studio.edgeimpulse.com/
2. Click **"Create new project"**
3. Name: "Wristband Health Monitor"
4. Project type: **Time series data**

### Upload Training Data

1. Navigate to **"Data acquisition"**
2. Click **"Upload data"**
3. Upload your CSV file
4. Configure columns:
   - Label column: Activity type (if labeled)
   - Data columns: GSR, MAX, MPU
   - Timestamp: Auto-detect

### Label Data (Optional)

For activity classification:
- Label segments as: "resting", "active", "stressed", "sleeping"
- Use the built-in labeling tool

## Part 3: Create ML Pipeline

### Design Impulse

1. Go to **"Impulse design"**
2. Configure:
   ```
   Window size: 2000ms
   Window increase: 500ms
   Frequency: 10Hz
   ```

### Add Processing Block

Choose **"Spectral Analysis"** for time series:
- FFT length: 128
- Features: FFT, RMS, Spectral power
- Auto-generate features

### Add Learning Block

#### Option 1: Classification (Activity Recognition)
```
Neural Network Architecture:
- Input layer: Auto (from features)
- Hidden layers: 20 neurons, 10 neurons
- Output layer: 4 classes (rest, active, stressed, sleep)
- Activation: ReLU
```

#### Option 2: Anomaly Detection
```
K-means clustering:
- Number of clusters: 32
- Anomaly axes: All
```

## Part 4: Train Model

### Configure Training

```
Training settings:
- Learning rate: 0.0005
- Epochs: 50
- Batch size: 32
- Validation split: 20%
```

### Start Training

1. Click **"Start training"**
2. Monitor:
   - Training accuracy
   - Validation accuracy
   - Confusion matrix

### Expected Performance
```
Accuracy: >90%
Loss: <0.15
Inference time: <50ms
Model size: <100KB
```

## Part 5: Test Model

### Live Classification

1. Go to **"Live classification"**
2. Connect device or upload test samples
3. View real-time predictions

### Model Testing

1. Navigate to **"Model testing"**
2. Test on validation set
3. Review:
   - Accuracy per class
   - Confusion matrix
   - False positive/negative rates

## Part 6: Deploy to Arduino

### Download Model

1. Go to **"Deployment"**
2. Select **"Arduino library"**
3. Choose optimization:
   - **Unoptimized** (easiest to use)
   - **Quantized (int8)** (smaller, faster)
   - **EON Compiler** (best performance)

4. Click **"Build"**
5. Download `.zip` file

### Install Library in Arduino IDE

1. **Sketch → Include Library → Add .ZIP Library**
2. Select downloaded file
3. Library installed!

### Import in Sketch

```cpp
// Include Edge Impulse library
#include <wristband_health_monitor_inferencing.h>

// Define features buffer
float features[EI_CLASSIFIER_DSP_INPUT_FRAME_SIZE];
int feature_ix = 0;

void setup() {
  Serial.begin(115200);
  
  // Initialize Edge Impulse
  ei_printf("Edge Impulse inferencing ready\n");
  ei_printf("Model: %s\n", EI_CLASSIFIER_PROJECT_NAME);
}

void loop() {
  // Collect sensor data
  features[feature_ix++] = readGSR();
  features[feature_ix++] = readHeartRate();
  features[feature_ix++] = readMotion();
  
  // When buffer is full, run inference
  if (feature_ix >= EI_CLASSIFIER_DSP_INPUT_FRAME_SIZE) {
    // Create signal from features
    signal_t signal;
    numpy::signal_from_buffer(features, EI_CLASSIFIER_DSP_INPUT_FRAME_SIZE, &signal);
    
    // Run classifier
    ei_impulse_result_t result = { 0 };
    EI_IMPULSE_ERROR res = run_classifier(&signal, &result, false);
    
    if (res != EI_IMPULSE_OK) {
      ei_printf("ERR: Failed to run classifier (%d)\n", res);
      return;
    }
    
    // Print predictions
    ei_printf("Predictions:\n");
    for (size_t ix = 0; ix < EI_CLASSIFIER_LABEL_COUNT; ix++) {
      ei_printf("  %s: %.5f\n", 
        result.classification[ix].label, 
        result.classification[ix].value);
    }
    
    // Find highest prediction
    float max_value = 0;
    const char* prediction = "";
    for (size_t ix = 0; ix < EI_CLASSIFIER_LABEL_COUNT; ix++) {
      if (result.classification[ix].value > max_value) {
        max_value = result.classification[ix].value;
        prediction = result.classification[ix].label;
      }
    }
    
    ei_printf("\nPrediction: %s (%.2f%%)\n", prediction, max_value * 100);
    
    // Reset buffer
    feature_ix = 0;
  }
  
  delay(100); // 10Hz sampling
}
```

## Part 7: Time Series Forecasting

### Configure for Prediction

For forecasting future values:

1. **Impulse Design**:
   ```
   Window size: 5000ms (5 seconds of history)
   Window increase: 1000ms
   Prediction horizon: 2000ms (predict 2 seconds ahead)
   ```

2. **Learning Block**: Time series (LSTM)
   ```
   Architecture:
   - LSTM layer: 20 units
   - Dense layer: 10 units
   - Output: 3 values (GSR, MAX, MPU predictions)
   ```

### Forecasting Code Example

```cpp
void predictFuture() {
  // Collect last 5 seconds of data
  float features[50 * 3]; // 50 samples x 3 sensors at 10Hz
  
  // Run prediction
  ei_impulse_result_t result = { 0 };
  run_classifier(&signal, &result, false);
  
  // Access predictions
  float predicted_gsr = result.regression.predictions[0];
  float predicted_hr = result.regression.predictions[1];
  float predicted_motion = result.regression.predictions[2];
  
  Serial.print("Predicted GSR in 2s: ");
  Serial.println(predicted_gsr);
}
```

## Part 8: Anomaly Detection

### Enable Anomaly Detection

In Edge Impulse:
1. Add **"Anomaly Detection"** learning block
2. Configure K-means clustering
3. Train on "normal" data only

### Detect Anomalies

```cpp
void checkAnomaly() {
  // Run inference
  ei_impulse_result_t result = { 0 };
  run_classifier(&signal, &result, false);
  
  // Check anomaly score
  float anomaly_score = result.anomaly;
  
  if (anomaly_score > 0.5) {
    Serial.println("⚠️ ANOMALY DETECTED!");
    Serial.print("Score: ");
    Serial.println(anomaly_score);
    
    // Trigger alert
    sendAlert("Unusual health pattern detected");
  }
}
```

## Part 9: Advanced Features

### Feature Importance

Understand which sensors matter most:
- Edge Impulse shows feature importance in training results
- Focus data collection on high-importance sensors

### Online Learning (Future)

Edge Impulse supports:
- Transfer learning
- Model retraining with new data
- Continuous improvement

### Multi-Model Deployment

Run multiple models:
```cpp
// Model 1: Activity classification
run_activity_classifier(&signal, &activity_result);

// Model 2: Stress detection
run_stress_detector(&signal, &stress_result);

// Model 3: Anomaly detection
run_anomaly_detector(&signal, &anomaly_result);
```

## Public Model Access

Your public model: https://mltools.arduino.cc/public/388214/latest

### Share Your Model

1. Edge Impulse Studio → **"Dashboard"**
2. Click **"Make public"**
3. Share URL with community
4. Others can:
   - Clone your project
   - Download your model
   - See training pipeline

## Troubleshooting

### Model Too Large
- Reduce NN layer sizes
- Enable quantization (int8)
- Use EON Compiler

### Low Accuracy
- Collect more diverse training data
- Increase training epochs
- Adjust window size
- Add more features

### Inference Too Slow
- Optimize model (EON Compiler)
- Reduce sampling rate
- Use smaller window size

### Memory Issues
```cpp
// Check available memory
Serial.print("Free RAM: ");
Serial.println(ESP.getFreeHeap());
```

## Performance Optimization

### Buffer Management
```cpp
// Use circular buffer for efficiency
CircularBuffer<float, 150> sensor_buffer;

void loop() {
  sensor_buffer.push(readSensor());
  
  if (sensor_buffer.isFull()) {
    runInference(sensor_buffer.data());
  }
}
```

### Reduce Inference Frequency
```cpp
// Run every 5 seconds instead of continuously
if (millis() - last_inference > 5000) {
  runInference();
  last_inference = millis();
}
```

## Next Steps

- [ ] Collect at least 10 minutes of training data
- [ ] Label data for supervised learning
- [ ] Train initial model (>85% accuracy)
- [ ] Deploy and test on device
- [ ] Collect edge cases and retrain
- [ ] Enable anomaly detection
- [ ] Implement time series forecasting

## Resources

- [Edge Impulse Docs](https://docs.edgeimpulse.com/)
- [Arduino Tutorial](https://docs.edgeimpulse.com/docs/development-platforms/officially-supported-mcu-targets/arduino-nano-33-ble-sense)
- [Example Projects](https://docs.edgeimpulse.com/docs/tutorials/end-to-end-tutorials)
- [Forum](https://forum.edgeimpulse.com/)

---

Transform your wristband into an intelligent health monitoring system! 🧠✨
