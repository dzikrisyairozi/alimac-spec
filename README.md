# Alimac v1: Handheld Edge Device with Local Voice AI Assistant
## Technical Specification Document

---

## 1. Hardware Integration

### 1.1 System Architecture

#### 1.1.1 Computing Platform
- **Primary SoC**: Raspberry Pi 5 (preferred) or Raspberry Pi 4 Model B
  - **CPU**: Raspberry Pi 5: Broadcom BCM2712 (quad-core Cortex-A76 @ 2.4GHz)
  - **RAM**: 8GB LPDDR4X-4267 SDRAM (minimum 4GB for Pi 4)
  - **Storage**: 64GB+ high-speed microSD card (Class 10, UHS-I, A2 rating)
  - **Power Requirements**: 5V/3A via USB-C

#### 1.1.2 Audio Subsystem
- **Input**:
  - Primary microphone: MEMS digital microphone array (min. 2 mics for noise cancellation)
  - Specifications: 20Hz-20kHz frequency response, -42dB sensitivity, SNR >65dB
  - ADC resolution: 24-bit with hardware DSP for preprocessing
  - Sampling rate: 48kHz
  
- **Output**:
  - Primary speaker: 3W (min.) full-range driver with Class D amplifier
  - Frequency response: 100Hz-20kHz
  - SNR: >80dB
  - DAC resolution: 24-bit
  - Custom-designed acoustic chamber for optimized sound propagation

- **Audio Processing**:
  - Hardware DSP for audio preprocessing
  - Low-latency audio pipelines (<10ms processing delay)
  - Hardware echo cancellation
  - Automatic gain control
  - Real-time noise reduction

#### 1.1.3 Bluetooth Module
- **Specification**: Bluetooth 5.0 (with BLE support)
- **Chip**: Qualcomm QCC3040 or equivalent
- **Profiles**: A2DP, HSP, HFP
- **Codec Support**: SBC, AAC, aptX, aptX HD
- **Range**: Class 1 (up to 100m line-of-sight)
- **Latency**: <100ms audio latency
- **Multi-point connectivity**: Support for simultaneous connection to 2 devices

#### 1.1.4 Networking
- **Wi-Fi**:
  - Standard: IEEE 802.11ac (Wi-Fi 5) dual-band
  - Frequencies: 2.4GHz and 5GHz
  - Speed: Up to 433 Mbps
  - Hotspot capability with DHCP server
  - Range: 30m indoor coverage
  - Security: WPA3
  - Management: Custom firmware for access point configuration
  
- **Bluetooth LE** for proximal device discovery
- **Optional**: Ethernet port (RJ45) via USB adapter for development

#### 1.1.5 User Interface Components
- **Display**:
  - Type: 1.3" OLED display (128×64 resolution)
  - Interface: SPI/I2C
  - Refresh rate: 30Hz
  - Color: Monochrome with high contrast ratio (2000:1)
  - Visibility: 170° viewing angle
  
- **Controls**:
  - Volume adjustment: Digital rotary encoder with push-button function
  - Dedicated function button: Activation/deactivation with LED status indicator
  - Hardware mute switch for privacy
  
- **Indicators**:
  - RGB LED for status indication:
    - Blue pulsing: Listening
    - Yellow solid: Processing/Generating
    - Green pulsing: Speaking
    - Red: Error or low battery
  - Battery level indicator: 4-stage LED array

#### 1.1.6 Power Management
- **Battery**:
  - Type: Li-ion or LiPo 3.7V
  - Capacity: 5000mAh (minimum)
  - Expected runtime: 8+ hours of active use
  - Standby time: 72+ hours
  - Charging: USB-C PD compatible (5V/3A)
  - Charge time: 0-80% in under 2 hours
  
- **Power Management IC**:
  - Advanced PMICs for optimal power distribution
  - Hardware-level sleep modes
  - Intelligent power scheduling based on usage patterns
  - Temperature monitoring and thermal management
  - Over-voltage and over-current protection
  - Battery health monitoring and management

- **Power States**:
  - Deep sleep: <50mW consumption
  - Standby: <200mW consumption
  - Active listening: <1W consumption
  - Processing/speaking: <3W consumption
  - Peak operation: <5W consumption

### 1.2 Hardware Integration Points

#### 1.2.1 I/O Interfaces
- **GPIO Allocation**:
  - I2S for digital audio (2 pins)
  - SPI for display (4 pins)
  - I2C for sensors and peripherals (2 pins)
  - GPIO for physical controls (5+ pins)
  - UART for debug console (2 pins)
  
- **External Interfaces**:
  - USB-C port (for charging and development)
  - 3.5mm audio jack (optional, for line-in/line-out)

#### 1.2.2 Internal Bus Architecture
- **Primary Data Bus**: SPI at 50MHz for high-speed peripheral communication
- **Secondary Bus**: I2C at 400kHz for control and status communication
- **Audio Bus**: I2S for digital audio transfer

#### 1.2.3 Hardware Interrupt System
- **Critical Interrupts**:
  - Power state changes
  - Audio buffer thresholds
  - User input events
  - Battery critical alerts
- **Priority-based interrupt handling with hardware FIFO**

#### 1.2.4 Custom Hardware Components
- **Audio Amplifier Circuit**:
  - Class-D amplifier with 90% efficiency
  - Hardware volume control with anti-pop circuitry
  - Dynamic range compression for consistent volume levels
  
- **Power Distribution Board**:
  - Voltage regulation for all components
  - Power sequencing logic
  - Load switching for power conservation
  - Overcurrent/thermal protection

### 1.3 Integration Methodology

#### 1.3.1 Hardware-Software Interface
- **Device Drivers**:
  - Custom ALSA audio drivers for low-latency audio capture/playback
  - Display driver with framebuffer support
  - Power management driver with ACPI-like states
  - GPIO mapping interface for physical controls

#### 1.3.2 Hardware Abstraction Layer
- **Abstract interfaces for**:
  - Audio I/O (capture, playback, routing)
  - Display control (buffer management, rendering)
  - User input (debounced, event-based)
  - Power states (transitions, monitoring)
  - Networking (connection management, hotspot control)

#### 1.3.3 Calibration and Testing
- **Factory Calibration Procedures**:
  - Audio calibration for frequency response
  - Microphone sensitivity adjustment
  - Display color/contrast calibration
  - Battery capacity verification
  - Thermal performance verification
  - Radio performance testing

---

## 2. Software Architecture

### 2.1 System Software Stack

#### 2.1.1 Operating System
- **Base OS**: Custom Linux distribution based on Raspberry Pi OS Lite
- **Kernel Customizations**:
  - Real-time patches for audio processing
  - CPU governor optimized for AI workloads
  - Memory management optimized for inference
  - I/O schedulers tuned for flash storage
  - Power management extensions
- **Boot Configuration**:
  - Custom boot sequence optimized for rapid startup (<10 seconds to ready state)
  - Read-only root filesystem with overlay for system stability
  - A/B partition scheme for reliable OTA updates

#### 2.1.2 System Services
- **Service Management**: Systemd for process control
- **Core Services**:
  - `alimac-audio`: Audio capture and playback service
  - `alimac-network`: Wi-Fi/Bluetooth connectivity service
  - `alimac-ui`: User interface and input management
  - `alimac-power`: Power management and battery monitoring
  - `alimac-inference`: AI model execution orchestration
  - `alimac-webserver`: Lightweight web server for dashboard

#### 2.1.3 Security Framework
- **Secure Boot**: Verified boot process with hardware root of trust
- **Encryption**: Full disk encryption for user data
- **Access Control**: Fine-grained permission system
- **Network Security**: Isolated network stack for hotspot mode
- **Update Security**: Signed OTA updates with rollback protection

### 2.2 AI Processing Pipeline

#### 2.2.1 Speech-to-Text (STT) Component
- **Model Architecture**: Conformer or Whisper-style encoder-decoder
- **Model Size**: ~100MB optimized ONNX format
- **Performance Metrics**:
  - Latency: <500ms for 5-second utterance
  - Word Error Rate (WER): <5% for clean audio
  - Languages: English (primary), expandable via model swapping
- **Preprocessing Pipeline**:
  - Audio normalization
  - Voice activity detection
  - Noise suppression
  - Feature extraction (80-dim mel spectrogram)
- **Implementation**:
  - ONNX Runtime with CUDA/TensorRT acceleration if available
  - Quantization: INT8 precision for inference
  - Streaming capability for real-time transcription

#### 2.2.2 Text Generation Component
- **Model Architecture**: Attention-based decoder-only transformer
- **Model Size**: ~500MB-1GB optimized ONNX format
- **Performance Metrics**:
  - Latency: <1s for initial response, streaming thereafter
  - Throughput: 10+ tokens/second generation speed
  - Context window: 2048 tokens
- **Implementation**:
  - ONNX Runtime optimized for CPU inference
  - 8-bit quantization for memory efficiency
  - KV-cache optimization for faster inference
  - Context management for conversation continuity
- **Features**:
  - Domain-specific tuning for assistant-like responses
  - Customizable personality via prompt engineering
  - Command recognition for device control

#### 2.2.3 Text-to-Speech (TTS) Component
- **Model Architecture**: FastSpeech2 with HiFi-GAN vocoder
- **Model Size**: ~100MB optimized ONNX format
- **Performance Metrics**:
  - Latency: <500ms startup, real-time thereafter
  - Sample rate: 22.05kHz
  - Audio quality: MOS >4.0
- **Implementation**:
  - ONNX Runtime optimized for CPU inference
  - Mixed precision (FP16)
  - Audio generation in chunks for streaming output
- **Features**:
  - Expressive prosody with controllable parameters
  - Multiple voice options

#### 2.2.4 Pipeline Integration
- **Data Flow**:
  - Audio Capture → VAD → STT → Text Generation → TTS → Audio Playback
- **Buffer Management**:
  - Circular buffers for audio data
  - Thread-safe queues for inter-process communication
- **Concurrency Model**:
  - Pipeline parallelism for overlapping processing stages
  - Asynchronous processing with event-based coordination
- **Resource Management**:
  - Dynamic CPU core allocation based on workload
  - Memory budgeting for each pipeline stage
  - Thermal-aware throttling

### 2.3 Hardware Interface Layer

#### 2.3.1 Audio Subsystem
- **API**: ALSA with custom extensions
- **Implementation Language**: C/C++
- **Features**:
  - Circular buffer management
  - Latency monitoring
  - Dynamic resampling
  - Hardware-accelerated audio processing where available
- **Performance Targets**:
  - Input-to-output latency: <50ms
  - Buffer underrun prevention: 99.99% reliability

#### 2.3.2 Display Controller
- **API**: Frame buffer with custom compositor
- **Implementation Language**: C/C++
- **Features**:
  - Hardware-accelerated drawing primitives
  - Animation framework
  - Event-based UI updates
  - Power-efficient refresh strategies

#### 2.3.3 Input Processing
- **API**: Event-based input subsystem
- **Implementation Language**: C/C++
- **Features**:
  - Debounce logic for physical inputs
  - Gesture recognition for multi-function controls
  - Input event queuing and prioritization

#### 2.3.4 Power Management
- **API**: Custom power governor
- **Implementation Language**: C/C++
- **Features**:
  - State machine for power transitions
  - Workload prediction for proactive scaling
  - Temperature-aware throttling
  - Battery discharge profiling

### 2.4 Web Interface

#### 2.4.1 Web Server
- **Implementation**: Lightweight embedded HTTP server (e.g., Mongoose)
- **Performance**: Support for 5+ simultaneous connections
- **Security**: Basic authentication, HTTPS with self-signed certificates
- **Features**:
  - RESTful API for device control
  - WebSocket for real-time updates
  - Static file serving

#### 2.4.2 Frontend Application
- **Technology Stack**:
  - HTML5 + CSS3 + Vanilla JavaScript
  - Progressive enhancement for broad compatibility
  - Responsive design for mobile and desktop interfaces
- **UI Components**:
  - Device status dashboard
  - Settings configuration panel
  - Conversation history viewer
  - Voice model management
  - System diagnostics
- **Optimization**:
  - <100KB total bundle size
  - Asset minification and compression
  - Lazy loading for non-critical components

#### 2.4.3 Backend API
- **Architecture**: RESTful JSON API
- **Endpoints**:
  - `/api/system/status` - Device status information
  - `/api/system/settings` - Configuration management
  - `/api/conversations` - Conversation history
  - `/api/models` - AI model management
  - `/api/diagnostics` - System health monitoring
- **Implementation**:
  - Stateless design
  - Cache-friendly responses
  - Rate limiting for security

### 2.5 Data Management

#### 2.5.1 Local Database
- **Database Engine**: SQLite or LMDB
- **Schema Design**:
  - Conversations table
  - Settings table
  - System logs table
  - User preferences table
- **Performance Optimization**:
  - Proper indexing
  - Transaction batching
  - Connection pooling
  - Vacuum scheduling

#### 2.5.2 Data Persistence
- **Storage Strategy**:
  - Configuration: JSON files in `/etc/alimac/`
  - User data: SQLite database in `/var/lib/alimac/`
  - Logs: Rotating text files in `/var/log/alimac/`
  - Models: Optimized binary files in `/opt/alimac/models/`
- **Backup/Restore**:
  - Scheduled database backups
  - Export/import functionality
  - Configuration snapshots

### 2.6 Development and Deployment

#### 2.6.1 Build System
- **Build Tool**: CMake with custom scripts
- **Dependency Management**: Git submodules + package manifest
- **Compilation Optimization**:
  - Link-time optimization
  - Architecture-specific compilation flags
  - Binary size reduction techniques

#### 2.6.2 Testing Framework
- **Unit Tests**: Google Test for C/C++ components
- **Integration Tests**: Custom test harness
- **Performance Benchmarks**: Specialized tooling for AI inference
- **Continuous Integration**: GitHub Actions or similar

#### 2.6.3 Deployment
- **Image Creation**: Custom image building pipeline
- **OTA Updates**:
  - Delta updates for bandwidth efficiency
  - A/B partition scheme for failsafe updates
  - Version control with rollback capability
- **Factory Provisioning**:
  - Initial calibration procedure
  - Device ID assignment
  - Certificate generation

### 2.7 Software Architecture Visual Layout

#### 2.7.1 System Architecture Diagram

```
                                    +---------------------------+
                                    |     Alimac Device         |
                                    +-----------+---------------+
                                                |
                         +----------------------+----------------------+
                         |                      |                      |
            +------------v-----------+ +-----------------+  +----------v-----------+
            |   Hardware Interface   | |                 |  |      Web Server      |
            |        Layer           | |  AI Pipeline    |  |                      |
            |                        | |                 |  |   (Access via WiFi   |
            | +--------------------+ | |                 |  |     Hotspot/LAN)     |
            | |  Input Processing  +----->              |  |                      |
            | +--------------------+ | |                 |  | +------------------+ |
            |                        | |  +----------+  |  | |                  | |
            | +--------------------+ | |  |   STT    +------->  RESTful API     | |
            | |  Audio Subsystem   +----->          |  |  | |                  | |
            | +--------------------+ | |  +-----+----+  |  | +--------+--------+ |
            |                        | |        |       |  |          |          |
            | +--------------------+ | |  +-----v----+  |  | +--------v--------+ |
            | |  Display Controller| | |  |   Text   |  |  | |                  | |
            | +--------------------+ | |  |Generation|  |  | | WebSocket Server | |
            |                        | |  +-----+----+  |  | |                  | |
            | +--------------------+ | |        |       |  | +--------+--------+ |
            | |  Power Management  | | |  +-----v----+  |  |          |          |
            | +--------------------+ | |  |   TTS    |  |  | +--------v--------+ |
            |                        | |  +-----+----+  |  | |                  | |
            +--------+---------------+ |        |       |  | |  Static Content  | |
                     |                 +--------+-------+  | |                  | |
                     |                          |          | +------------------+ |
                     |                 +--------v-------+  |                      |
                     +----------------->                |  |                      |
                                      |  Audio Output   |  |                      |
                                      |                 |  |                      |
                                      +-----------------+  +----------------------+
```

#### 2.7.2 Service Communication Flow

```
+----------------+    +----------------+    +----------------+    +----------------+
|                |    |                |    |                |    |                |
| Audio Capture  +--->| Speech-to-Text +--->| Text Generation+--->| Text-to-Speech |
|  Service       |    |  Service       |    |  Service       |    |  Service       |
|                |    |                |    |                |    |                |
+-------+--------+    +----------------+    +------+---------+    +-------+--------+
        ^                                          |                      |
        |                                          |                      |
        |                                          v                      v
+-------+--------+                         +-------+--------+    +-------+--------+
|                |                         |                |    |                |
| Input Events   |                         | Database       |    | Audio Output   |
| Handler        |                         | Service        |    | Service        |
|                |                         |                |    |                |
+-------+--------+                         +-------+--------+    +----------------+
        ^                                          ^
        |                                          |
        |                                          |
+-------+--------+    +----------------+    +------+---------+
|                |    |                |    |                |
| UI Controller  |<---+ Web Server     +<---+ API Gateway    |
|                |    |                |    |                |
+----------------+    +-------+--------+    +----------------+
                              ^
                              |
                     +--------+--------+
                     |                 |
                     | Client          |
                     | (Web Browser)   |
                     |                 |
                     +-----------------+
```

### 2.8 Code Repository Structure

The Alimac software will be organized into a monorepo structure with clear separation of concerns:

```
alimac/
├── LICENSE
├── README.md
├── docs/
│   ├── architecture/
│   ├── api/
│   ├── deployment/
│   └── user_manual/
├── src/
│   ├── core/                  # Core system components
│   │   ├── audio/             # Audio capture and playback 
│   │   ├── display/           # Display and UI rendering
│   │   ├── input/             # Input device handling
│   │   ├── power/             # Power management
│   │   └── system/            # System service management
│   │
│   ├── ai/                    # AI processing pipeline
│   │   ├── stt/               # Speech-to-Text implementation
│   │   ├── nlu/               # Natural Language Understanding
│   │   ├── generation/        # Text generation model
│   │   ├── tts/               # Text-to-Speech implementation
│   │   └── pipeline/          # Pipeline orchestration
│   │
│   ├── network/               # Networking components
│   │   ├── wifi/              # WiFi and hotspot management
│   │   ├── bluetooth/         # Bluetooth connectivity 
│   │   └── security/          # Network security
│   │
│   ├── web/                   # Web interface
│   │   ├── server/            # Embedded web server
│   │   ├── api/               # RESTful API implementation
│   │   └── frontend/          # Web dashboard (HTML, CSS, JS)
│   │
│   └── utils/                 # Shared utilities
│       ├── logging/           # Logging system
│       ├── config/            # Configuration management
│       └── security/          # Security utilities
│
├── models/                    # AI model files (gitignored, downloaded during build)
│   ├── stt/
│   ├── generation/
│   └── tts/
│
├── tools/                     # Development and build tools
│   ├── build/                 # Build scripts
│   ├── deploy/                # Deployment utilities
│   ├── test/                  # Testing frameworks
│   └── benchmark/             # Performance benchmarking
│
├── scripts/                   # System scripts
│   ├── install/               # Installation scripts
│   ├── update/                # Update procedures
│   └── maintenance/           # Maintenance utilities
│
├── tests/                     # Test suite
│   ├── unit/                  # Unit tests
│   ├── integration/           # Integration tests
│   └── performance/           # Performance tests
│
└── CMakeLists.txt             # Main build configuration
```

### 2.9 Binary Protection and Anti-Reverse Engineering

To protect the Alimac software from reverse engineering while maintaining performance on resource-constrained devices, we will implement a multi-layered protection strategy:

#### 2.9.1 Binary Obfuscation Approach

1. **Code Obfuscation Techniques**
   - **Control Flow Flattening**: Transform the program flow into a state machine with dispatchers.
   - **Dead Code Insertion**: Insert non-functional code to complicate analysis.
   - **String Encryption**: Encrypt all string literals in the binary.
   - **Function Inlining**: Eliminate identifiable function boundaries.
   - **Register Reassignment**: Frequently change register allocations.

2. **Build-time Protection**
   - **Symbol Stripping**: Remove all symbolic information from compiled binaries.
   - **Link-Time Optimization (LTO)**: Merge object files for further optimization that obscures original structure.
   - **Custom Section Layouts**: Non-standard ELF/binary section layouts to confuse disassemblers.
   - **Compile with `-fvisibility=hidden`**: Limit exported symbols.

3. **Runtime Protection**
   - **Anti-Debugging Techniques**: 
     - Detect attached debuggers using `ptrace` checks
     - Monitor execution timing to detect single-stepping
     - Process tree inspection
   - **Memory Protection**:
     - Guard pages around critical memory regions
     - Runtime randomization of memory layouts (ASLR-like)
     - Self-modifying code for critical sections

4. **Model File Protection**
   - **Custom Model Format**: Store AI models in a proprietary encrypted format.
   - **Model Splitting**: Split models into fragments loaded at runtime.
   - **Just-in-Time Decryption**: Only decrypt model weights when needed, clear from memory after use.

#### 2.9.2 Secure Boot and Runtime Flow

```
    ┌────────────────┐      ┌────────────────┐      ┌────────────────┐
    │                │      │                │      │                │
    │ Secure Boot    │─────>│ Integrity      │─────>│ Binary         │
    │ Process        │      │ Verification   │      │ Decryption     │
    │                │      │                │      │                │
    └────────────────┘      └────────────────┘      └───────┬────────┘
                                                            │
                                                            │
    ┌────────────────┐      ┌────────────────┐      ┌───────▼────────┐
    │                │      │                │      │                │
    │ Runtime        │<─────│ Memory         │<─────│ Service        │
    │ Protection     │      │ Protection     │      │ Initialization │
    │                │      │                │      │                │
    └────────────────┘      └────────────────┘      └────────────────┘
```

#### 2.9.3 Encrypted Deployment Process

1. **Image Building**:
   ```
   Source Code → Compilation → Obfuscation → Binary Encryption → Signed Image
   ```

2. **Model Preparation**:
   ```
   AI Models → Quantization → Model Splitting → Encryption → Packaging
   ```

3. **Deployment Flow**:
   ```
   Encrypted Image → Hardware TEE Verification → Secure Decryption → Execution
   ```

#### 2.9.4 Key Management System

The system will employ a hierarchical key management system with the following structure:

```
Root Key (Factory-Set)
  │
  ├── Device Master Key (Unique per device)
  │     │
  │     ├── Binary Decryption Key
  │     ├── Model Decryption Key
  │     └── Configuration Decryption Key
  │
  └── Update Authorization Key
        │
        └── OTA Verification Keys
```

This key structure ensures that even if one layer is compromised, the entire system remains secure. The Root Key will be protected using hardware security features when available (TPM or secure element).

#### 2.9.5 Implementation Guidelines

1. **Toolchain Requirements**:
   - LLVM/Clang compiler toolchain with obfuscation plugins
   - Custom post-processing tools for binary hardening
   - Hardware-specific security features utilization

2. **Build Process Integration**:
   - Obfuscation steps integrated into CMake build system
   - Automated key generation and management
   - CI/CD pipeline with security verification gates

3. **Testing Strategy**:
   - Security testing against common reverse engineering tools
   - Performance impact assessment of protection measures
   - Validation of correct runtime behavior after protection

---

## 3. Hardware Design

### 3.1 Physical Form Factor

#### 3.1.1 Dimensions and Weight
- **Size**: Approximately 120mm × 70mm × 25mm (length × width × thickness)
- **Weight**: <200g including battery
- **Volume**: <210cm³

#### 3.1.2 Enclosure Design
- **Construction**:
  - Two-piece clamshell design with ultrasonic welding
  - Material: ABS plastic with soft-touch coating
  - Internal aluminum frame for structural rigidity and heat dissipation
- **Sealing**: IP54 rated for dust and splash resistance
- **Color Options**: Matte black, off-white, and sage green

#### 3.1.3 Mechanical Features
- **Button Design**:
  - Tactile feedback with 200g actuation force
  - 2mm travel distance
  - Rated for 100,000 cycles
- **Volume Control**:
  - Knurled aluminum rotary encoder
  - 24 detents per revolution
  - Integrated push button functionality
- **Display Window**:
  - Scratch-resistant acrylic lens
  - Anti-glare coating
  - Optical bonding to display for improved visibility

### 3.2 Thermal Management

#### 3.2.1 Passive Cooling System
- **Heat Dissipation Strategy**:
  - Copper heat spreader connected to internal frame
  - Strategically placed thermal vents
  - Thermal interface material between components and heat spreader
- **Thermal Budget**:
  - Maximum case temperature: 45°C at 25°C ambient
  - Maximum component temperature: 80°C
  - Thermal resistance: <5°C/W from SoC to ambient

#### 3.2.2 Active Cooling (if needed)
- **Fan Specification**:
  - 30mm × 30mm × 7mm low-profile fan
  - Airflow: 2.5 CFM
  - Noise level: <25dBA at 30cm
  - PWM speed control based on temperature
- **Thermal Throttling Logic**:
  - Progressive performance reduction at 75°C
  - Critical shutdown at 85°C

### 3.3 PCB Design

#### 3.3.1 Main Board
- **Board Size**: 100mm × 60mm, 4-layer design
- **Layer Stack**:
  - Top layer: Signal routing and components
  - Inner layer 1: Ground plane
  - Inner layer 2: Power planes
  - Bottom layer: Signal routing and components
- **Manufacturing Specifications**:
  - FR-4 substrate, 1.6mm thickness
  - 1oz copper
  - ENIG surface finish
  - Minimum trace width/spacing: 8mil/8mil
  - Minimum drill size: 0.3mm

#### 3.3.2 Component Layout
- **Zones**:
  - High-speed digital (SoC, memory)
  - Power management
  - Audio processing
  - RF section (Wi-Fi/Bluetooth)
- **EMI Considerations**:
  - Proper ground planes
  - Signal integrity for high-speed traces
  - RF shielding for wireless modules
  - Separation of analog and digital grounds

#### 3.3.3 Power Distribution
- **Power Rails**:
  - 5V input from USB-C
  - 3.7V from battery
  - 3.3V for digital logic
  - 1.8V for core components
  - 1.2V for memory
- **Protection Circuitry**:
  - Reverse polarity protection
  - Overcurrent protection
  - ESD protection on external interfaces

### 3.4 Acoustic Design

#### 3.4.1 Speaker Enclosure
- **Acoustic Chamber**:
  - Tuned port design for enhanced bass response
  - Internal volume: ~5cc
  - Damping material to reduce resonance
- **Speaker Mounting**:
  - Vibration isolation gaskets
  - Rigid mounting structure to prevent buzzing

#### 3.4.2 Microphone Placement
- **Array Configuration**:
  - Linear array of 2 MEMS microphones
  - 25mm spacing for improved directivity
  - Acoustic baffles for isolation
- **Noise Isolation**:
  - Physical isolation from internal components
  - Vibration damping material
  - Wind noise reduction mesh

### 3.5 User Interface Design

#### 3.5.1 Visual Elements
- **Display Integration**:
  - Flush-mounted with minimal bezel
  - High-contrast visibility in various lighting conditions
  - Anti-glare treatment
- **LED Indicators**:
  - Light pipe design for diffused illumination
  - Brightness control based on ambient light

#### 3.5.2 Tactile Elements
- **Button Design**:
  - Contoured shapes for blind operation
  - Distinct tactile feedback
  - Different sizes and textures for differentiation
- **Grip Texture**:
  - Ergonomic contours
  - Non-slip texture on sides

#### 3.5.3 Accessibility Considerations
- **High-contrast visual indicators**
- **Tactile differentiation of controls**
- **Haptic feedback for user actions**
- **Audio confirmations for state changes**

### 3.6 Manufacturing and Assembly

#### 3.6.1 Bill of Materials
- **Component Selection Criteria**:
  - Industrial temperature range (-20°C to 70°C)
  - Minimum 5-year availability
  - RoHS compliance
  - Standardized packaging for automated assembly
- **Supply Chain Considerations**:
  - Multiple source options for critical components
  - Lead time management strategy
  - Component lifecycle management

#### 3.6.2 Assembly Process
- **Manufacturing Flow**:
  - PCB fabrication
  - SMT assembly
  - Through-hole component insertion
  - Functional testing
  - Enclosure assembly
  - Final testing and calibration
- **Quality Control**:
  - Automated optical inspection
  - In-circuit testing
  - Functional testing at multiple stages
  - Burn-in testing for reliability

#### 3.6.3 Serviceability
- **Repair Considerations**:
  - Modular design for component replacement
  - Accessible fasteners
  - Labeled internal components
- **Upgrade Path**:
  - Potential for storage expansion
  - Software-upgradable features
  - User-replaceable battery

### 3.7 Packaging and Accessories

#### 3.7.1 Retail Packaging
- **Box Design**:
  - Compact dimensions: 150mm × 100mm × 50mm
  - Recyclable materials
  - Protective inserts
- **Accessories**:
  - USB-C charging cable (1m)
  - Quick start guide
  - Warranty information

#### 3.7.2 Optional Accessories
- **Protective case**
- **Desktop charging stand**
- **Extended battery pack**
- **Premium earphones/headset**

---

## 4. Additional Considerations

### 4.1 Regulatory Compliance
- **Certifications Required**:
  - FCC Part 15 (USA)
  - CE Mark (Europe)
  - RoHS
  - WEEE
  - UL/CSA safety
- **Testing Requirements**:
  - EMI/EMC testing
  - Safety testing
  - RF exposure evaluation
  - Battery transportation testing

### 4.2 Sustainability
- **Environmental Considerations**:
  - Energy-efficient design
  - Recyclable materials
  - Reduced packaging
  - RoHS compliance
- **Product Lifecycle**:
  - Minimum 3-year support period
  - Repair program
  - Recycling program

### 4.3 Cost Targets
- **Bill of Materials**: $60-$80 per unit at scale
- **Manufacturing Cost**: $15-$20 per unit at scale
- **Target Retail Price**: $199-$249
