--- 
title: "React vs. Angular Part 1: First Principles"
date: 2019-02-14T23:45:08-05:00
draft: false
---


##### Introduction #####

Before we get into the nitty gritty of React vs. Angular, it's worth understanding the basic reason for their existence.  

Tero Parviainen summarizes this quite well:

> The basic task [of UI frameworks] is to take the internal state of the program and map it to something visible on the screen. You take an assortment of objects, arrays, strings, and numbers and end up with a tree of text paragraphs, forms, links, buttons, and images. On the web, the former is usually represented as JavaScript data structures (the model), and the latter as the Document Object Model (DOM).

<div style="text-align:center"><img style="height:200px;width:500px" src="/images/posts/react_vs_angular_part_1/data_projection.svg" alt="Data Projection" /></div>

[Source](http://teropa.info/blog/2015/03/02/change-and-its-detection-in-javascript-frameworks.html)


Tero calls this process data projection.  

Building on Tero's idea, I like to think of the framework as the glue that maps Javascript data (the model) to the DOM.  In the early days, when front-end frameworks didn't really exist, developers had to write their own glue code.  Libraries like jQuery existed to make the process easier.

As the web evolved and the front-end become more complex, the glue code evolved along with it.  Straight jQuery yielded to lightweight frameworks like Backbone.js, and eventually to more complex frameworks like React, Angular, Ember.js and the like.

##### A Little History #####

Back before 2010 or so, there weren't any real de-facto front-end frameworks to speak of.  Most of the DOM was not generated within the browser.  The DOM was generated server-side, and the browser merely acted as a 'dumb' consumer of the server-generated HTML.

For example, if you clicked a 'Submit' button on a form, the process involved the following:

>1. The web page unloaded
>1. The browser would send data to the server
>1. The server would render a new page
>1. The server would send back the newly rendered page to the browser
>1. The browser displayed the new page

This was a simple approach, but rather slow, since a full client to server roundtrip was needed on every action.  These days, browsers have gotten extremely fast rendering the DOM and running Javascript (particularly fueled by Google's V8 engine in Chrome) that the whole process can be done right in the browser, allowing for much faster updates.

In the next part of this series, we'll examine React and Angular, and how they tackle the problem of data projection in different ways.
