Inspecting Angular built bundle
========================================================================================================================

September 6, 2022

Quick one. I have been working with Angular a lot, since my job pretty much revolved around it.
I also have a couple of side projects running on Angular, so naturally I'm somewhat curious to know what is inside the
final build of `main.js`.

A quick Google reveals that people were using [Webpack Bundle Analyzer](https://www.npmjs.com/package/webpack-bundle-analyzer),
but I did not get much from it... 

```sh
yarn build --stats-json
npx webpack-bundle-analyzer dist/foo/stats.json
```

I see a huge blob of green, with `main.ts + 209 modules (concatenated)`. No ways of knowing anything else...

Running back quickly to Google, there are quotes that Webpack Bundle Analyzer is no longer the cool kid!

> The Angular team **strongly recommends** to only use **source-map-explorer** to analyze your bundle size instead of _webpack-bundle-analyzer_
> 
> &mdash; [StackOverflow](https://stackoverflow.com/questions/46567781/angular-cli-output-how-to-analyze-bundle-files)

```sh
yarn build --sourceMap=true
npx source-map-explorer dist/foo/main.*.js
```

Voila, a clean, interactable webpage appears, with the file size of each dependency.

TIL, RxJS takes up 26.84KB in the bundle.

---

#### References

- https://www.digitalocean.com/community/tutorials/angular-angular-webpack-bundle-analyzer
- https://stackoverflow.com/questions/46567781/angular-cli-output-how-to-analyze-bundle-files
