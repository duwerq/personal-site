---
title: "Upload Image from React-Native to Rails API"
layout: post
date: 2016-12-17 22:44
image: /assets/images/markdown.jpg
headerImage: false
tag:
- react-native
- ruby on rails
blog: true
author: matthewmarks
description: Upload Image from React-Native to Rails API
---

** *The intent of this tutorial is to show how to pick an image inside React-Native, take the base64 encoded data, upload it to a rails server, decode the image and then upload the image to Amazon S3.*

We’ll start with what collecting the User’s chosen image.  First we’ll be installing react-native-image-picker (https://github.com/marcshilling/react-native-image-picker) . Follow their installation directions and then declare the module inside your app:

{% highlight js %}
var ImagePicker = require('react-native-image-picker');
{% endhighlight %}

We’ll create a function and declare an options variable with our desired settings:

{% highlight js %}
_updateImage() {

  let options = {
    quality: 1.0,
    maxWidth: 500,
    maxHeight: 500,
    storageOptions: {
      skipBackup: true
    }
  };

}
{% endhighlight %}

Then we’ll call ImagePicker, the user will select a photo, we’ll take the base64 data and send it to an action that will upload the image data to our rails app.

{% highlight js %}
  ImagePicker.showImagePicker(options, (response) => {
    console.log('Response = ', response);

    if (response.didCancel) {
      console.log('User cancelled photo picker');
    }
    else if (response.error) {
      console.log('ImagePicker Error: ', response.error);
    }
    else if (response.customButton) {
      console.log('User tapped custom button: ', response.customButton);
    }
    else {
      let source = null;

      if (response.origURL) {
        if (response.uri.includes(".gif")) {
          source = {uri: 'data:image/' + 'gif' + ';base64,' + response.data, isStatic: true};
        } else if (response.uri.includes(".png")) {
          source = {uri: 'data:image/' + 'png' + ';base64,' + response.data, isStatic: true};
        } else {
          source = {uri: 'data:image/' + 'jpeg' + ';base64,' + response.data, isStatic: true};
        }
      }
      return this.props.uploadImage(source.uri)
    }
  })

}
{% endhighlight %}

So let’s go a little more into detail over what’s happening here.  First we’re following react-native-image-pickers instructions verbatim:

{% highlight js %}
ImagePicker.showImagePicker(options, (response) => {
    console.log('Response = ', response);

    if (response.didCancel) {
      console.log('User cancelled photo picker');
    }
    else if (response.error) {
      console.log('ImagePicker Error: ', response.error);
    }
    else if (response.customButton) {
      console.log('User tapped custom button: ', response.customButton);
    }
    else {
{% endhighlight %}

This bit of code afterwards checks the image url and constructs the base64 encoded image uri with the correct image type and assigns it to the variable source.

{% highlight js %}

let source = null;

if (response.origURL) {
        if (response.uri.includes(".gif")) {
          source = {uri: 'data:image/' + 'gif' + ';base64,' + response.data, isStatic: true};
        } else if (response.uri.includes(".png")) {
          source = {uri: 'data:image/' + 'png' + ';base64,' + response.data, isStatic: true};
        } else {
          source = {uri: 'data:image/' + 'jpeg' + ';base64,' + response.data, isStatic: true};
        }
      }
{% endhighlight %}

The final code returns the full base64 image uri to an action that is dispatched through Redux.

{% highlight js %}
return this.props.uploadImage(source.uri)
{% endhighlight %}

Our dispatched action will make an API call to our rails application with our base64 image url.  Our request to the rails app should include a payload similar to this:

{% highlight js %}
payload = {
            "user": {
              user_image: data:image/jpeg;base64,d39if98……(very long string)
            }
          }
{% endhighlight %}

Now when the API call is received by our rails application, we’ll need to decode the image.  This next part of the tutorial I have to give all credit to Sebastian Dobrincu, so much so that I’m not going to reinvent the wheel because he did a phenomenal job the first time.  Follow his directions here.  The only recommendation I can make is to to his convert_data_uri_to_upload(obj_hash) method.  Inside he declares an img_params variable that includes the name of the file, the type of image data the image data itself.

{% highlight ruby %}
img_params = {:filename => "image.#{image_data[:extension]}", :type => image_data[:type], :tempfile => temp_img_file}
{% endhighlight %}

When you set:
{% highlight ruby %}
  filename => "image.#{image_data[:extension]}"
{% endhighlight %}

Change 
{% highlight ruby %}"image.#{image_data[:extension]}" {% endhighlight %} 

to something like 

{% highlight ruby %}
"image#{obj_hash[:image_id]}.#{image_data[:extension]}" 
{% endhighlight %}  

The reason we replace the word image in the filename with 

{% highlight ruby %}
"#obj_hash[:image_id]"
{% endhighlight %} 

is to add a unique a filename for each image.  If left the way it is, each new image will have the same name as the one before it and when it gets uploaded to Amazon S3 in the same bucket, it will continue to overwrite the previous image instead of adding to a collection of images.  This is a small detail but can save you a little bit of frustration.

Now if I’m being honest, most of this tutorial is really someone else's instructions and me connecting the dots with a bit of insight.  That being said though, I hope you find some use from it and if you have any questions or comments, they’re always welcome.
