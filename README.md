# MediConnect - Healthcare Appointment Management App

Mediconnect explores the use of large language models in healthcare applications, focusing on conversational AI for medical assistance and appointment scheduling. The project studies how AI-driven interaction can improve usability and accessibility in healthcare systems.
A comprehensive Flutter application for healthcare management, featuring hospital search, appointment booking, AI-powered chatbot assistance, and user profile management.

## 📱 App Overview

MediConnect is a full-featured healthcare app that connects patients with healthcare providers. The app includes:

- **Hospital Search & Discovery**: Find nearby hospitals with detailed information
- **Appointment Booking**: Schedule appointments with healthcare providers
- **AI Chatbot**: Get medical assistance and information via Gemini AI
- **User Profile Management**: Manage personal health information and appointment history
- **Location-based Services**: GPS integration for finding nearby medical facilities

## 🛠 Technology Stack

- **Frontend**: Flutter (Dart)
- **Backend**: Firebase (Firestore, Authentication)
- **AI Integration**: Google Gemini AI API
- **State Management**: Provider Pattern
- **Location Services**: Geolocator plugin
- **Maps Integration**: Google Maps
- **UI Components**: Material Design 3

## 🚀 Development Journey

### Phase 1: Project Setup & Environment Configuration

#### Initial Flutter Project Creation
```bash
flutter create mediconnect
cd mediconnect
```

#### Environment Setup Challenges & Solutions

**Problem**: Windows path handling issues due to spaces in user directory
```
C:\Users\Sai Nithish Adithya\mediconnect
```

**Solution**: Configured custom Gradle home to avoid path issues
```bash
# Set environment variables
$env:JAVA_HOME="C:\Program Files\Android\Android Studio\jbr"
$env:GRADLE_USER_HOME="C:\gradle-home"
$env:GRADLE_OPTS="-Xmx2G -XX:MaxMetaspaceSize=1G"
```

**Gradle Configuration** (`android/gradle.properties`):
```properties
org.gradle.jvmargs=-Xmx2048M -Dkotlin.daemon.jvm.options="-Xmx2048M"
android.useAndroidX=true
android.enableJetifier=true
org.gradle.user.home=C:/gradle-home
```

### Phase 2: Core Dependencies & Project Structure

#### Key Dependencies (`pubspec.yaml`)
```yaml
dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.8
  firebase_core: ^3.15.2
  firebase_auth: ^5.7.0
  cloud_firestore: ^5.6.12
  provider: ^6.1.2
  geolocator: ^13.0.4
  permission_handler: ^11.4.0
  google_generative_ai: ^0.4.6
  go_router: ^14.8.1
  intl: ^0.19.0
  http: ^1.2.2
  speech_to_text: ^7.0.0
  flutter_tts: ^4.1.0
  flutter_chat_ui: ^1.6.15
  flutter_markdown: ^0.6.23

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^5.0.0
```

#### Project Structure
```
mediconnect/
├── lib/
│   ├── main.dart                    # App entry point
│   ├── models/                      # Data models
│   │   ├── hospital.dart
│   │   ├── appointment.dart
│   │   └── user_model.dart
│   ├── screens/                     # UI screens
│   │   ├── home_screen.dart
│   │   ├── new_hospitals_screen.dart
│   │   ├── hospital_details_screen.dart
│   │   ├── appointment_booking_screen.dart
│   │   ├── chatbot_screen.dart
│   │   └── profile_screen.dart
│   ├── services/                    # Business logic
│   │   ├── auth_service.dart
│   │   ├── hospital_service.dart
│   │   ├── appointment_service.dart
│   │   └── ai_chat_service.dart
│   ├── providers/                   # State management
│   │   └── user_provider.dart
│   └── widgets/                     # Reusable components
├── android/                         # Android configuration
├── ios/                             # iOS configuration
└── web/                             # Web configuration
```

### Phase 3: Firebase Integration

#### Firebase Setup Process

1. **Created Firebase Project**: `mediconnect-745ea`
2. **Downloaded Configuration Files**:
   - `android/app/google-services.json`
   - `ios/Runner/GoogleService-Info.plist`
   - `web/firebase-config.js`

3. **Firebase Initialization** (`lib/main.dart`):
```dart
import 'package:firebase_core/firebase_core.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  runApp(const MyApp());
}
```

4. **Authentication Service** (`lib/services/auth_service.dart`):
```dart
class AuthService {
  final FirebaseAuth _auth = FirebaseAuth.instance;
  final FirebaseFirestore _firestore = FirebaseFirestore.instance;

  // Sign up with email and password
  Future<UserCredential?> signUpWithEmailAndPassword(
      String email, String password, String name) async {
    try {
      UserCredential result = await _auth.createUserWithEmailAndPassword(
        email: email,
        password: password,
      );
      
      // Create user document in Firestore
      await _firestore.collection('users').doc(result.user!.uid).set({
        'name': name,
        'email': email,
        'createdAt': FieldValue.serverTimestamp(),
      });
      
      return result;
    } catch (e) {
      print('Sign up error: $e');
      return null;
    }
  }
}
```

### Phase 4: Core App Features Implementation

#### 4.1 Hospital Search & Discovery

**Hospital Model** (`lib/models/hospital.dart`):
```dart
class Hospital {
  final String id;
  final String name;
  final String address;
  final String phone;
  final String email;
  final String imageUrl;
  final double rating;
  final double distance;
  final List<String> specializations;
  final Map<String, dynamic> location;

  Hospital({
    required this.id,
    required this.name,
    required this.address,
    required this.phone,
    required this.email,
    required this.imageUrl,
    required this.rating,
    required this.distance,
    required this.specializations,
    required this.location,
  });

  factory Hospital.fromFirestore(DocumentSnapshot doc) {
    Map<String, dynamic> data = doc.data() as Map<String, dynamic>;
    return Hospital(
      id: doc.id,
      name: data['name'] ?? '',
      address: data['address'] ?? '',
      phone: data['phone'] ?? '',
      email: data['email'] ?? '',
      imageUrl: data['imageUrl'] ?? '',
      rating: (data['rating'] ?? 0.0).toDouble(),
      distance: (data['distance'] ?? 0.0).toDouble(),
      specializations: List<String>.from(data['specializations'] ?? []),
      location: data['location'] ?? {},
    );
  }
}
```

**Hospital Service** (`lib/services/hospital_service.dart`):
```dart
class HospitalService {
  final FirebaseFirestore _firestore = FirebaseFirestore.instance;

  Future<List<Hospital>> getNearbyHospitals() async {
    try {
      QuerySnapshot snapshot = await _firestore
          .collection('hospitals')
          .orderBy('rating', descending: true)
          .limit(20)
          .get();

      return snapshot.docs.map((doc) => Hospital.fromFirestore(doc)).toList();
    } catch (e) {
      print('Error fetching hospitals: $e');
      return [];
    }
  }
}
```

#### 4.2 Appointment Booking System

**Appointment Model** (`lib/models/appointment.dart`):
```dart
class Appointment {
  final String id;
  final String userId;
  final String hospitalId;
  final String hospitalName;
  final String doctorName;
  final String specialization;
  final DateTime dateTime;
  final String status;
  final String notes;

  Appointment({
    required this.id,
    required this.userId,
    required this.hospitalId,
    required this.hospitalName,
    required this.doctorName,
    required this.specialization,
    required this.dateTime,
    required this.status,
    this.notes = '',
  });
}
```

**Appointment Booking UI** (`lib/screens/appointment_booking_screen.dart`):
```dart
class AppointmentBookingScreen extends StatefulWidget {
  final Hospital hospital;
  
  const AppointmentBookingScreen({Key? key, required this.hospital}) : super(key: key);
}

// Key features:
// - Date and time selection
// - Doctor specialization selection
// - Patient details form
// - Appointment confirmation
```

### Phase 5: AI Chatbot Integration

#### 5.1 Initial OpenAI Integration (Later Migrated to Gemini)

**Initial Setup**:
```dart
// Initial OpenAI implementation
class AiChatService {
  static const String _baseUrl = 'https://api.openai.com/v1/chat/completions';
  
  Future<String> sendMessage(String message) async {
    // OpenAI API implementation
  }
}
```

#### 5.2 Migration to Google Gemini AI

**Problem**: Switched to Gemini AI for better integration and cost-effectiveness

**Gemini AI Service** (`lib/services/ai_chat_service.dart`):
```dart
import 'package:google_generative_ai/google_generative_ai.dart';

class AiChatService {
  static const String _modelName = 'gemini-pro';
  late final GenerativeModel _model;
  late final ChatSession _chatSession;

  AiChatService(String apiKey) {
    _model = GenerativeModel(
      model: _modelName,
      apiKey: apiKey,
      generationConfig: GenerationConfig(
        temperature: 0.7,
        topK: 40,
        topP: 0.95,
        maxOutputTokens: 1024,
      ),
      systemInstruction: Content.system(
        'You are MediBot, a helpful medical assistant for the MediConnect app. '
        'Provide accurate, helpful medical information while always recommending '
        'users consult with healthcare professionals for serious concerns.'
      ),
    );
    _chatSession = _model.startChat();
  }

  Future<String> sendMessage(String message) async {
    try {
      final response = await _chatSession.sendMessage(Content.text(message));
      return response.text ?? 'Sorry, I could not process your request.';
    } catch (e) {
      print('Error sending message to Gemini: $e');
      return 'Sorry, I encountered an error. Please try again.';
    }
  }
}
```

#### 5.3 Chatbot UI Implementation

**Chat Screen** (`lib/screens/chatbot_screen.dart`):
```dart
class ChatbotScreen extends StatefulWidget {
  @override
  _ChatbotScreenState createState() => _ChatbotScreenState();
}

class _ChatbotScreenState extends State<ChatbotScreen> {
  final List<types.Message> _messages = [];
  final _user = const types.User(id: 'user');
  final _bot = const types.User(id: 'bot', firstName: 'MediBot');
  late AiChatService _aiChatService;
  
  // Predefined suggested messages
  final List<String> _suggestedMessages = [
    "What are the symptoms of flu?",
    "How to maintain good health?",
    "When should I see a doctor?",
    "What is a healthy diet?",
    "How to manage stress?",
    "Exercise recommendations"
  ];

  @override
  void initState() {
    super.initState();
    _initializeAI();
    _addWelcomeMessage();
  }

  void _initializeAI() {
    const String? geminiApiKey = String.fromEnvironment('GEMINI_API_KEY');
    if (geminiApiKey == null || geminiApiKey.isEmpty) {
      _addErrorMessage('AI service not configured properly');
      return;
    }
    _aiChatService = AiChatService(geminiApiKey);
  }
}
```

### Phase 6: UI/UX Improvements & Bug Fixes

#### 6.1 Overflow Issues Resolution

**Problem**: RenderFlex overflow errors in hospital listing screens

**Solution**: Applied responsive design patterns
```dart
// Before (causing overflow)
Row(
  children: [
    Text(hospital.phone),
    Text(hospital.email),
  ],
)

// After (responsive fix)
Row(
  children: [
    Flexible(
      child: Text(
        hospital.phone,
        overflow: TextOverflow.ellipsis,
      ),
    ),
    SizedBox(width: 8),
    Flexible(
      child: Text(
        hospital.email,
        overflow: TextOverflow.ellipsis,
      ),
    ),
  ],
)
```

#### 6.2 Chatbot UI Improvements

**Problem**: Suggested messages blocking chat content

**Solution**: Integrated suggestions within scrollable area
```dart
Widget _buildSuggestedMessages() {
  return Container(
    margin: const EdgeInsets.all(8),
    padding: const EdgeInsets.all(12),
    decoration: BoxDecoration(
      color: Colors.grey[100],
      borderRadius: BorderRadius.circular(12),
      border: Border.all(color: Colors.grey[300]!),
    ),
    child: Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text(
          'Suggested Questions:',
          style: TextStyle(
            fontWeight: FontWeight.bold,
            color: Colors.grey[700],
          ),
        ),
        SizedBox(height: 8),
        Wrap(
          spacing: 8,
          runSpacing: 4,
          children: _suggestedMessages.take(6).map((message) {
            return _buildSuggestionChip(message);
          }).toList(),
        ),
      ],
    ),
  );
}
```

### Phase 7: API Key Management & Security

#### 7.1 Environment Variables Setup

**Build Configuration**:
```bash
# Development run command
flutter run --dart-define=AI_PROVIDER=gemini --dart-define="GEMINI_API_KEY=YOUR_API_KEY"
```

**PowerShell Script** (`run_app.ps1`):
```powershell
# Set environment variables for development
$env:JAVA_HOME="C:\Program Files\Android\Android Studio\jbr"
$env:GRADLE_USER_HOME="C:\gradle-home"
$env:GRADLE_OPTS="-Xmx2G -XX:MaxMetaspaceSize=1G"


```

#### 7.2 API Key Retrieval in Code

```dart
class _ChatbotScreenState extends State<ChatbotScreen> {
  void _initializeAI() {
    // Retrieve API key from environment
    const String? geminiApiKey = String.fromEnvironment('GEMINI_API_KEY');
    
    if (geminiApiKey == null || geminiApiKey.isEmpty) {
      _addErrorMessage('AI service not configured properly');
      return;
    }
    
    _aiChatService = AiChatService(geminiApiKey);
  }
}
```

### Phase 8: Navigation & Routing

#### 8.1 Router Configuration

**App Router** (`lib/main.dart`):
```dart
final GoRouter _router = GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/hospitals',
      builder: (context, state) => const NewHospitalsScreen(),
    ),
    GoRoute(
      path: '/hospital-details',
      builder: (context, state) {
        final hospital = state.extra as Hospital;
        return HospitalDetailsScreen(hospital: hospital);
      },
    ),
    GoRoute(
      path: '/book-appointment',
      builder: (context, state) {
        final hospital = state.extra as Hospital;
        return AppointmentBookingScreen(hospital: hospital);
      },
    ),
    GoRoute(
      path: '/chatbot',
      builder: (context, state) => const ChatbotScreen(),
    ),
    GoRoute(
      path: '/profile',
      builder: (context, state) => const ProfileScreen(),
    ),
  ],
);
```

#### 8.2 Bottom Navigation

**Main Navigation** (`lib/screens/home_screen.dart`):
```dart
class HomeScreen extends StatefulWidget {
  @override
  _HomeScreenState createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {
  int _currentIndex = 0;
  
  final List<Widget> _pages = [
    HomePage(),
    NewHospitalsScreen(),
    ChatbotScreen(),
    ProfileScreen(),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: _pages[_currentIndex],
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _currentIndex,
        onTap: (index) => setState(() => _currentIndex = index),
        type: BottomNavigationBarType.fixed,
        items: const [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Home'),
          BottomNavigationBarItem(icon: Icon(Icons.local_hospital), label: 'Hospitals'),
          BottomNavigationBarItem(icon: Icon(Icons.chat), label: 'AI Chat'),
          BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Profile'),
        ],
      ),
    );
  }
}
```

## 🔧 Build & Deployment

### Development Environment Setup

1. **Install Flutter SDK**
2. **Setup Android Studio** with Android SDK
3. **Configure Environment Variables**:
   ```bash
   $env:JAVA_HOME="C:\Program Files\Android\Android Studio\jbr"
   $env:GRADLE_USER_HOME="C:\gradle-home"
   $env:GRADLE_OPTS="-Xmx2G -XX:MaxMetaspaceSize=1G"
   ```

### Running the Application

#### Development Mode
```bash
# Basic run
flutter run

# With API configuration
flutter run --dart-define=AI_PROVIDER=gemini --dart-define="GEMINI_API_KEY=YOUR_API_KEY"

# Release build
flutter build apk --release --dart-define=AI_PROVIDER=gemini --dart-define="GEMINI_API_KEY=YOUR_API_KEY"
```

#### Using PowerShell Script
```bash
./run_app.ps1
```

## 🛠 Troubleshooting & Solutions

### Common Issues Encountered

#### 1. Windows Path Issues
**Problem**: Gradle build failures due to spaces in Windows user directory
**Solution**: Set custom `GRADLE_USER_HOME` to path without spaces

#### 2. Speech-to-Text Plugin Issues
**Problem**: Compilation errors with `speech_to_text: ^6.6.2`
**Solution**: Updated to stable version `^7.0.0`

#### 3. UI Overflow Errors
**Problem**: RenderFlex overflow in hospital listings
**Solution**: Used `Flexible` widgets with `TextOverflow.ellipsis`

#### 4. Memory Leaks in State Management
**Problem**: `setState()` called after dispose
**Solution**: Check `mounted` property before calling `setState()`

```dart
void _someAsyncFunction() async {
  // ... async operations
  if (mounted) {
    setState(() {
      // Update state
    });
  }
}
```

#### 5. API Integration Challenges
**Problem**: OpenAI API integration complexity
**Solution**: Migrated to Google Gemini AI for simpler integration

## 📱 Features Overview

### Core Features
- ✅ User Authentication (Firebase Auth)
- ✅ Hospital Search & Discovery
- ✅ Appointment Booking System
- ✅ AI-Powered Medical Chatbot (Gemini AI)
- ✅ User Profile Management
- ✅ Location-based Hospital Search
- ✅ Real-time Chat Interface
- ✅ Responsive UI Design

### Technical Features
- ✅ State Management with Provider
- ✅ Firebase Firestore Integration
- ✅ Environment-based Configuration
- ✅ Navigation with GoRouter
- ✅ Error Handling & User Feedback
- ✅ Material Design 3 UI
- ✅ Cross-platform Compatibility

## 🔮 Future Enhancements

### Planned Features
- [ ] Push Notifications for Appointments
- [ ] Telemedicine Video Consultation
- [ ] Health Records Integration
- [ ] Payment Gateway Integration
- [ ] Multi-language Support
- [ ] Offline Mode Support
- [ ] Advanced Search Filters
- [ ] Doctor Reviews & Ratings
- [ ] Health Reminders & Alerts
- [ ] Integration with Wearable Devices

### Technical Improvements
- [ ] Unit & Integration Testing
- [ ] CI/CD Pipeline Setup
- [ ] Performance Optimization
- [ ] Accessibility Improvements
- [ ] Advanced State Management (Bloc/Riverpod)
- [ ] Microservices Architecture
- [ ] Advanced Analytics Integration

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 👨‍💻 Developer

**Sai Nithish Adithya** **& Sharanya**
- Environment: Windows with PowerShell
- Development Period: 2024-2025
- Flutter Version: Latest Stable

## 🙏 Acknowledgments

- Flutter Team for the excellent framework
- Firebase Team for backend services
- Google AI Team for Gemini API
- Open source community for various plugins and packages

---

*This README documents the complete journey of building MediConnect from initial setup to a fully functional healthcare management application with AI integration.*
