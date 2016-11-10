Image to Individual Pixels in HTML and CSS
===

November 9, 2016

Original question from [>>/g/57436638](https://archive.rebeccablacktech.com/g/thread/S57436638).

> Are there any tools which convert images to a collection of html pixels which depict that image?

Its a little bit confusing, but what OP had in mind was to make something like this.

```html
<div id="parent" style="position:relative">
    <div class="child" style="position:absolute; width:1px; height:1px; background:#fff;
        top:0px; left:0px;"></div>

    <div class="child" style="position:absolute; width:1px; height:1px; background:#fff;
        top:0px; left:1px"></div>

    <!-- and so on -->
</div>
```

So its a pure HTML/CSS representation of an image, pixel by pixel. I had explored the concept before this, by crafting several icons by the pixels with HTML/CSS (with JS to inject the coordinates). Of course, it still looks bad, and when different DPIs come to play, everything is a mess.

With some free time at hand, I decided to try it out with plain, browser-side JS. Let me tell you first that everything here is a mistake. Generating the HTML/CSS is extremely slow, and is barely manageable with a square 128 pixels image. After that, attempting to open a saved HTML with all the pixels is also likely to kill your browser. Chrome seems to be able to handle it better (multi-process something something I think?), while Firefox just dies.

The whole source can be found in [Gist](https://gist.github.com/altbdoor/0c47bc4034449645285fb70913bea6d9). It is interesting to code for this, but another language would be more appropriate. But when one considers the fact that the output HTML can barely be opened by browsers...

---

#### References

- http://stackoverflow.com/questions/13938686/can-i-load-a-local-file-into-an-html-canvas-element
- http://stackoverflow.com/questions/667045/getpixel-from-html-canvas
