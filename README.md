# Flutter tips and snippets

A list of personal flutter tips and snippets

## VSCode snippets

<details>
  <summary>dart.json</summary>

``` json
{
	// Place your snippets for dart here. Each snippet is defined under a snippet name and has a prefix, body and 
	// description. The prefix is what is used to trigger the snippet and the body will be expanded and inserted. Possible variables are:
	// $1, $2 for tab stops, $0 for the final cursor position, and ${1:label}, ${2:another} for placeholders. Placeholders with the 
	// same ids are connected.
	// Example:
	// "Print to console": {
	// 	"prefix": "log",
	// 	"body": [
	// 		"console.log('$1');",
	// 		"$2"
	// 	],
	// 	"description": "Log output to console"
	// }
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
  }
}
```

</details>

## Stateful class with parameters:

<details>
  <summary>Code</summary>

``` flutter
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

``` Flutter
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

**Reference**: https://stackoverflow.com/questions/51605131/how-to-add-the-widgets-dynamically-to-column-in-flutter
<details>
  <summary>Code</summary>

``` flutter
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

``` flutter
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

## Firebase

### Firebase cloud database rules example
**Reference**: https://firebase.google.com/docs/firestore/security/rules-conditions

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

</details>

### Firebase storage rule example

**Reference**: https://firebase.google.com/docs/reference/security/storage

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

