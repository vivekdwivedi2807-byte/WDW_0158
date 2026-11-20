# Live messaging setup (Firebase Realtime Database)

The chat interface is in `index.html`. `chat.HTML` remains the passcode screen and opens `index.html` after the demo code is accepted.

## 1. Create the Firebase services

1. Open the Firebase Console and create a project.
2. Add a Web App in Project settings and copy its config values into `firebase-config.js`.
3. In **Authentication > Sign-in method**, enable **Anonymous** sign-in.
4. In **Realtime Database**, create a database and copy its URL into `firebase-config.js` as `databaseURL`.
5. In **Realtime Database > Rules**, replace the rules with the rules below and publish them.

## 2. Database rules for this demo

```json
{
  "rules": {
    "rooms": {
      "$roomCode": {
        "members": {
          "$uid": {
            ".write": "auth != null && auth.uid === $uid && newData.val() === true",
            ".validate": "newData.val() === true"
          }
        },
        "messages": {
          ".read": "auth != null && root.child('rooms').child($roomCode).child('members').child(auth.uid).val() === true",
          "$messageId": {
            ".write": "auth != null && root.child('rooms').child($roomCode).child('members').child(auth.uid).val() === true && !data.exists() && newData.child('uid').val() === auth.uid",
            ".validate": "newData.hasChildren(['uid', 'name', 'text', 'createdAt'])",
            "uid": { ".validate": "newData.isString()" },
            "name": { ".validate": "newData.isString() && newData.val().length <= 30" },
            "text": { ".validate": "newData.isString() && newData.val().length <= 1000" },
            "createdAt": { ".validate": "newData.isNumber()" },
            "$other": { ".validate": false }
          }
        }
      }
    }
  }
}
```

The room invite code is a secret link-like code: share it only with the person you want to join. Anyone who gets the code can join this learning demo. It is not a substitute for account-based access control.

## 3. Run it

Open `index.html` with VS Code's **Live Server** extension. The other person must open the same hosted page, enter their display name, and join using the 24-character invite code. Do not test by opening the HTML as a `file://` URL; browser modules and Firebase need a local web server or deployed HTTPS site.

This prototype sends messages through Firebase Realtime Database. It does not encrypt chat contents end-to-end, so use sample messages only.
