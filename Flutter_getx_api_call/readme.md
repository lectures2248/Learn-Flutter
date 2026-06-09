# Flutter MVC Posts App 
### Building a REST API Consumer App using MVC Pattern and GetX

---

## Part 1 — Understanding the MVC Pattern

MVC stands for Model, View, Controller. It is a way of organizing your code so that different parts of the application have different responsibilities. Think of it like a restaurant.

- The **Model** is the kitchen. It knows what the data looks like and how to prepare it.
- The **Controller** is the waiter. It goes to the kitchen, gets the food (data), and brings it to the table.
- The **View** is the dining area. It just shows things to the customer (the user). It does not cook anything itself.

Why do we separate things like this? Because when your app grows bigger, you do not want everything tangled together. If there is a bug in how data is fetched, you check the controller or the service. If there is a layout problem, you check the view. Each piece has one job.

In our project, the folder structure will look like this:

```
lib/
  main.dart
  models/
    post_model.dart
  services/
    api_service.dart
  controllers/
    home_controller.dart
  screen/
    home_screen.dart
```

We will also use a package called **GetX**. GetX handles state management for us. State management basically means: when data changes, the screen should automatically update to show the new data. GetX makes this very simple compared to the older ways of doing it in Flutter.

---

## Part 2 — Project Setup

### Step 1: Create a New Flutter Project


### Step 2: Add the Required Packages

Open `pubspec.yaml` and add the following under `dependencies`:

```yaml

  get: ^4.6.6
  http: ^1.2.1
```

The `get` package is GetX — our state management tool. The `http` package allows us to make network requests (talk to APIs over the internet).


This downloads the packages so your project can use them.

### Step 3: Create the Folder Structure

Inside the `lib` folder, create these folders:
- `models`
- `services`
- `controllers`
- `screen`

You can create them manually in your file explorer or use the terminal. The structure helps you know where to find each piece of your code. Six months from now, when you come back to this project, you will thank yourself for organizing it properly.

---

## Part 3 — The Model

### File: `lib/models/post_model.dart`

```dart
class PostModel {
  final int userId;
  final int id;
  final String title;
  final String body;

  PostModel({
    required this.userId,
    required this.id,
    required this.title,
    required this.body,
  });

  factory PostModel.fromJson(Map<String, dynamic> json) {
    return PostModel(
      userId: json['userId'] as int,
      id:     json['id']     as int,
      title:  json['title']  as String,
      body:   json['body']   as String,
    );
  }
}
```

### What Is a Model?

A model is a Dart class that represents your data. It is a blueprint. When the API returns data, it comes as raw JSON — basically just text with key-value pairs. We need to convert that raw JSON into proper Dart objects so we can work with them easily in our app.

Look at what the API returns for a single post:

```json
{
  "userId": 1,
  "id": 1,
  "title": "sunt aut facere repellat provident...",
  "body": "quia et suscipit suscipit recusandae..."
}
```

Our model class has four fields that match exactly: `userId`, `id`, `title`, and `body`. The `final` keyword means these values cannot change after the object is created — posts are read-only data, which is the correct approach here.

### The Constructor

```dart
PostModel({
  required this.userId,
  required this.id,
  required this.title,
  required this.body,
});
```

This is a named constructor. The `required` keyword means all four fields must be provided when you create a PostModel object. You cannot create a PostModel with missing data. This prevents bugs.

### What Is a Factory Constructor?
  A factory constructor is a special constructor that does some work before creating and returning an object — unlike a         regular constructor which just directly creates one. In our case, that "work" is reading values out of a JSON dictionary     and packaging them into a clean PostModel object.

```dart
factory PostModel.fromJson(Map<String, dynamic> json) {
  return PostModel(
    userId: json['userId'] as int,
    id:     json['id']     as int,
    title:  json['title']  as String,
    body:   json['body']   as String,
  );
}
```

Think of it this way. Imagine the API hands you a sealed envelope. Inside the envelope is a piece of paper with this written on it:

```
userId: 1
id: 1
title: sunt aut facere...
body: quia et suscipit...
```

That envelope is the JSON. Your app cannot directly use an envelope — it needs a proper `PostModel` object with real Dart fields. The factory constructor is the person who opens the envelope, reads each line, and fills in the PostModel form for you.

In Dart, a regular constructor just creates a new object when you call it — nothing more. A `factory` constructor is allowed to do some work first before returning the object. In our case, that work is reading the JSON map and picking out the correct values.

`Map<String, dynamic>` is how Dart represents a JSON object. Think of it as a dictionary: the keys are always strings (like `"userId"`, `"title"`), and the values can be any type — a number, a piece of text, true/false, etc. That is why the value side is called `dynamic` — it can be anything.

When we write `json['userId'] as int`, we are doing two things at once:
- `json['userId']` — look up the key `"userId"` in the map and get its value
- `as int` — tell Dart "I know this value is an integer, treat it as one"

This is called a type cast. We do the same for every field: grab it from the map, cast it to the right type, and pass it into the PostModel constructor.

So the factory constructor takes the messy raw JSON map and neatly packages it into a clean PostModel object. Every time you work with APIs in Flutter, you will write something like this. It is a standard pattern.

The name `fromJson` is a convention. It is not a Flutter or Dart requirement — it is just what every developer writes so that other developers immediately understand what this constructor does.

---

## Part 4 — The Service

### File: `lib/services/api_service.dart`

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

import '../models/post_model.dart';

class ApiService {
  static const String _url = 'https://jsonplaceholder.typicode.com/posts';

  Future<List<PostModel>> fetchPosts() async {
    final response = await http.get(Uri.parse(_url));

    if (response.statusCode == 200) {
      final List<dynamic> jsonList = jsonDecode(response.body);

      return jsonList
          .map((item) => PostModel.fromJson(item))
          .toList();

    } else {
      throw Exception('API error! Status code: ${response.statusCode}');
    }
  }
}
```

### What Is a Service?

Technically MVC has three parts: Model, View, Controller. But in real apps, we add a Service layer as well. The service is responsible for one thing only: talking to the outside world. In our case, talking to the internet.

By putting the network code in a separate service class, our controller never has to worry about how HTTP requests work. The controller just calls `fetchPosts()` and gets back a list of PostModel objects. Clean separation.

### Breaking Down the Code

**The URL:**
```dart
static const String _url = 'https://jsonplaceholder.typicode.com/posts';
```

`static const` means this value belongs to the class itself, not to any individual instance. There is one URL for the whole class. The underscore `_` before the name means it is private — only code inside this file can access it. This is good practice because the URL is an internal implementation detail.

**The method signature:**
```dart
Future<List<PostModel>> fetchPosts() async {
```

`async` means this function runs asynchronously. Network calls take time — they could take half a second or five seconds depending on the internet connection. We cannot just freeze the entire app while waiting. `async` lets the app keep running other things while we wait for the response.

`Future<List<PostModel>>` means this function will eventually return a List of PostModel objects — but not immediately. A Future is a promise that the data will arrive at some point.

**Making the HTTP call:**
```dart
final response = await http.get(Uri.parse(_url));
```

`http.get()` makes a GET request to the URL. `Uri.parse()` converts our URL string into a proper URI object that the http package understands. `await` means: pause here and wait until the response comes back before moving to the next line. We can only use `await` inside an `async` function.

**Checking the response:**
```dart
if (response.statusCode == 200) {
```

HTTP status code 200 means "OK — everything went fine." If the server is down, or the URL is wrong, or there is no internet, we will get a different code. 404 means not found, 500 means server error, etc. We only process the data if the status is 200.

**Decoding the JSON:**
```dart
final List<dynamic> jsonList = jsonDecode(response.body);
```

`response.body` is the raw response text — a big string of JSON. `jsonDecode()` converts that string into Dart objects. Since the API returns a JSON array (a list of posts), `jsonDecode` gives us a `List<dynamic>`.

**Converting each item to a PostModel:**
```dart
return jsonList
    .map((item) => PostModel.fromJson(item))
    .toList();
```

`.map()` goes through every item in the list and transforms it. For each `item` (which is a raw JSON map), we call `PostModel.fromJson(item)` to convert it into a proper PostModel object. `.toList()` converts the result back into a List. In the end we get `List<PostModel>` — a clean list of model objects.

**Handling errors:**
```dart
} else {
  throw Exception('API error! Status code: ${response.statusCode}');
}
```

If the status code is not 200, we throw an exception. This propagates the error up to whoever called `fetchPosts()`. The controller will catch it and handle it gracefully.

---

## Part 5 — The Controller

### File: `lib/controllers/home_controller.dart`

```dart
import 'package:get/get.dart';

import '../models/post_model.dart';
import '../services/api_service.dart';

class HomeController extends GetxController {

  final ApiService _apiService = ApiService();

  var isLoading = false.obs;
  var errorMessage = ''.obs;
  var postList = <PostModel>[].obs;

  @override
  void onInit() {
    super.onInit();
    fetchPosts();
  }

  Future<void> fetchPosts() async {
    isLoading.value = true;
    errorMessage.value = '';

    try {
      final List<PostModel> posts = await _apiService.fetchPosts();
      postList.assignAll(posts);

    } catch (e) {
      errorMessage.value = 'error: ${e.toString()}';

    } finally {
      isLoading.value = false;
    }
  }
}
```

### What Is the Controller?

The controller is the brain of the operation. It holds the data, manages the state (loading, error, success), and decides when to call the service. The view (the screen) does not talk to the service directly — it only talks to the controller.

### GetxController

```dart
class HomeController extends GetxController {
```

By extending `GetxController`, our class gets all of GetX's state management abilities. GetX will automatically manage the lifecycle of this controller — creating it, initializing it, and destroying it when it is no longer needed.

### Observable Variables — The Core of GetX

```dart
var isLoading = false.obs;
var errorMessage = ''.obs;
var postList = <PostModel>[].obs;
```

The `.obs` at the end is the magic. `.obs` stands for observable. It turns a regular variable into a reactive variable. When an observable variable changes its value, any widget on the screen that is listening to it will automatically rebuild.

Think of it like a newspaper subscription. You subscribe once. Every morning (every time the variable changes), a fresh newspaper (updated UI) is delivered to you automatically. You do not have to go pick it up yourself.

Without GetX, you would need to call `setState()` manually or use a `StreamBuilder`. GetX handles all of that behind the scenes for you.

- `isLoading` — tracks whether we are currently waiting for the API response
- `errorMessage` — holds any error text if something goes wrong
- `postList` — holds the list of posts once they arrive

### The onInit Method

```dart
@override
void onInit() {
  super.onInit();
  fetchPosts();
}
```

`onInit()` is a lifecycle method from `GetxController`. It runs automatically the moment this controller is created. It is like a constructor but specifically for initialization work.

`super.onInit()` calls the parent class's `onInit()` method first — always do this when overriding lifecycle methods. Then we immediately call `fetchPosts()` so that the data loads as soon as the screen opens. The user should not have to press a button to see the posts.

### The fetchPosts Method

```dart
Future<void> fetchPosts() async {
  isLoading.value = true;
  errorMessage.value = '';
```

First we set `isLoading` to true. Notice we use `.value` — this is how you read or write the actual value of an observable. The observable is a wrapper around the value, so `.value` unwraps it.

Setting `isLoading.value = true` automatically tells every listening widget: "Hey, I changed. Rebuild yourself." The screen will immediately show a loading spinner.

We also clear the `errorMessage` in case there was a previous error.

```dart
try {
  final List<PostModel> posts = await _apiService.fetchPosts();
  postList.assignAll(posts);
```

We call the service inside a try-catch block. If the service throws an exception, the catch block handles it. If it succeeds, we get back our list of PostModel objects.

`postList.assignAll(posts)` replaces all items in the observable list with the new ones. We use `assignAll` instead of just assigning because `postList` is an observable list — it has special methods that trigger UI updates. If you just did `postList = posts`, you would break the reactivity.

```dart
} catch (e) {
  errorMessage.value = 'error: ${e.toString()}';
} finally {
  isLoading.value = false;
}
```

If anything goes wrong (no internet, server error, etc.), the exception is caught here and stored in `errorMessage`. The `finally` block runs no matter what — success or failure. We always set `isLoading` to false at the end because we are done loading either way.

---

## Part 6 — The Screen (View)

### File: `lib/screen/home_screen.dart`

```dart
import 'package:flutter/material.dart';
import 'package:get/get.dart';

import '../controllers/home_controller.dart';

class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final HomeController controller = Get.put(HomeController());

    return Scaffold(
      appBar: AppBar(
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        title: const Text('Posts App'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            tooltip: 'Refresh',
            onPressed: () => controller.fetchPosts(),
          ),
        ],
      ),
      body: Obx(() {
        if (controller.isLoading.value) {
          return const Center(
            child: CircularProgressIndicator(),
          );
        }

        if (controller.errorMessage.value.isNotEmpty) {
          return Center(
            child: Padding(
              padding: const EdgeInsets.all(24),
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  const Icon(Icons.wifi_off, color: Colors.red, size: 64),
                  const SizedBox(height: 16),
                  Text(
                    controller.errorMessage.value,
                    textAlign: TextAlign.center,
                    style: const TextStyle(fontSize: 15, color: Colors.red),
                  ),
                  const SizedBox(height: 24),
                  ElevatedButton.icon(
                    onPressed: () => controller.fetchPosts(),
                    icon: const Icon(Icons.refresh),
                    label: const Text('Try Again'),
                  ),
                ],
              ),
            ),
          );
        }

        if (controller.postList.isEmpty) {
          return const Center(
            child: Text('No posts found.', style: TextStyle(fontSize: 16)),
          );
        }

        return ListView.builder(
          itemCount: controller.postList.length,
          itemBuilder: (context, index) {
            final post = controller.postList[index];

            return Card(
              margin: const EdgeInsets.symmetric(horizontal: 12, vertical: 6),
              elevation: 3,
              child: ListTile(
                leading: CircleAvatar(
                  backgroundColor: Theme.of(context).colorScheme.primary,
                  child: Text(
                    '${post.id}',
                    style: const TextStyle(
                      color: Colors.white,
                      fontSize: 12,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                ),
                title: Text(
                  post.title,
                  style: const TextStyle(fontWeight: FontWeight.w600),
                ),
                subtitle: Text(
                  'User ID: ${post.userId}',
                  style: TextStyle(color: Colors.grey[600], fontSize: 12),
                ),
                trailing: const Icon(Icons.arrow_forward_ios, size: 14),
              ),
            );
          },
        );
      }),
    );
  }
}
```

### Why StatelessWidget?

We use `StatelessWidget` instead of `StatefulWidget`. Normally, if your screen needs to update itself, you would use `StatefulWidget` and call `setState()`. But since GetX handles all the state updates for us through observables, we do not need `StatefulWidget` at all. The screen itself is stateless — GetX manages the state externally.

### Injecting the Controller

```dart
final HomeController controller = Get.put(HomeController());
```

`Get.put()` creates an instance of `HomeController` and registers it in GetX's dependency injection system. Dependency injection is just a fancy way of saying: GetX holds onto the controller for you and gives it to you when you ask. After this line, the controller is running — `onInit()` has been called, and `fetchPosts()` has started.

If you called `Get.put()` on a second screen and the controller was already created, GetX would give you the same instance rather than creating a new one. This is useful when multiple screens need the same controller.

### The Obx Widget

```dart
body: Obx(() {
  // everything inside here rebuilds automatically
}),
```

`Obx` is a widget from GetX that listens to observable variables. Any observable that is read inside `Obx` is automatically tracked. When any of those observables change, the `Obx` widget rebuilds its content.

In our case, `Obx` watches `isLoading`, `errorMessage`, and `postList`. When the controller sets `isLoading.value = true`, `Obx` rebuilds and shows the loading spinner. When the posts arrive and `postList` gets filled, `Obx` rebuilds again and shows the list.

### The Three States of a Network Screen

Any screen that loads data from the internet has three possible states. Our code handles all three:

**State 1 — Loading:**
```dart
if (controller.isLoading.value) {
  return const Center(
    child: CircularProgressIndicator(),
  );
}
```
While the API call is in progress, show a spinner in the center of the screen. Users need to know something is happening.

**State 2 — Error:**
```dart
if (controller.errorMessage.value.isNotEmpty) {
  return Center(
    child: Column(
      children: [
        Icon(Icons.wifi_off, color: Colors.red, size: 64),
        Text(controller.errorMessage.value),
        ElevatedButton(
          onPressed: () => controller.fetchPosts(),
          child: Text('Try Again'),
        ),
      ],
    ),
  );
}
```
If something went wrong, show an error icon, the error message, and a retry button. Notice the retry button calls `controller.fetchPosts()` again. Always give users a way to recover from errors.

**State 3 — Success:**
```dart
return ListView.builder(
  itemCount: controller.postList.length,
  itemBuilder: (context, index) {
    final post = controller.postList[index];
    return Card( ... );
  },
);
```
Show the list. `ListView.builder` is efficient — it only builds the widgets that are currently visible on screen, not all 100 at once. For a list with 100 items this might not matter much, but if you had 10,000 items, it would matter enormously.

### Reading Data from the Model in the View

Inside `itemBuilder`, we access the individual post like this:

```dart
final post = controller.postList[index];
```

Then we use `post.id`, `post.title`, `post.userId` to display data. Notice that the view only reads from the model — it never modifies it. The view is read-only. All changes go through the controller.

---
