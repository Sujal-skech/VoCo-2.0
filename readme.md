# VoCo

VoCo is a voice-first learning app built with SwiftUI. It combines pronunciation practice, spoken navigation, quizzes, stories, and learning courses in an accessible iOS experience.

## Features

- **Pronunciation practice:** Listen to sample sentences, record your voice, and replay your recording.
- **Hands-free navigation:** Use spoken commands to move between learning areas and control supported activities.
- **Talk with VoCo:** Ask educational questions using the AI service. The app includes a local fallback for when its backend is unavailable.
- **Learning activities:** Browse technical and non-technical courses, take quizzes, and read stories.
- **Multilingual support:** Choose from Indian and international language options for the app's speech features.
- **Native iOS services:** Uses system speech synthesis, speech recognition, audio recording, and biometric authentication APIs.

## Requirements

- macOS with Xcode installed to build and run the iOS app
- iOS 16 or later (deployment target)
- Swift 5.9 or later for Swift Package Manager builds and tests
- Python 3 for the optional AI backend

## Run the iOS app

1. Open `VoCoApp.xcodeproj` in Xcode.
2. Select the **VoCoApp** scheme and an iOS simulator or connected device.
3. Build and run the app.

The app launches with mock authentication, sample sentences, and a mock AI service. This mode is suitable for exploring the app without Firebase or a running AI backend.

To run the Swift package tests on macOS:

```sh
swift test
```

The package target contains the reusable app code and tests; the SwiftUI application entry point is built by the Xcode project.

## Optional Gemini backend

The Python backend keeps the Gemini API key on the server rather than embedding it in the iOS app. It uses only Python's standard library.

1. Obtain a Gemini API key and configure it in the backend environment. Do not commit the key or share it with the app:

   **macOS / Linux**

   ```sh
   cd backend
   export GEMINI_API_KEY="your-key"
   python3 server.py
   ```

   **Windows PowerShell**

   ```powershell
   Set-Location backend
   $env:GEMINI_API_KEY = "your-key"
   python server.py
   ```

   Alternatively, create `backend/.env` with `GEMINI_API_KEY=your-key`. Keep this file private.

2. Confirm that the server is running:

   ```sh
   curl http://localhost:8080/health
   ```

   The server defaults to port `8080`. You can change it with the `PORT` environment variable. Without a configured Gemini key, the backend still starts and uses local fallback responses.

   This lightweight server is intended for development. Before exposing it beyond a trusted development network, add appropriate transport security, authentication, and request controls.

3. Point the app's `GeminiAIService` at the server. The service currently has a development URL as its default, so set `backendURL` to the correct address for your environment. For an iOS simulator, this is usually `http://127.0.0.1:8080`; for a physical device, use the development computer's reachable LAN address.

4. The default app entry point uses mock services. To use the live service container, update the `VoCoMainAppView` construction in `App/VoCoApp.swift`:

   ```swift
   let aiService = GeminiAIService()
   aiService.backendURL = URL(string: "http://127.0.0.1:8080")!
   VoCoMainAppView(container: .live(aiService: aiService))
   ```

   Replace the URL with the backend's reachable address as needed. The live container selects the Firebase-backed authentication and sentence adapters. Firebase SDK dependencies and Firebase app initialization are not currently configured in this project; add and configure them before relying on live Firebase authentication or Firestore data. Without the Firestore SDK, sentence practice uses bundled sample sentences.

### Backend endpoints

| Method | Path | Purpose |
| --- | --- | --- |
| `GET` | `/health` | Backend health and configuration status |
| `POST` | `/api/chat` | Conversational AI response |
| `POST` | `/api/interpret_command` | Interpret a spoken navigation command |

## Project layout

| Path | Description |
| --- | --- |
| `App/` | SwiftUI application entry point and iOS configuration |
| `Views/` | SwiftUI screens, grouped by app feature |
| `ViewModels/` | State and workflows for authentication, pronunciation, quizzes, and voice commands |
| `Models/` | App data models and standard learning content |
| `Services/` | Native, mock, Firebase, and AI service implementations |
| `Tests/` | Swift package unit tests |
| `backend/` | Optional Python Gemini API proxy |

## Permissions

The app requests microphone and speech-recognition access for recording and voice features. Face ID access is used for biometric authentication. iOS displays the permission prompts when these features require access.