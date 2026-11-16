# QNX 8.0 Real-Time Industrial Machine Monitoring and Fault Notification System

## Project Overview

This project implements a real-time industrial machine monitoring and fault notification prototype using QNX Neutrino RTOS 8.0 and C.

The system follows a client-server architecture in which three independent sensor-client processes simulate temperature, electrical current and vibration measurements. A central monitoring server receives telemetry requests, validates message integrity, manages a bounded shared-memory queue and processes sensor records using periodic timer events.

Threshold violations are classified as faults, recorded with timestamps and communicated through sensor-specific fault reports with client acknowledgement.

## System Architecture

The application consists of:

- Three independent sensor-client processes.
- A central QNX monitoring server.
- Named message-passing communication.
- A bounded shared-memory queue.
- A timer-driven worker thread.
- Sensor-specific fault detection and notification.

## Technologies Used

| Category | Technology |
|---|---|
| Programming Language | C |
| Operating System | QNX Neutrino RTOS 8.0 |
| Development Environment | QNX Momentics IDE |
| IPC | QNX Message Passing |
| Vectored I/O | MsgSendv() |
| Shared Memory | POSIX shm_open(), mmap() |
| Synchronization | pthread mutex and condition variable |
| Multithreading | POSIX Threads |
| Timer Management | POSIX Timers |
| Event Notification | QNX Pulses |

## Key Implementation Features

### 1. Concurrent Sensor Clients

Implemented three independent sensor-client processes representing:

- Temperature monitoring
- Electrical current monitoring
- Vibration monitoring

Each client generates 30 deterministic telemetry samples, resulting in 90 records per complete test run.

### 2. QNX Message-Passing IPC

Implemented named server communication using QNX message-passing APIs.

- Server registration using name_attach().
- Client connection using name_open().
- Request and response communication.
- Vectored message transfer using MsgSendv().
- Separate header and payload buffers.

Each request transfers a 65,536-byte payload using two I/O vectors.

### 3. Shared-Memory Queue Management

Implemented a bounded 16-slot shared-memory queue using:

- shm_open()
- ftruncate()
- mmap()
- SLOT_FREE and SLOT_READY states

The server manages queue insertion, record selection and removal while protecting shared queue metadata.

### 4. Process Synchronization and Multithreading

Implemented process-shared pthread synchronization using:

- pthread mutexes
- pthread condition variables
- PTHREAD_PROCESS_SHARED attributes
- A dedicated server worker thread

These mechanisms coordinate queue updates and timer-driven record processing.

### 5. Timer-Driven Processing and Pulse Notifications

Configured a periodic POSIX timer with a 100 ms interval.

Timer events generate QNX pulses to trigger server-side processing. After processing a notification, the server sends a completion pulse to the originating client.

### 6. Fault Detection and Reporting

Implemented threshold-based fault classification for three simulated sensor types.

| Sensor | Fault Threshold |
|---|---|
| Temperature | ≥ 80 °C |
| Current | > 12 A |
| Vibration | ≥ 4.5 RMS units |

The system records fault events with sensor identity, sample sequence, measured value and timestamps.

Sensor-specific fault reports are delivered to clients, with acknowledgements checked by the server.

## System Configuration

| Parameter | Configuration |
|---|---|
| Sensor Clients | 3 |
| Samples per Client | 30 |
| Total Telemetry Records | 90 |
| Shared-Memory Queue Capacity | 16 slots |
| Payload Size | 65,536 bytes |
| Timer Interval | 100 ms |
| Worker Threads | 1 |

## Test Results

The final evaluated software test demonstrated:

| Performance Metric | Observed Result |
|---|---:|
| Temperature Samples | 30 |
| Current Samples | 30 |
| Vibration Samples | 30 |
| Total Records Processed | 90/90 |
| Temperature Faults | 6 |
| Current Faults | 9 |
| Vibration Faults | 9 |
| Total Fault Events | 24 |
| Completion Notifications | 90 |
| Fault Reports Received | 3/3 |

**Final Machine Status:** MOTOR_INSPECTION_REQUIRED

All 90 simulated readings were processed, and all three sensor-specific fault reports were successfully delivered and acknowledged.

## Project Limitations

- Sensor measurements are generated using deterministic software data rather than physical sensors.
- The evaluated version does not include hardware sensor interfaces.
- Long-duration reliability and worst-case response-time testing were not performed.
- Queue saturation and malformed-message fault-injection testing were not demonstrated in the reported successful run.

## Future Enhancements

- Integrate physical temperature, current and vibration sensors.
- Add configurable CSV-based telemetry input.
- Implement queue saturation and recovery testing.
- Evaluate processing latency, timer jitter and throughput.
- Develop a live monitoring dashboard connected to the QNX server.
- Improve fault recovery and controlled shutdown mechanisms.

## Contributors

- Athulya Sakthi M
- Visham Kumar V
- Lakshana S

Department of Electrical and Electronics Engineering  
Amrita Vishwa Vidyapeetham, Coimbatore, India
