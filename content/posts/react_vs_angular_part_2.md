---
title: "React vs. Angular Part 2: Change Detection"
date: 2019-02-17T19:52:50-05:00
draft: false
---

##### Introduction #####
In [Part 1](/posts/react_vs_angular_part_1) we discussed how the essential job of a UI framework is to project data.  The UI framework takes Javascript arrays, objects, strings, numbers, etc (the model) and coverts them into the DOM tree (the view).

However, projecting data, though obviously important, is not actually the hard part of the UI framework implementation.  Rather, the difficult part is to determine when the model changes, and thus when to update the DOM.  For example, when a user clicks the 'Submit' button on a form, and we get back new data from the server, we must detect that there is new data, and then update the DOM.  This is called change detection.  

One of the important differences between React and Angular is their approach to detecting changes in the data.  

##### Angular Change Detection - Dirty Checking #####

Angular separates view and logic by using a template to represent the view and Javascript component code to represent logic.  

In our case, we're particularly interested in the template, because any data tied to the template view is what should be checked for changes.

You specify data bindings in an Angular template with an expression such as `{{foo.x}}`:

```html
<p>{{foo.x}}</p>
```

When data is specified in such a manner, Angular now knows it needs to watch for changes on `foo.x` and update the `<p>` node in the DOM when `foo.x` changes.  In order to do this, Angular creates a watcher on `foo.x`.  A watcher is a function that gets executed when any browser event is fired (mouse clicks, keyboard events, API calls, etc).

<div style="text-align:center"><img style="height:200px;width:500px" src="/images/posts/react_vs_angular_part_2/angular_onchange_watch.svg" alt="Angular Watchers" /></div>

[Source](http://teropa.info/blog/2015/03/02/change-and-its-detection-in-javascript-frameworks.html)

In Angular 2+, a watcher is called a `ChangeDetectorRef`.  Each component you define in Angular has a `ChangeDetectorRef` automatically generated for it at compile-time.  

In our example, the `ChangeDetectorRef` holds onto the last value of `foo.x`.  Whenever a new event is fired, the `ChangeDetectorRef` checks if the value of `foo.x` changed (this is what we call dirty checking).  If `foo.x` did change in response to the event, it will re-render the DOM element associated with the template in which `foo.x` is referenced.

The benefit of Angular's approach to change detection is that Angular creates `ChangeDetectorRef`s during compile time and automatically calls them in response to events so you aren't required to (unlike React where you need to manually call  the `setState` method).

However, there is one clear drawback to this approach.  On every event, Angular must run all `ChangeDetectorRef`s for every component since it doesn't know what might have changed for any given event.  Though there is a way to speed this up via use of [Observables](https://medium.com/@luukgruijs/understanding-creating-and-subscribing-to-observables-in-angular-426dbf0b04a3) or [Immutable data structures](https://blog.thoughtram.io/angular/2016/02/22/angular-2-change-detection-explained.html#immutable-objects), you'll have to explicitly understand where/how to use them and write your components in the appropriate manner to utilize them.

##### Angular Change Detection - Algorithm #####

The following is a more detailed view of the change detection algorithm in Angular.  I provide it here only for the curious.  Don't worry if you don't completely understand all the details discussed here!

As I mentioned, the `ChangeDetectorRef` is the central class responsible for detecting and updating changes to the DOM.  I provide a conceptual version of the class below:

```javascript

function onBrowserEvent() {
    for each node in DOM:
        node.changeDetectorRef.detectChanges();
}

// This class gets instantiated for every component 
// created in the Angular app at runtime and
// assigned to a node
class ChangeDetectorRef {

    // Pass in the domNode that this ChangeDetectorRef is 
    // responsible for, and the bindingFunction that 
    // we execute on every browser event.
    
    // The binding function is created at compile-time by the 
    // Angular compiler, and it is guaranteed to return 
    // the most up-to-date value  for foo.x
    constructor(domNode, bindingFunction) {
        this.domNode = domNode;
        this.boundValue = this.domNode.value;
        this.bindingFunction = bindingFunction;
    }

    // This function gets called by Angular on every browser event
    detectChanges() {
            
        // get the existing bound value
        const oldValue = this.boundValue;
        // get the new value from the bound function associated 
        // with this node.
        const newValue = this.bindingFunction()
        // If the value changed
        if (oldValue !== newValue) {
            // Set the value for this node in the actual DOM
            this.domNode.value = newValue;
            // Record this boundValue separately as the new value 
            // associated with this DOM node
            this.boundValue = newValue;
        }
    }
}
```

Although this is not exactly how the `ChangeDetectorRef` class is implemented internally, it is conceptually similar.

It's worth noting that in the above code, any read/update operations performed directly on the DOM is expensive.  Therefore, we store the DOM value separately as `this.boundValue` and read from that.  Also, we only update the value in the DOM if it has changed.

##### React Change Detection - Virtual DOM #####

Unlike Angular, where `ChangeDetectorRef`s attach to every component and DOM components are updated only when they are changed, React re-renders the *entire DOM* on every change.  This may seem cumbersome and inefficient, but in fact, React doesn't render directly to the DOM.  It renders to the virtual DOM.  The virtual DOM is a light Javascript structure of objects that represents the actual DOM.

Every time a change occurs, the virtual DOM is recreated in its entirety.  Remember how Angular keeps track of both the old and new values for a bound element in order to see if anything changed?  Well, React does something similar, but for the *entire DOM*.  

React keeps 2 Virtual DOMs around: one virtual DOM that represents the DOM before the change, and a second one that represents the DOM after the change.  It then diffs both Virtual DOMs and figures out the set of DOM elements that changed between them, and then applies those changes to the actual DOM.

<div style="text-align:center"><img style="height:320px;width:520px" src="/images/posts/react_vs_angular_part_2/react_onchange_vdom_change.svg" alt="Angular Watchers" /></div>

[Source](http://teropa.info/blog/2015/03/02/change-and-its-detection-in-javascript-frameworks.html)


The benefits of React's approach with virtual DOM diffing is that you don't need to setup change detectors for each individual component at compile-time as Angular does.  Rather, you are continuously diffing new virtual DOMs at run-time in response to `setState` calls in the React code.  

However, although virtual DOM diffing is quite fast, is not necessarily faster than Angular's `ChangeDetectorRef`s.  This is because React needs to re-create a new virtual DOM and diff it against the old one on every change.  With Angular, we're merely diffing new and old bound values.

##### React Change Detection - Algorithm #####

The following is a more detailed view of the change detection algorithm in React: 

```javascript
function reactRender(reactComponent, node){
  //get the current virtual DOM
  const currentVdom = currentDomForNode(node);
  //call the render method to get the new vdom
  const newVdom = reactComponent.render();
  //diff the old and new vdoms to get a list of changes to the DOM
  const changes = reactDiff(currentVdom, newVdom);
  //apply those changes to the real DOM
  changes.forEach(reactApplyChangeToDom);
}
```

One of the nice things about the React architecture is that Virtual DOM diffing is an elegant and simple concept, making the high-level code easy to understand.

