# Class 1: Product Form with Category Dropdown from Database



# STEP 1: BUILD THE FORM UI

First we build only the design. No controller, no database, no logic. Just widgets, so you can see the screen and fix the layout.

`lib/screens/add_product_screen.dart`

```dart
import 'package:flutter/material.dart';

class AddProductScreen extends StatelessWidget {
  const AddProductScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Add Product')),

      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: [

            // PRODUCT NAME
            TextField(
              decoration: const InputDecoration(
                labelText: 'Product Name',
                border: OutlineInputBorder(),
              ),
            ),

            const SizedBox(height: 16),

            // PRICE
            TextField(
              keyboardType: TextInputType.number,
              decoration: const InputDecoration(
                labelText: 'Price',
                prefixText: 'Rs. ',
                border: OutlineInputBorder(),
              ),
            ),

            const SizedBox(height: 16),

            // DESCRIPTION
            TextField(
              maxLines: 4,
              decoration: const InputDecoration(
                labelText: 'Description',
                alignLabelWithHint: true,
                border: OutlineInputBorder(),
              ),
            ),

            const SizedBox(height: 16),

            // CATEGORY LABEL
            const Align(
              alignment: Alignment.centerLeft,
              child: Text('Category'),
            ),

            const SizedBox(height: 6),

            // CATEGORY DROPDOWN
            Container(
              padding: const EdgeInsets.symmetric(horizontal: 12),
              decoration: BoxDecoration(
                border: Border.all(color: Colors.grey),
                borderRadius: BorderRadius.circular(4),
              ),
              child: DropdownButton<String>(
                value: null,
                hint: const Text('Select a category'),
                isExpanded: true,
                underline: const SizedBox(),
                items: const [
                  DropdownMenuItem(value: '1', child: Text('Electronics')),
                  DropdownMenuItem(value: '2', child: Text('Sports')),
                  DropdownMenuItem(value: '3', child: Text('Books')),
                ],
                onChanged: (value) {},
              ),
            ),

            const SizedBox(height: 32),

            // SAVE BUTTON
            ElevatedButton(
              onPressed: () {},
              child: const Text('Save Product'),
            ),

          ],
        ),
      ),
    );
  }
}
```

Run it now. The form appears.

### What to notice

**The dropdown opens but nothing gets selected.** Tap Electronics and it goes back to "Select a category". That is not a bug. `value: null` is hardcoded, so nothing can change it. A dropdown needs somewhere to store the selection. That storage is what the controller gives us in Step 2.

**Why `DropdownButton` needs a Container.** `TextField` draws its own label and border. `DropdownButton` does not. It is a plain widget. So we draw the label ourselves with a `Text` above it, and the border ourselves with a `Container`.

| Property | Why |
|---|---|
| `underline: const SizedBox()` | removes the default grey line, since we already drew a border |
| `isExpanded: true` | makes it take the full width. Without it a long category name overflows |
| `hint` | the text shown when `value` is null |

**Categories are hardcoded right now.** That is temporary. In Step 2 they come from Supabase.

---

# STEP 2: WRITE THE CONTROLLER

Now the logic. All of it goes in one file, and none of it goes in the screen.

`lib/controllers/product_controller.dart`

## 2.1 The state

```dart
import 'package:flutter/material.dart';
import 'package:get/get.dart';
import 'package:supabase_flutter/supabase_flutter.dart';

class ProductController extends GetxController {
  final supabase = Supabase.instance.client;

  // text controllers
  final nameController = TextEditingController();
  final priceController = TextEditingController();
  final descriptionController = TextEditingController();

  // error messages. null means no error
  final nameError = RxnString();
  final priceError = RxnString();
  final descriptionError = RxnString();
  final categoryError = RxnString();

  // dropdown
  final categories = <Map<String, dynamic>>[].obs;
  final selectedCategoryId = RxnString();

  // flags
  final isLoadingCategories = true.obs;
}
```

`TextEditingController` lets us read what the user typed. `nameController.text` gives the text.

`RxnString` is a String that is allowed to be null. Null means the field is fine. A message means show it in red. These five variables are what replace `Form` and `GlobalKey<FormState>`.

`selectedCategoryId` is a String even though `cat_id` is a number in the database. Our dropdown is `DropdownButton<String>`, so every value inside it must be a String. We convert it back to a number in Class 3.

## 2.2 Start and cleanup

```dart
  @override
  void onInit() {
    super.onInit();
    loadCategories();
  }

  @override
  void onClose() {
    nameController.dispose();
    priceController.dispose();
    descriptionController.dispose();
    super.onClose();
  }
```

`onInit` is GetX's version of `initState`. It runs once when the screen opens, so this is where we fetch categories.

`onClose` is `dispose`. Text controllers hold listeners, so they must be disposed or your app leaks memory.

## 2.3 Load categories from Supabase

```dart
  Future<void> loadCategories() async {
    try {
      final data = await supabase
          .from('categories')
          .select('cat_id, category_name')
          .order('category_name');

      categories.value = List<Map<String, dynamic>>.from(data);
    } catch (e) {
      showMessage('Could not load categories', '$e');
    } finally {
      isLoadingCategories.value = false;
    }
  }
```

`select('cat_id, category_name')` fetches only the two columns we need.

Supabase returns a `List<dynamic>`. `List<Map<String, dynamic>>.from(data)` converts it to the type our dropdown expects.

`finally` runs whether the call worked or failed. Without it, one network error leaves the loading spinner turning forever.

## 2.4 Validation

```dart
  bool validateForm() {

    // name
    final name = nameController.text.trim();
    if (name.isEmpty) {
      nameError.value = 'Product name is required';
    } else if (name.length < 3) {
      nameError.value = 'Name must be at least 3 characters';
    } else {
      nameError.value = null;
    }

    // price
    final priceText = priceController.text.trim();
    final price = double.tryParse(priceText);
    if (priceText.isEmpty) {
      priceError.value = 'Price is required';
    } else if (price == null) {
      priceError.value = 'Enter a valid number';
    } else if (price <= 0) {
      priceError.value = 'Price must be greater than zero';
    } else {
      priceError.value = null;
    }

    // description
    final desc = descriptionController.text.trim();
    if (desc.isEmpty) {
      descriptionError.value = 'Description is required';
    } else if (desc.length < 10) {
      descriptionError.value = 'Description must be at least 10 characters';
    } else {
      descriptionError.value = null;
    }

    // category
    if (selectedCategoryId.value == null) {
      categoryError.value = 'Please select a category';
    } else {
      categoryError.value = null;
    }

    // form is valid only if every error is null
    return nameError.value == null &&
        priceError.value == null &&
        descriptionError.value == null &&
        categoryError.value == null;
  }
```

Every field gets an answer, either a message or null. Then we return true only when all four are null.

Two rules to remember:

`trim()` first, always. Without it a user types three spaces and passes validation.

Use `double.tryParse`, not `double.parse`. `tryParse` returns null when the text is not a number. `parse` crashes the app.

| Input | tryParse result |
|---|---|
| 500 | 500.0 |
| 999.99 | 999.99 |
| abc | null |
| empty | null |

## 2.5 Temporary save

Real saving comes in Class 3. Today we only prove the validation works.

```dart
  void saveProduct() {
    if (!validateForm()) return;
    showMessage('Valid', 'Category id is ${selectedCategoryId.value}');
  }

  void showMessage(String title, String message) {
    Get.snackbar(
      title,
      message,
      snackPosition: SnackPosition.BOTTOM,
      margin: const EdgeInsets.all(12),
    );
  }
}
```

`Get.snackbar` does not need a `BuildContext`. That is exactly why it can live inside a controller.

---

# STEP 3: THE BINDING

`lib/bindings/product_binding.dart`

```dart
import 'package:get/get.dart';
import '../controllers/product_controller.dart';

class ProductBinding extends Bindings {
  @override
  void dependencies() {
    Get.lazyPut<ProductController>(() => ProductController());
  }
}
```

Now attach it to your existing route:

```dart
GetPage(
  name: '/add-product',
  page: () => const AddProductScreen(),
  binding: ProductBinding(),
),
```

`lazyPut` creates the controller only when the screen actually needs it, and GetX deletes it automatically when you leave the screen.

---

# STEP 4: CONNECT THE SCREEN TO THE CONTROLLER

Now go back to the screen and change four things:

1. `StatelessWidget` becomes `GetView<ProductController>`
2. the body gets wrapped in one `Obx`
3. every `TextField` gets a `controller` and an `errorText`
4. the dropdown reads its items and its value from the controller

Full updated file:

```dart
import 'package:flutter/material.dart';
import 'package:get/get.dart';

import '../controllers/product_controller.dart';

class AddProductScreen extends StatelessWidget {
  const AddProductScreen({super.key});

  @override
  Widget build(BuildContext context) {

  final controller = Get.put(ProductController());
    return Scaffold(
      appBar: AppBar(title: const Text('Add Product')),

      body: Obx(() {

        // spinner while categories are loading
        if (controller.isLoadingCategories.value) {
          return const Center(child: CircularProgressIndicator());
        }

        return SingleChildScrollView(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [

              // PRODUCT NAME
              TextField(
                controller: controller.nameController,
                decoration: InputDecoration(
                  labelText: 'Product Name',
                  border: const OutlineInputBorder(),
                  errorText: controller.nameError.value,
                ),
              ),

              const SizedBox(height: 16),

              // PRICE
              TextField(
                controller: controller.priceController,
                keyboardType: TextInputType.number,
                decoration: InputDecoration(
                  labelText: 'Price',
                  prefixText: 'Rs. ',
                  border: const OutlineInputBorder(),
                  errorText: controller.priceError.value,
                ),
              ),

              const SizedBox(height: 16),

              // DESCRIPTION
              TextField(
                controller: controller.descriptionController,
                maxLines: 4,
                decoration: InputDecoration(
                  labelText: 'Description',
                  alignLabelWithHint: true,
                  border: const OutlineInputBorder(),
                  errorText: controller.descriptionError.value,
                ),
              ),

              const SizedBox(height: 16),

              // CATEGORY LABEL
              const Align(
                alignment: Alignment.centerLeft,
                child: Text('Category'),
              ),

              const SizedBox(height: 6),

              // CATEGORY DROPDOWN
              Container(
                padding: const EdgeInsets.symmetric(horizontal: 12),
                decoration: BoxDecoration(
                  border: Border.all(
                    color: controller.categoryError.value == null
                        ? Colors.grey
                        : Colors.red,
                  ),
                  borderRadius: BorderRadius.circular(4),
                ),
                child: DropdownButton<String>(
                  value: controller.selectedCategoryId.value,
                  hint: const Text('Select a category'),
                  isExpanded: true,
                  underline: const SizedBox(),
                  items: controller.categories.map((category) {
                    return DropdownMenuItem<String>(
                      value: category['cat_id'].toString(),
                      child: Text(category['category_name']),
                    );
                  }).toList(),
                  onChanged: (value) {
                    controller.selectedCategoryId.value = value;
                    controller.categoryError.value = null;
                  },
                ),
              ),

              // CATEGORY ERROR MESSAGE
              if (controller.categoryError.value != null)
                Padding(
                  padding: const EdgeInsets.only(top: 6, left: 4),
                  child: Text(
                    controller.categoryError.value!,
                    style: const TextStyle(color: Colors.red, fontSize: 12),
                  ),
                ),

              const SizedBox(height: 32),

              // SAVE BUTTON
              ElevatedButton(
                onPressed: controller.saveProduct,
                child: const Text('Save Product'),
              ),

            ],
          ),
        );
      }),
    );
  }
}
```

### What changed and why

**`GetView<ProductController>`** gives you a ready `controller` property. You do not write `Get.find()` anywhere.

**One `Obx` around the whole body.** GetX watches every `.value` you read inside it and rebuilds when any of them changes. One Obx is enough for the entire screen. There is no `setState` in this file, and there is no `StatefulWidget`.

**`errorText`** is a normal property of `InputDecoration`. That is the reason we do not need `TextFormField`, a `Form`, or a form key. The message comes straight from the controller.

**`keyboardType: TextInputType.number`** only changes which keyboard a phone shows. It is not validation. On the web the user can still type letters, which is why `validateForm()` still has to check.

**The dropdown now works.** `value` reads from the controller and `onChanged` writes back to it. Because that variable is reactive, the Obx redraws and the selected name stays on screen.

We also set the border to red when there is a category error, and print the message underneath. `DropdownButton` has no `errorText` of its own, so we do that part by hand.

Three dropdown rules. Break any of them and the app crashes:

- item values and `selectedCategoryId` must both be `String`
- no two items can have the same value
- the current value must exist in the items list

---


