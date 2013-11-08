---
layout: post
title: "Basic Designer Skills"
date: 2013-11-08 13:45
comments: true
categories: [iOS]
---

As an iOS developer, to learn some basic skills of a designer is good for us, which, for the most part, refer to the basic usage of Photoshop. In this case, when we receive a psd file from the designer, we can start to work without the help of designers to cut images and mark all the information in the comps. In this article, we'll cover topics about image seperation, image cropping, reading necessary infomation from the psd file etc. BTW I'm using Photoshop CS6.
###Image Seperation
Suppose we want to cut the twitter icon from the following image:

{% img /images/2013/11/twitter-button.png %}

first we want to select the twitter icon, so click on this icon on the left side bar of PS:

{% img /images/2013/11/ps-select-icon.png %}

Next maybe a new feature of CS6, we check the `Auto-Select` box and choose `Layer`:
 
{% img /images/2013/11/ps-auto-select.png %}

So that when we click on some components on the image, the right side bar (the layers tree) will highlight the exact layer of the component you click on, and when that happens, it means you have selected the component, in our case, the twitter icon:

{% img /images/2013/11/ps-twitter-select.png %}

Then we right click the selected layer on right side bar, choose `Duplicate layer` on the popup menu:

{% img /images/2013/11/ps-twitter-select-right-click-menu.png %}

Then in the dialog, we select `New` and click `OK`:

{% img /images/2013/11/ps-duplicate-layer.png %}

Then a new tab will show like this:

{% img /images/2013/11/ps-twitter-new-tab.png %}

Then we select this icon from the left side bar:

{% img /images/2013/11/ps-slice-icon.png %}

Use `'command'+'+'` to zoom in, and slice around the twitter icon with any size you like:

{% img /images/2013/11/ps-twitter-slice.png %}

I choose the size 70 * 58. You'd better make your slice width and height all even numbers. On the top menu, select `File->Save For Web`, in the popup dialog, click `Save`, in the next popup dialog, choose your file name and click `Save`:

{% img /images/2013/11/ps-save.png %}

The resulting icon will be in ~/Desktop/Images folder.

###Image Cropping
When we show a button, say, 200 pixels width, we don't actually need to cut a background image with 200 pixels width, a small one will do. Suppose we want a background icon for this button:

{% img /images/2013/11/button-bg.png %}

we select the layer as described above, duplicated layers, and open it in the new tab:

{% img /images/2013/11/button-bg-new-tab.png %}

Then we select this icon from the left side bar:

{% img /images/2013/11/direct-selection-tool.png %}

Click on the edge of the image, it'll show some little squares around the corner:

{% img /images/2013/11/button-bg-square.png %}

In the blank area we click and drag the cursor to select all four squares on the right side(or left side), and press `shift` + KEY left to narrow the background image to some suitable size:

{% img /images/2013/11/button-bg-narrow.png %}

The left steps are the same as described above.

###Read Text Infomation
To read text font, select this icon from left side bar:

{% img /images/2013/11/ps-text.png %}

Click on some label, and it'll show information on the top menu bar:

{% img /images/2013/11/ps-text-font.png %}

To read color infomation, select `Window->Info` on the top menu, info panel will show:

{% img /images/2013/11/ps-info-panel.png %}

When we move the cursor to the zoomed in label, we can read RGB values for text and its shadow.

###Read Position Infomation
Almost the same as reading text infomation, all position information can be read from the infomation panel. 

