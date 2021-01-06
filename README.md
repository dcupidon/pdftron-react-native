# PDFTron React Native Wrapper

- [Prerequisites](#prerequisites)
- [Preview](#preview)
- [Installation](#installation)
- [Usage](#usage)
- [Components](#components)
- [License](#license)

## Prerequisites
- No license key is required for trial. However, a valid commercial license key is required after trial.
- npm or yarn
- PDFTron SDK >= 6.10.0
- react-native >= 0.60.0

## Preview

**Android** |  **iOS**
:--:|:--:
<img alt='demo-android' src='https://pdftron.s3.amazonaws.com/custom/websitefiles/android/react-native-ui-demo.gif' height="800" /> | <img alt='demo-ios' src='https://pdftron.s3.amazonaws.com/custom/websitefiles/ios/react-native-ui-demo.gif' height="800" />

## Legacy UI

Version `2.0.2` is the last stable release for the legacy UI.

The release can be found here: https://github.com/PDFTron/pdftron-react-native/releases/tag/legacy-ui.

## Installation

0. If using yarn, do: `yarn global add react-native-cli`

1. First, follow the official getting started guide on [setting up the React Native environment](https://reactnative.dev/docs/environment-setup), [setting up the iOS and Android environment](https://reactnative.dev/docs/environment-setup), and [creating a React Native project](https://reactnative.dev/docs/environment-setup), the following steps will assume your app is created through `react-native init MyApp`.

2. In `MyApp` folder, install `react-native-pdftron` by calling:
    ```shell
    yarn add github:PDFTron/pdftron-react-native
    yarn add @react-native-community/cli --dev
    yarn add @react-native-community/cli-platform-android --dev
    yarn add @react-native-community/cli-platform-ios --dev
    yarn install
    ```
    or
    ```shell
    npm install github:PDFTron/pdftron-react-native --save
    npm install @react-native-community/cli --save-dev
    npm install @react-native-community/cli-platform-android --save-dev
    npm install @react-native-community/cli-platform-ios --save-dev
    ```

### Android

1. Add the following in your `android/app/build.gradle` file:

    ```diff
    android {
        compileSdkVersion rootProject.ext.compileSdkVersion

        compileOptions {
            sourceCompatibility JavaVersion.VERSION_1_8
            targetCompatibility JavaVersion.VERSION_1_8
        }

        defaultConfig {
            applicationId "com.reactnativesample"
            minSdkVersion rootProject.ext.minSdkVersion
            targetSdkVersion rootProject.ext.targetSdkVersion
            versionCode 1
            versionName "1.0"
    +       multiDexEnabled true
    +       manifestPlaceholders = [pdftronLicenseKey:PDFTRON_LICENSE_KEY]
        }

        dependencies {
    +       implementation "androidx.multidex:multidex:2.0.1"
        }

        ...
    }
    ```

2. Add the following to your `android/build.gradle` file:
	```diff
	buildscript {
	    ext {
		buildToolsVersion = "28.0.3"
	+	minSdkVersion = 21
		compileSdkVersion = 28
		targetSdkVersion = 28
	    }
	    // ...
	}
	```
3. In your `android/gradle.properties` file. Add the following line to it:
    ``` diff
    # Add the PDFTRON_LICENSE_KEY variable here. 
    # For trial purposes leave it blank.
    # For production add a valid commercial license key.
    PDFTRON_LICENSE_KEY=
    ```
4. Add the following to your `android/app/src/main/AndroidManifest.xml` file:

    ```diff
    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
        package="com.myapp">
        
		<uses-permission android:name="android.permission.INTERNET" />
		<!-- Required to read and write documents from device storage -->
	+	<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
		<!-- Required if you want to record audio annotations -->
	+	<uses-permission android:name="android.permission.RECORD_AUDIO" />

      <application
        ...
    +   android:largeHeap="true"
    +   android:usesCleartextTraffic="true">

        <!-- Add license key in meta-data tag here. This should be inside the application tag. -->
    +   <meta-data
    +       android:name="pdftron_license_key"
    +       android:value="${pdftronLicenseKey}"/>

        <activity
          android:name=".MainActivity"
          android:label="@string/app_name"
          android:configChanges="keyboard|keyboardHidden|orientation|screenSize"
    -     android:windowSoftInputMode="adjustResize"
    +     android:windowSoftInputMode="adjustPan"
    +     android:theme="@style/CustomAppTheme">
          <intent-filter>
              <action android:name="android.intent.action.MAIN" />
              <category android:name="android.intent.category.LAUNCHER" />
          </intent-filter>
        </activity>
        <activity android:name="com.facebook.react.devsupport.DevSettingsActivity" />
      </application>
    </manifest>
    ```

5. In your `android\app\src\main\java\com\myapp\MainApplication.java` file, change `Application` to `MultiDexApplication`:
    ```diff
    - import android.app.Application;
    + import androidx.multidex.MultiDexApplication;
    ...
    - public class MainApplication extends Application implements ReactApplication {
    + public class MainApplication extends MultiDexApplication implements ReactApplication {
    ```

6. Replace `App.js` with what is shown [here](#usage)
7. Finally in the root project directory, run `react-native run-android`.

### iOS

1. Open `Podfile` in the `ios` folder, add the followng line to the `target 'MyApp' do ... end` block:

    ```
    target 'MyApp' do
        # ...
        pod 'PDFNet', podspec: 'https://www.pdftron.com/downloads/ios/cocoapods/pdfnet/latest.podspec'
        # ...
    end
    ```

2. In the `ios` folder, run `pod install`.
3. (Optional) If you need a close button icon, you will need to add the PNG resources to `MyApp` as well, i.e. `ic_close_black_24px`.
4. Replace `App.js` with what is shown [here](#usage).
5. Finally in the root project directory, run `react-native run-ios`.

## Usage

Replace `App.js` with the following:

```javascript
import React, { Component } from 'react';
import {
  Platform,
  StyleSheet,
  Text,
  View,
  PermissionsAndroid,
  BackHandler,
  NativeModules,
  Alert
} from 'react-native';

import { DocumentView, RNPdftron } from 'react-native-pdftron';

type Props = {};
export default class App extends Component<Props> {

  constructor(props) {
    super(props);

    this.state = {
      permissionGranted: Platform.OS === 'ios' ? true : false
    };

    RNPdftron.initialize("Insert commercial license key here after purchase");
    RNPdftron.enableJavaScript(true);
  }

  componentDidMount() {
    if (Platform.OS === 'android') {
      this.requestStoragePermission();
    }
  }

  async requestStoragePermission() {
    try {
      const granted = await PermissionsAndroid.request(
        PermissionsAndroid.PERMISSIONS.WRITE_EXTERNAL_STORAGE
      );
      if (granted === PermissionsAndroid.RESULTS.GRANTED) {
        this.setState({
          permissionGranted: true
        });
        console.log("Storage permission granted");
      } else {
        this.setState({
          permissionGranted: false
        });
        console.log("Storage permission denied");
      }
    } catch (err) {
      console.warn(err);
    }
  }

  onLeadingNavButtonPressed = () => {
    console.log('leading nav button pressed');
    if (Platform.OS === 'ios') {
      Alert.alert(
        'App',
        'onLeadingNavButtonPressed',
        [
          {text: 'OK', onPress: () => console.log('OK Pressed')},
        ],
        { cancelable: true }
      )
    } else {
      BackHandler.exitApp();
    }
  }

  render() {
    if (!this.state.permissionGranted) {
      return (
        <View style={styles.container}>
          <Text>
            Storage permission required.
          </Text>
        </View>
      )
    }

    const path = "https://pdftron.s3.amazonaws.com/downloads/pl/PDFTRON_mobile_about.pdf";

    return (
      <DocumentView
        document={path}
        showLeadingNavButton={true}
        leadingNavButtonIcon={Platform.OS === 'ios' ? 'ic_close_black_24px.png' : 'ic_arrow_back_white_24dp'}
        onLeadingNavButtonPressed={this.onLeadingNavButtonPressed}
      />
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#F5FCFF',
  }
});
```

- (iOS) For app bundle file path:

```javascript
document="sample"
```

- (Android) For local storage file path:

```javascript
document="file:///storage/emulated/0/Download/sample.pdf"
```

- (Android) For raw resource path (include file extension):

```javascript
document="android.resource://mypackagename/raw/sample.pdf"
```

- (Android) For content Uri:

```javascript
document="content://..."
```

## API

### RNPdftron
- [initialize](#initialize)
- [enableJavaScript](#enableJavaScript)
- [getVersion](#getVersion)
- [getPlatformVersion](#getPlatformVersion)
- [encryptDocument](#encryptDocument)

#### initialize
Used to initialize PDFTron SDK with your PDFTron commercial license key. You can run PDFTron in demo mode by passing an empty string.

Parameters:

Name | Type | Description
--- | --- | ---
licenseKey | string | your PDFTron license key

```js
RNPdftron.initialize('your_license_key');
```

#### enableJavaScript
To enable JavaScript for PDFTron SDK.

Parameters:

Name | Type | Description
--- | --- | ---
enabled | bool | whether to enable or disable JavaScript

```js
RNPdftron.enableJavaScript(true);
```

#### getVersion
To get the current PDFNet version.

Promise Parameters:

Name | Type | Description
--- | --- | ---
version | string | current PDFNet version

Return a promise.

```js
RNPdftron.getVersion().then((version) => {
  console.log("Current PDFNet version:", version);
})
```

#### getPlatformVersion
 To get the version of current platform (Android/iOS).

Promise Parameters:

Name | Type | Description
--- | --- | ---
platformVersion | string | current platform version (Android/iOS)

Return a promise.

```js
RNPdftron.getPlatformVersion().then((platformVersion) => {
  console.log("App currently running on:", platformVersion);
})
```

#### encryptDocument
Used to encrypt (password-protect) a document. **Note**: This function does not lock the document it cannot be used it while the document is opened in the viewer.

Parameters:

Name | Type | Description
--- | --- | ---
file path | string | the local file path to the file
password | string | the password you would like to set
current password | string | the current password, use empty string if no password

Return a promise.

Example:

```js
RNPdftron.encryptDocument("/sdcard/Download/new.pdf", "1111", "").then(() => {
  console.log("done password");
});
```

### DocumentView

A React component for displaying documents of different types such as PDF, docx, pptx, xlsx and various image formats.

#### Props
- [document](#document)
- [password](#password)
- [leadingNavButtonIcon](#leadingNavButtonIcon)
- [showLeadingNavButton](#showLeadingNavButton)
- [onLeadingNavButtonPressed](#onLeadingNavButtonPressed)
- [onDocumentLoaded](#onDocumentLoaded)
- [onDocumentError](#onDocumentError)
- [disabledElements](#disabledElements)
- [disabledTools](#disabledTools)
- [customHeaders](#customHeaders)
- [readOnly](#readOnly)
- [thumbnailViewEditingEnabled](#thumbnailViewEditingEnabled)
- [annotationAuthor](#annotationAuthor)
- [continuousAnnotationEditing](#continuousAnnotationEditing)
- [selectAnnotationAfterCreation](#selectAnnotationAfterCreation)
- [fitMode](#fitMode)
- [layoutMode](#layoutMode)
- [initialPageNumber](#initialPageNumber)
- [pageNumber](#pageNumber)
- [onPageChanged](#onPageChanged)
- [onZoomChanged](#onZoomChanged)
- [topToolbarEnabled](#topToolbarEnabled)
- [bottomToolbarEnabled](#bottomToolbarEnabled)
- [annotationToolbars](#annotationToolbars)
- [hideDefaultAnnotationToolbars](#hideDefaultAnnotationToolbars)
- [hideAnnotationToolbarSwitcher](#hideAnnotationToolbarSwitcher)
- [hideTopToolbars](#hideTopToolbars)
- [hideTopAppNavBar](#hideTopAppNavBar)
- [hideToolbarsOnTap](#hideToolbarsOnTap)
- [pageIndicatorEnabled](#pageIndicatorEnabled)
- [showSavedSignatures](#showSavedSignatures)
- [isBase64String](#isBase64String)
- [padStatusBar](#padStatusBar)
- [autoSaveEnabled](#autoSaveEnabled)
- [hideAnnotationMenu](#hideAnnotationMenu)
- [annotationMenuItems](#annotationMenuItems)
- [overrideAnnotationMenuBehavior](#overrideAnnotationMenuBehavior)
- [onAnnotationMenuPress](#onAnnotationMenuPress)
- [longPressMenuEnabled](#longPressMenuEnabled)
- [longPressMenuItems](#longPressMenuItems)
- [overrideLongPressMenuBehavior](#overrideLongPressMenuBehavior)
- [onLongPressMenuPress](#onLongPressMenuPress)
- [overrideBehavior](#overrideBehavior)
- [onBehaviorActivated](#onBehaviorActivated)
- [pageChangeOnTap](#pageChangeOnTap)
- [useStylusAsPen](#useStylusAsPen)
- [multiTabEnabled](#multiTabEnabled)
- [tabTitle](#tabTitle)
- [signSignatureFieldsWithStamps](#signSignatureFieldsWithStamps)
- [followSystemDarkMode](#followSystemDarkMode)
- [collabEnabled](#collabEnabled)
- [currentUser](#currentUser)
- [currentUserName](#currentUserName)
- [onExportAnnotationCommand](#onExportAnnotationCommand)
- [onAnnotationsSelected](#onAnnotationsSelected)
- [onAnnotationChanged](#onAnnotationChanged)
- [annotationPermissionCheckEnabled](#annotationPermissionCheckEnabled)
- [onFormFieldValueChanged](#onFormFieldValueChanged)
- [onBookmarkChanged](#onBookmarkChanged)
- [hideThumbnailFilterModes](#hideThumbnailFilterModes)
- [onToolChanged](#onToolChanged)

##### document
string, required

The path to the document.

Example:

```js
<DocumentView
  document={'sample.pdf'}
/>
```

##### password
string, optional

The path to the document.

Example:

```js
<DocumentView
  password={'password'}
/>
```

##### leadingNavButtonIcon
string, optional

The icon path to the leading navigation button. The button would use the specified icon if it is a valid, or the default icon otherwise.

Example:

```js
<DocumentView
  leadingNavButtonIcon={Platform.OS === 'ios' ? 'ic_close_black_24px.png' : 'ic_arrow_back_white_24dp'}
/>
```

##### showLeadingNavButton
bool, optional, default to true

Defines whether to show the leading navigation button.

```js
<DocumentView
  showLeadingNavButton={true}
/>
```

##### onLeadingNavButtonPressed
function, optional

This function is called when the leading navigation button is pressed.

```js
<DocumentView
  onLeadingNavButtonPressed = {() => { 
    console.log('The leading nav has been pressed'); 
  }}
/>
```

##### onDocumentLoaded
function, optional

This function is called when the document finishes loading.

```js
<DocumentView
  onDocumentLoaded = {(path) => { 
    console.log('The document has finished loading:', path); 
  }}
/>
```

##### onDocumentError
function, optional

This function is called when document opening encounters an error.

```js
<DocumentView
  onDocumentError = {(error) => { 
    console.log('Error occured during document opening:', error); 
  }}
/>
```

##### disabledElements
array of string, optional, default to none

Defines buttons to be disabled for the viewer. Strings should be [Config.Buttons](./src/Config/Config.js) constants.

```js
<DocumentView
  disabledElements={[Config.Buttons.userBookmarkListButton]}
/>
```

##### disabledTools
array of string, optional, default to none

Defines tools to be disabled for the viewer. Strings should be [Config.Tools](./src/Config/Config.js) constants.

```js
<DocumentView
  disabledTools={[Config.Tools.annotationCreateLine, Config.Tools.annotationCreateRectangle]}
/>
```

##### customHeaders
object, optional

Defines custom headers to use with HTTP/HTTPS requests.

```js
<DocumentView
  customHeaders={{headerKey: 'headerValue'}}
/>
```

##### readOnly
bool, optional, default to false

Defines whether the viewer is read-only. If true, the UI will not allow the user to change the document.

```js
<DocumentView
  readOnly={true}
/>
```

##### thumbnailViewEditingEnabled
bool, optional, default to true

Defines whether user could modify through thumbnail view.

```js
<DocumentView
  thumbnailViewEditingEnabled={true}
/>
```

##### annotationAuthor
string, optional

Defines the author name for all annotations in the current document. Exported xfdfCommand would contain this information.

```js
<DocumentView
  annotationAuthor={'PDFTron'}
/>
```

##### continuousAnnotationEditing
bool, optional, default to true

Defines whether annotations could be continuously edited.

```js
<DocumentView
  continuousAnnotationEditing={true}
/>
```

##### selectAnnotationAfterCreation
bool, optional, default to true

Defines whether an annotation should be selected after its creation.

```js
<DocumentView
  selectAnnotationAfterCreation={true}
/>
```
##### fitMode
string, optional, default value is 'FitWidth'

Defines the fit mode of the viewer. String should be one of [Config.FitMode](./src/Config/Config.js) constants.

```js
<DocumentView
  fitMode={Config.FitMode.FitPage}
/>
```

##### layoutMode
string, optional, default value is 'Continuous'

Defines the layout mode of the viewer. String should be one of [Config.LayoutMode](./src/Config/Config.js) constants.

```js
<DocumentView
  layoutMode={Config.LayoutMode.FacingContinuous}
/>
```

##### initialPageNumber
number, optional

Defines the initial page number that viewer lands on when the document opens.

```js
<DocumentView
  initialPageNumber={5}
/>
```

##### pageNumber
number, optional

Defines the page number that viewer lands on. Different from [`initialPageNumber`](#initialPageNumber), changing this prop value at runtime will change the page accordingly.

```js
<DocumentView
  pageNumber={5}
/>
```

##### onPageChanged
function, optional

This function is called when the page number has been changed.

Parameters:

Name | Type | Description
--- | --- | ---
previousPageNumber | int | the previous page number
pageNumber | int | the current page number

```js
<DocumentView
  onPageChanged = {({previousPageNumber, pageNumber}) => {
    console.log('Page number changes from', previousPageNumber, 'to', pageNumber); 
  }}
/>
```

##### onZoomChanged
function, optional

This function is called when the zoom scale has been changed.

Parameters:

Name | Type | Description
--- | --- | ---
zoom | double | the current zoom ratio of the document

```js
<DocumentView
  onZoomChanged = {(zoom) => {
    console.log('Current zoom ratio is', zoom); 
  }}
/>
```

##### topToolbarEnabled
bool, optional, default to true

Deprecated. Use [`hideTopAppNavBar`](#hideTopAppNavBar) prop instead.

##### bottomToolbarEnabled
bool, optional, default to true

Defines whether the bottom toolbar of the viewer is enabled.

```js
<DocumentView
  bottomToolbarEnabled={false}
/>
```

##### annotationToolbars
array of object, options (one of [Config.DefaultToolbars](./src/Config/Config.js) constants or custom toolbar object)

Defines custom toolbars. If passed in, default toolbars will no longer appear.
It is possible to mix and match with default toolbars. See example below:

```js
const myToolbar = {
  [Config.CustomToolbarKey.Id]: 'myToolbar',
  [Config.CustomToolbarKey.Name]: 'myToolbar', 
  [Config.CustomToolbarKey.Icon]: Config.ToolbarIcons.FillAndSign,
  [Config.CustomToolbarKey.Items]: [Config.Tools.annotationCreateArrow, Config.Tools.annotationCreateCallout, Config.Buttons.undo]
};

...
<Documentview
  annotationToolbars={[Config.DefaultToolbars.Annotate, myToolbar]}
/>
```
##### hideDefaultAnnotationToolbars
array of string, optional, default to none

Defines which default annotation toolbars should be hidden. Note that this prop should be used when [`annotationToolbars`](#annotationToolbars) is not defined. Strings should be [Config.DefaultToolbars](./src/Config/Config.js) constants

```js
<DocumentView
  hideDefaultAnnotationToolbars={[Config.DefaultToolbars.Annotate, Config.DefaultToolbars.Favorite]}
/>
```

##### hideAnnotationToolbarSwitcher
bool, optional, default to false

Defines whether to show the toolbar switcher in the top toolbar.

```js
<DocumentView
  hideAnnotationToolbarSwitcher={false}
/>
```

##### hideTopToolbars
bool, optional, default to false

Defines whether to hide both the top app nav bar and the annotation toolbar.

```js
<DocumentView
  hideTopToolbars={false}
/>
```

##### hideTopAppNavBar
bool, optional, default to false

Defines whether to hide the top nav app bar.

```js
<DocumentView
  hideAnnotationToolbarSwitcher={false}
/>
```

##### hideToolbarsOnTap
bool, optional, default to true

Defines whether an unhandled tap in the viewer should toggle the visibility of the top and bottom toolbars. When false, the top and bottom toolbar visibility will not be toggled and the page content will fit between the bars, if any.

```js
<DocumentView
  hideToolbarsOnTap={false}
/>
```

##### pageIndicatorEnabled
bool, optional, default to true

Defines whether to show the page indicator for the viewer.

```js
<DocumentView
  pageIndicatorEnabled={true}
/>
```

##### showSavedSignatures
bool, optional, default to true

Defines whether to show saved signatures for reusing in signature mode.

```js
<DocumentView
  showSavedSignatures={true}
/>
```

##### isBase64String
bool, optional, default to false

If true, [`document`](#document) prop will be treated as a base64 string. 

When viewing a document initialized with a base64 string (i.e. a memory buffer), a temporary file is created on Android, and no temporary path is created on iOS.

```js
<DocumentView
  isBase64String={true}
  document={'...'} // base 64 string
/>
```

##### padStatusBar
bool, optional, default to false, android only

Defines whether the viewer will add padding to take account of status bar.

```js
<DocumentView
  padStatusBar={true}
/>
```

##### autoSaveEnabled
bool, optional, default to true

Defines whether document is automatically saved for the viewer.

```js
<DocumentView
  autoSaveEnabled={true}
/>
```

##### hideAnnotationMenu
array of string, optional, default to none

Defines annotation types that will not show the default annotation menu. Strings should be [Config.Tools](./src/Config/Config.js) constants.

```js
<DocumentView
  hideAnnotationMenu={[Config.Tools.annotationCreateArrow, Config.Tools.annotationEraserTool]}
/>
```

##### annotationMenuItems
array of string, optional, default to containing all the items

Defines menu items that can show when an annotation is selected. Strings should be [Config.AnnotationMenu](./src/Config/Config.js) constants.

```js
<DocumentView
  annotationMenuItems={[Config.AnnotationMenu.search, Config.AnnotationMenu.share]}
/>
```
##### overrideAnnotationMenuBehavior
array of string, optional, default to none

Defines menu items that should skip default behavior. Strings should be [Config.AnnotationMenu](./src/Config/Config.js) constants.

```js
<DocumentView
  overrideAnnotationMenuBehavior={[Config.AnnotationMenu.copy]}
/>
```

##### onAnnotationMenuPress
function, optional

This function is called if the pressed annotation menu item is passed in to [`overrideAnnotationMenuBehavior`](#overrideAnnotationMenuBehavior)

Parameters:

Name | Type | Description
--- | --- | ---
annotationMenu | string | One of [Config.AnnotationMenu](./src/Config/Config.js) constants, representing which item has been pressed
annotations | array | An array of `{id, rect}` objects, where `id` is the annotation identifier and `rect={x1, y1, x2, y2}` specifies the annotation's screen rect.

```js
<DocumentView
  onAnnotationMenuPress = {({annotationMenu, annotations}) => {
    console.log('Annotation menu item', annotationMenu, 'has been pressed');
    annotations.forEach(annotation => {
      console.log('The id of selected annotation is', annotation.id);
      console.log('The lower left corner of selected annotation is', annotation.x1, annotation.y1);
    });
  }}
/>
```

##### longPressMenuEnabled
bool, optional, default to true

Defines whether to show menu of options after long press on text or blank space in the viewer

```js
<DocumentView
  longPressMenuEnabled={true}
/>
```

##### longPressMenuItems
array of string, optional, default to containing all the items

Defines menu items that can show when long press on text or blank space. Strings should be [Config.LongPressMenu](./src/Config/Config.js) constants.

```js
<DocumentView
  longPressMenuItems={[Config.LongPressMenu.copy, Config.LongPressMenu.read]}
/>
```

##### overrideLongPressMenuBehavior
array of strings, optional, default to none

Defines menu items on long press that should skip default behavior.

```js
<DocumentView
  overrideLongPressMenuBehavior={[Config.LongPressMenu.search]}
/>
```

##### onLongPressMenuPress
function, optional

This function is called if the pressed long press menu item is passed in to [`overrideLongPressMenuBehavior`](#overrideLongPressMenuBehavior)

Parameters:

Name | Type | Description
--- | --- | ---
longPressMenu | string | One of [Config.LongPressMenu](./src/Config/Config.js) constants, representing which item has been pressed
longPressText | string | the selected text if pressed on text, empty otherwise

```js
<DocumentView
  onLongPressMenuPress = {({longPressMenu, longPressText}) => {
    console.log('Long press menu item', longPressMenu, 'has been pressed');
    if (longPressText !== '') {
      console.log('The selected text is', longPressText);
    }
  }}
/>
```

##### overrideBehavior
array of string, optional, default to none

Defines actions that should skip default behavior, such as external link click. Strings should be [Config.Actions](./src/Config/Config.js) constants.

```js
<DocumentView
  overrideBehavior={[Config.Actions.linkPress]}
/>
```

##### onBehaviorActivated
function, optional

This function is called if the activated behavior is passed in to [`overrideBehavior`](#overrideBehavior)

Parameters:

Name | Type | Description
--- | --- | ---
action | string | One of [Config.Actions](./src/Config/Config.js) constants, representing which action has been activated
data | object | A JSON object that varies depending on the action

Data param table:

Action | Param
--- | ---
[`Config.Actions.linkPress`](./src/Config/Config.js) | key: `url`, value: the link pressed

```js
<DocumentView
  onBehaviorActivated = {({action, data}) => {
    console.log('Activated action is', action);
    if (action === Config.Actions.linkPress) {
      console.log('The external link pressed is', data.url);
    }
  }}
/>
```

##### pageChangeOnTap
bool, optional, default to true

Defines whether page should change on tap when viewer is in horizontal viewing mode

```js
<DocumentView
  pageChangeOnTap={true}
/>
```

##### useStylusAsPen
bool, optional, default to true

Defines whether stylus should act as a pen in pan mode. If false, it will act as a finger.

```js
<DocumentView
  useStylusAsPen={true}
/>
```

##### multiTabEnabled
bool, optional, default to false

Defines whether viewer will show tabs for documents opened. Changing the [`document`](#document) prop value will cause a new tab to be opened with the associated file.

```js
<DocumentView
  multiTabEnabled={true}
/>
```

##### tabTitle
string, optional, default to file name

Set the tab title if [`multiTabEnabled`](#multiTabEnabled) is true.

```js
<DocumentView
  multiTabEnabled={true} // requirement
  tabTitle={'tab1'}
/>
```

##### signSignatureFieldsWithStamps
bool, optional, default to false

Defines wether signature field will be signed with image stamp.
This is useful if you are saving XFDF to remote source.

```js
<DocumentView
  signSignatureFieldsWithStamps={true}
/>
```

##### followSystemDarkMode
bool, optional, Android only, default to true

Defines whether UI will appear in dark color when System is dark mode. Otherwise it will use viewer setting instead.

```js
<DocumentView
  signSignatureFieldsWithStamps={false}
/>
```

##### collabEnabled
bool, optional, default to false

Defines whether to use the collaboration mode. If true then `currentUser` must be set as well for collaboration mode to work

```js
<DocumentView
  collabEnabled={true}
  currentUser={'Pdftron'}
/>
```

##### currentUser
string, required if [`collabEnabled`](#collabEnabled) is set to true

Defines the current user. Annotations created will has its title matching this string.

```js
<DocumentView
  collabEnabled={true}
  currentUser={'Pdftron'}
/>
```

##### currentUserName
string, optional

Defines the current user name. Will set the user name only if [`collabEnabled`](#collabEnabled) is true and [`currentUser`](#currentUser) is defined.

```js
<DocumentView
  collabEnabled={true}
  currentUser={'Pdftron'}
  currentUserName={'Hello_World'}
/>
```

##### onExportAnnotationCommand
function, optional

This function is called if a change has been made to annotations in the current document. Unlike [`onAnnotationChanged`](#onAnnotationChanged), this function has xfdfCommand as its parameter.

Parameters:

Name | Type | Description
--- | --- | ---
action | string | the action that occurred (add, delete, modify)
xfdfCommand | string | an xfdf string containing info about the edit

```js
<DocumentView
  onExportAnnotationCommand = {({action, xfdfCommand}) => {
    console.log('Annotation edit action is', action);
    console.log('The exported xfdfCommand is', xfdfCommand);
  }}
/>
```

##### onAnnotationsSelected
function, optional

This function is called if annotations are selected.

Parameters:

Name | Type | Description
--- | --- | ---
annotations | array | array of annotation data in the format `{id: string, pageNumber: number, rect: {x1: number, y1: number, x2: number, y2: number}}`, representing the selected annotations

```js
<DocumentView
  onAnnotationsSelected = {({annotations}) => {
    annotations.forEach(annotation => {
      console.log('The id of selected annotation is', annotation.id);
      console.log('It is in page', annotation.pageNumber);
      console.log('Its lower left corner has coordinate', annotation.rect.x1, annotation.rect.y1);
    });
  }}
/>
```

##### onAnnotationChanged
function, optional

This function is called if a change has been made to annotations in the current document. Unlike `onExportXfdfCommand`, this function has readable annotation objects as its parameter.

Parameters:

Name | Type | Description
--- | --- | ---
action | string | the action that occurred (add, delete, modify)
annotations | array | array of annotation data in the format `{id: string, pageNumber: number}`, representing the annotations that have been changed

```js
<DocumentView
  onAnnotationChanged = {({action, annotations}) => {
    console.log('Annotation edit action is', action);
    annotations.forEach(annotation => {
      console.log('The id of changed annotation is', annotation.id);
      console.log('It is in page', annotation.pageNumber);
    });
  }}
/>
```

##### annotationPermissionCheckEnabled
bool, optional, default to false

Defines whether annotation's flags will be taken into account when it is selected, for example, a locked annotation can not be resized or moved.

```js
<DocumentView
  annotationPermissionCheckEnabled={true}
/>
```

##### onFormFieldValueChanged
function, optional

This function is called if a change has been made to form field values.

Parameters:

Name | Type | Description
--- | --- | ---
fields | array | array of field data in the format `{fieldName: string, fieldValue: string}`, representing the fields that have been changed

```js
<DocumentView
  onFormFieldValueChanged = {({fields}) => {
    console.log('Annotation edit action is', action);
    annotations.forEach(annotation => {
      console.log('The id of changed annotation is', annotation.id);
      console.log('It is in page', annotation.pageNumber);
    });
  }}
/>
```

##### onBookmarkChanged
function, optional

This function is called if a change has been made to bookmarks.

Parameters:

Name | Type | Description
--- | --- | ---
bookmarkJson | string | the list of current bookmarks in JSON format

```js
<DocumentView
  onBookmarkChanged = {({bookmarkJson}) => {
    console.log('Bookmarks have been changed. Current bookmark collection is', bookmarkJson);
  }}
/>
```

##### hideThumbnailFilterModes
array of string, optional

Defines filter modes that should be hidden in the thumbnails browser. Strings should be [Config.ThumbnailFilterMode](./src/Config/Config.js) constants

```js
<DocumentView
  hideThumbnailFilterModes={[Config.ThumbnailFilterMode.Annotated]}
/>
```

##### onToolChanged
function, optional

This function is called when the current tool changes to a new tool

Parameters:

Name | Type | Description
--- | --- | ---
previousTool | string | the previous tool (one of the [Config.Tools](./src/Config/Config.js) constants or "unknown tool"), representing the tool before change
tool | string | the current tool (one of the [Config.Tools](./src/Config/Config.js) constants or "unknown tool"), representing the current tool

```js
<DocumentView
  onToolChanged = {({previousTool, tool}) => {
    console.log('Tool has been changed from', previousTool, 'to', tool);
  }}
/>
```

Example:

```js
import { DocumentView, Config } from 'react-native-pdftron';
<DocumentView
  ref={(c) => this._viewer = c}
  document={path}
  showLeadingNavButton={true}
  leadingNavButtonIcon={Platform.OS === 'ios' ? 'ic_close_black_24px.png' : 'ic_arrow_back_white_24dp'}
  onLeadingNavButtonPressed={() => {}}
  onDocumentLoaded={(path) => { console.log('Document is loaded at', path); }}
  disabledElements={[Config.Buttons.searchButton, Config.Buttons.shareButton]}
  disabledTools={[Config.Tools.annotationCreateLine, Config.Tools.annotationCreateRectangle]}
  customHeaders={{Foo: bar}}
  onPageChanged={({previousPageNumber, pageNumber}) => { console.log('page changed'); }}
  onAnnotationChanged={({action, annotations}) => { console.log('annotations changed'); }}
  annotationPermissionCheckEnabled={false}
  onBookmarkChanged={({bookmarkJson}) => { console.log('bookmark changed'); }}
  hideThumbnailFilterModes={[Config.ThumbnailFilterMode.Annotated]}
  onToolChanged={({previousTool,tool}) => { console.log('tool changed'); }}
/>
```

#### Methods
- [setToolMode](#setToolMode)
- [commitTool](#commitTool)
- [getPageCount](#getPageCount)
- [importAnnotations](#importAnnotations)
- [exportAnnotations](#exportAnnotations)
- [flattenAnnotations](#flattenAnnotations)
- [deleteAnnotations](#deleteAnnotations)
- [saveDocument](#saveDocument)
- [setFlagForFields](#setFlagForFields)
- [setValuesForFields](#setValuesForFields)
- [importAnnotationCommand](#importAnnotationCommand)
- [handleBackButton](#handleBackButton)
- [selectAnnotation](#selectAnnotation)
- [setFlagsForAnnotations](#setFlagsForAnnotations)
- [setPropertiesForAnnotation](#setPropertiesForAnnotation)
- [getPageCropBox](#getPageCropBox)
- [importBookmarkJson](#importBookmarkJson)
- [setCurrentPage](#setCurrentPage)
- [getDocumentPath](#getDocumentPath)
- [closeAllTabs](#closeAllTabs)

##### setToolMode
To set the current tool mode.

Parameters:

Name | Type | Description
--- | --- | ---
toolMode | string | One of [Config.Tools](./src/Config/Config.js) string constants, representing to tool mode to set

```js
this._viewer.setToolMode(Config.Tools.annotationCreateFreeHand);
```

##### commitTool
Commits the current tool, only available for multi-stroke ink and poly-shape.

Returns a Promise.

Promise Parameters:

Name | Type | Description
--- | --- | ---
committed | bool | true if either ink or poly-shape tool is committed, false otherwise

```js
this._viewer.commitTool().then((committed) => {
  // committed: true if either ink or poly-shape tool is committed, false otherwise
});
```

##### getPageCount
To get the current page count of the document.

Returns a Promise.

Promise Parameters:

Name | Type | Description
--- | --- | ---
pageCount | int | the current page count of the document

```js
this._viewer.getPageCount().then((pageCount) => {
  console.log('pageCount', pageCount);
});
```

##### importAnnotations
To import XFDF annotation string to the current document.

Parameters:

Name | Type | Description
--- | --- | ---
xfdf | string | annotation string in XFDF format for import

Returns a Promise.

```js
const xfdf = '<?xml version="1.0" encoding="UTF-8"?>\n<xfdf xmlns="http://ns.adobe.com/xfdf/" xml:space="preserve">\n\t<annots>\n\t\t<circle style="solid" width="5" color="#E44234" opacity="1" creationdate="D:20190729202215Z" flags="print" date="D:20190729202215Z" page="0" rect="138.824,653.226,236.28,725.159" title="" /></annots>\n\t<pages>\n\t\t<defmtx matrix="1.333333,0.000000,0.000000,-1.333333,0.000000,1056.000000" />\n\t</pages>\n\t<pdf-info version="2" xmlns="http://www.pdftron.com/pdfinfo" />\n</xfdf>';
this._viewer.importAnnotations(xfdf);
```

##### exportAnnotations
To extract XFDF from the current document.

Parameters:

Name | Type | Description
--- | --- | ---
options | object | key: annotList, type: array. If specified, annotations with the matching id and pageNumber will be exported; otherwise, export all annotations in the current document

Returns a Promise.

Promise Parameters:

Name | Type | Description
--- | --- | ---
xfdf | string | annotation string in XFDF format

Without options:

```js
this._viewer.exportAnnotations().then((xfdf) => {
  console.log('XFDF for all annotations:', xfdf);
});
```

With options:

```js
// annotList is an array of annotation data in the format {id: string, pageNumber: int}
const annotations = [{id: 'annot1', pageNumber: 1}, {id: 'annot2', pageNumber: 3}];
this._viewer.exportAnnotations({annotList: annotations}).then((xfdf) => {
  console.log('XFDF for 2 specified annotations', xfdf);
});
```

##### flattenAnnotations
To flatten the forms and (optionally) annotations in the current document.

Parameters:

Name | Type | Description
--- | --- | ---
formsOnly | bool | Defines whether only forms are flattened. If false, all annotations will be flattened

Returns a Promise.

```js
// flatten forms and annotations in the current document.
this._viewer.flattenAnnotations(false);
```

##### deleteAnnotations
To delete the specified annotations in the current document.

Parameters:

Name | Type | Description
--- | --- | ---
annotations | array | Defines which annotation to be deleted. Each element is in the format {id: string, pageNumber: int}

Returns a Promise.

```js
// delete annotations in the current document.
this._viewer.deleteAnnotations([
    {
        id: 'annotId1',
        pageNumber: 1,
    },
    {
        id: 'annotId2',
        pageNumber: 2,
    }
]);
```

##### saveDocument
To save the current document.

Returns a Promise.

Promise Parameters:

Name | Type | Description
--- | --- | ---
filePath | string | the location of the saved document

```js
this._viewer.saveDocument().then((filePath) => {
  console.log('saveDocument:', filePath);
});
```

##### setFlagForFields
To set a field flag value on one or more form fields.

Parameters:

Name | Type | Description
--- | --- | ---
fields | array | list of field names for which the flag should be set
flag | int | flag to be set. Number should be a [`Config.FieldFlags`](./src/Config/Config.js) constant
value | bool | value to set for flag

Returns a Promise.

```js
this._viewer.setFlagForFields(['First Name', 'Last Name'], Config.FieldFlags.ReadOnly, true);
```

##### setValuesForFields
Set field values on one or more form fields.

Note: the old function `setValueForFields` is deprecated. Please use this one instead.

Parameters:

Name | Type | Description
--- | --- | ---
fieldsMap | object | map of field names and values which should be set

Returns a Promise.

```js
this._viewer.setValuesForFields({
  'textField1': 'Test',
  'textField2': 1234,
  'checkboxField1': true,
  'checkboxField2': false,
  'radioButton1': 'Yes',
  'radioButton2': 'No'
});
```

##### importAnnotationCommand
Import remote annotation command to local document.

Parameters:

Name | Type | Description
--- | --- | ---
xfdfCommand | string | the XFDF command string
initialLoad | bool | whether this is for initial load. Will be false by default

Returns a Promise.

```js
const xfdfCommand = 'xfdfCommand <?xml version="1.0" encoding="UTF-8"?><xfdf xmlns="http://ns.adobe.com/xfdf/" xml:space="preserve"><add><circle style="solid" width="5" color="#E44234" opacity="1" creationdate="D:20201218025606Z" flags="print" date="D:20201218025606Z" name="9d0f2d63-a0cc-4f06-b786-58178c4bd2b1" page="0" rect="56.4793,584.496,208.849,739.369" title="PDF" /></add><modify /><delete /><pdf-info import-version="3" version="2" xmlns="http://www.pdftron.com/pdfinfo" /></xfdf>';
this._viewer.importAnnotationCommand(xfdf);

```

##### handleBackButton
To handle back button in search mode. Android only.

Returns a Promise.

Promise Parameters:

Name | Type | Description
--- | --- | ---
handled | bool | whether the back button is handled successfully

```js
this._viewer.handleBackButton().then((handled) => {
  if (!handled) {
    BackHandler.exitApp();
  }
});
```

##### selectAnnotation
To select the specified annotation in the current document.

Parameters:

Name | Type | Description
--- | --- | ---
id | string | the id of the target annotation
pageNumber | integer | the page number where the targe annotation is located. It is 1-indexed

Return a Promise.

```js
// select annotation in the current document.
this._viewer.selectAnnotation('annotId1', 1);
```

##### setFlagsForAnnotations
To set flags for specified annotations in the current document. The `flagValue` controls whether a flag will be set to or removed from the annotation.

Note: the old function `setFlagForAnnotations` is deprecated. Please use this one.

Parameters:

Name | Type | Description
--- | --- | ---
annotationFlagList | array | A list of annotation flag operations. Each element is in the format {id: string, pageNumber: int, flag: [Config.AnnotationFlags](./src/Config/Config.js) constants, flagValue: bool}

Return a Promise.

```js
//  Set flag for annotations in the current document.
this._viewer.setFlagsForAnnotations([
    {
        id: 'annotId1',
        pageNumber: 1,
        flag: Config.AnnotationFlags.noView,
        flagValue: true
    },
    {
        id: 'annotId2',
        pageNumber: 5,
        flag: Config.AnnotationFlags.lockedContents,
        flagValue: false
    }
]);
```

##### setPropertiesForAnnotation
To set properties for specified annotation in the current document, if it is valid. 

Note: the old function `setPropertyForAnnotation` is deprecated. Please use this one.

Parameters:

Name | Type | Description
--- | --- | ---
annotationId | string | the unique id of the annotation
pageNumber | integer | the page number where annotation is located. It is 1-indexed
propertyMap | object | an object containing properties to be set. Available properties are listed below

Properties in propertyMap:

Name | Type | Markup exclusive | Example
--- | --- | --- | ---
rect | object | no | {x1: 1, y1: 2, x2: 3, y2: 4}
contents | string | no | "contents"
subject | string | yes | "subject"
title | string | yes | "title"
contentRect | object | yes | {x1: 1, y1: 2, x2: 3, y2: 4}

Return a promise.

```js
// Set properties for annotation in the current document.
this._viewer.setPropertiesForAnnotation('Pdftron', 1, {
  rect: {
    x1: 1.1,    // left
    y1: 3,      // bottom
    x2: 100.9,  // right
    y2: 99.8    // top
  },
  contents: 'Hello World',
  subject: 'Sample',
  title: 'set-prop-for-annot'
});
```

##### getPageCropBox
To get the crop box for specified page as a JSON object.

Parameters:

Name | Type | Description
--- | --- | ---
pageNumber | integer | the page number for the target crop box. It is 1-indexed

Return a Promise.

Promise Parameters:

Name | Type | Description
--- | --- | ---
cropBox | object | an object with information about position (`x1`, `y1`, `x2` and `y2`) and size (`width` and `height`)

```js
this._viewer.getPageCropBox(1).then((cropBox) => {
  console.log('bottom-left coordinate:', cropBox.x1, cropBox.y1);
  console.log('top-right coordinate:', cropBox.x2, cropBox.y2);
  console.log('width and height:', cropBox.width, cropBox.height);
});
```

##### importBookmarkJson
Imports user bookmarks to the document. The input needs to be a valid bookmark JSON format.

Parameters:

Name | Type | Description
--- | --- | ---
bookmarkJson | String | needs to be in valid bookmark JSON format, for example {"0": "Page 1"}. The page numbers are 1-indexed

Return a Promise.

```js
this._viewer.importBookmarkJson("{\"0\": \"Page 1\", \"3\": \"Page 4\"}");
```

##### setCurrentPage
Set current page of the document.

Parameters:

Name | Type | Description
--- | --- | ---
pageNumber | integer | the page number for the target crop box. It is 1-indexed

Return a Promise.

Promise Parameters:

Name | Type | Description
--- | --- | ---
success | bool | whether the setting process was successful

```js
this._viewer.setCurrentPage(4).then((success) => {
  if (success) {
    console.log("Current page is set to 4.");
  }
});
```

##### getDocumentPath
Return the path of the current document.

Return a Promise.

Promise Parameters:

Name | Type | Description
--- | --- | ---
path | string | the document path

```js
this._viewer.getDocumentPath().then((path) => {
  console.log('The path to current document is: ' + path);
});
```

##### closeAllTabs
Closes all tabs in multi-tab environment.

Return a Promise.

```js
// Do this only when DocumentView has multiTabEnabled = true
this._viewer.closeAllTabs();
});
```

## Contributing
See [Contributing](./CONTRIBUTING.md)

## License
See [License](./LICENSE)
![](https://onepixel.pdftron.com/pdftron-react-native)
