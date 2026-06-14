## **PART 1 - PRODUCT FORM WITH CATEGORY DROPDOWN** 

Before creating CRUD operations, we first need a proper form. 

Our product form will contain: 

1. Product Name 

2. Product Price 

3. Product Description 

4. Category Dropdown 

5. Product Image 

Today we will focus on building the form correctly before connecting database. 

Why? 

Because every CRUD application starts with collecting user input. 

## **STEP 1 : CREATE CONTROLLERS** 

Inside State Class: 

final productNameController = TextEditingController(); 

final productPriceController = TextEditingController(); 

final productDescriptionController = TextEditingController(); 

Explanation: 

Controllers help us read values from TextFields. 

Example: 

User types: 

Laptop 

Controller stores: 

1 

Laptop 

## **STEP 2 : CREATE FORM KEY** 

final _formKey = GlobalKey<FormState>(); 

Explanation: 

Form Key is used for validation. 

Without Form Key: 

We cannot easily validate all fields together. 

## **STEP 3 : CREATE CATEGORY LIST** 

List categories = []; 

String? selectedCategoryId; 

Explanation: 

categories 

Stores categories fetched from database. 

Example: 

[ {cat_id:1,category_name:"Electronics"}, {cat_id:2,category_name:"Sports"} ] 

selectedCategoryId 

Stores selected dropdown value. 

Example: 

1 

## **STEP 4 : CREATE FORM WIDGET** 

Form( key: _formKey, child: Column( children: [ 

2 

```
],
```

), ) 

Explanation: 

All fields should be inside Form widget. 

This allows validation. 

## **STEP 5 : PRODUCT NAME FIELD** 

TextFormField( controller: productNameController, 

decoration: const InputDecoration( labelText: "Product Name", border: OutlineInputBorder(), ), 

validator: (value){ 

```
if(value==null||
value.isEmpty){
return
"Product name required";
}
returnnull;
```

}, ) 

Explanation: 

validator runs when user submits form. 

If field is empty: 

Error appears. 

If field has value: 

Validation passes. 

3 

## **STEP 6 : PRODUCT PRICE FIELD** 

TextFormField( controller: productPriceController, 

keyboardType: TextInputType.number, 

decoration: const InputDecoration( labelText: "Price", border: OutlineInputBorder(), ), 

validator: (value){ 

```
if(value==null||
value.isEmpty){
return"Price required";
}
if(double.tryParse(value)
==null){
return
"Enter valid price";
}
returnnull;
```

}, ) 

Explanation: 

double.tryParse() 

Checks whether entered value is a number. 

Valid: 

500 2000 999.99 

Invalid: 

abc 

4 

hello 

price 

## **STEP 7 : DESCRIPTION FIELD** 

TextFormField( controller: productDescriptionController, 

maxLines: 4, 

decoration: const InputDecoration( labelText: "Description", border: OutlineInputBorder(), ), 

validator: (value){ 

```
if(value==null||
value.isEmpty){
return
"Description required";
}
if(value.length<10){
return
"Minimum 10 characters";
}
returnnull;
```

}, ) 

Explanation: 

Description should contain enough detail. 

Bad: 

Good 

Nice 

Best 

5 

Valid: 

Gaming laptop with 16GB RAM 

## **STEP 8 : CATEGORY DROPDOWN** 

DropdownButtonFormField<String>( 

value: selectedCategoryId, 

items: categories.map((category){ 

```
returnDropdownMenuItem(
value:
category["cat_id"]
.toString(),
child:Text(
category["category_name"],
),
);
```

}).toList(), 

onChanged: (value){ 

```
setState(() {
  selectedCategoryId =
      value;
});
```

}, 

decoration: const InputDecoration( labelText: "Category", border: OutlineInputBorder(), ), 

validator: (value){ 

```
if(value==null){
```

6 

```
return
"Please select category";
}
returnnull;
```

}, ) 

Explanation: 

Dropdown displays categories. 

Example: 

Electronics Sports Books 

User selects: 

Electronics 

selectedCategoryId stores: 

1 

## **STEP 9 : SAVE BUTTON** 

ElevatedButton( 

onPressed: (){ 

```
if(_formKey.currentState!
    .validate()){
  print("Form Valid");
}
```

}, 

child: const Text("Save"), ) 

Explanation: 

validate() 

7 

Runs all validators. 

If every field passes: 

Save operation starts. 

If any field fails: 

Error messages appear. 

## **PART 2 - IMAGE PICKER USING XFILE ** 

We will use: 

image_picker 

Why XFile? 

Because it works on: 

Android iOS Web Windows Mac 

Same code everywhere. 

## **STEP 1 : INSTALL PACKAGE** 

pubspec.yaml 

image_picker: ^1.1.2 

Run: 

flutter pub get 

## **STEP 2 : IMPORT PACKAGE** 

import 'package:image_picker/image_picker.dart'; 

8 

## **STEP 3 : CREATE VARIABLES** 

final ImagePicker picker = ImagePicker(); 

XFile? selectedImage; 

Explanation: 

selectedImage stores picked image. 

Initially: 

null 

After selection: 

image stored 

## **STEP 4 : CREATE PICK IMAGE FUNCTION** 

Future<void> pickImage() async { 

final XFile? image = await picker.pickImage( 

```
source: ImageSource.gallery,
```

); 

if(image != null){ 

```
setState(() {
  selectedImage = image;
});
```

} 

} 

================================================== UNDERSTANDING EACH LINE 

================================================== 

9 

pickImage() 

Opens gallery. 

ImageSource.gallery 

Select image from gallery. 

await 

Wait for user selection. 

if(image != null) 

User selected image. 

setState() 

Refresh UI. 

================================================== STEP 5 : IMAGE PICK BUTTON ================================================== 

ElevatedButton( 

onPressed: pickImage, 

child: const Text("Select Image"), 

) 

## **STEP 6 : DISPLAY IMAGE** 

selectedImage == null 

? const Text("No Image Selected") 

Image.network( selectedImage!.path, height: 200, 

) 

10 

Explanation: 

Web uses URL style path. 

XFile works nicely on web. 

For mobile later you can use: 

Image.file() 

when required. 
## **PART 3 - IMAGE UPLOAD TO SUPABASE ** 


Now we will upload image to Storage. 

After upload: 

Image URL will be stored in database. 

Flow: 

Select Image ↓ Upload To Storage ↓ Get Public URL ↓ Insert Product Data ↓ Save URL In Database 

## **STEP 1 : CREATE STORAGE BUCKET** 

Bucket Name: 

products 

Make Bucket Public. 

Why Public? 

Because images should display directly. 

## **STEP 2 : CREATE UPLOAD FUNCTION** 

Future<String?> uploadImage() async { 

if(selectedImage == null){ 

11 

```
returnnull;
```

} 

final bytes = await selectedImage! .readAsBytes(); 

final fileName = DateTime.now() .millisecondsSinceEpoch .toString(); 

await supabase.storage 

```
.from('products')
.uploadBinary(
fileName,
bytes,
);
```

final imageUrl = 

```
  supabase.storage
  .from('products')
  .getPublicUrl(
      fileName);
```

return imageUrl; } 

================================================== UNDERSTANDING EACH LINE ================================================== 

readAsBytes() 

Converts image into bytes. 

Storage uploads bytes. 

fileName 

12 

Creates unique image name. 

Example: 

1747852145 

uploadBinary() 

Uploads image. 

Works on: 

Web Android iOS 

getPublicUrl() 

Generates image URL. 

Example: 

https://xxxxx.supabase.co/storage/v1/object/public/products/image1.jpg 

================================================== STEP 3 : CREATE SAVE PRODUCT FUNCTION ================================================== 

Future<void> saveProduct() async { 

if(!_formKey.currentState! .validate()){ 

```
return;
```

} 

final imageUrl = await uploadImage(); 

await supabase 

```
  .from("products")
```

```
  .insert({
```

13 

```
"product_name":
    productNameController.text,
"price":
    double.parse(
      productPriceController.text,
    ),
"description":
    productDescriptionController.text,
"category_id":
    int.parse(
      selectedCategoryId!,
    ),
"image_url":
    imageUrl,
```

}); } ================================================== COMPLETE SAVE FLOW ================================================== 

User Opens Form ↓ 

Enter Product Name ↓ 

Enter Price ↓ 

Enter Description ↓ Select Category ↓ 

Select Image ↓ 

Validation Runs ↓ 

Image Uploads To Storage ↓ 

Public URL Generated ↓ 

Product Inserted Into Database ↓ 

14 

Product Saved Successfully 

================================================== WHAT STUDENTS LEARNED 

================================================== 

## 1. Form Validation 

TextFormField Validator Form Key 

## 1. Dropdown Handling 

DropdownButtonFormField 

Selected Category 

1. Cross Platform Image Picker 

Image Picker 

XFile 

## 1. Supabase Storage 

uploadBinary() 

getPublicUrl() 

## 1. Product Insert Logic 

Insert Product Data 

Save Image URL 

Connect Product With Category 

This is the foundation of every real-world E-Commerce, Inventory Management, POS, and Product Management application. 

15 

