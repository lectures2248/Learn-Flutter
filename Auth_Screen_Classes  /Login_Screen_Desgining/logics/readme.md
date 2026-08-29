# LoginController — Step by Step

## Step 1: Empty field check
```dart
if (emailController.text.isEmpty || passwordController.text.isEmpty) {
  Get.snackbar("Error", "Email and Password are required", ...);
  return;
}
```
If email or password is empty, show an error and stop.

## Step 2: Email format check
```dart
else if (!GetUtils.isEmail(emailController.text)) {
  Get.snackbar("Error", "Please Write a Vaild email", ...);
  return;
}
```
`GetUtils.isEmail()` checks the email is a valid format. If not, show an error and stop.

## Step 3: Log in with Supabase
```dart
final response = await Supabase.instance.client.auth.signInWithPassword(
  email: emailController.text,
  password: passwordController.text,
);
final user = response.user;
```
This sends the email/password to Supabase and actually authenticates the user. If successful, `user` contains the logged-in user's data, including their `id`.

## Step 4: Null check
```dart
if (user == null) {
  Get.snackbar("Login Failed", "Invalid email or password", ...);
  return;
}
```
Safety check in case `user` comes back null.

## Step 5: Check if the user is an admin
```dart
final adminCheck = await Supabase.instance.client
    .from('admins')
    .select()
    .eq('id', user.id)
    .maybeSingle();
```
Looks up the `admins` table for a row matching `user.id`.
- `maybeSingle()` returns `null` if no row matches (won't throw an error like `single()` would).

## Step 6: Navigate based on result
```dart
if (adminCheck != null) {
  Get.offAll(() => const AdminDashboard());
} else {
  Get.offAll(() => const HomeScreen());
}
```
- Row found → user is an admin → go to `AdminDashboard`.
- No row → regular user → go to `HomeScreen`.
- `Get.offAll()` clears the navigation stack, so the user can't go back to the login screen.

## Step 7: Error handling
```dart
} on AuthException catch (e) {
  Get.snackbar("Login Failed", e.message, ...);
} catch (e) {
  Get.snackbar("Error", "Something went wrong. Please try again.", ...);
}
```
- `AuthException` catches Supabase auth errors (wrong password, etc.) and shows the specific message.
- Generic `catch` handles any other unexpected error.

## Step 8: Cleanup
```dart
@override
void onClose() {
  emailController.dispose();
  passwordController.dispose();
  super.onClose();
}
```
