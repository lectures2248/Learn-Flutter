# Add to Cart - Complete Step by Step Guide

This guide explains, step by step, how the **Add to Cart** feature is built on top of the
**Product Fetching** that is already done.


---

## Step 1: Create the CartItem Model

A "Product" is just the item from the API (id, title, price, image...).
A "Cart Item" is different - it is **a product PLUS a quantity** (how many the user added).

📄 File: `lib/models/cart_item_model.dart`

```dart
import 'product_model.dart';

class CartItem {
  final ProductModel product;
  int quantity;

  CartItem({required this.product, this.quantity = 1});

  double get totalPrice => product.price * quantity;
}
```

### Explanation (easy words)
- `product` → stores the full product info (title, image, price, etc.) so we don't repeat data.
- `quantity` → how many of this product the user added. Starts at `1` by default.
- `totalPrice` → a **getter** (calculated automatically). It is `price × quantity`.
  - Example: price = $10, quantity = 3 → totalPrice = $30.
  - We never store this value, it is always calculated fresh.

> Think of `CartItem` as a "shopping cart card" — it remembers WHICH product and HOW MANY.

---

## Step 2: Add Cart State to the Controller

Open `lib/controllers/product_controller.dart`. This already has `products`, `isLoading`,
`fetchProducts()` etc. Now we add the cart logic in the SAME controller so both screens can
access it.

### 2.1 - Add a list to hold cart items

```dart
var cartItems = <CartItem>[].obs;
```

- `<CartItem>[]` → an empty list that will hold `CartItem` objects.
- `.obs` → makes it **observable**. This is the GetX magic word — it means
  "whenever this list changes, automatically rebuild any UI that is watching it."

### 2.2 - Add two helper getters (for showing totals)

```dart
int get totalItems => cartItems.fold(0, (sum, item) => sum + item.quantity);

double get totalPrice =>
    cartItems.fold(0.0, (sum, item) => sum + item.totalPrice);
```

- `totalItems` → adds up the `quantity` of every cart item.
  - Example: Item A (qty 2) + Item B (qty 1) = totalItems = 3.
  - This number is shown as the badge on the cart icon.
- `totalPrice` → adds up the `totalPrice` of every cart item.
  - This is the grand total shown at the bottom of the Cart screen.
- `.fold(startValue, (sum, item) => ...)` → simple way to "loop and add up" in Dart.
  Start with 0 (or 0.0), then for every item in the list, add something to `sum`.

### 2.3 - Add the `addToCart` method

```dart
void addToCart(ProductModel product) {
  final index = cartItems.indexWhere((item) => item.product.id == product.id);
  if (index >= 0) {
    cartItems[index].quantity++;
    cartItems.refresh();
  } else {
    cartItems.add(CartItem(product: product));
  }
}
```

### Explanation line by line
1. `cartItems.indexWhere((item) => item.product.id == product.id)`
   → Search the cart: "Is this product **already** in the cart?"
   → Returns the **position (index)** of the item if found, or `-1` if not found.
2. `if (index >= 0)` → means YES, this product is already in the cart.
   - `cartItems[index].quantity++` → just increase its quantity by 1 (e.g. 1 → 2).
   - `cartItems.refresh()` → tells GetX "something inside the list changed, please rebuild the UI".
     (We need this because changing `quantity` on an object doesn't automatically notify GetX —
     only adding/removing from the list itself does that automatically.)
3. `else` → means NO, this is a NEW product for the cart.
   - `cartItems.add(CartItem(product: product))` → create a brand new `CartItem` with
     quantity = 1 (default) and add it to the list.

> 💡 In simple words: "If it's already in the cart, just add +1 to its quantity.
> If it's not in the cart yet, add it as a new item."

### 2.4 - Add increase / decrease / remove methods

```dart
void increaseQuantity(int productId) {
  final index = cartItems.indexWhere((item) => item.product.id == productId);
  if (index >= 0) {
    cartItems[index].quantity++;
    cartItems.refresh();
  }
}

void decreaseQuantity(int productId) {
  final index = cartItems.indexWhere((item) => item.product.id == productId);
  if (index < 0) return;
  if (cartItems[index].quantity > 1) {
    cartItems[index].quantity--;
    cartItems.refresh();
  } else {
    cartItems.removeAt(index);
  }
}

void removeFromCart(int productId) {
  cartItems.removeWhere((item) => item.product.id == productId);
}
```

### Explanation
- `increaseQuantity(productId)` → find the item by its product id, and `quantity++`.
- `decreaseQuantity(productId)`:
  - If quantity is more than 1 → just reduce it by 1 (`quantity--`).
  - If quantity is already 1 → remove the item completely from the cart
    (because quantity can't go to 0, that means "no longer in cart").
- `removeFromCart(productId)` → directly deletes the item from the cart list, no matter
  what the quantity is. `removeWhere` automatically notifies GetX (no `.refresh()` needed)
  because we are changing the LIST itself, not a property inside an item.

>  Important rule to remember:
> - Changing the **list** (`add`, `removeAt`, `removeWhere`) → GetX detects it automatically.
> - Changing a **property inside an object** (`item.quantity++`) → you MUST call
>   `cartItems.refresh()` yourself, otherwise the UI won't update.

---

## Step 3: Update the Product Screen UI (Add to Cart button)

Open `lib/views/product_view.dart`. The product list (`ListView.builder`) is already working
from the fetching step. Now we add a button on each product row.

### 3.1 - Add the "Add to Cart" button per product

```dart
trailing: Obx(() {
  final index = controller.cartItems.indexWhere((item) => item.product.id == product.id);
  final qty = index >= 0 ? controller.cartItems[index].quantity : 0;
  return ElevatedButton(
    onPressed: () => controller.addToCart(product),
    child: Text(qty > 0 ? 'Added ($qty)' : 'Add to Cart'),
  );
}),
```

### Explanation
- `Obx(() { ... })` → a special GetX widget. It means "watch the observable variables used
  inside, and rebuild ONLY this small widget whenever they change" (not the whole screen).
- `controller.cartItems.indexWhere(...)` → check if THIS product is already in the cart.
- `qty` → how many of this product are currently in the cart (0 if not added yet).
- `ElevatedButton`:
  - `onPressed: () => controller.addToCart(product)` → when tapped, call the method we
    wrote in Step 2.3.
  - `child: Text(...)` → shows **"Add to Cart"** if qty is 0,
    or **"Added (qty)"** if it's already in the cart (e.g. "Added (3)").

> 💡 Because this button is wrapped in its OWN `Obx`, only the button text updates when you
> tap it — the rest of the product row (image, title, price) does NOT rebuild. This is good
> for performance.

### 3.2 - Add a Cart icon (with item count badge) to open the Cart screen

```dart
floatingActionButton: Obx(() => FloatingActionButton(
      backgroundColor: Colors.amber,
      onPressed: () => Get.to(() => CartView()),
      child: Badge(
        label: Text('${controller.totalItems}'),
        isLabelVisible: controller.totalItems > 0,
        child: const Icon(Icons.shopping_cart),
      ),
    )),
```

### Explanation
- `floatingActionButton` → the round button that floats at the bottom-right of the screen.
- Wrapped in `Obx(() => ...)` → so the badge number updates live whenever `totalItems` changes.
- `Badge`:
  - `label: Text('${controller.totalItems}')` → shows the total quantity of all items
    (e.g. "3") as a small red circle on top of the cart icon.
  - `isLabelVisible: controller.totalItems > 0` → only show the badge number if the cart
    is NOT empty. If cart is empty, badge is hidden.
  - `child: Icon(Icons.shopping_cart)` → the cart icon itself.
- `onPressed: () => Get.to(() => CartView())` → when tapped, **navigate** to the `CartView`
  screen (this is GetX's simple way of doing `Navigator.push`).

---

## Step 4: Build the Cart Screen UI

Create / open `lib/views/cart_view.dart`.

### 4.1 - Get access to the SAME controller

```dart
class CartView extends StatelessWidget {
  CartView({super.key});

  final ProductController controller = Get.find();
  ...
}
```

### Explanation
- `Get.find()` → "find the controller that ALREADY EXISTS" (the same one created in
  `ProductView` using `Get.put()`).
- This is the key reason the cart works across screens — `ProductView` and `CartView`
  are both talking to the **exact same `ProductController` instance**, so they share the
  same `cartItems` list.

### 4.2 - Show "Cart is empty" message OR the list of cart items

```dart
body: Obx(() {
  if (controller.cartItems.isEmpty) {
    return const Center(child: Text('Your cart is empty'));
  }

  return ListView.builder(
    itemCount: controller.cartItems.length,
    itemBuilder: (context, index) {
      final item = controller.cartItems[index];
      return ListTile(
        leading: Image.network(
          item.product.image,
          width: 50,
          height: 50,
          fit: BoxFit.cover,
        ),
        title: Text(item.product.title, maxLines: 1, overflow: TextOverflow.ellipsis),
        subtitle: Text('\$${item.totalPrice.toStringAsFixed(2)}'),
        trailing: Row(
          mainAxisSize: MainAxisSize.min,
          children: [
            IconButton(
              icon: const Icon(Icons.remove_circle_outline),
              onPressed: () => controller.decreaseQuantity(item.product.id),
            ),
            Text('${item.quantity}'),
            IconButton(
              icon: const Icon(Icons.add_circle_outline),
              onPressed: () => controller.increaseQuantity(item.product.id),
            ),
          ],
        ),
      );
    },
  );
}),
```

### Explanation
- `Obx(() { ... })` → wraps the WHOLE body so it rebuilds whenever `cartItems` changes
  (item added, removed, quantity changed).
- `if (controller.cartItems.isEmpty)` → if the cart has nothing, just show
  **"Your cart is empty"** text in the center.
- `ListView.builder` → builds one row per cart item.
- Each row (`ListTile`):
  - `leading` → product image (small thumbnail).
  - `title` → product name.
  - `subtitle` → shows `totalPrice` for THIS item (price × quantity), e.g. "$30.00".
  - `trailing` → a small `Row` with 3 widgets:
    - `-` button → calls `decreaseQuantity(productId)`.
    - `Text('${item.quantity}')` → shows current quantity (e.g. "2").
    - `+` button → calls `increaseQuantity(productId)`.

> 💡 Try it: Tap `-` until quantity becomes 0 → the item disappears from the cart
> automatically (because of the logic we wrote in `decreaseQuantity`).

### 4.3 - Show the Grand Total at the bottom

```dart
bottomNavigationBar: Obx(() {
  if (controller.cartItems.isEmpty) return const SizedBox.shrink();
  return Padding(
    padding: const EdgeInsets.all(16),
    child: Text(
      'Total: \$${controller.totalPrice.toStringAsFixed(2)}',
      style: const TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
    ),
  );
}),
```

### Explanation
- `bottomNavigationBar` → a widget pinned to the bottom of the screen.
- If cart is empty → `SizedBox.shrink()` → shows nothing (an invisible widget of size 0).
- Otherwise → shows **"Total: $XX.XX"** using `controller.totalPrice` (the getter we wrote
  in Step 2.2, which adds up all `item.totalPrice` values).
- `.toStringAsFixed(2)` → formats the number to always show 2 decimal places (e.g. `30.5`
  becomes `"30.50"`).

---

## Step 5: Make sure everything is wired together (main.dart)

📄 File: `lib/main.dart`

```dart
import 'package:flutter/material.dart';
import 'package:get/get.dart';
import 'views/product_view.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return GetMaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Product App',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
      ),
      home: ProductView(),
    );
  }
}
```

### Explanation
- We MUST use `GetMaterialApp` instead of plain `MaterialApp`.
  Why? Because `Get.to()` (navigation) and `Get.find()` / `Get.put()` (controller sharing)
  only work properly when the app is wrapped in `GetMaterialApp`.
- `home: ProductView()` → the first screen shown is the product list screen.
- Inside `ProductView`, the line `Get.put(ProductController())` creates the controller
  ONE TIME and makes it available everywhere (including `CartView`) via `Get.find()`.

---
https://www.mediafire.com/file/o203fl6vl1unvtw/productapp.rar/file
