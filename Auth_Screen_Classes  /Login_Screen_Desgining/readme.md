# Flutter Login App
---


## Part 1 

### Step 1.0: Set SplashScreen as the home

Instead of showing a plain Scaffold, we set `SplashScreen()` as the first screen. `home` is the widget Flutter shows when the app launches — it is our entry screen.

```dart
import 'package:flutter/material.dart';
import 'splash_screen.dart';               // <--- import the splash screen file

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: SplashScreen(),                // <--- app opens with SplashScreen first
    );
  }
}
```
---


## Part 2 

### Step 1.1: Create the base StatefulWidget

We use `StatefulWidget` because the splash screen needs to run code when it first appears — specifically, starting a timer. That startup logic lives in `initState()`, which only exists in `StatefulWidget`.

```dart
import 'package:flutter/material.dart';

class SplashScreen extends StatefulWidget {
  const SplashScreen({super.key});

  @override
  State<SplashScreen> createState() => _SplashScreenState();
}

class _SplashScreenState extends State<SplashScreen> {

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Text("Splash Screen"),
      ),
    );
  }
}
```

---

### Step 2: Add initState() and override it

`initState()` is called exactly once — right when the widget is inserted into the screen for the first time. This is where we start the 3-second timer. We must call `super.initState()` first, always.

```dart
class _SplashScreenState extends State<SplashScreen> {

  @override
  void initState() {
    super.initState();              // <--- always call super first
    _navigateToLogin();             // <--- starts the timer immediately on screen load
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Text("Splash Screen"),
      ),
    );
  }
}
```

> **Note:** `initState()` runs before `build()`. So the timer starts even before the user sees the splash screen visually.

---

### Step 3: Write the _navigateToLogin() method

This method waits 3 seconds then pushes the `LoginScreen`. We use `Future.delayed()` for the wait. We use `pushReplacement` — not `push` — so the user cannot press the back button and return to the splash screen.

```dart
class _SplashScreenState extends State<SplashScreen> {

  @override
  void initState() {
    super.initState();
    _navigateToLogin();
  }

  void _navigateToLogin() async {                        // <--- async because we use await
    await Future.delayed(Duration(seconds: 3));           // <--- pauses for exactly 3 seconds
    Navigator.pushReplacement(                            // <--- replaces splash, no back button
      context,
      MaterialPageRoute(builder: (context) => LoginScreen()),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(child: Text("Splash Screen")),
    );
  }
}
```

> **Important:** `pushReplacement` removes the SplashScreen from the navigation stack. If we used `push`, pressing back on LoginScreen would bring the user back to the splash screen — which is wrong.

---

### Step 4: Add the full-screen gradient Container

Same gradient as the login screen — gold to cream. Both screens share the same colors so the app feels consistent. We size the Container to the full device screen using `MediaQuery`.

```dart
class _SplashScreenState extends State<SplashScreen> {

  @override
  void initState() {
    super.initState();
    _navigateToLogin();
  }

  void _navigateToLogin() async {
    await Future.delayed(Duration(seconds: 3));
    Navigator.pushReplacement(
      context,
      MaterialPageRoute(builder: (context) => LoginScreen()),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Container(
        height: MediaQuery.of(context).size.height,   // <--- full screen height
        width: MediaQuery.of(context).size.width,     // <--- full screen width
        decoration: BoxDecoration(
          gradient: LinearGradient(
            colors: [
              Color(0xFFE0C375),                      // <--- gold (top)
              Color(0xFFFAE7CB),                      // <--- cream (bottom)
            ],
          ),
        ),
        child: Center(child: Text("Splash Screen")),
      ),
    );
  }
}
```

---

### Step 5: Replace Center with Column and add the Icon

We replace the placeholder `Center` with a `Column` so we can stack the icon, app name, and subtitle vertically. The `Icon` widget gives us a centered lock icon that matches the login theme.

```dart
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,   // <--- center everything vertically
          children: [
            Icon(
              Icons.lock_rounded,                        // <--- lock icon for the brand
              size: 80,                                  // <--- large splash icon
              color: Colors.black87,
            ),
          ],
        ),
```

---

### Step 6: Add the app title Text below the Icon

The app title appears right below the icon. We add `SizedBox` between them as spacing. `fontWeight.bold` and `fontSize: 32` make it feel like a proper splash headline.

```dart
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(
              Icons.lock_rounded,
              size: 80,
              color: Colors.black87,
            ),
            SizedBox(height: 20),               // <--- space between icon and title
            Text(
              "Welcome",
              style: TextStyle(
                fontSize: 32,                   // <--- large headline
                fontWeight: FontWeight.bold,
                color: Colors.black87,
              ),
            ),
          ],
        ),
```

---

### Step 7: Add the subtitle "Please wait..." below the title

A small subtitle tells the user the app is loading. `Colors.black54` gives it a lighter tone than the title so there is a clear visual hierarchy — bold title above, soft subtitle below.

```dart
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(Icons.lock_rounded, size: 80, color: Colors.black87),
            SizedBox(height: 20),
            Text(
              "Welcome",
              style: TextStyle(fontSize: 32, fontWeight: FontWeight.bold, color: Colors.black87),
            ),
            SizedBox(height: 10),               // <--- space between title and subtitle
            Text(
              "Please wait...",
              style: TextStyle(
                fontSize: 14,
                color: Colors.black54,          // <--- lighter color = secondary text
              ),
            ),
          ],
        ),
```

---

### Step 8: Add the import for LoginScreen at the top

Now that we reference `LoginScreen` in `_navigateToLogin()`, we must import its file. Without this import the app will throw a compilation error.

```dart
import 'package:flutter/material.dart';
import 'login_screen.dart';                 // <--- required to use LoginScreen()

class SplashScreen extends StatefulWidget {
  const SplashScreen({super.key});

  @override
  State<SplashScreen> createState() => _SplashScreenState();
}

class _SplashScreenState extends State<SplashScreen> {

  @override
  void initState() {
    super.initState();
    _navigateToLogin();
  }

  void _navigateToLogin() async {
    await Future.delayed(Duration(seconds: 3));
    Navigator.pushReplacement(
      context,
      MaterialPageRoute(builder: (context) => LoginScreen()),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Container(
        height: MediaQuery.of(context).size.height,
        width: MediaQuery.of(context).size.width,
        decoration: BoxDecoration(
          gradient: LinearGradient(
            colors: [
              Color(0xFFE0C375),
              Color(0xFFFAE7CB),
            ],
          ),
        ),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(Icons.lock_rounded, size: 80, color: Colors.black87),
            SizedBox(height: 20),
            Text(
              "Welcome",
              style: TextStyle(fontSize: 32, fontWeight: FontWeight.bold, color: Colors.black87),
            ),
            SizedBox(height: 10),
            Text(
              "Please wait...",
              style: TextStyle(fontSize: 14, color: Colors.black54),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

```
******---[ SplashScreen Complete ]---******
```

---

## Part 3 — Login Screen Structure (`login_screen.dart`)

---

### Step 1: Create the base StatefulWidget

We use `StatefulWidget` because the password field needs to toggle between hidden and visible. That rebuild requires `setState()`, which only `StatefulWidget` can call.

```dart
import 'package:flutter/material.dart';

class LoginScreen extends StatefulWidget {
  const LoginScreen({super.key});

  @override
  State<LoginScreen> createState() => _LoginScreenState();
}

class _LoginScreenState extends State<LoginScreen> {

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Text("Login Screen"),
      ),
    );
  }
}
```

---

### Step 2: Add the full-screen Container

We wrap everything in a `Container` sized to the full screen. This will hold the gradient background. Without explicit `height` and `width`, the Container shrinks to fit its content instead of filling the screen.

```dart
class _LoginScreenState extends State<LoginScreen> {

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Container(
        height: MediaQuery.of(context).size.height,   // <--- full device height
        width: MediaQuery.of(context).size.width,     // <--- full device width
        child: Center(
          child: Text("Login Screen"),
        ),
      ),
    );
  }
}
```

---

### Step 3: Add BoxDecoration with LinearGradient

Now the Container gets its visual appearance — a two-color gradient from gold at the top to cream at the bottom. This matches the splash screen, giving the app a consistent look.

```dart
      body: Container(
        height: MediaQuery.of(context).size.height,
        width: MediaQuery.of(context).size.width,
        decoration: BoxDecoration(                    // <--- visual decoration for Container
          gradient: LinearGradient(
            colors: [
              Color(0xFFE0C375),                      // <--- gold (top)
              Color(0xFFFAE7CB),                      // <--- cream (bottom)
            ],
          ),
        ),
        child: Center(child: Text("Login Screen")),
      ),
```

> **Note:** Once you use `decoration`, you cannot also set `color:` directly on the Container. Everything visual must go inside `BoxDecoration`.

---

### Step 4: Replace Center with Column + add the title

`Column` stacks children top-to-bottom. `mainAxisAlignment.center` pushes the entire group of children to the vertical middle. We add the "Login Here!!" title as the first child.

```dart
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,  // <--- center all children vertically
          children: [
            Text(
              "Login Here!!",
              style: TextStyle(
                fontWeight: FontWeight.bold,
                fontSize: 26,                           // <--- large title text
              ),
            ),
            SizedBox(height: 50),                       // <--- space below title before fields
          ],
        ),
```


---

## Part 4 — Login Screen Input Fields

---

### Step 1: Declare the two TextEditingControllers

We declare controllers before `build()`. Each controller links to one `TextField` and lets us read the user's input using `.text`. Two fields = two controllers.

```dart
class _LoginScreenState extends State<LoginScreen> {

  TextEditingController email_controller    = TextEditingController();  // <--- for email field
  TextEditingController password_controller = TextEditingController();  // <--- for password field

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Container(
        height: MediaQuery.of(context).size.height,
        width: MediaQuery.of(context).size.width,
        decoration: BoxDecoration(
          gradient: LinearGradient(
            colors: [Color(0xFFE0C375), Color(0xFFFAE7CB)],
          ),
        ),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text("Login Here!!", style: TextStyle(fontWeight: FontWeight.bold, fontSize: 26)),
            SizedBox(height: 50),
          ],
        ),
      ),
    );
  }
}
```

> **Important:** If you don't assign a controller, you cannot read what the user typed. `controller.text` is the only way to get the value programmatically when the user taps Login.

---

### Step 2: Build the white rounded Container for the Email field

Every field uses the same visual box: a white `Container` with rounded corners, wrapped in `Padding` for left-right margin. The Container is the visual box — the `TextField` sits inside it.

```dart
// Add inside Column children, after SizedBox(height: 50)

Padding(
  padding: const EdgeInsets.symmetric(horizontal: 50),  // <--- left-right space from edges
  child: Container(
    height: 50,
    width: 350,
    decoration: BoxDecoration(
      color: Colors.white,                               // <--- white input box
      borderRadius: BorderRadius.circular(15),           // <--- rounded corners
    ),
    child: Center(child: Text("Email goes here")),       // placeholder for now
  ),
),
SizedBox(height: 20),
```

---

### Step 3: Add the TextField inside the Email Container

Now we replace the placeholder `Center` with the actual `TextField`. `InputBorder.none` removes the default underline — the Container already gives the field its border. `contentPadding` vertically centers the hint text inside the 50px box.

```dart
Padding(
  padding: const EdgeInsets.symmetric(horizontal: 50),
  child: Container(
    height: 50,
    width: 350,
    decoration: BoxDecoration(
      color: Colors.white,
      borderRadius: BorderRadius.circular(15),
    ),
    child: TextField(
      controller: email_controller,                      // <--- links to the email controller
      decoration: InputDecoration(
        hintText: "Enter Email Address",
        prefixIcon: Icon(Icons.email, color: Colors.black),
        border: InputBorder.none,                        // <--- no underline
        contentPadding: EdgeInsets.only(top: 14),        // <--- centers text vertically in box
      ),
    ),
  ),
),
SizedBox(height: 20),
```

---

### Step 4: Declare the password visibility boolean and toggle method

Before building the password field, we add the state variable and the method that flips it. `change_visibilty = true` means the password is hidden by default — correct UX.

```dart
class _LoginScreenState extends State<LoginScreen> {

  bool change_visibilty = true;              // <--- true = password hidden on load

  TextEditingController email_controller    = TextEditingController();
  TextEditingController password_controller = TextEditingController();

  void toggle_visiblity() {
    setState(() {
      change_visibilty = !change_visibilty;  // <--- flips true to false, triggers rebuild
    });
  }

  // ... build() continues below unchanged
}
```

> **Note:** `setState()` is required. Without it, the boolean flips in memory but Flutter never knows to rebuild — the icon and text masking never update visually.

---

### Step 5: Build the Password field Container

Same visual box as the email field — white Container, rounded corners, same padding. Only the content inside will be different. We set it up first as a shell.

```dart
// Add after the email SizedBox(height: 20)

Padding(
  padding: const EdgeInsets.symmetric(horizontal: 50),
  child: Container(
    height: 50,
    width: 350,
    decoration: BoxDecoration(
      color: Colors.white,
      borderRadius: BorderRadius.circular(15),
    ),
    child: Center(child: Text("Password goes here")),    // placeholder for now
  ),
),
SizedBox(height: 12),
```

---

### Step 6: Add the Password TextField with obscureText and suffixIcon

Now we replace the placeholder with the actual `TextField`. Two things make it different from the email field: `obscureText` hides characters as dots, and `suffixIcon` is the eye button that calls `toggle_visiblity()`.

```dart
Padding(
  padding: const EdgeInsets.symmetric(horizontal: 50),
  child: Container(
    height: 50,
    width: 350,
    decoration: BoxDecoration(
      color: Colors.white,
      borderRadius: BorderRadius.circular(15),
    ),
    child: TextField(
      controller: password_controller,
      obscureText: change_visibilty,                       // <--- true=dots, false=visible text
      decoration: InputDecoration(
        hintText: "Enter Password",
        prefixIcon: Icon(Icons.lock, color: Colors.black),
        border: InputBorder.none,
        contentPadding: EdgeInsets.only(top: 14),
        suffixIcon: IconButton(
          onPressed: () => toggle_visiblity(),             // <--- calls toggle on every tap
          icon: Icon(
            change_visibilty ? Icons.visibility : Icons.visibility_off,  // <--- switches icon
            color: Colors.black,
          ),
        ),
      ),
    ),
  ),
),
SizedBox(height: 12),
```

> **Important:** When `setState` runs, Flutter re-reads `change_visibilty` and updates both `obscureText` and the ternary icon simultaneously — text masking and icon always stay in sync.


---

## Part 5 — Forget Password & Create Account Row

---

### Step 1: Add a Row with spaceBetween alignment

We need two texts side by side — "Forget Password ?" on the left and "Create an Account" on the right. `Row` places children horizontally. `MainAxisAlignment.spaceBetween` pushes the first child to the far left and the second to the far right automatically.

```dart
// Add after the password SizedBox(height: 12)

Padding(
  padding: const EdgeInsets.symmetric(horizontal: 50),   // <--- same margin as the fields
  child: Row(
    mainAxisAlignment: MainAxisAlignment.spaceBetween,    // <--- left and right ends
    children: [
      Text("Forget Password ?"),
      Text("Create an Account"),
    ],
  ),
),
SizedBox(height: 30),
```

---

### Step 2: Wrap "Forget Password ?" in GestureDetector and style it

`GestureDetector` makes any widget tappable. We wrap the Text so tapping it can later trigger a navigation or dialog. We also apply styling so it looks like a clickable link.

```dart
Padding(
  padding: const EdgeInsets.symmetric(horizontal: 50),
  child: Row(
    mainAxisAlignment: MainAxisAlignment.spaceBetween,
    children: [

      GestureDetector(
        onTap: () {
          // forget password logic goes here
        },
        child: Text(
          "Forget Password ?",
          style: TextStyle(
            fontSize: 13,
            color: Colors.black87,                        // <--- visible but not loud
            fontWeight: FontWeight.w500,                  // <--- medium weight, not bold
          ),
        ),
      ),

      Text("Create an Account"),                          // will be styled in next step
    ],
  ),
),
SizedBox(height: 30),
```

---

### Step 3: Wrap "Create an Account" in GestureDetector and style it

Same treatment as "Forget Password ?" — wrapped in `GestureDetector` for tap handling, styled to match. Both texts now look and behave consistently.

```dart
Padding(
  padding: const EdgeInsets.symmetric(horizontal: 50),
  child: Row(
    mainAxisAlignment: MainAxisAlignment.spaceBetween,
    children: [

      GestureDetector(
        onTap: () {
          // forget password logic
        },
        child: Text(
          "Forget Password ?",
          style: TextStyle(
            fontSize: 13,
            color: Colors.black87,
            fontWeight: FontWeight.w500,
          ),
        ),
      ),

      GestureDetector(
        onTap: () {
          // navigate to signup screen
        },
        child: Text(
          "Create an Account",
          style: TextStyle(
            fontSize: 13,
            color: Colors.black87,                        // <--- same style as forget password
            fontWeight: FontWeight.w500,
          ),
        ),
      ),

    ],
  ),
),
SizedBox(height: 30),
```

---

## Part 6 — Login Button

---

### Step 1: Add the GestureDetector as the button wrapper

We use `GestureDetector` instead of `ElevatedButton` to have complete control over the button's visual design. `onTap` is where the login action will go.

```dart
// Add after SizedBox(height: 30)

GestureDetector(
  onTap: () {
    // login logic goes here
  },
  child: Center(child: Text("Login")),   // placeholder for now
),
```

---

### Step 2: Add the styled black Container as the button body

The actual button appearance is a black `Container` with rounded corners. We give it a fixed size and center the text inside it.

```dart
GestureDetector(
  onTap: () {
    print(email_controller.text);        // <--- reads email on tap
    print(password_controller.text);     // <--- reads password on tap
  },
  child: Container(
    height: 50,
    width: 100,
    decoration: BoxDecoration(
      color: Colors.black,               // <--- black button background
      borderRadius: BorderRadius.circular(15),
    ),
    child: Center(
      child: Text(
        "Login",
        style: TextStyle(
          color: Colors.white,           // <--- white text on black background
          fontWeight: FontWeight.bold,
          fontSize: 18,
        ),
      ),
    ),
  ),
),
```
