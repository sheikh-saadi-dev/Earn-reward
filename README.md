# Earn Reward App (Flutter)

ভিডিও দেখে পয়েন্ট আয় করুন, ডেইলি রিওয়ার্ড সংগ্রহ করুন, এবং bKash/Nagad-এ উত্তোলন করুন —
উত্তোলনের প্রতিটি অনুরোধ সরাসরি আপনার Google Sheet-এ সেভ হয়।

## যা যা আছে
- **Splash → Login (Google + Phone OTP) → Home**
- **Home** — Rewarded video ad দেখে পয়েন্ট আয়
- **Daily Reward** — ৭ দিনের স্ট্রিক বোনাস
- **Withdraw** — bKash/Nagad ফর্ম, সাবমিট হলে Google Sheet-এ সেভ হয়
- **Withdraw History** — অতীতের সব অনুরোধ ও status
- **Profile** — ইউজার তথ্য, স্ট্যাটস, লগ আউট

---

## ধাপ ১ — প্রজেক্ট রেডি করুন

এই ফোল্ডারে শুধু `lib/`, `pubspec.yaml` এবং `apps_script/` আছে (Android/iOS নেটিভ
ফোল্ডার নেই)। প্রথমে একটা নতুন Flutter প্রজেক্ট বানিয়ে এই ফাইলগুলো কপি করুন:

```bash
flutter create earn_reward_app
cd earn_reward_app
# এখন এই প্রজেক্টের lib/ ফোল্ডার ও pubspec.yaml দিয়ে replace করুন
flutter pub get
```

---

## ধাপ ২ — Firebase সেটআপ (Google + Phone লগইনের জন্য)

1. [Firebase Console](https://console.firebase.google.com) এ একটা নতুন প্রজেক্ট বানান।
2. Authentication → Sign-in method থেকে **Google** এবং **Phone** দুটোই enable করুন।
3. FlutterFire CLI দিয়ে কনফিগার করুন (সবচেয়ে সহজ পদ্ধতি):
   ```bash
   dart pub global activate flutterfire_cli
   flutterfire configure
   ```
   এটা স্বয়ংক্রিয়ভাবে `google-services.json` (Android), `GoogleService-Info.plist` (iOS)
   এবং `firebase_options.dart` বসিয়ে দেবে।
4. Android-এ SHA-1 ফিঙ্গারপ্রিন্ট Firebase Console-এ যোগ করতে হবে (Google Sign-In এর জন্য বাধ্যতামূলক):
   ```bash
   cd android && ./gradlew signingReport
   ```
   পাওয়া SHA-1 টা Firebase Console → Project Settings → Your apps এ যোগ করুন।

---

## ধাপ ৩ — AdMob সেটআপ (আপনার আগে থেকেই অ্যাকাউন্ট আছে)

1. AdMob Console-এ গিয়ে একটা **Rewarded** ad unit বানান (Android ও iOS আলাদা করে)।
2. `lib/services/ad_service.dart` ফাইলে গিয়ে টেস্ট আইডির জায়গায় আপনার real Ad Unit ID বসান:
   ```dart
   // TEST ID — replace with your real Android rewarded ad unit id
   return 'ca-app-pub-3940256099942544/5224354917';
   ```
3. `android/app/src/main/AndroidManifest.xml` এর `<application>` ট্যাগের ভেতরে আপনার
   AdMob **App ID** যোগ করুন:
   ```xml
   <meta-data
       android:name="com.google.android.gms.ads.APPLICATION_ID"
       android:value="ca-app-pub-XXXXXXXXXXXXXXXX~YYYYYYYYYY"/>
   ```
4. iOS-এর জন্য `ios/Runner/Info.plist` এ একই App ID `GADApplicationIdentifier` কী হিসেবে যোগ করুন।

> ⚠️ টেস্ট আইডি দিয়ে প্লে স্টোরে পাবলিশ করা যাবে না — রিলিজের আগে অবশ্যই real ID বসাতে হবে।

---

## ধাপ ৪ — Google Sheets-এ Withdraw ডেটা সেভ করা

কোনো পেইড ব্যাকএন্ড ছাড়াই, Google Apps Script দিয়ে সরাসরি Sheet-এ ডেটা যাবে:

1. একটা নতুন Google Sheet বানান (নাম দিন যেমন "Withdraw Requests")।
2. Sheet-এ **Extensions → Apps Script** এ যান।
3. `apps_script/Code.gs` ফাইলের পুরো কোড কপি করে পেস্ট করুন।
4. উপরের ডানদিকে **Deploy → New deployment** ক্লিক করুন।
   - Type: **Web app**
   - Execute as: **Me**
   - Who has access: **Anyone**
5. Deploy করার পর যে URL পাবেন সেটা কপি করুন।
6. `lib/services/sheets_service.dart` ফাইলে গিয়ে বসান:
   ```dart
   static const String _webhookUrl = 'আপনার_deployment_URL_এখানে';
   ```

এখন থেকে প্রতিটি withdraw request "Withdraws" নামের একটা শীটে অটোমেটিক সারি হিসেবে যোগ হবে
(Request ID, User, Method, Account Number, Points, Amount, Date, Status)।
Status কলাম ম্যানুয়ালি "Pending" → "Approved" এ পরিবর্তন করে আপনি নিজে ট্র্যাক করতে পারবেন।

---

## ধাপ ৫ — পয়েন্ট ও উত্তোলনের নিয়ম বদলাতে চাইলে

`lib/services/points_service.dart` ফাইলের উপরের দিকের কনস্ট্যান্টগুলো বদলান:

```dart
static const int pointsPerAd = 20;        // প্রতি ভিডিওতে পয়েন্ট
static const int maxAdsPerDay = 15;       // দৈনিক সর্বোচ্চ ভিডিও
static const int pointsPerBDT = 1000;     // ১০০০ পয়েন্ট = ১ টাকা
static const int minWithdrawPoints = 50000; // ন্যূনতম উত্তোলন = ৫০ টাকা
```

---

## ধাপ ৬ — রান করুন

```bash
flutter pub get
flutter run
```

## গুরুত্বপূর্ণ নোট
- বর্তমানে পয়েন্ট **স্থানীয়ভাবে (on-device)** সেভ হয় (`SharedPreferences`)। কেউ অ্যাপ
  আনইনস্টল করলে পয়েন্ট হারিয়ে যাবে এবং এক ডিভাইসের পয়েন্ট অন্য ডিভাইসে সিংক হবে না। বড়
  পরিসরে চালাতে চাইলে ভবিষ্যতে এটাকে Firestore-এ (uid অনুযায়ী) সরিয়ে নেওয়া ভালো — এই জন্য
  `PointsService` ফাইলটা এমনভাবে লেখা যাতে শুধু এই একটা ফাইল বদলালেই backend সুইচ করা যায়।
- Fraud/multi-account withdraw abuse ঠেকাতে বাস্তব প্রোডাক্টে সাধারণত Cloud Function দিয়ে
  server-side validation করা হয় — এই ভার্সনে সেটা নেই, শুরুর জন্য যথেষ্ট সহজ রাখা হয়েছে।
- Google Play নীতিমালা অনুযায়ী "পয়েন্টের বিনিময়ে সত্যিকারের টাকা" দেওয়া অ্যাপের ক্ষেত্রে
  বিশেষ রিভিউ প্রক্রিয়া ও শর্ত আছে — পাবলিশ করার আগে Play Console-এর নীতিমালা ভালোভাবে
  পড়ে নেওয়ার পরামর্শ থাকলো।
