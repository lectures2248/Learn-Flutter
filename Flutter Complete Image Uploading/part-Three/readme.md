# Class 3: Uploading the Image and Saving the Product


## Setup

### Products table

Make it in the Supabase dashboard.

Table name: `products`

| Column | Type |
|---|---|
| product_id | int8, primary key, identity |
| product_name | text |
| price | numeric |
| description | text |
| category_id | int8, foreign key to categories.cat_id |
| image_url | text |

### Storage bucket

Storage > New bucket

- Name: `products`
- Turn **Public bucket** on

Then open the bucket policies and allow **insert** for anon.

The bucket must be public. We save a plain URL in the table and open it later in a browser. A private bucket makes that URL stop working.

---

# STEP 1: UPLOAD THE IMAGE

Open `product_controller.dart` and add this function.

```dart
  Future<String> uploadImage() async {
    final fileName =
        '${DateTime.now().millisecondsSinceEpoch}_${pickedFile!.name}';

    await supabase.storage.from('products').uploadBinary(
          fileName,
          imageBytes.value!,
          fileOptions: FileOptions(contentType: pickedFile!.mimeType),
        );

    return supabase.storage.from('products').getPublicUrl(fileName);
  }
```

| Line | Why |
|---|---|
| `millisecondsSinceEpoch` | makes the name unique, so two users uploading `photo.jpg` do not overwrite each other |
| `imageBytes.value!` | the bytes we already read in Class 2 |
| `uploadBinary` | uploads bytes, so it works on web too |
| `contentType` | tells the browser it is a picture, not a file to download |
| `getPublicUrl` | gives the link we save in the table |

The link looks like this:

```
https://yourproject.supabase.co/storage/v1/object/public/products/1747852145_photo.jpg
```

We did not read the file again. We used the same bytes from Class 2.

---

# STEP 2: SAVE THE PRODUCT

Add one variable with your other flags:

```dart
  final isSaving = false.obs;
```

Now delete the temporary `saveProduct` from Class 1 and write this one:

```dart
  Future<void> saveProduct() async {
    if (!validateForm()) return;

    isSaving.value = true;

    try {
      final imageUrl = await uploadImage();

      await supabase.from('products').insert({
        'product_name': nameController.text.trim(),
        'price': double.parse(priceController.text.trim()),
        'description': descriptionController.text.trim(),
        'category_id': int.parse(selectedCategoryId.value!),
        'image_url': imageUrl,
      });

      clearForm();
      showMessage('Success', 'Product saved');
    } catch (e) {
      showMessage('Error', '$e');
    }

    isSaving.value = false;
  }
```

Three things to notice:

**The first line stops everything.** If the form is not valid we return, and no upload happens.

**`isSaving`** disables the button while saving, so the user cannot press Save five times.

**`double.parse`, not `tryParse`.** In Class 1 we used `tryParse` because we were checking the text. Now validation already proved it is a number, so `parse` is safe.

`int.parse(selectedCategoryId.value!)` turns the String back into a number, because `category_id` is a number column. We only made it a String so the dropdown would work.

---

# STEP 3: CLEAR THE FORM

```dart
  void clearForm() {
    nameController.clear();
    priceController.clear();
    descriptionController.clear();
    selectedCategoryId.value = null;
    pickedFile = null;
    imageBytes.value = null;

    nameError.value = null;
    priceError.value = null;
    descriptionError.value = null;
    categoryError.value = null;
    imageError.value = null;
  }
```

Clear the errors too. If you only clear the text, the red messages stay on an empty form.

---

# STEP 4: THE SAVE BUTTON

In `add_product_screen.dart`, change only the button at the bottom:

```dart
              // SAVE BUTTON
              ElevatedButton(
                onPressed: controller.isSaving.value
                    ? null
                    : controller.saveProduct,
                child: controller.isSaving.value
                    ? const CircularProgressIndicator()
                    : const Text('Save Product'),
              ),
```

`onPressed: null` disables a button in Flutter. It turns grey and stops responding.

Nothing else in the screen changes.

---

---
