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

## Provider (manage state change)

**Reference**: <https://alligator.io/flutter/state-management/>

Dependencies: [provider](https://pub.dev/packages/provider)

<details>
  <summary>lib/shared/provider.dart</summary>

``` dart
import 'package:flutter/material.dart';

class Data extends ChangeNotifier {
  Map data = {};

  void updateData(input) {
    data = input;
    notifyListeners();
  }
}

```

</details>

<details>
  <summary>lib/main.dart</summary>

``` dart
import 'package:mypackage/shared/provider.dart';
...
class _MyHomePageState extends State<MyHomePage> {
  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider<Data>(
      ...
    )
  }
}
```

</details>

<details>
  <summary>use provider</summary>

``` dart
import 'package:provider/provider.dart';
import 'package:mypackage/shared/provider.dart';
...
// Update data in provider
Provider.of<Data>(context).updateData({...});
// Get data from provider
print(Provider.of<Data>(context).data['myfield']);
```

</details>

## Multilanguage

**Reference**: <https://medium.com/flutter-community/flutter-internationalization-the-easy-way-using-provider-and-json-c47caa4212b2>
**NOTE**: This example use multiple provider that allow a different provider for data management (similar to the code in the `Provider` section)

Dependencies: [provider](https://pub.dev/packages/provider), [shared_preferences](https://pub.dev/packages/shared_preferences)

Create your locale files as json in `locale/` and remember to add them in `assets` section in `pubspec.yaml` .

<details>
  <summary>locale/en.json</summary>

``` json
{
  "welcome": "Welcome!"
}
```

</details>

<details>
  <summary>shared/provider.dart</summary>

``` dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

...

class AppLanguage extends ChangeNotifier {
  Locale _appLocale = Locale('en');

  Locale get appLocal => _appLocale ?? Locale("en");
  fetchLocale() async {
    var prefs = await SharedPreferences.getInstance();

    if (prefs.getString('language_code') == null) {
      _appLocale = Locale('en');
      return Null;
    }
    _appLocale = Locale(prefs.getString('language_code'));
    return Null;
  }

  void changeLanguage(Locale type) async {
    var prefs = await SharedPreferences.getInstance();
    if (_appLocale == type) {
      return;
    }
    if (type == Locale("it")) {
      _appLocale = Locale("it");
      await prefs.setString('language_code', 'it');
      await prefs.setString('countryCode', 'IT');
    } else {
      _appLocale = Locale("en");
      await prefs.setString('language_code', 'en');
      await prefs.setString('countryCode', 'US');
    }
    notifyListeners();
  }
}
```

</details>

<details>
  <summary>lib/shared/localization.dart</summary>

``` dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart'; // for rootBundle
import 'dart:convert'; // for json

class AppLocalizations {
  final Locale locale;

  AppLocalizations(this.locale);

  // Helper method to keep the code in the widgets concise
  // Localizations are accessed using an InheritedWidget "of" syntax
  static AppLocalizations of(BuildContext context) {
    return Localizations.of<AppLocalizations>(context, AppLocalizations);
  }

  // Static member to have a simple access to the delegate from the MaterialApp
  static const LocalizationsDelegate<AppLocalizations> delegate =
  _AppLocalizationsDelegate();

  Map<String, String> _localizedStrings;

  Future<bool> load() async {
    // Load the language JSON file from the "lang" folder
    String jsonString =
    await rootBundle.loadString('locale/${locale.languageCode}.json');
    Map<String, dynamic> jsonMap = json.decode(jsonString);

    _localizedStrings = jsonMap.map((key, value) {
      return MapEntry(key, value.toString());
    });

    return true;
  }

  // This method will be called from every widget which needs a localized text
  String translate(String key) {
    return _localizedStrings[key];
  }
}

class _AppLocalizationsDelegate
    extends LocalizationsDelegate<AppLocalizations> {
  // This delegate instance will never change (it doesn't even have fields!)
  // It can provide a constant constructor.
  const _AppLocalizationsDelegate();

  @override
  bool isSupported(Locale locale) {
    // Include all of your supported language codes here
    return ['en', 'it'].contains(locale.languageCode);
  }

  @override
  Future<AppLocalizations> load(Locale locale) async {
    // AppLocalizations class is where the JSON loading actually runs
    AppLocalizations localizations = new AppLocalizations(locale);
    await localizations.load();
    return localizations;
  }

  @override
  bool shouldReload(_AppLocalizationsDelegate old) => false;
}
```

</details>

<details>
  <summary>lib/main.dart</summary>

``` dart
...

import 'package:flutter_localizations/flutter_localizations.dart';
import 'package:provider/provider.dart';
import 'package:mypackage/shared/localization.dart';
import 'package:mypackage/shared/provider.dart';
...
void main() async {
  WidgetsFlutterBinding.ensureInitialized(); // avoid error
  AppLanguage appLanguage = AppLanguage();
  await appLanguage.fetchLocale();
  runApp(MyApp(
    appLanguage: appLanguage,
  ));
//  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  final AppLanguage appLanguage;

  MyApp({this.appLanguage});

  @override
  Widget build(BuildContext context) {
    return MultiProvider(
        providers: [
          ChangeNotifierProvider<Data>(
            create: (context) => Data(),
          ),
          ChangeNotifierProvider<AppLanguage>(
            create: (context) => AppLanguage(),
          ),
        ],
        child: Consumer<AppLanguage>(builder: (context, model, child) {
          return MaterialApp(
            locale: model.appLocal,
            title: 'Title',
            theme: ThemeData(
              primarySwatch: Colors.blue,
            ),
            supportedLocales: [
              Locale('en', 'US'),
              Locale('it', 'IT'),
            ],
            localizationsDelegates: [
              AppLocalizations.delegate,
              GlobalMaterialLocalizations.delegate,
              GlobalWidgetsLocalizations.delegate,
            ],
            home: MyHomePage(title: 'Title'),
            navigatorObservers: <NavigatorObserver>[observer],
          );
        }));
  }
}

```

</details>

</details>
<details>
  <summary>lib/pages/settings.dart (optional settings page)</summary>

``` dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'package:mypackage/shared/provider.dart';
import 'package:mypackage/shared/localization.dart';

class SettingsPage extends StatefulWidget {
  final String title = 'Settings';

  @override
  State<StatefulWidget> createState() => _SettingsPageState();
}

class _SettingsPageState extends State<SettingsPage> {
  String _selectedLanguage;
  List _availableLanguages = ["en", "it"];

  List<DropdownMenuItem<String>> _dropDownMenuItems;

  @override
  void initState() {
    _dropDownMenuItems = getDropDownMenuItems();
    _selectedLanguage = _dropDownMenuItems[0].value;
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Builder(builder: (BuildContext context) {
        return ListView(
          scrollDirection: Axis.vertical,
          children: <Widget>[
            Text(AppLocalizations.of(context).translate('language')),
            Center(
              child: new DropdownButton(
                value: _selectedLanguage,
                items: _dropDownMenuItems,
                onChanged: changedDropDownItem,
              ),
            ),
          ],
        );
      }),
    );
  }

  List<DropdownMenuItem<String>> getDropDownMenuItems() {
    List<DropdownMenuItem<String>> items = new List();
    for (String role in _availableLanguages) {
      items.add(new DropdownMenuItem(value: role, child: new Text(role)));
    }
    return items;
  }

  void changedDropDownItem(String selectedLanguage) {
    var appLanguage = Provider.of<AppLanguage>(context);
    appLanguage.changeLanguage(Locale(selectedLanguage));
    setState(() {
      _selectedLanguage = selectedLanguage;
    });
  }
}
```

</details>

<details>
  <summary>use translation</summary>

``` dart
import 'package:mypackage/shared/localization.dart';
...
print(AppLocalizations.of(context).translate('welcome'));
```

</details>

## Create a list based on array of results

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

### Progress

* circular loading widget: `CircularProgressIndicator()` 

### Stack widget is useful to create overlap widgets

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

Be sure you have as root widget in body definition:

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

**Explain**: `CONDITION` must return `true` or `false` 
You can use this method to return a widget based on condition:

``` dart
Widget myTest() {
  return CONDITION
    ? Text("Visible if true")
    : Text("Visible if false");

}
```

### Switch
**Reference**: <https://stackoverflow.com/questions/53066664/flutter-android-preferencescreen-settings-page>
``` dart
SwitchListTile(
  value: false,
  title: Text("This is a SwitchPreference"),
  onChanged: (value) {},
),
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
List<MyObjectClass> docs = request.documents
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

#### Update a document by id

``` dart
await Firestore.instance.collection('mycollection').document(id).updateData({...});
```

#### Update with field remove

``` dart
await Firestore.instance.collection('mycollection').document(id).updateData({myUnusedField: FieldValue.delete(), ...});
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

### Firebase storage - store images

**Reference**: <https://www.c-sharpcorner.com/article/upload-image-file-to-firebase-storage-using-flutter>

Dependencies: [cached_network_image](https://pub.dev/packages/cached_network_image), [image_picker](https://pub.dev/packages/image_picker), [firebase_storage](https://pub.dev/packages/firebase_storage)

**NOTE**: This example use the firebase storage rule defined in the other section

<details>
<summary>lib/shared/file_storage.dart</summary>

``` dart
import 'package:firebase_storage/firebase_storage.dart';
import 'dart:io';
import 'package:path/path.dart' as Path;

class StorageRepository {
  Future<Map<String, String>> uploadImage(File _image, String owner, String role) async {
    if (role != "allowedRole") {
      throw 'Not allowed';
    }
    String fileName = 'mydirectory/${Path.basename(_image.path)}';
    StorageReference storageReference = FirebaseStorage.instance
        .ref()
        .child(fileName);
    StorageUploadTask uploadTask = storageReference.putFile(
      _image,
      StorageMetadata(
        customMetadata: <String, String>{'owner': owner, 'role': role},
      ),
    );
    await uploadTask.onComplete;
    String fileURL = await storageReference.getDownloadURL();
    return {"fileUrl": fileURL, "fileName": fileName};
  }

  Future<void> deleteImage(String filename, String role) async {
    if (role != "allowedRole") {
      throw 'Not allowed';
    }
    final StorageReference firebaseStorageRef =
        FirebaseStorage.instance.ref().child(filename);

    await firebaseStorageRef.delete();
    return true;
  }
}
```

</details>

<details>
<summary>add image</summary>

``` dart
import 'package:image_picker/image_picker.dart'; // For Image Picker

....
// add the widgets (use a steteful class)
                Text('Selected Image'),
                _image != null
                    ? Image.file(
                        _image,
                        height: 150,
                      )
                    : Container(/*height: 150*/),
                _image == null
                    ? RaisedButton(
                        child: Text('Choose File'),
                        onPressed: chooseFile,
                        color: Colors.cyan,
                      )
                    : Container(),
                _image != null
                    ? RaisedButton(
                        child: Text('Clear Selection'),
                        onPressed: clearSelection,
                      )
                    : Container(),
  ....

  Future chooseFile() async {
    await ImagePicker.pickImage(source: ImageSource.gallery).then((image) {
      setState(() {
        _image = image;
      });
    });
  }

  void clearSelection() {
    setState(() {
      _image = null;
    });
  }

....
  // call this function for upload
  void _uploadImage() async {
    ...
    // upload image
    if (_image != null) {
      StorageRepository storageRepository = new StorageRepository();
      Map fileInfo = await storageRepository.uploadImage(_image, userId, role);
      String fileUrl = fileInfo["fileUrl"];
      String fileName = fileInfo["fileName"];
    }
    ....
  }

```

</details>

<details>
<summary>update/delete image</summary>

``` dart
import 'package:image_picker/image_picker.dart'; // For Image Picker
import 'package:flutter_cache_manager/flutter_cache_manager.dart';
....
// need a stateful widget

  File _image;
  bool _needToUpdateImage = false;
  bool _isDeleted = false;
  ...
          Text('Selected Image'),
          _image == null && _isDeleted == false
              ? sharedWidgets.renderImage(fileUrl)
              : Container(),
          _image != null
              ? Image.file(
                  _image,
                  height: 150,
                )
              : Container(/*height: 150*/),
          _image == null
              ? Row(children: <Widget>[
                  RaisedButton(
                    child: Text('Choose File'),
                    onPressed: chooseFile,
                    color: Colors.cyan,
                  ),
                  fileInfo["fileUrl"] != null
                      ? RaisedButton(
                          child: Text('Delete image'),
                          onPressed: () {
                            _confirmDelete(fileName);
                          })
                      : Container()
                ])
              : Container(),
          _image != null
              ? RaisedButton(
                  child: Text('Clear Selection'),
                  onPressed: clearSelection,
                )
              : Container(),
....

  Future chooseFile() async {
    await ImagePicker.pickImage(source: ImageSource.gallery).then((image) {
      setState(() {
        _image = image;
        _needToUpdateImage = true;
      });
    });
  }

  void clearSelection() {
    setState(() {
      _image = null;
    });
  }

  void _confirmDelete(fileInfo) {
    showDialog(
      context: context,
      builder: (BuildContext context) {
        // return object of type Dialog
        return AlertDialog(
          title: new Text("Confirm image delete"),
          content: new Text("Are you sure?"),
          actions: <Widget>[
            // usually buttons at the bottom of the dialog
            new FlatButton(
              child: new Text("Close"),
              onPressed: () {
                Navigator.of(context).pop();
              },
            ),
            new FlatButton(
                child: new Text("Confirm"),
                onPressed: () async {
                  _deleteImage(fileInfo["fileName"]);
                  Navigator.of(context).pop();
                }),
          ],
        );
      },
    );
  }


  Future _deleteImage(fileInfo) async {
    try {
      ...
      // delete image
      StorageRepository storageRepository = new StorageRepository();
      await storageRepository.deleteImage(fileInfo["fileName"], role);
      // remove from cache: https://github.com/Baseflow/flutter_cache_manager
      await DefaultCacheManager().removeFile(fileInfo["fileUrl"]);

      setState(() {
        _image = null;
        _isDeleted = true;
      });
    } catch (e) {
      ...
    }
  }

  void _updateImage(fileInfo) async {
    try {
      ...
      if (_needToUpdateImage) {
        StorageRepository storageRepository = new StorageRepository();
        // delete old image
        await storageRepository.deleteImage(fileInfo["fileName"], role);
        // remove from cache: https://github.com/Baseflow/flutter_cache_manager
        await DefaultCacheManager().removeFile(fileInfo["fileUrl"]);
        //  upload new image
        Map<String, String> fileInfo =
            await storageRepository.uploadImage(_image, userId, role);
        String fileUrl = fileInfo["fileUrl"];
        String fileName = fileInfo["fileName"];
      }
      ....
    } catch (e) {
      ....
    }
  }
....

```

</details>

<details>
<summary>render image</summary>

``` dart
import 'package:cached_network_image/cached_network_image.dart';

.....

CachedNetworkImage(
  imageUrl: imageUrl,
  placeholder: (context, url) => CircularProgressIndicator(), // use circular progress when is loading
  errorWidget: (context, url, error) => Icon(Icons.error),
)
```

</details>

### Firebase admin
- Error: `function ignored because the firestore emulator does not exist or is not running.` Solution: change `serve` in `package.json` with: `npm run build && firebase serve --only functions,firestore`
**Reference**: https://github.com/firebase/functions-samples/issues/572#issuecomment-504032472

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

