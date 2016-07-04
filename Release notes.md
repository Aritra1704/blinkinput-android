# Release notes

## 2.7.0
- support for parsing Vehicle Identification Numbers (VINs)
- renamed BlinkOCRActivity to SegmentScanActivity
- added RandomScanActivity which is similar to SegmentScanActivity but it does not force the user to scan text segments in the predefined order
- improved autofocus support on SGS6 and SGS7
- fixed memory leak in RecognitionProcessCallback, leak was caused by Recognizer singleton holding reference to both Context and MetadataListener even after termination

## 2.6.0
- added support for using detectors to perform detection of various documents
- added support for combination of detectors with BlinkOCR recognizer to perform OCR of only parts of detected documents
    - refer to documentation and demo apps for example
- updated BlinkOCRActivity, through ScanConfiguration it is now possible to make some scan field optional (user can skip it) and set the field size

## 2.5.0
- FailedDetectionMetadata, PointsDetectionMetadata and QuadDetectionMetadata have been replaced with DetectionMetadata which now holds a DetectorResult
    - DetectorResult is more flexible as it allows more different detection types to be added in future
- fixed several possible crashes in camera management
- fixed autofocus bug on LG devices when metering areas or non-default zoom level were set
- fixed autofocus bug on LG G4 (not related to bug above)
- added option to set the OCR document type in BlinkOCREngineOptions
- introduced new setting options for generic AmountParser:
    - set parser to Arabic-Indic mode (amounts with Arabic-Indic digits)
    - allow amounts with space separated digit groups (thousands)
    - set ideal (expected) number of digits before decimal point
- added factory method `createFromPreset` to generic AmountParserSettings that creates the settings from one of the available presets (`GENERIC`, `LARGE_AMOUNT`) 

## 2.4.0
- reconfigureRecognizers method now throws an error if phone does not have autofocus and at least one of new recognizers require it
- raw resources are now packed as assets
- fixed bug with isScanningPaused which sometimes returned bogus value and caused scanning to work even if initial scanning was set to be paused
- support for scanning custom camera frames via DirectAPI
- fixed bug on some devices causing it to never start scanning if device was not shaken
- increased OCR engine initialisation speed
- improved Frame Quality Estimation on low-end devices (fixed regression introduced in v1.6.0)
- added new options to BlinkOcrEngineOptions
- added RegexParser which can parse almost any regular expression from OCR result

## 2.3.0
- support detecting on activity flip event
- fixed crash in RecognizerCompatibility on ARMv7 without NEON
- added RecognizerCompatibility to javadoc
- fixed NPE in BarcodeDetailedData

## 2.2.0
- improved performance of conversion of `Image` object into `Bitmap`
- fixed crash that could be caused by quickly restarting camera activity
- fixed bug in camera layout in certain aspect ratios of camera view
- fixed bug in segment scan when put on landscape activity
- fixed bug in handling `setMeteringAreas`
- `setMeteringAreas` now receives boolean indicating whether set areas should be rotated with device
- added support for specifying camera aspect mode from XML

## 2.1.0
- fixed crash that occurred when detection image was being sent from native to Java
- fixed issue that cause irrationaly refusal to scan anything on high-end devices
- fixed bug in scanning 1D barcodes with ZXing when scanning region was set
	- the bug caused not to honor set region
- when embedding `RecognizerView` into custom scan activity, you no longer need to take care of whether runtime permissions are set or not. You can now simply pass all lifecycle events to `RecognizerView` and if camera permission is denied, a new callback method `onCameraPermissionDenied` of `CameraEventsListener` will be invoked to give you chance to ask user for permissions
	- please refer to updated demo apps for example of new callback
- **IMPORTANT** - `onScanningDone` callback method does not automatically pause scanning loop anymore. As soon as `onScanningDone` method ends, scanning will be automatically resumed without resetting state
	- if you need to reset state, please call `resetRecognitionState` in your implementation of `onScanningDone`
	- if you need to have scanning paused after `onScanningDone` ends, please call `pauseScanning` in your implementation of `onScanningDone`. Do not forget to call `resumeScanning` to resume scanning after it has been paused.
- `pauseScanning` and `resumeScanning` calls are now counted, i.e. if you call `pauseScanning` twice, you will also need to call `resumeScanning` twice to actually resume scanning
	- this is practical if you show multiple onboarding views over camera and you want the scanning paused while each is shown and you do not know in which order they will be dismissed. Now you can simply call `pauseScanning` on showing the onboarding view and `resumeScanning` on dismissing it, regardless of how many onboarding views you have
	- if you want to show onboarding help first time your scan activity starts, you can call `setInitialScanningPaused(true)` which will ensure that first time camera is started, the scanning will not automatically start - you will need to call `resumeScanning` to start scanning after your onboarding view is dismissed
- added support for `x86_64` architecture


## 2.0.0
- new API which is easier to understand, but is not backward compatible. Please check [README](README.md) and updated demo applications for more information.
- removed support for ARMv7 devices which do not support NEON SIMD
	- this enabled us to increase recognition speed at cost of not supporting old devices like those using NVIDIA Tegra 2
	- you can check [this article](https://microblink.zendesk.com/hc/en-us/articles/206113151-Removing-support-for-devices-without-NEON-SIMD-extensions) for more information about NEON and why we use it
- added official support for Android 6.0 and it's runtime camera permissions
	- if using provided _BlinkOCRActivity_, the logic behind asking user to give camera permission is handled internally
	- if integrating using custom UI, you are required to ask user to give you permission to use camera. To make this easier, we have provided a _CameraPermissionManager_ class which does all heavylifting code about managing states when asking user for camera permission. Refer to demo apps to see how it is used.
- BlinkOCR now depends on appcompat-v7 library, instead of full android-support library.
	- even older versions of BlinkOCR required only features from appcompat-v7 so we now decided to make appcompat-v7 as dependency because it is much smaller than full support library and is also default dependency of all new Android apps.
- completely rewritten JNI layer which now gives much lower overhead in communication between Java and native code
	- in our internal tests, this yielded up to 3 times better performance in OCR
- fixed issue with Nexus 5X upside down camera
- when using DirectAPI, recognized Bitmap is not recycled anymore so it can be reused


## 1.9.0
- fixed auto-exposure bug on Nexus 6
- improved OCR quality and performance
- fixed autofocus issue on devices that do not support continuous autofocus
- support for defining camera video resolution preset
	- to define video resolution preset via Intent, use `BlinkOCRActivity.EXTRAS_CAMERA_VIDEO_PRESET`
	- to define video resolution preset on `RecognizerView`, use method `setVideoResolutionPreset`

## 1.8.0
- added support for scanning barcodes
- fixed race condition causing memory leak or rare crashes
- fixed `NullPointerException` in `BaseCameraView.dispatchTouchEvent`
- fixed bug that caused returning scan result from old video frame
- fixed `NullPointerException` in camera2 management
- fixed rare race condition in gesture recognizer
- fixed segmentation fault on recognizer reconfiguration operation
- fixed freeze when camera was being quickly turned on and off
- fixed bug in IBAN parser that caused not to parse IBAN's that contain letters, such as in UK, Ireland or Netherlands
- ensured `RecognizerView` lifecycle methods are called on UI thread
- ensure `onCameraPreviewStarted` is not called if camera is immediately closed after start before the call should have taken place
- ensure `onScanningDone` is not called after `RecognizerView` has been paused, even if it had result ready just before pausing
- default maximum number of chars in `raw parser` is now 3000 (it used to be 600)
- it is now possible to define maximum allowed number of char recognition variants via `BlinkOCREngineOptions`. Default value is `0`.
- when calling `onDisplayOcrResult` callback, make sure OCR char recognition variants are not sent to Java - this is both slow and not required
- added support for using _BlinkOCR_ as camera capture API. To do that, implement following:
	- when using `RecognizerView` do not call `setRecognitionSettings` or call it with `null` or empty array
	- implement `ImageListener` interface and set the listener with `setImageListener`
	- as a reminder - you can process video frames obtained that way using direct API method `recognizeImageWithSettings`
- reorganized integration demo apps
	- `BlinkOCRSegmentDemo` shows how to use simple Intent-based API to scan little text segments. It also shows you how to create a custom scan activity for scanning little text segments.
	- `BlinkOCRFullScreen` shows how to perform full camera frame generic OCR, how to draw OCR results on screen and how to obtain `OcrResult` object for further processing. This app also shows how to scan Code128 or Code39 barcode on same screen that is used for OCR.
	- `BlinkOCRDirectAPI` shows how to perform OCR of `Bitmap` object obtained from camera
	- all demo apps now use Maven integration method because it is much easier than importing AAR manually
- **removed** parsers specific to country standards - these are now available as part of our [PhotoPay](https://microblink.com/photopay) product
	- removed croatian parsers
	- removed serbian parsers
	- removed macedonian parsers
	- removed swedish parsers

## 1.7.1
- fixed NullPointerException when RecognizerSettings array element was `null`

## 1.7.0
- [ImageListener](https://blinkocr.github.io/blinkocr-android/com/microblink/image/ImageListener.html) can now receive [DEWARPED](https://blinkocr.github.io/blinkocr-android/com/microblink/image/ImageType.html#DEWARPED) images and [Image](https://blinkocr.github.io/blinkocr-android/com/microblink/image/Image.html) now contains information about its [Orientation](https://blinkocr.github.io/blinkocr-android/com/microblink/hardware/orientation/Orientation.html)
- [recognizeBitmap](https://blinkocr.github.io/blinkocr-android/com/microblink/view/recognition/RecognizerView.html#recognizeBitmap(android.graphics.Bitmap, com.microblink.view.recognition.ScanResultListener)) method can now receive orientation of given [Bitmap](https://developer.android.com/reference/android/graphics/Bitmap.html)
- it is now possible to recognize [Image](https://blinkocr.github.io/blinkocr-android/com/microblink/image/Image.html) objects directly, without slow conversion into [Bitmap](https://developer.android.com/reference/android/graphics/Bitmap.html)
- removed method `resumeScanningWithoutStateReset` - method `resumeScanning` of [RecognizerView](https://blinkocr.github.io/blinkocr-android/com/microblink/view/recognition/RecognizerView.html) now receives `boolean` indicating whether internal state should be reset

## 1.6.0
- removed dependency to deprecated [Horizontal Variable ListView](https://github.com/sephiroth74/HorizontalVariableListView) - default activity now only requires [Android support library](https://developer.android.com/tools/support-library/index.html)
- new and improved android camera management
	- on devices that support it, utilize [Camera2 API](https://developer.android.com/reference/android/hardware/camera2/package-summary.html) for better per frame camera control
	- new and improved algorithm for choosing which frame is of good enough quality to be processed - there is now less latency from initialization of camera until first scan result

## 1.5.0
- added support for Macedonian parsers
- added date parser
- fixed crash in DirectAPI when recognizer was terminated in the middle of recognition process
- removed support for ARMv6 processors because those are too slow for OCR

## 1.4.0
- added support for Serbian parsers
- fixed camera orientation detection when RecognizerView is not initialized with Activity context

## 1.3.0
- added support for Croatian and Swedish parsers
- added more fonts to OCR model file

## 1.2.0
- support for controlling camera zoom level
- support "easy" integration mode with provided BlinkOCRActivity - check README
- Raw parser is now more customizable - check javadoc for class RawParserSettings
	- character whitelist can now be defined
	- maximum and minimum height of text line can be defined
	- color vs. grayscale image processing support
	- maximum expected number of chars can now be defined

## 1.1.0
- support for parsing e-mails
- introduced new licence key format (generate your free licence key on [https://microblink.com/login](https://microblink.com/login) or contact us at [http://help.microblink.com](http://help.microblink.com)

## 1.0.0

- Initial release
- Support for parsing amounts and IBANs