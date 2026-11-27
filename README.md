Create a complete, production-ready native Android application in Kotlin using Jetpack Compose for automatically filtering and fast-accepting ride orders from Rapido Captain (`com.rapido.rider`) and Uber Driver (`com.ubercab.driver`) apps using an Android AccessibilityService.

### Core Functional Requirements:

1. **Super-Fast Auto-Accept Engine (AccessibilityService):**
   - Monitor on-screen UI nodes (`AccessibilityEvent.TYPE_WINDOW_STATE_CHANGED` and `TYPE_WINDOW_CONTENT_CHANGED`) in real time with zero artificial delay so incoming order cards are parsed within milliseconds.
   - Extract key ride details dynamically from the screen text nodes:
     - **Platform:** Rapido Captain or Uber Driver.
     - **Ride Fare / Amount (₹):** Parse numeric currency values (e.g., `₹145`, `₹ 85`).
     - **Pickup Distance (km):** Parse pickup distance (e.g., `1.2 km`, `800 m` converted to `0.8 km`).
     - **Drop Distance (km):** Parse trip/drop distance (e.g., `6.5 km`).
     - **Pickup & Drop Area Names:** Extract location/address strings shown on the order card.
   - **Instant Click Execution:** As soon as all active filter rules match, immediately locate the `"Accept"` button node (by text/contentDescription/viewId) and trigger `performAction(AccessibilityNodeInfo.ACTION_CLICK)`—with a fallback gesture dispatch (`dispatchGesture`) at the button's exact screen bounding box (`getBoundsInScreen`) if standard node click is blocked.
   - Include a Master Toggle (Start/Stop Radar) and a floating/persistent foreground notification showing active status.

2. **Manual Filter Configuration Screen (Saved via DataStore):**
   - **Minimum Fare (₹):** Input field / slider to set minimum acceptable order amount (e.g., `₹50`).
   - **Maximum Pickup Distance (km):** Input field to set max pickup radius (e.g., `2.5 km`).
   - **Drop Distance Range (km):** Min and Max drop distance inputs (e.g., `1.0 km` to `25.0 km`).
   - **Preferred Area / Zone Whitelist:** Allow the user to add, edit, toggle, or delete custom area keywords (pre-populate defaults with: `Barajaguli`, `Jaguli`, `Haringhata`, `Kalyani`, `Kanchrapara`, `Chakdaha`). Include an option to match keywords against either Pickup Location, Drop Location, or Both (case-insensitive substring matching).
   - **Platform Toggles:** Separate switches to enable/disable auto-accept for Rapido Captain and Uber Driver independently.

3. **Order History & Analytics Log (Room Database):**
   - Automatically log every incoming ride order detected by the service into a local Room database.
   - Store: Timestamp, Platform (Rapido/Uber), Fare Amount (₹), Pickup Distance, Drop Distance, Pickup/Drop Area text, and Status (`ACCEPTED` or `SKIPPED` along with the exact skip reason, e.g., `"Pickup > 2.5 km"` or `"Area mismatch"`).
   - **History UI:** Show summary counters at the top (`Total Scanned`, `Accepted`, `Skipped`, and `Total Accepted Earnings ₹`), filter tabs (`All`, `Accepted`, `Skipped`), and a button to clear history.

4. **Setup, Permissions & Built-in Simulator Tab:**
   - Provide one-tap status cards and direct intent buttons to enable **Accessibility Service** (`Settings.ACTION_ACCESSIBILITY_SETTINGS`) and **Battery Optimization Exemption** so the service is never killed in the background.
   - Include a **Built-in Test Simulator** where the user can input sample fare, pickup km, drop km, and area name to test whether their current filter rules trigger an `ACCEPTED` or `SKIPPED` result without needing a live ride.

### Technical & Project Architecture:
- Use **Kotlin**, **Jetpack Compose (Material 3)**, **Room Database**, **Jetpack DataStore Preferences**, and **Kotlin Coroutines/Flow**.
- Include the complete `AndroidManifest.xml`, `accessibility_service_config.xml` (configured with `canRetrieveWindowContent="true"` and `canPerformGestures="true"`), and clean, error-free Gradle Kotlin DSL files (`build.gradle.kts`, `settings.gradle.kts`, `gradle/libs.versions.toml`) targeting SDK 34/35 and JDK 17 so the project compiles cleanly on GitHub Actions (`gradle assembleDebug`) without missing dependencies or version conflicts.
- Support both English and Bengali UI labels toggleable from the top app bar.
