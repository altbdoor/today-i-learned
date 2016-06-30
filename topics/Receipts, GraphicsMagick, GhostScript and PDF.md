Receipts, GraphicsMagick, GhostScript and PDF
===

June 30, 2016

Short story, parents always have a bunch of receipt PDFs to print out every month. In order to save on ink, I usually crop the PDF out as an image, and join them all together in a Word document before printing them out. Then I thought to myself, this should be automatable, right?

So I have a number of PDFs, each containing a single receipt. Each receipt takes up about half of the page, so optimally it would be nice to merge two receipts into one. Logic-wise, I do not think it would be an issue. Rather, is there any program that can do it, on Windows?

I found [GraphicsMagick](http://www.graphicsmagick.org/) (abbreviated as GM) and its capability of converting PDFs into images. Of course, there is also [ImageMagick](http://www.imagemagick.org/script/index.php), but personally I have had good experience with GM and batch image conversion. ImageMagick and GM share a lot of similarity, so what was done here should be doable with ImageMagick as well. GM has a dependency on [GhostScript](http://www.ghostscript.com/) to read/write PDFs. So I installed GM 1.3.24 (Q8) and GhostScript 9.19.

> Beware if you are out to get the installer file for GM. The download page conveniently points to [SourceForge](https://sourceforge.net/projects/graphicsmagick/files/), and the installer files are in the `graphicsmagick-binaries` folder. I have mistaken it to be in the `graphicsmagick` folder, and a higher download count with the `graphicsmagick` folder does not help either.
> 
> Additionally, you have to make sure that GM and GhostScript are installed with the same architecture. And, in case you are curious about what is Q8 (or Q16), GM has an [answer](http://www.graphicsmagick.org/INSTALL-windows.html#retrieve-install-package) for that. GM should add itself into the computer's `%PATH%`, so you can call GM directly with `gm` in the CMD.

After a number of trial and errors, I got a complete command to convert the PDFs into PNGs first, and a command to combine two PNGs into one.

```sh
# convert pdf to png
gm convert -density 200 \  # convert the pdf with dpi of 200. high density == high quality
    input.pdf \  # input pdf
    -quality 100 \  # the image output quality
    -crop 1460x940+95+285 \  # cropped out only the relevant section of receipt
    -colorspace Gray \  # only needed it to be grayscale
    -bordercolor white -border 120x200 \  # padding the image with a border
    output.png  # output png

# combine two png
gm convert input_1.png input_2.png \  # input is two png files
    -gravity South -chop 0x200 \  # chop out the earlier padding at the bottom
    -append \  # combine them vertically
    input_1.png  # output it, and replace the first png file
```

So with all the images ready, I need to convert it back to a printable format. At first I was looking for a Excel-CSV equivalent with Word, but I cannot find anything good. After a long search, only did I realize I could have just convert it back to PDF again. GM conveniently comes with a parameter `-page`, which accepts A4 as a value.

However, assuming we have an odd number of receipts, the last image would be, in a sense, more horizontal than vertical. Somehow I cannot force GM to convert the last image into a portrait PDF. I thought of padding the image until its "more vertical than horizontal", but that feels hacky. Luckily, I was able to find a similar question in [StackExchange](http://unix.stackexchange.com/questions/20026/convert-images-to-pdf-how-to-make-pdf-pages-same-size).

With that, the final command to convert the images into one PDF is finalized.

```sh
gm convert input_1.png input_2.png \  # input images
    -compress jpeg \  # convert into jpeg for smaller pdf file size
    -resize 1140x \  # resize the image into 1140px by width
    -extent 1240x1753-50 \  # extend the image into an a4 size paper
    -gravity center \  # make sure the image is placed in center
    -units PixelsPerInch \  # define the unit for density
    -density 150x150 \  # set density at 150
    -repage 1240x1753 \  # set page into a4
    output_compiled.pdf  # output it as pdf
```

While the commands are done, having to type them in is a hassle as well. I used Python at first, for a simple script to find all PDFs in current directory, and do the processing. But optimally, it would be nice to bundle it into an `.exe` and just double click. `py2exe` results in a pretty large file (as far as I can remember). In the end, I decided to use C# with `csc.exe`. With the `csc.exe` from .NET 2.0, it should work on every Windows version.

I did not include the Python or C# source here, but you can find them on [Gist](https://gist.github.com/altbdoor/008fbaeb8932169b6530d2fd6c4197b0). Pardon the lack of try-catch-throw and bad coding with C#, I made a lot of safe assumptions and it has been awhile since I last touched C#.


#### References

- http://www.graphicsmagick.org/
- http://www.imagemagick.org/script/index.php
- http://www.ghostscript.com/
- http://www.graphicsmagick.org/INSTALL-windows.html#retrieve-install-package
- http://unix.stackexchange.com/questions/20026/convert-images-to-pdf-how-to-make-pdf-pages-same-size
