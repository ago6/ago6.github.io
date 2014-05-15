---
layout: post
title: "Instagram jQuery Plugin Fix"
date: 2012-09-15 11:58
comments: true
categories: tech
---
When I decided I wanted to incorporate an Instagram "widget" of some sort on my site, I decided on the [jquery.instagram.js plugin](http://potomak.github.com/jquery-instagram/) since it looked easy to use and flexible.

However when I finally got it working on the site, I noticed there were some issues with the stream.

<!-- more -->

I had taken all the right steps. I set up the plugin and wanted it to display 4 images at the foot of the site. I set the required variables, including `show: 4`, as necessary for the plugin, along with setting up a button that when pressed will show you the next 4 images in the stream.

After getting the javascript to work I took a look at it live on the site, and after some additional CSS work to get it to look how I liked, every including the "show more" button seemed to be as it should. Happy with how things were working, I went back and made the necessary changes for the plugin to show images from my stream, which was a simple change of my getting and setting the `userId` and `accessToken` variables.

Going back to seeing it live, everything looked good and I saw the latest 4 images from my stream as expected. Awesome! I then went and pressed the "more" button expecting to see the next 4 images, but was instead greeted with a "no more results" error.

"Well, that's not right. I definitely have more than 4 images in my stream." I double checked everything and decided that there was an issue somewhere.

### to the source! ###

I went to the `jquery.instagram.js` file and started adding some `console.log`s throughout the file to see if I could spot anything that might shed some light.

While stepping through the code it looked like the "more" type button does an API request to a predefined pagination url, which is set on the initial API request to instagram. Pretty straight forward. So a request is made to Instagram, Instagram returns X images along with a url to receive the next X images, and this button was making a request to that pagination url and showing those images. After looking through that portion of the code everything seemed to be in order and not the cause of the issue...

I then took a look at the response to the API request the plugin was making to instagram. The response contained almost a dozen images, even though I only wanted to show 4. So I took a look at the code that actually shows images;

```javascript
if(limit > 0) {
  for(var i = 0; i < limit; i++) {
    that.append(createPhotoElement(res.data[i]));
  }
}
```

So it just takes the first 4 (or however many you set as to be shown) images from the response and shows them. Okay, fair enough.

But now we're getting close to the issue.

So Instagram's response to the API call includes the images along with the pagination URL for the next batch for that query, assuming there are more results for the initial query than images the response is currently returning. Now in my stream I currently have 8 or so images, and on that initial API call to Instagram, Instagram was returning all 8 images, so there was no pagination URL returned. Now the plugin was doing the request (and in my case returning all 8 of my images), and just showing the first 4 on the page. And when you click the "more" button to paginate, it fails, because Instagram has nothing more to give you (even though the plugin only showed 4)!

Plugin is doing the request, is given images 1 - 10 (default behavior), shows the first 4, then asks for images 11-20, and shows the first 4 from *that* request. In the end, we're only shown images 1-4, and 11-14, completely skipping 12 images.

Okay, now that I know the problem, what's the fix? Do I rewrite the button to iterate through the results returned instead of relying on the pagination URL? That probably won't be a minor rewrite.

After looking through Instagram's API documentation I believe I found the easiest solution. When doing an API request to Instagram, you can specify the amount of images you want back, with the pagination URL feature continuing to work as expected. So if you have 12 images in your stream, and you do a request for the first 4, along with being returned the images, you'll be given the pagination URL for images 5 - 8.

Perfect, incorporating this allows me to fix the issue with modifying as little of the existing code as possible.

I made the change locally, adding the `show` parameter defined when using the `instagram` function as a `count` parameter in the API request sent to Instagram, so Instagram will only return what I want to show, along with a pagination URL for the next batch if I choose to show them. Saved the new version, tested it on the site, and poof, problem resolved!

### Now to Contribute to the Open Source Project ###

Since this is an issue that is affecting everyone using this plugin, and I already did the leg work on discovering and resolving it, it only makes sense to contribute to the project.

I went ahead and forked the repo, created the branch, and did a pull request for my succinct diff, and it was approved and merged into master the next day. Hooray for open source!

If you ever need a javascript plugin for Instagram, I recommend taking a look at [jquery.instagram.js](https://github.com/potomak/jquery-instagram). It's popular for a reason!
