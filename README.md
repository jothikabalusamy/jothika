# A-personalized-travel-planning-and-tracking-app A personalized travel planning and tracking app would provide users with a seamless, customized experience for planning trips and tracking their journey. Here's an outline of how the app could function:

1. User Profile and Preferences
Personal Information: Allow users to create a profile with their travel preferences (budget, preferred destinations, travel style).
Travel Goals: Set specific travel goals, such as visiting a certain number of countries, exploring certain themes (e.g., adventure, history, food), or experiencing different activities.
Calendar Integration: Sync with the user’s calendar to plan trips and track holidays.
2. Travel Planning
Destination Suggestions: Based on user preferences, suggest destinations with detailed information (best time to visit, top attractions, and local experiences).
Custom Itineraries: Allow users to create itineraries with activities, accommodations, transportation options, and day-by-day plans.
Budget Planner: Include tools to set and track travel budgets, with cost breakdowns for flights, accommodation, meals, etc.
Collaborative Planning: Allow friends or travel groups to collaborate on planning trips and share itineraries.
Accommodation & Transport Booking: Integrated booking services for flights, trains, hotels, car rentals, etc.
Local Recommendations: Include personalized recommendations for restaurants, activities, and hidden gems based on preferences or past travels.
Building a personalized travel planning and tracking app involves multiple components and features, and the code required will vary depending on the platform (Android/iOS) and tools you want to use. Below is a basic structure and sample code to get started with some of the features of the app using **Flutter** (a popular framework for building cross-platform apps) and **Firebase** for backend services.

### Features Overview:
1. **User Authentication**
2. **Trip Planning and Management**
3. **Expense Tracker**
4. **Real-time Notifications**
5. **Trip Tracker**
6. **Offline Access**

### Technologies Used:
- **Flutter** for frontend (cross-platform).
- **Firebase Authentication** for user login/signup.
- **Firebase Firestore** for database storage.
- **Firebase Cloud Messaging (FCM)** for real-time notifications.

---
code


### 1. **Setting up Flutter Project**

First, create a new Flutter project:

```bash
flutter create travel_planner_app
cd travel_planner_app
```

### 2. **Add Dependencies in `pubspec.yaml`**

```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^2.18.0
  firebase_auth: ^4.3.0
  cloud_firestore: ^3.1.0
  provider: ^6.2.3
  google_sign_in: ^5.4.0
  firebase_messaging: ^13.0.3
```

Run the following command to install the dependencies:

```bash
flutter pub get
```

### 3. **Firebase Setup**

Before coding, you need to set up Firebase for your Flutter app. Visit the [Firebase Console](https://console.firebase.google.com/), create a new project, and follow the instructions to integrate Firebase with your Flutter app for **Authentication**, **Firestore**, and **FCM**.

---

### 4. **Code Implementation**

#### **a. User Authentication (Login/Signup)**

In `lib/main.dart`:

```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_auth/firebase_auth.dart';
import 'package:google_sign_in/google_sign_in.dart';
import 'home_page.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Travel Planner',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: AuthPage(),
    );
  }
}

class AuthPage extends StatelessWidget {
  final FirebaseAuth _auth = FirebaseAuth.instance;

  Future<User?> _signInWithGoogle() async {
    final GoogleSignIn googleSignIn = GoogleSignIn();
    final GoogleSignInAccount? googleUser = await googleSignIn.signIn();
    final GoogleSignInAuthentication googleAuth = await googleUser!.authentication;
    
    final AuthCredential credential = GoogleAuthProvider.credential(
      accessToken: googleAuth.accessToken,
      idToken: googleAuth.idToken,
    );

    final UserCredential userCredential = await _auth.signInWithCredential(credential);
    return userCredential.user;
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Travel Planner - Login"),
      ),
      body: Center(
        child: ElevatedButton(
          onPressed: () async {
            User? user = await _signInWithGoogle();
            if (user != null) {
              Navigator.pushReplacement(
                context,
                MaterialPageRoute(builder: (context) => HomePage(user: user)),
              );
            }
          },
          child: Text("Sign in with Google"),
        ),
      ),
    );
  }
}
```

#### **b. Home Page and Firestore Integration**

`lib/home_page.dart`:

```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:flutter/material.dart';
import 'package:firebase_auth/firebase_auth.dart';

class HomePage extends StatefulWidget {
  final User user;
  HomePage({required this.user});

  @override
  _HomePageState createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  final FirebaseAuth _auth = FirebaseAuth.instance;
  final FirebaseFirestore _firestore = FirebaseFirestore.instance;
  String _tripTitle = "";

  void _addTrip() async {
    if (_tripTitle.isNotEmpty) {
      await _firestore.collection('trips').add({
        'userId': widget.user.uid,
        'title': _tripTitle,
        'date': Timestamp.now(),
      });
      setState(() {
        _tripTitle = "";
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Travel Planner"),
        actions: [
          IconButton(
            icon: Icon(Icons.exit_to_app),
            onPressed: () async {
              await _auth.signOut();
              Navigator.pushReplacement(
                context,
                MaterialPageRoute(builder: (context) => AuthPage()),
              );
            },
          ),
        ],
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          children: [
            TextField(
              onChanged: (value) {
                setState(() {
                  _tripTitle = value;
                });
              },
              decoration: InputDecoration(
                labelText: 'Enter Trip Title',
                border: OutlineInputBorder(),
              ),
            ),
            SizedBox(height: 10),
            ElevatedButton(
              onPressed: _addTrip,
              child: Text("Add Trip"),
            ),
            SizedBox(height: 20),
            Expanded(
              child: StreamBuilder<QuerySnapshot>(
                stream: _firestore.collection('trips').where('userId', isEqualTo: widget.user.uid).snapshots(),
                builder: (context, snapshot) {
                  if (!snapshot.hasData) {
                    return Center(child: CircularProgressIndicator());
                  }

                  var trips = snapshot.data!.docs;
                  return ListView.builder(
                    itemCount: trips.length,
                    itemBuilder: (context, index) {
                      var trip = trips[index];
                      return ListTile(
                        title: Text(trip['title']),
                        subtitle: Text("Added on ${trip['date'].toDate()}"),
                      );
                    },

Output:


--------------------------------------
|      Travel Planner               |
|------------------------------------|
|  [ Text Field: Enter Trip Title ]  |
|  [ Add Trip Button ]               |
--------------------------------------
|   Trip 1: "Paris Trip"            |
|   Added on: 2024-11-22            |
|------------------------------------|
|   Trip 2: "Italy Vacation"        |
|   Added on: 2024-10-15            |
|------------------------------------|
|   Trip 3: "Japan Adventure"       |
|   Added on: 2024-09-05            |
--------------------------------------


A personalized travel planning and tracking app

presented by


JOTHIKA. B ( Head) 


KABILAN. G


KABILAN. R


YUVARAJ. M
