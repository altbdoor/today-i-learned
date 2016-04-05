Input Number Polyfill
===

April 5, 2016

Part of a project requires me to simulate an `<input type="number">` for the input fields. While it can be seen that it is widely supported in [caniuse.com](http://caniuse.com/#feat=input-number), its not exactly accurate in behavior. For one, Firefox appears to allow users to input alphabets, only to warn them when the form is submitted.

Originally, another developer had used an answer from [StackOverflow](http://stackoverflow.com/questions/995183/how-to-allow-only-numeric-0-9-in-html-inputbox-using-jquery) as a solution, but the edge cases are infinitely possible. The idea of adding another edge case into the if-else block really puts me off. So I went looking for a more solid solution.

Of course, its all over StackOverflow, with most of the solutions suggesting the `keypress` event with a bunch of if-else blocks (again). However, I did went through with some of the non-accepted solutions, and was pretty surprised to find this [answer](http://stackoverflow.com/questions/995183/how-to-allow-only-numeric-0-9-in-html-inputbox-using-jquery#32004562).

> You can use on input event like this:

I would consider myself pretty experienced with JQuery (despite only learning about [Event Delegation](https://learn.jquery.com/events/event-delegation/) properly not too long ago), but I have not heard of the `input` event. At first, I thought it was a custom JQuery event, until I found its documentation on [Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/Events/input).

With `oninput`, every thing falls in its right place very quickly.

```js
$('input[type="number"]').on('input', function () {
    this.value = this.value.replace(/[^0-9\.]/g,'');
});
```

Pretty good, except for the fact that the regular expression allows `123.45.67`, which is invalid. So it is another Google search for a proper `is_numeric` implementation in JavaScript to use. No surprises, as [StackOverflow](http://stackoverflow.com/questions/9716468/is-there-any-function-like-isnumeric-in-javascript-to-validate-numbers) comes to the rescue again.

```js
$('input[type="number"]').on('input', function () {
    var val = this.value;
    
    if (!isNaN(parseFloat(val)) && isFinite(val)) {
        return;
    }
    else {
        this.value = this.value.replace(/[^0-9\.]/g,'');
    }
});
```

Still imperfect. `parseFloat('123.45.67')` returns `123.45`. So its time to whip up some more regular expression.

```js
$('input[type="number"]').on('input', function () {
    var val = this.value;
    
    if (
        val != '' && /^\d+(\.(\d+)?)?$/.test(val) &&
        !isNaN(parseFloat(val)) && isFinite(val)
    ) {
        return;
    }
    else {
        val = val.replace(/[^0-9\.]/g,'');
        
        if (/^\d+(\.(\d+)?)?$/.test(val)) {
            this.value = val;
        }
        else {
            var firstPeriod = val.indexOf('.'),
                secondPeriod = val.indexOf('.', firstPeriod + 1);
            
            this.value = val.substr(0, secondPeriod);
        }
    }
});
```

This part was a little extra tricky. Because the regular expression has to be true when a user inputs `123.`, before the decimals come in. Then, an extra handler is introduced to `substr` the value to before the second decimal, effectively turning `123.45.67` to `123.45`.

By the time I was done with this, I was informed that we need to allow even more weird characters (brackets, spaces, slashes), and it appears that it was unfeasible for me to continue even further, so I just gave up. However, I found some crucial information from here, so you might want to pay attention to these extra details.

1. [`parseFloat` does not handle internationalization](http://stackoverflow.com/questions/12694455/javascript-parsefloat-in-different-cultures). Some countries use comma as a decimal separator, but `parseFloat` does not recognize them. Might need to convert these values back and forth.

1. `oninput` is not compatible with old browsers (some wacky issues with IE 9 and below), [even with JQuery](https://forum.jquery.com/topic/html5-oninput-event). Not a concern if the application is bleeding edge.

1. I found a [JQuery plugin by spicyj](https://github.com/spicyj/jquery-splendid-textchange), but it appears to be not maintained anymore. From the issues and pull requests, it looks like a user named [milang](https://github.com/milang) has fixed these issues, but the pull request was not merged. So it might be better to use the [fork](https://github.com/pandell/jquery-splendid-textchange/).

Would be nice to revisit this someday (maybe as a full-blown plugin), but for today, its just something I learned.

---

#### References

- http://caniuse.com/#feat=input-number
- http://stackoverflow.com/questions/995183/how-to-allow-only-numeric-0-9-in-html-inputbox-using-jquery
- https://developer.mozilla.org/en-US/docs/Web/Events/input
- http://stackoverflow.com/questions/9716468/is-there-any-function-like-isnumeric-in-javascript-to-validate-numbers
- http://stackoverflow.com/questions/12694455/javascript-parsefloat-in-different-cultures
- https://forum.jquery.com/topic/html5-oninput-event
- https://github.com/spicyj/jquery-splendid-textchange
- https://github.com/pandell/jquery-splendid-textchange/
