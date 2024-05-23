# {Project_Name}


# iOS Project Technical documentation

## Services

Management classes should be implemented in **service** style.  
This allows to implement mocked / fake implementations during development or testing purpose without lot of changes in main code.

Example of services usage:

```swift
/// Service protocol
protocol ServiceProtocol {
	func data(completion: @escaping (data: Data, error: Error) -> Void)
}

/// Service implementation
class ServiceImplementation: ServiceProtocol {

	func data(completion: @escaping (data: Data, error: Error) -> Void) {
		// Implementation body
	}
}

/// Class
class ViewController: UIViewController {
	
	var service: ServiceProtocol = ServiceImplementation.init()
	
	func viewDidLoad() {
		super.viewDidLoad()
		
		service.data() { data, error in
			// handling
		}
	}
}

/// Test
class ViewControllerTest: XCTestCase {

	func testFakeService() {
		let vc = ViewController.init...
		vc.service = ServiceImplementationFake.init()
		
		// Test body
	}
}
```


## Network

Network endpoints should be divided in logically-separated files with `enum` type and implement `URLEndpointProtocol`.  
Parameters should be passed as `case` parameters and used in `parameters` variable.  
Endpoint address needs to be declared in `endpoint` variable.  

Example of endpoint file:  

```swift

enum SomeUrlEndpoint: URLEndpointProtocol {

	case simple
	case paramsGet(param: Int)
	case paramsPost(param1: String, param2: Int)
	
	var endpoint: String {        
		switch self {
		 
		case .simple: 					return "/users/profile/"
		case .paramsGet(let param): 	return "/files/\(param)"
		case .paramsPost:				return "/files/"
		}
	}
	
	var parameters: Parameters? {
		switch self {
		
		case .paramsPost(let param1, let param2):
			return [
				"key1"		  : param1,
				"another_key" : param2
			]
		
		return nil
		}
	}
}
```

**Already implemented network services**:  

1. Auth:
	- `login` - authorize with `uid` (username or email) and `pass`
	- `login` - step 1 in phone authorization
	- `loginPhoneConfirm` - step 2 in phone authorization 
	- `forgotPass` - send forgot pass mail for user with `uid`
	- `changePass` - change current user password
	- `register` - create new user with `email` and `pass`
	- `loginFacebook` - authorize with social network Facebook
	- `loginGoogle` - authorize with social network Google


## Errors

### Errors
There are few already implemented error types:  

**NetworkError**:  

- `requestError` - something wrong with request or its parameters
- `authError` - wrong authentification
- `serverError` - failed to get server response
- `parsingError` - failed to parse received response
- `invalidToken` - wrong token authentification

**AppError**:

- `permissionsDeniedPhotoLibrary` - permissions denied for Library
- `permissionDeniedCamera` - permissions denied for Camera
- `permissionDeniedLocation` - permissions denied for Location
- `imagePickFailed` - failed to get image from UIImagePickerController
- `fieldRequired` - required field is empty
- `invalidValue` - wrong value entered
- `custom(text: String)`

### ErrorsDict
`ErrorsDict` is typealias for `Dictionary<String, Error>`, where key - is name of error-related field and value - is error directly. Used in network requests for combining few independent errors into single object

### ErrorDisplayable
Protocol for components which can display errors on UI (textFields, imageViews etc). Requires only one variable `error` with type of `Error?`.  
Example of `ErrorDisplayable` implementation is `PlaceholderTF`. It contains special label `errorLabel`, which is shown or hidden, depends on `error` value  

### BaseVC errors handling method
There are few methods in `BaseVC` for error handling:

- `handleErrors` - looks for key-related `ErrorDisplayable` component and pass error-value to it. Also stores last not-displayable error (error without related displayable components) and display it as alert.  

- `errorDisplayable(for identifier: String)` - iterate over all views of VC and search for view with `accessibilityIdentifier`, equal to passed `identifier` (key from `ErrorsDict`)

- `showAlert` - display passed error / error text as alert. There are two implementations:
	- `error: Error` - display alert with passed title (default - "Error") and message `error.localizedDescription` 
	- `message: String, action: UIAlertAction_Block` - display alert with passed title (default - nil), message, buttonTitle (default - "OK") and add passed action with alert button


## Permissions

### PermissionsManager
`PermissionsManager` it's an object, who handle permission statuses and requests permission for next features:

- camera
- library
- location

Check permission status - `PermissionManager.feature.status`.  

Available statuses:

- notDetermined
- allowed
- denied

Request permission - `PermissionManager.feature.askPermission(completionBlock: { (granted) in ... }`

### LocationManager
Wrapper for `CLLocationManager`.  
Methods: 

- `startUpdatingLocation(delegate:)` - handle location with delegate
- `startUpdatingLocation(locationUpdatedBlock:)` - handle location with closure block
- `stopUpdatingLocation`

### ImagePicker
Class for retrieving images for default sources: camera (if available) and library.  

Methods:

- `openSelectSourceSheet(from:)` - display action sheet with list of available sources over passed VC
- `openCameraPicker(from:)` - open `UIImagePickerController` for camera directly
- `openLibraryPicker(from:)` - open `UIImagePickerController` for library directly

You can get response via delegate methods:

- `imagePickerPicked(image:)`
- `imagePickerFailed(error:)`
- `imagePickerCancelled`


## VC instantiate
Implemented protocol `StoryboardInstatiatable` for simpler VC instantiate from storyboards.  
1. Fill `StoryboardName` enum with your storyboards info.  
2. When add VC to storyboard, set its `Storyboard ID` with same value, as class


![Screenshot](https://user-images.githubusercontent.com/15200601/54596123-aea43100-4a3c-11e9-8227-e097e5e8da24.png).  
3. Add `StoryboardInstatiatable` protocol to your class and add `storyboardName` variable:

```swift
class ExampleVC: BaseVC, StoryboardInstatiatable {
        
    // MARK: - Variables
    
    static let storyboardName: StoryboardName = .auth
    
```
4. Now you can instantiate it in next way:  
`let vc = ExampleVC.instantiateVC()`


## Default Screens

There are already implemented some common used screens with simple UI and logic.  
You can remove unnecessary or append specific parts

1. LoginVC  
**Purpose**: authorize user view email / username / phone / social accounts fb or google

2. LoginPhoneConfirmVC  
**Purpose**: enter confirmation code from sms

3. ForgotPasswordVC  
**Purpose**: send auth details to user email

4. RegistrationVC  
**Purpose**: create new user with basic email - pass pair


## Reusable cells 
`/Views/TVC/`

- `ImageTVC`  
Simple TVC with clickable rounded UIImageView. Designed for user avatar displaying and editing on Profile screen.
	-  `picture` - UIImage, which should be displayed in cell. Rounded by default.
	-  `imagePressed` - image pressed closure block. Passes current image value. Possible usage: display action sheet with source of new avatar photo.

	
- `TextFieldTVC`  
TVC with textField. Outlet is internal, so no addition variables needed.  
Current textField class is `PlaceholderTF` - Google Material textField implementation by [Viktor Olesenko](https://github.com/OlesenkoViktor).  
**Warning**: current `PlaceholderTF` don't detect it's height correctly. Please use manual size calculations

- `SwitcherTVC`  
TVC with label and switcher.
	- `isOn` - wrapper for switcher `isOn` variable.
	- `title` - text for cell label.
	- `valueChangedBlock` - switcher value changed closure block. Passes new switcher value.
