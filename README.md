# Custom events and unbinding

## Overview

We've used custom events already, but let's look at using jqLite to add events to our elements. We also need to unbind these events.

## Objectives

- Describe using DOM events
- Describe the lifecycle of events and potential memory leak issues
- Create a native DOM event
- Unbind the native DOM event using $destroy

## jqLite events

jqLite allows us to use `.on` and `.off` on our `element` inside the link function. `.off()` will remove all event listeners that we've added to the element, and `.on` let's us add them for different event types, such as `click`.

If we don't unbind our events, we may suffer with issues later down the line as our element will no longer exist but our event callbacks may still be fired. If we're updating our element inside of our callback, we will get errors because the element no longer exists.

Let's add a click event to our element:

```js
function ourDirective() {
  return {
    transclude: true,
    template: [
      '<div>',
        'Click on me!',
      '</div>'
    ].join(''),
    link: function (scope, element) {
        element.on('click', function () {
            alert('You clicked me!');
        });
    }
  };
}

angular
  .module('app', [])
  .directive('ourDirective', ourDirective);
```

Awesome! Now we can add native DOM events to our element. Here we can also target `window`, and `document`, etc, as we might have to change our directives behaviour when the browser resizes or when the user scrolls the page. But how do we know when to unbind these events?

## $destroy

Luckily, we get an event fired when our directive is *about* to be unmounted from the DOM. We can subscribe to this event using `scope.$on`.

```js
function ourDirective() {
  return {
    transclude: true,
    template: [
      '<div>',
        'Click on me!',
      '</div>'
    ].join(''),
    link: function (scope, element) {
        scope.$on('$destroy', function () {
           alert('About to be destroyed!');
        });
    }
  };
}

angular
  .module('app', [])
  .directive('ourDirective', ourDirective);
```

This will alert us when our directive is about to be removed from the DOM. Now, we can unsubscribe from these events. Luckily, as explained earlier, we can use `.off()`!

```js
function ourDirective() {
  return {
    transclude: true,
    template: [
      '<div>',
        'Click on me!',
      '</div>'
    ].join(''),
    link: function (scope, element) {
        element.on('click', function () {
           alert('You clicked me!');
        });

        scope.$on('$destroy', function () {
		   element.off();
		});
    }
  };
}

angular
  .module('app', [])
  .directive('ourDirective', ourDirective);
```

Sorted! We've now created new events and removed the listeners when our directive gets destroyed.