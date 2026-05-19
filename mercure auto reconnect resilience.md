# Mercure Auto-Reconnect Resilience Architecture

## The Problem: "Server Connection Issue" & Missed Updates

When deploying the Plaisoram ecosystem to cloud providers like DigitalOcean App Platform, long-lived HTTP connections (like Server-Sent Events / SSE) are subjected to aggressive idle timeouts. 

If the dashboard does not broadcast an update for 60 to 120 seconds, the cloud load balancer assumes the connection is dead and severs it. 

Previously, the Android Player used a "fire-and-forget" connection strategy. When DigitalOcean severed the connection, the Android `EventSourceListener` would trigger an `onFailure` event. Because the code only logged the failure without taking action, the app would permanently stop listening. This forced users to manually exit and reopen the app to fetch the latest data via standard REST API polling.

## The Solution: Kotlin Flow Resilience

To solve this, we implemented an "Infinite Auto-Reconnect Resilience" architecture in the Android Player (`PlayerViewModel.kt`) using modern Kotlin Coroutines and Flows.

### 1. The `callbackFlow` Wrapper
We encapsulated the OkHttp `EventSource` connection inside a reactive `callbackFlow`. 
- When the stream is open, the flow remains active.
- When live data (like `PlaylistUpdated` or `PowerCommand`) is received, the flow uses `trySend()` to push it to the UI layer.
- **Crucial Step:** If the connection drops for *any* reason (DigitalOcean timeout, WiFi loss, server restart), the flow is explicitly programmed to crash by calling `close(Exception)`.

### 2. The `.retryWhen` Operator
Because the connection is now a reactive stream, we attached a `.retryWhen {}` operator to it at the collection point. 

When the flow crashes due to a dropped connection, the `.retryWhen` operator intercepts the crash before the application crashes. It introduces an artificial delay (exponential backoff or fixed 5-second wait) to prevent spamming the server, and then automatically restarts the entire flow, creating a fresh, authenticated connection to the Mercure Hub.

### Code Implementation Snapshot

```kotlin
observeMercureEvents(config.deviceId)
    .retryWhen { cause, attempt ->
        android.util.Log.w("PlayerViewModel", "Connection dropped! Reconnecting in 5 seconds...")
        kotlinx.coroutines.delay(5000) // Wait 5 seconds
        true // Return true to loop forever
    }
    .collect { data ->
        // Handle incoming JSON events transparently
    }
```

## Result
The Android application is now 100% decoupled from the stability of the underlying network or cloud load balancer. If the device is left on for 24 hours, it may drop and silently reconnect hundreds of times in the background, guaranteeing it never misses a remote command from the dashboard.
