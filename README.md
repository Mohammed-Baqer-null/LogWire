# LogWire

A lightweight Android logcat reader and forwarder library that automatically captures and centralizes logs from multiple applications.

## What is LogWire?

LogWire provides a simple way to collect logs from different Android apps and view them in a centralized logger application. Instead of using ADB or checking logs separately for each app, LogWire automatically forwards all logs to a receiver service where you can monitor them in real-time.

### Features:
- Automatic initialization via ContentProvider
- Process-specific log filtering
- AIDL-based inter-process communication
- Thread-safe operations
- Real-time log monitoring

## How It Works

LogWire operates with a **two-part architecture**:

1. **Target Application (Log Producer)**: The LogWire library must be integrated into the application(s) you want to monitor. It automatically captures logs from these apps and forwards them via IPC.

2. **Logger Application (Log Receiver)**: A centralized application that receives and displays logs from all target applications in real-time using `LogReceiverService`.

### Integration Requirements

- **Target Apps**: Include the LogWire library as a dependency. The library will automatically initialize and forward logs to the logger application.
- **Logger App**: Implement `LogReceiverService` to receive and display logs from all connected target applications.

## Important: Configuration for Custom Projects

If you're integrating LogWire into a project that is **not** the Android Code Studio IDE source, you must update the following configurations:

### 1. Update Package Names

In `LWProperties.kt`, line 41:
```kotlin
const val PACKAGE_NAME: String = "com.your.logger.package" // Change this to YOUR logger app package name
```

This package name should match the package of your **logger application** (the app that receives and displays logs).

### 2. Implementation Example (Logger Application)

LogWire provides a ready-to-use service `LogReceiverService.kt` that can be easily integrated into your logger application's activities or fragments.

#### Basic Usage:
```kotlin
private var logService: LogReceiverService? = null
private var isBound = false

private val logListener = object : LogReceiverService.LogListener {
    override fun onLogReceived(log: LogEntry) {
        // Handle received log entry
        val formattedLog = "${log.getFormattedTime()} [${log.level.label}] ${log.tag}: ${log.message}"
        // Display in TextView, append to editor, etc.
    }
}

private val serviceConnection = object : ServiceConnection {
    override fun onServiceConnected(name: ComponentName?, service: IBinder?) {
        logService = LogReceiverService.getInstance()
        isBound = true
        logService?.addListener(logListener)
    }
    
    override fun onServiceDisconnected(name: ComponentName?) {
        logService?.removeListener(logListener)
        logService = null
        isBound = false
    }
}

// Start and bind to the service
val intent = Intent(requireContext(), LogReceiverService::class.java)
requireContext().startService(intent)
requireContext().bindService(intent, serviceConnection, Context.BIND_AUTO_CREATE)

// Don't forget to unbind when done
override fun onDestroy() {
    super.onDestroy()
    if (isBound) {
        logService?.removeListener(logListener)
        requireContext().unbindService(serviceConnection)
        isBound = false
    }
}
```

## Requirements

- USB Debugging enabled on the device
- LogWire library integrated into target applications
- Logger application with `LogReceiverService` implementation

## License

LogWire is free software licensed under the GNU Lesser General Public License v3.0 or later.

## Author

Created by Mohammed-baqer-null  
GitHub: https://github.com/Mohammed-baqer-null
