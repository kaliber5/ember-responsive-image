# ember-responsive-image

[![Build Status](https://travis-ci.org/kaliber5/ember-responsive-image.svg?branch=master)](https://travis-ci.org/kaliber5/ember-responsive-image)
[![npm version](https://badge.fury.io/js/ember-responsive-image.svg)](https://badge.fury.io/js/ember-responsive-image)
[![Code Climate](https://codeclimate.com/github/kaliber5/ember-responsive-image/badges/gpa.svg)](https://codeclimate.com/github/kaliber5/ember-responsive-image)
[![Ember Observer Score](https://emberobserver.com/badges/ember-responsive-image.svg)](https://emberobserver.com/addons/ember-responsive-image)

An ember-cli addon to automatically generate resized images and use them in `img` tags with the `srcset` attribute.

This is very useful for responsive web apps to optimize images for a wide range of devices (smartphones, tablets, desktops etc.). All browsers with [support for the `srcset` attribute](http://caniuse.com/#search=srcset) will automatically load the most appropriate resized image for the given device, e.g. based on screen size and density (high dpi "retina" screens).

Built on top of the awesome [sharp](https://github.com/lovell/sharp) library.

## Getting started

### Install in ember-cli application

In your application's directory:

```bash
ember install ember-responsive-image
```

### Install sharp

On Windows, Mac and Linux `npm` / `yarn` will download and install the `sharp` image processing library 
automatically. For other environments please consult the [installation](http://sharp.dimens.io/en/stable/install/#installation)
instructions.

## Basic Usage

Add the configuration to your `config/environment.js`

```js
module.exports = function(environment) {
  var ENV = {
    'responsive-image': {
      sourceDir: 'assets/images/generate',
      destinationDir: 'assets/images/responsive',
      quality: 80,
      supportedWidths: [2048, 1536, 1080, 750, 640],
      removeSourceDir: true,
      justCopy: false,
      extensions: ['jpg', 'jpeg', 'png', 'gif']
    }
  }
}
```

If you need different configurations, you can make the `responsive-image` config an array:

```js
module.exports = function(environment) {
  var ENV = {
    'responsive-image': [
        {
          sourceDir: 'assets/images/generateLarge',
          destinationDir: 'assets/images/responsiveLarge',
          quality: 80,
          supportedWidths: [2048, 1536, 1080],
          removeSourceDir: true,
          justCopy: false,
          extensions: ['jpg', 'jpeg', 'png', 'gif']
        },
        {
          sourceDir: 'assets/images/generateSmall',
          destinationDir: 'assets/images/responsiveSmall',
          quality: 80,
          supportedWidths: [750, 640, 320],
          removeSourceDir: true,
          justCopy: false,
          extensions: ['jpg', 'jpeg', 'png', 'gif']
        }
    ]
  }
}
```


### Options

* **sourceDir:** The folder with the origin images.
* **destinationDir:** This folder will contain the generated Images. It will be created, if not existing. Must not be the same as sourceDir.
* **supportedWidths:** These are the widths of the resized images.
* **removeSourceDir:** If true, the sourceDir will be removed from the build.
* **justCopy:** If true, the images will just be copied without resizing. This is useful for development builds to speed things up, but should be false for production.
* **extensions:** Array of file extensions. Only files with these extensions will be resized, others will be ignored. Avoid errors with files like `.DS_Store`.

Put one or more images in the source folder (in this case 'assets/images/generate/'), like 'myImage.png', and build the project. The resized images will be generated into the destination directory ('assets/images/responsive'):
```
myImage640w.png
myImage750w.png
myImage1080w.png
myImage1536w.png
myImage2048w.png
```

**Note:** If the width of your origin image is less than the generated should be, the image will be generated unresized.

## Service

The `responsive-image` service provides the available images with the sizes for a given origin image, and also retrieves the image that fits for the current screen size.
```js
let availableImages = responsiveImageService.getImages("myImage.png");
/**
avaliableImages contains now: 
[
    {width: 640, height: 320, image: "/assets/images/responsive/myImage640w.png"},
    {width: 750, height: 375, image: "/assets/images/responsive/myImage750w.png"},
    ...
    {width: 2048, height: 1012, image: "/assets/images/responsive/myImage2048w.png"}
]
*/

let imageData = responsiveImageService.getImageDataBySize("myImage.png", 100); // The size argument is in ´vw´, 100 is the default and can be omitted
// {width: 750, height: 375, image: "/assets/images/responsive/myImage750w.png"}

let fittingImage = responsiveImageService.getImageBySize("myImage.png", 100); // The size argument is in ´vw´, 100 is the default and can be omitted
// "/assets/images/responsive/myImage1080w.png"
```

The base width to calculate the necessary image width is the `screen.width` assigned to the `screenWidth` property of the services. If this doesn't fit your needs, you can assign an other value, e.g. `document.documentElement.clientWidth`. 

## Helper

The `responsive-image-resolve` helper provides the image url that fits for the current screen size. The first parameter is the name of the origin image. 
The second argument is the width in `vw` and has a default value of `100`, so it can be omitted. 

```hbs
{{responsive-image-resolve "myImage.png" 100}}
```

will result in

```html
/assets/images/responsive/myImage1080w.png
```

## Components
### The responsive-image component

In a template you can use the responsive-image component. The image argument is required and must be one of the origin files:

```hbs
{{responsive-image image="myImage.png"}}
```

This will generate an `img` tag with the resized images as the [`srcset` attribute](https://developer.mozilla.org/de/docs/Web/HTML/Element/img#attr-srcset), so the browser can decide, which image fits the needs:
```html
<img id="ember308" src="/assets/images/responsive/myImage1080w.png" srcset="/assets/images/responsive/myImage640w.png 640w, /assets/images/responsive/myImage750w.png 750w, /assets/images/responsive/myImage1080w.png 1080w, /assets/images/responsive/myImage1536w.png 1536w, /assets/images/responsive/myImage2048w.png 2048w" class="ember-view">
```

The image in the `src` attribute is calculated by the component and will be used by browsers without `srcset` support.

Other attributes like `alt`, `className`, `width` or `height` are optional:

```hbs
{{responsive-image image="myImage.png" className="my-css-class" alt="This is my image"}}
```

```html
<img id="ember308" src="..." srcset="..." class="ember-view my-css-class" alt="This is my image">
```

If your image width is not '100vw', say 70vw for example, you can specify the `size` (only `vw` is supported as a unit by now):
```hbs
{{responsive-image image="myImage.png" size="70"}}
```

```html
<img id="ember308" src="..." srcset="..." sizes="70vw">
```

You can also replace the [`sizes` attribute](https://developer.mozilla.org/de/docs/Web/HTML/Element/img#attr-sizes) if your responsive image width is more complicated like:
```hbs
{{responsive-image image="myImage.png" sizes="(min-width: 800px) 800px, 100vw"}}
```

```html
<img id="ember308" src="..." srcset="..." sizes="(min-width: 800px) 800px, 100vw">
```

### The responsive-background component

In a template you can use the `responsive-background` component. The image argument is required and must be one of the origin files:

```hbs
{{responsive-background image="myImage.png"}}
```

This will generate an `div` tag with an image as a background image, which fits the needs:
```html
<div id="ember308" style="background-image: url('/assets/images/responsive/myImage1080w.png')" class="ember-view"></div>
```

Like the `responsive-image` component, you can pass a size:
```hbs
{{responsive-background image="myImage.png" size="50"}}
```

```html
<div id="ember308" style="background-image: url('/assets/images/responsive/myImage640w.png')" class="ember-view"></div>
```

## Mixins
### The responsive-image mixin

This mixin binds the url of the best fitting image to the source attribute, based in the values provided by the `image` and `size` attribute. It also get the `responsiveImage` service injected.

### The responsive-background mixin

This mixin binds the url of the best fitting image as the background url to the elements style attribute, based in the values provided by the `image` and `size` attribute. It also get the `responsiveImage` service injected.

## Extensibility hooks
### Extend the image processing

During the image process the addon calls the `preProcessImage` and the `postProcessImage` hooks for each origin image and supported width. Here you can add custom image process steps, like a watermark integration. The first hook will be called just before the addon's image process calls applies, the latter after. You can register your callbacks by calling the addon's `addImagePreProcessor` or `addImagePostProcessor` function before the addon's `postprocessTree` was called.
In both cases the callback function you provide must have the following signature:

```javascript
  function preProcessor(sharp, image, width, configuration)
  {
    // do something with the sharp-object...
    return sharp;
  }
```
* **sharp:** [sharp](https://github.com/lovell/sharp) object with the current origin image
* **image:** the name of the origin image file
* **width:** the width of the resulting resized image of the current process
* **configuration:** the configuration for the current image processing (from environments configuration)

The callback must return a `sharp`-object or a Promise resolves to it.

**Note:** In addition to the callback, you can also pass an optional target object that will be set as `this` on the context. This is a good way to give your function access to the current object.

**Note:** If you set `justCopy` to `true` in your configuration, your callback will be called, but the result doesn't take effect to the resulting image (because it's just a copy).

For an example see [ember-lazy-responsive-image](https://github.com/kaliber5/ember-lazy-responsive-image/blob/master/index.js)

### Extend the metadata

Before the addon injects the generated metadata into the build, a `extendMetadata`-hook is called for each origin image. The `metadata`-object contains the information for the addon's `ResponsiveImage`-Service. Here you can add custom metadata. You can register your callbacks by calling the addon's `addMetadataExtension` function before the addon's `postprocessTree` was called.
The callback function you provide must have the following signature:

```javascript
  function customMetadata(image, metadata, configuration)
  { 
    // do something with the metadata-object...  
    return metadata;
  }
```
* **image:** the name of the origin image file
* **metadata:** object with the metadata of the generated images
* **configuration:** the configuration for the image generation (from environments configuration)

The callback must return an object with the extended metadata.

**Note:** In addition to the callback, you can also pass an optional target object that will be set as `this` on the context. This is a good way to give your function access to the current object.

For an example see [ember-lazy-responsive-image](https://github.com/kaliber5/ember-lazy-responsive-image/blob/master/index.js) and the extended [ResponsiveImageService](https://github.com/kaliber5/ember-lazy-responsive-image/blob/master/addon/services/responsive-image.js)

## Lazy loading and LQIP (Low Quality Image Placeholder)

For lazy-loading and LQIP support, see [ember-lazy-responsive-image](https://github.com/kaliber5/ember-lazy-responsive-image).

## Tests

Sometimes you can get in trouble in integration tests and get an assertion `'There is no data for image ...'` if your subject relates to this addon. Because the `ResponsiveImageService` gets necessary data injected in an instance-initializer, and instance-initializers won't be called in the integration tests. To fix this, you can provided ``setupResponsiveImage()` test helper in tests:

```js
import { setupRenderingTest} from 'ember-qunit';
import { setupResponsiveImage } from 'ember-responsive-image/test-support';

module('Integration: My Image Component', function(hooks) {
  setupResponsiveImage(hooks);

  test(...);
});
```
