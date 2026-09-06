# Class 2: Adding a Product Image

## Setup

Add the package to `pubspec.yaml`:

```yaml
  image_picker: ^1.1.2
```

```bash
flutter pub get
```


# STEP 1: THE UI

Open `add_product_screen.dart`. Add these widgets at the **top** of the Column, before the Product Name field.

```dart
              // IMAGE BOX
              Container(
                height: 180,
                decoration: BoxDecoration(
                  border: Border.all(color: Colors.grey),
                  borderRadius: BorderRadius.circular(8),
                ),
                child: const Center(child: Text('No image selected')),
              ),

              const SizedBox(height: 8),

              // SELECT IMAGE BUTTON
              OutlinedButton(
                onPressed: () {},
                child: const Text('Select Image'),
              ),

              const SizedBox(height: 24),
```

Run it. You see a grey box and a button that does nothing.

Just like the dropdown in Class 1, the box has nowhere to store an image yet. We fix that in Step 2.

---

# STEP 2: THE CONTROLLER

Open `product_controller.dart`.

## 2.1 Two new imports

```dart
import 'dart:typed_data';
import 'package:image_picker/image_picker.dart';
```

## 2.2 Three new variables

Add these with your other variables:

```dart
  final ImagePicker picker = ImagePicker();

  XFile? pickedFile;
  final imageBytes = Rxn<Uint8List>();
  final imageError = RxnString();
```

| Variable | What it holds |
|---|---|
| `picker` | the object that opens the gallery |
| `pickedFile` | the file we picked (its name and type) |
| `imageBytes` | the image itself, as bytes |
| `imageError` | the red message, same as `nameError` |

`Uint8List` just means a list of bytes. An image is bytes.

`Rxn<Uint8List>` means bytes that can be null. Null means no image picked yet.

## 2.3 The pick function

```dart
  Future<void> pickImage() async {
    final XFile? image = await picker.pickImage(
      source: ImageSource.gallery,
      maxWidth: 1200,
      imageQuality: 85,
    );

    if (image == null) return;

    pickedFile = image;
    imageBytes.value = await image.readAsBytes();
    imageError.value = null;
  }
```

| Line | What it does |
|---|---|
| `picker.pickImage(...)` | opens the gallery and waits |
| `ImageSource.gallery` | pick from photos. Use `ImageSource.camera` for the camera |
| `maxWidth` and `imageQuality` | make the photo smaller. 4 MB becomes about 200 KB |
| `if (image == null) return;` | the user pressed back. Nothing to do |
| `readAsBytes()` | reads the image into bytes |
| `imageError.value = null` | clears the red message |

## 2.4 Why we read bytes

`image_picker` gives you an `XFile`, not a `File`.

To show the image we will use `Image.memory(bytes)`.

Do not use `Image.file()`. It needs `dart:io`, and `dart:io` does not exist on the web. Your web build will refuse to compile.

Bytes work everywhere. Same code on Chrome and on a phone, and in Class 3 the upload takes bytes too.

## 2.5 Add the image to validation

Inside `validateForm()`, add this after the category check:

```dart
    // image
    if (imageBytes.value == null) {
      imageError.value = 'Please select a product image';
    } else {
      imageError.value = null;
    }
```

Then add one more line to the return at the bottom:

```dart
    return nameError.value == null &&
        priceError.value == null &&
        descriptionError.value == null &&
        categoryError.value == null &&
        imageError.value == null;
```

---

# STEP 3: CONNECT IT

Go back to `add_product_screen.dart` and replace the widgets from Step 1 with these:

```dart
              // IMAGE BOX
              if (controller.imageBytes.value == null)
                Container(
                  height: 180,
                  decoration: BoxDecoration(
                    border: Border.all(
                      color: controller.imageError.value == null
                          ? Colors.grey
                          : Colors.red,
                    ),
                    borderRadius: BorderRadius.circular(8),
                  ),
                  child: const Center(child: Text('No image selected')),
                )
              else
                Image.memory(
                  controller.imageBytes.value!,
                  height: 180,
                  fit: BoxFit.cover,
                ),

              const SizedBox(height: 8),

              // SELECT IMAGE BUTTON
              OutlinedButton(
                onPressed: controller.pickImage,
                child: const Text('Select Image'),
              ),

              // IMAGE ERROR MESSAGE
              if (controller.imageError.value != null)
                Padding(
                  padding: const EdgeInsets.only(top: 6),
                  child: Text(
                    controller.imageError.value!,
                    style: const TextStyle(color: Colors.red, fontSize: 12),
                  ),
                ),

              const SizedBox(height: 24),
```

Three things to notice:

**`if` inside the children list.** If the condition is true the widget is added, otherwise it is skipped. That is how the grey box turns into the photo.

**`Image.memory`** draws bytes. That is the widget that works on both web and mobile.

**No new `Obx`.** The one from Class 1 already wraps the whole body. It redraws by itself when `imageBytes.value` changes.

---

