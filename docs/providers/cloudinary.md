# Cloudinary Provider

## Usage

```hbs
<ResponsiveImage @src={{responsive-image-cloudinary-provider "path/to/uploaded/image.jpg"}}/>
```

### Transformations

Besides the transformations that the addon itself implicitly adds (related to resizing images)
you can add your own [Cloudinary transformations](https://cloudinary.com/documentation/transformation_reference) by using the
`transformations` parameter:

```hbs
<ResponsiveImage @src={{responsive-image-cloudinary-provider "path/to/uploaded/image.jpg" transformations="e_sharpen:400,e_grayscale"}}/>
```

### Image formats

By default, all supported image formats (PNG, JPEG, WEBP, AVIF) are referenced in the generated `<source>` tags.
You can tweak that using the `formats` argument:

```hbs
<ResponsiveImage @src={{responsive-image-cloudinary-provider "path/to/uploaded/image.jpg" formats=(array "webp" "avif")}}/>
```

### Remote files

The provider supports Cloudinary's [`fetch`](https://cloudinary.com/documentation/fetch_remote_images) mode to automatically fetch
images from a remote source and deliver them (with optional custom transformations) through Cloudinary's CDN. Simply
pass a fully qualified http(s) URL:

```hbs
<ResponsiveImage @src={{responsive-image-cloudinary-provider "https://upload.wikimedia.org/wikipedia/commons/1/13/Benedict_Cumberbatch_2011.png" transformations="e_grayscale"}}/>
```
