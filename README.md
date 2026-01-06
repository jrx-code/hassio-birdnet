# Hassio BirdNET-Go Integration

This project integrates [BirdNET-Go](https://github.com/tphakala/birdnet-go) with **Home Assistant** using MQTT. It analyzes audio from RTSP streams (e.g., IP cameras) to automatically recognize bird species and sends the detection data (species name, scientific name, confidence, image) to Home Assistant sensors.

## 📂 Project Structure

```text
.
├── birdnet
│   ├── config
│   │   └── config.yaml      # Main BirdNET-Go configuration
│   ├── data                 # Persistent data (DB, clips)
│   └── docker-compose.yaml  # Docker service definition
└── Hassio
    ├── lovelace-card.yaml   # UI Card configuration
    └── sensors.yaml         # MQTT Sensor configuration
```

## 🚀 Setup Guide

### 1. BirdNET-Go Configuration

Navigate to `birdnet/config/config.yaml`. You **must** update the following sections to match your specific environment:

*   **Location:** Set your `latitude` and `longitude` in the `birdnet` section. This significantly improves recognition accuracy by filtering species native to your area.
*   **Audio Sources:** Update the `rtsp` section with your camera streams.
    ```yaml
    rtsp:
      urls:
        - rtsp://user:password@192.168.1.10:554/stream1
        # - rtsp://192.168.1.11:554/stream2
    ```
*   **MQTT Broker:** Configure the connection to your Home Assistant MQTT broker.
    ```yaml
    mqtt:
      broker: tcp://192.168.1.XX:1883
      username: mqtt_user
      password: mqtt_password
    ```
*   **Weather (Optional):** Add your OpenWeatherMap API Key in the `weather` section if you want weather data correlated with detections.

### 2. Docker Deployment

Run the container using Docker Compose. Ensure you are in the `birdnet` directory:

```bash
cd birdnet
docker-compose up -d
```

The Web UI will be available at `http://your-server-ip:8085`.

### 3. Home Assistant Integration

#### Sensor Configuration
Add the content of `Hassio/sensors.yaml` to your Home Assistant configuration (e.g., in `configuration.yaml` or `mqtt.yaml` depending on your setup).

**Attributes extracted:**
*   `state`: Common Name
*   `scientific_name`: Scientific Name
*   `confidence`: Confidence level (%)
*   `time`: Time of detection
*   `image_url`: Bird Image URL (fetched from Wikimedia)
*   `camera`: Source Camera Name

#### Dashboard Card
Add a **Markdown** card to your Lovelace dashboard using the code from `Hassio/lovelace-card.yaml`.

**Requirements:**
*   [card-mod](https://github.com/thomasloven/lovelace-card-mod) (via HACS) - required for custom styling (locking the image height).

**Card Code Preview:**
```yaml
type: markdown
card_mod:
  style: |
    ha-card {
      height: 234px !important;
      overflow: hidden !important;
    }
content: >
  {{ state_attr('sensor.birdnet_go', 'time') }} : **{{ states('sensor.birdnet_go') }}** 
  {{ state_attr('sensor.birdnet_go', 'scientific_name') }} 
  <img src="{{ state_attr('sensor.birdnet_go', 'image_url') }}"/>
```

## ⚙️ Technical Details

*   **Docker Image:** Uses `ghcr.io/tphakala/birdnet-go:nightly` for the latest features and optimizations.
*   **Ports:** Web UI runs on port `8085` (mapped from 8080 internal).
*   **Storage:** 
    *   Clips are retained for 30 days or until disk usage hits 70%.
    *   Minimum 10 clips are always kept.
*   **Filters:** 
    *   **Privacy Filter:** Masks human voices (confidence 0.05).
    *   **Dog Bark Filter:** Ignores dog barks (confidence 0.1).
*   **Sensitivity:** Detection threshold is set to `0.8` with dynamic thresholding enabled (trigger 0.9).

## 🤝 Credits

*   Based on [BirdNET-Go](https://github.com/tphakala/birdnet-go) by **tphakala**.
*   Original BirdNET framework by **Cornell Lab of Ornithology** and **Chemnitz University of Technology**.

