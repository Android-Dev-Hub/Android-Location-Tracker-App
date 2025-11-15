# Android Location Tracker App

A simple Android app that tracks device location and logs or uploads it. This README gives setup steps, required permissions, example code snippets, and notes about privacy and background location.

---

## Features

* Get current location (latitude, longitude).
* Continuous location updates (foreground / background).
* Option to save location locally or send to server (example included).
* Simple UI showing coordinates and start/stop tracking buttons.

---

## Requirements

* Android Studio (4.0+)
* Android SDK (API 23+/Android 6.0+) — background location needs API 29+ handling.
* Kotlin (recommended) or Java

---

## Important permissions

Add these to `AndroidManifest.xml` (runtime request is required for dangerous permissions):

```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<!-- If you want background location updates (API 29+) -->
<uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
<uses-permission android:name="android.permission.INTERNET" />
```

---

## Gradle dependencies

In `build.gradle (app)` add:

```gradle
implementation "com.google.android.gms:play-services-location:21.0.1" // check latest
implementation "androidx.core:core-ktx:1.9.0"
implementation "androidx.appcompat:appcompat:1.6.1"
implementation "com.google.android.material:material:1.8.0"
```

---

## Basic app flow

1. Ask runtime permission for location.
2. Use FusedLocationProviderClient to request location updates.
3. Show location in UI and optionally send to server.
4. Stop updates when user stops tracking.

---

## Example: `MainActivity.kt` (Kotlin)

```kotlin
package com.example.locationtracker

import android.Manifest
import android.content.pm.PackageManager
import android.os.Bundle
import android.widget.Button
import android.widget.TextView
import androidx.activity.result.contract.ActivityResultContracts
import androidx.appcompat.app.AppCompatActivity
import androidx.core.app.ActivityCompat
import com.google.android.gms.location.*

class MainActivity : AppCompatActivity() {
    private lateinit var fusedClient: FusedLocationProviderClient
    private lateinit var locCallback: LocationCallback
    private lateinit var locRequest: LocationRequest

    private lateinit var tvLatLng: TextView
    private lateinit var btnStart: Button
    private lateinit var btnStop: Button

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        tvLatLng = findViewById(R.id.tvLatLng)
        btnStart = findViewById(R.id.btnStart)
        btnStop = findViewById(R.id.btnStop)

        fusedClient = LocationServices.getFusedLocationProviderClient(this)

        locRequest = LocationRequest.Builder(Priority.PRIORITY_HIGH_ACCURACY, 5000L)
            .setMinUpdateDistanceMeters(5f)
            .build()

        locCallback = object : LocationCallback() {
            override fun onLocationResult(result: LocationResult) {
                val loc = result.lastLocation ?: return
                val text = "Lat: ${loc.latitude}, Lon: ${loc.longitude}"
                tvLatLng.text = text
                // TODO: Save or send this location to server here
            }
        }

        btnStart.setOnClickListener { startTracking() }
        btnStop.setOnClickListener { stopTracking() }

        // Request runtime permissions
        requestPermissionLauncher.launch(Manifest.permission.ACCESS_FINE_LOCATION)
    }

    private val requestPermissionLauncher = registerForActivityResult(
        ActivityResultContracts.RequestPermission()
    ) { isGranted: Boolean ->
        if (!isGranted) {
            tvLatLng.text = "Location permission denied"
        }
    }

    private fun startTracking() {
        if (ActivityCompat.checkSelfPermission(
                this,
                Manifest.permission.ACCESS_FINE_LOCATION
            ) != PackageManager.PERMISSION_GRANTED
        ) {
            requestPermissionLauncher.launch(Manifest.permission.ACCESS_FINE_LOCATION)
            return
        }
        fusedClient.requestLocationUpdates(locRequest, locCallback, mainLooper)
        tvLatLng.text = "Tracking started..."
    }

    private fun stopTracking() {
        fusedClient.removeLocationUpdates(locCallback)
        tvLatLng.text = "Tracking stopped"
    }

    override fun onDestroy() {
        super.onDestroy()
        fusedClient.removeLocationUpdates(locCallback)
    }
}
```

---

## AndroidManifest (important parts)

```xml
<application
    android:label="LocationTracker"
    android:icon="@mipmap/ic_launcher">

    <activity android:name=".MainActivity">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>
    </activity>

    <!-- If you implement a foreground service for background tracking: -->
    <!--
    <service
        android:name=".LocationUpdatesService"
        android:foregroundServiceType="location" />
    -->

</application>
```

---

## Background tracking notes

* Android limits background location access. For continuous tracking use a **foreground service** (show a persistent notification) and request `ACCESS_BACKGROUND_LOCATION` for API 29+.
* Follow Play Store policies and explain to users why you need background location.

---

## Example: Send location to server (simple POST)

```kotlin
// Use your networking library (OkHttp, Retrofit). Simple pseudo-code:
fun sendLocation(lat: Double, lon: Double) {
    // Build JSON and POST to your server endpoint
}
```

---

## UI layout example (`activity_main.xml`)

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:padding="16dp"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:id="@+id/tvLatLng"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Lat, Lon" />

    <Button
        android:id="@+id/btnStart"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Start" />

    <Button
        android:id="@+id/btnStop"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Stop" />
</LinearLayout>
```

---

## Privacy

* Always inform users that you collect location data.
* Provide a clear reason and an easy way to stop tracking.
* Do not collect or send personally-identifying data unless you have user consent.

---

## Troubleshooting

* If you get no location: check device location is ON, app has permission, and Google Play Services is available.
* If background updates stop: implement foreground service and check battery optimization settings.

---

## License

This project is provided as example code. Use as you like but respect user privacy.

---

If you want, I can add a full `LocationUpdatesService` example, or Java versions of the code.
