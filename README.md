# Flutter tips and snippets

A list of personal flutter tips and snippets
**NOTE** All this snippets here was tested and/or have some personalizations, but there're the right reference to all sources where code is found

## VSCode snippets

<details>
  <summary>dart.json</summary>

``` json
{
  "Provider": {
  "prefix": "provider",
    "body": [
      "Provider.of<$1>(context).$2"
    ]
  },
  "Widget": {
    "prefix": "widget",
    "body": [
      "Widget $1() {$2}"
    ]
  },
  "Stateful": {
    "prefix": "stateful",
    "body": [
      "class $1 extends StatefulWidget {",
      "@override",
      "State<StatefulWidget> createState() => $1State();",
      "}",
      "class $1State extends State<$1> {",
      "@override",
      "Widget build(BuildContext context) {",
      "return $2",
      "}",
      "}"
    ]
  },
  "Stateless": {
    "prefix": "stateless",
    "body": [
      "class $1 extends StatelessWidget {",
      "@override",
      "Widget build(BuildContext context) {",
      "return $2",
      "}",
      "}"
    ]
  },
  "Future": {
    "prefix": "future",
    "body": [
      "Future<$1> $2($3) async {",
      "return $4;",
      "}"
    ]
  }
}
```

</details>

## Stateful class with parameters

<details>
  <summary>Code</summary>

``` dart
class MyClass extends StatefulWidget {
  final Type myField;

  const MyClass({Key key, this.myField}) : super(key: key);
  @override
  State<StatefulWidget> createState() => MyClassState();
}

class MyClassState extends State<ResultCard> {
  ...
}
```

</details>

Then: `MyClass _myinstance = new MyClass(myField: "myValue");` 

## Create a list based on array of results:

<details>
  <summary>Code</summary>

``` dart
ListView.builder(
  shrinkWrap: true, // use this to avoid rendere in column error
  padding: const EdgeInsets.all(8),
  itemCount: _getMyResultFunction,
  itemBuilder: (BuildContext context, int index) {
    return Container(
      height: 100,
      child: Center(child: Text(_searchResults[index].data["name"])),
    );
  })
```

</details>

### Dynamically add elements to list

**Reference**: <https://stackoverflow.com/questions/51605131/how-to-add-the-widgets-dynamically-to-column-in-flutter>
<details>
  <summary>Code</summary>

``` dart
  // 
  var _searchResults = List<Widget>();

  ...

  for (var i = 0; i < results.length; i++) {
    _searchResults.add(resultCard(results[i].data));
  }

  ...

  // clear the list
  _searchResults.clear();
```

</details>

## Layout

### **Stack** widget is useful to create overlap widgets

<details>
<summary>Code</summary>

``` dart
    return Directionality(
        textDirection: TextDirection.ltr,
        // stack allow widget overlapping for gamepad
        child: Stack(children: [
          level.widget,
          Joypad(
            onChange: (Offset delta) => level.movePlayer(delta),
          )
        ]));
  }
  ```

</details>

### Scrollable view

Be sure you have in the root widget:

``` dart
SingleChildScrollView(
  child: ...
)
```

### Visibility

**Reference**: <https://www.woolha.com/tutorials/flutter-hide-show-widget-using-visibility>

``` dart
Visibility(
  visible: CONDITION,
  child: ...
)
```

**Explain**: `CONDITION` is a that return `true` 
You can use this method to return a widget based on condition:

``` dart
Widget myTest() {
  return CONDITION
    ? Text("Visible if true")
    : Text("Visible if false");

}
```

### Dialogs

**Reference**: <https://medium.com/@nils.backe/flutter-alert-dialogs-9b0bb9b01d28>

``` dart
showDialog(
  context: context,
  builder: (BuildContext context) {
    // return object of type Dialog
    return AlertDialog(
      title: new Text("Alert Dialog title"),
      content: new Text("Alert Dialog body"),
      actions: <Widget>[
        // usually buttons at the bottom of the dialog
        new FlatButton(
          child: new Text("Close"),
          onPressed: () {
            Navigator.of(context).pop();
          },
        ),
      ],
    );
  },
);
```

## Firebase

### Firebase database - rules example

**Reference**: <https://firebase.google.com/docs/firestore/security/rules-conditions>

**Explain**: Create a collection `users` where not authenticated user can create an user, but only the owner can read, update and delete is informations. Create a collection `mycollection` where read is public, update and delete is allowed only if the authenticated user has the same id passed in `owner` field in the request data and create only if the authenticated user role is equal to `allowedRole` .
 
<details>
<summary>Code</summary>

``` text
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    match /users/{userId} {
      allow read, update, delete: if request.auth.uid == userId;
      allow create: if request.auth.uid != null;
    }
    match /mycollection/{myId} {
      allow create: if get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == "allowedRole"
      allow update, delete: if request.auth.uid == resource.data.owner;
      allow read: if request.auth.uid != null;
    }
  }
}
```

**IMP** `userId` in the record is the firebase documentID that is passed to the rule with `{userId}` , so you can pass the documentID and use it to check rule policy

</details>

### Firebase database - use stream widget

**Reference**: <https://inducesmile.com/google-flutter/how-to-use-streambuilder-with-firestore-in-flutter/>

<details>
<summary>Code</summary>

``` dart
StreamBuilder(
  stream: _myGetStreamFromFirebaseFunction,
  builder: (context, snapshot) {
    if (!snapshot.hasData) {
      return Text("Loading..");
    }
    // Your data is in snapshot.data
    // Your error is in snapshot.error
  },
),
```

</details>

**Extra**: in the get function insert something like:

``` dart
Stream<DocumentSnapshot> results = Firestore.instance.collection('mycollection').document(id).snapshots();`
```

### Firebase database - query

#### Get document in a collection based on id and transform to map

``` dart
Map request = await Firestore.instance
  .collection("mycollection")
  .document(id)
  .get()
  .then((DocumentSnapshot ds) => ds.data);
```

#### Get documents by query and transform to list

**Reference**: <https://stackoverflow.com/questions/57152017/how-do-i-convert-streamquerysnapshot-to-listmyobject>

``` dart
QuerySnapshot request = await Firestore.instance
  .collection("mycollection")
  .where('field', isEqualTo: "value")
  .limit(10) // optional
  .getDocuments();
List<Tip> docs = request.documents
  .map((doc) => MyObjectClass(doc.documentID, doc.data["..."], ....))
  .toList();
```

**Extra**: You can refer to id in where clause using FieldPath.documentId, example used to get all records with id in an array:

``` dart
.where(FieldPath.documentId, whereIn: ["..", ...])
```

**Reference**: <https://stackoverflow.com/questions/52468310/is-fieldpath-supported-in-flutter>

#### Delete a document by id

``` dart
await Firestore.instance.collection('mycollection').document(id).delete();
```

### Update a document by id

``` dart
await Firestore.instance.collection('mycollection').document(id).updateData({...});
```

### Firebase storage - rule example

**Reference**: <https://firebase.google.com/docs/reference/security/storage>

**Explain**: only authenticated user can read, user with `allowedRole` passed as metadata can create files, and only file owner can update or delete the file

<details>
<summary>Code</summary>

``` text
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /{allPaths=**} {
      allow read: if request.auth != null;
      allow create: if request.auth != null && request.resource.metadata.role == "allowedRole";
      allow update, delete: if resource.metadata["owner"] == request.auth.uid;
    }
  }
}
```

</details>

## MISC

### Upload a file on android emulator

Open Andorid Studio. Go to "Device File Explorer" which is on the bottom right of android studio, if you can't find it, use the search feature from the top-right button.
If you have more than one device connected, select the device you want from the drop-down list on top.
mnt>sdcard is the location for SD card on the emulator.
Right click on the folder and click Upload. See the image below.

### Dart class with optional parameter
``` dart
class MyInfo {
  String id;
  String value;
  String optional;

  MyInfo(String id, String value, {String optional = ''}) {
    this.id = id;
    this.value = value;
    this.optional = optional
  }
}
MyInfo complete = myInfo("test", "test", optional: "test");
MyInfo incomplete = myInfo("test1", "test1");
```
