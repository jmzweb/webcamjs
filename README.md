# WebcamJS

## HTML5 Webcam Image Capture Library with Flash Fallback

WebcamJS is a small (~2K minified and gzipped) standalone JavaScript library for capturing still images from your computer's camera, and delivering them to you as JPEG or PNG [http://en.wikipedia.org/wiki/Data_URI_scheme](Data URIs).  The images can then be displayed in your web page, rendered into a canvas, or submitted to your server via a standard form.  WebcamJS uses [http://dev.w3.org/2011/webrtc/editor/getusermedia.html](HTML5 getUserMedia), but provides an automatic and invisible Flash fallback.

WebcamJS is based on my old [https://code.google.com/p/jpegcam/](JPEGCam) project, but has been redesigned for the modern web.  Instead of relying on Flash and only being able to submit images directly to a server, WebcamJS delivers your images as client-side Data URIs, and it uses HTML5 getUserMedia where available.  Flash is only used if your browser doesn't support getUserMedia, and the fallback is handled automatically.

[http://pixlcore.com/demos/webcamjs/basic.html](Check out a live demo here!)

## QuickStart Guide

Host the `webcam.js` and `webcam.swf` files on your web server, and drop in this HTML snippet:

```html
	<script src="webcam.js"></script>

	<div id="my_camera" style="width:320px; height:240px;"></div>
	<div id="my_result"></div>

	<script language="JavaScript">
		Webcam.attach( '#my_camera' );
		
		function take_snapshot() {
			var data_uri = Webcam.snap();
			document.getElementById('my_result').innerHTML = '<img src="'+data_uri+'"/>';
		}
	</script>

	<a href="javascript:void(take_snapshot())">Take Snapshot</a>
```

This will create a live camera view in the `my_camera` DIV, and when the **Take Snapshot** link is clicked it will take a still snapshot, convert it to a JPEG, and deliver a Data URI which is inserted into the `my_result` DIV as a standard `<IMG SRC>` tag.

Data URIs may be passed around like any URL, and can be submitted to your server by stuffing them into a form field (see below for example of this).

## Configuration

If you want to override the default settings, just call `Webcam.set()` and pass in a hash with any of the following keys:

| Param Name | Default Value | Notes |
--------------------------------------
| `width` | (Auto) | Width of the live camera viewer in pixels, defaults to the actual size of the DOM element. |
| `height` | (Auto) | Height of the live camera viewer in pixels, defaults to the actual size of the DOM element. |
| `dest_width` | (Auto) | Width of the captured camera image in pixels, defaults to the live viewer size. |
| `dest_height` | (Auto) | Height of the captured camera image in pixels, defaults to the live viewer size. |
| `image_format` | jpeg | Desired image format of captured image, may be "jpeg" or "png". |
| `jpeg_quality` | 90 | For JPEG images, this is the desired quality, from 0 (worst) to 100 (best). |

Here is an example of overriding some parameters.  Remember to call this *before* you attach the viewer.

```javascript
	Webcam.set({
		width: 320,
		height: 240,
		dest_width: 640,
		dest_height: 480,
		image_format: 'jpeg',
		jpeg_quality: 90
	});
	
	// Attach camera here
```

## Initialization

WebcamJS is initialized and activated by "attaching" a live camera viewer to a DOM element.  The DOM element must already be created and empty.  Pass in a ID or CSS selector to the `Webcam.attach()` function.  Example:

```javascript
	Webcam.attach( '#my_camera' );
```

This will activate the user's webcam, ask for the appropriate permission, and begin showing a live camera image in the specified DOM element.

## Snapping a Picture

To snap a picture, just call the `Webcam.snap()` function.  The image data will be returned as a Data URI, which you can then display in your web page, or submit to a server.  Example:

```javascript
	var data_uri = Webcam.snap();
	
	document.getElementById('my_result').innerHTML = '<img src="'+data_uri+'"/>';
```

## Customizing Image Size

WebcamJS will automatically size the live camera viewer based on the DOM element it is attached to.  However, you can override this by setting the `width` and/or `height` parameters:

```javascript
	Webcam.set({
		width: 320,
		height: 240
	});
	
	// Attach camera here
```

The size of the captured JPEG / PNG image is set to match the live camera viewer by default.  However, you can override this by setting the `dest_width` and/or `dest_height`.  Note that you can set the destination image size different than the viewer size.  So you can have a small live viewer, but capture a large image.  Example:

```javascript
	Webcam.set({
		width: 320,
		height: 240,
		dest_width: 640,
		dest_height: 480,
	});
	
	// Attach camera here
```

## Setting an Alternate SWF Location

By default WebcamJS looks for the SWF file in the same directory as the JS file.  If you are hosting the SWF in a different location, please set it using the `Webcam.setSWFLocation()` function.  Example:

```javascript
	Webcam.setSWFLocation("/path/to/the/webcam.swf");
```

## Reset (Shutdown)

To shut down the live camera preview and reset the system, call `Webcam.reset()`.  This removes any DOM elements we added, including a Flash movie if applicable, and resets everything in the library to the initial state.  Example:

```javascript
	Webcam.reset();
```

To use the library again after resetting, you must call `Webcam.attach()` and pass it your DOM element.

## Custom Events

WebcamJS fires a number of events you can intercept using a JavaScript hook system.  Events are fired when the library is fully loaded, when the camera is live, and when an error occurs.  To register an event listener, call the `Webcam.on()` function, passing an event name and callback function.  Here is a table of the available event types:

| Event Name | Notes |
--------------------------------------
| `load` | Fires when the library finishes loading. |
| `live` | Fires when the user's camera goes live (i.e. showing a live preview). |
| `error` | Fires when an error occurs (your callback function is passed an error string). |

Example:

```javascript
	Webcam.on( 'load', function() {
		// library is loaded
	} );
	
	Webcam.on( 'live', function() {
		// camera is live, showing preview image
	} );
	
	Webcam.on( 'error', function(err) {
		// an error occurred (see 'err')
	} );
```

By default the `error` event shows a JavaScript alert dialog, but if you register your own event handler this action is suppressed, and your function is called instead.

## Submitting Images to a Server

WebcamJS delivers your images by way of a client-side JavaScript Data URI.  The binary image data is encoded with Base64 and stuffed into the URI.  It is up to you to send this data to your server and decode it.  There are many ways to do this, but here is the easiest.

First, add a hidden text element to a form:

```html
	<form id="myform" method="post" action="myscript.php">
		<input id="mydata" type="hidden" name="mydata" value=""/>
	</form>
```

Then, when you snap your picture, stuff the Data URI into the form field value (minus the header), and submit the form:

```javascript
	var data_uri = Webcam.snap();
	var raw_image_data = data_uri.replace(/^data\:image\/\w+\;base64\,/, '');
	
	document.querySelector('#mydata').value = raw_image_data;
	document.querySelector('#myform').submit();
```

Finally, on your server, grab the form data as if it were a plain text field, decode the Base64, and you have your binary image file.  Example here in PHP, assumes JPEG format:

```php
	$encoded_data = $_POST['mydata'];
	$binary_data = base64_decode( $encoded_data );
	
	// save to server (beware of permissions)
	$result = file_put_contents( 'webcam.jpg', $binary_data );
	if (!$result) die("Could not save image!  Check file permissions.");
```

## License

The MIT License (MIT)

Copyright (c) 2012 - 2014 Joseph Huckaby

Permission is hereby granted, free of charge, to any person obtaining a copy of
this software and associated documentation files (the "Software"), to deal in
the Software without restriction, including without limitation the rights to
use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of
the Software, and to permit persons to whom the Software is furnished to do so,
subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS
FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR
COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER
IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
