# Cornerstone
[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/bigcommerce/cornerstone)
![tests](https://github.com/bigcommerce/cornerstone/workflows/Theme%20Bundling%20Test/badge.svg?branch=master)

Stencil's Cornerstone theme is the building block for BigCommerce theme developers to get started quickly developing premium quality themes on the BigCommerce platform.

## Demo repository and upstream updates

`origin` is `Space48/apps-cornerstone-demo`. Configure the official source once
with `git remote add upstream https://github.com/bigcommerce/cornerstone.git`,
then fetch updates with `git fetch upstream --tags`.

Keep `master` as the deployed integration branch: do not rebase or force-push it.
Pushing or merging into `master` deploys the theme. Keep each upgrade on a
versioned `demo/cornerstone-<version>` branch based on an official release tag,
with the demo commits replayed on top. Merge upgrade PRs using **Create a merge
commit**, not squash or rebase merging, to preserve upstream ancestry.

The original demo fork shares ancestor `a9b47cd1` with BigCommerce and has ten
project-only commits:

| Purpose | Original commits |
| --- | --- |
| Deployment, configuration sync and CI | `8dd8010e`, `ce3cf714`, `01fa90f6`, `b88f4459`, `0afa0fd1`, `57a5ffdd` |
| Hyper Search mounts | `8eb7cd14` |
| Product size-guide query and tab | `cc3e4e86`, `17438a81`, `63664857` |

`demo/cornerstone-6.22.0` rebases these changes onto tag `6.22.0`, dropping
`b88f4459` because its Node 18 CI setting is obsolete. It also backports
BigCommerce's Node 24 migration (`b82b2320`) and updates the demo workflows'
checkout, setup-node and cache actions to v4, matching upstream CI.

For the next upgrade, start from the previous patch branch, not from `master`.
For example, when a `6.23.0` release tag exists:

```sh
git fetch upstream --tags
git switch -c demo/cornerstone-6.23.0 demo/cornerstone-6.22.0
git rebase --onto 6.23.0 6.22.0
nvm install
nvm use
npm ci
npx grunt check
npx stencil bundle --name bundle
```

Resolve conflicts by retaining the demo integrations alongside upstream fixes.
Drop backports only when the target release already provides them. Install
`@bigcommerce/stencil-cli` separately if `stencil` is not available. Preview
Hyper Search and products with and without size guides against the demo store
before merging; a successful bundle cannot verify the external app or live
metafield data.

Make future demo changes on the latest patch branch (or a feature branch based
on it), then merge them into `master` through a PR. Keep Page Builder
configuration-sync changes on the patch branch too, so the next rebase retains
them. Do not copy upstream commits individually, except for explicit backports
such as the Node 24 migration above.

### Stencil Utils
[Stencil-utils](https://github.com/bigcommerce/stencil-utils) is our supporting library for our events and remote interactions.

## JS API
When writing theme JavaScript (JS) there is an API in place for running JS on a per page basis. To properly write JS for your theme, the following page types are available to you:

* "pages/account/addresses"
* "pages/account/add-address"
* "pages/account/add-return"
* "pages/account/add-wishlist"
* "pages/account/recent-items"
* "pages/account/download-item"
* "pages/account/edit"
* "pages/account/return-saved"
* "pages/account/returns"
* "pages/account/payment-methods"
* "pages/auth/login"
* "pages/auth/account-created"
* "pages/auth/create-account"
* "pages/auth/new-password"
* "pages/blog"
* "pages/blog-post"
* "pages/brand"
* "pages/brands"
* "pages/cart"
* "pages/category"
* "pages/compare"
* "pages/errors"
* "pages/gift-certificate/purchase"
* "pages/gift-certificate/balance"
* "pages/gift-certificate/redeem"
* "global"
* "pages/home"
* "pages/order-complete"
* "pages/page"
* "pages/product"
* "pages/search"
* "pages/sitemap"
* "pages/subscribed"
* "pages/account/wishlist-details"
* "pages/account/wishlists"

These page types will correspond to the pages within your theme. Each one of these page types map to an ES6 module that extends the base `PageManager` abstract class.

```javascript
    export default class Auth extends PageManager {
        constructor() {
            // Set up code goes here; attach to internals and use internals as you would 'this'
        }
    }
```

### JS Template Context Injection
Occasionally you may need to use dynamic data from the template context within your client-side theme application code.

Two helpers are provided to help achieve this.

The inject helper allows you to compose a JSON object with a subset of the template context to be sent to the browser.

```
{{inject "stringBasedKey" contextValue}}
```

To retrieve the parsable JSON object, just call `{{jsContext}}` after all of the `{{@inject}}` calls.

For example, to setup the product name in your client-side app, you can do the following if you're in the context of a product:

```html
{{inject "myProductName" product.title}}

<script>
// Note the lack of quotes around the jsContext handlebars helper, it becomes a string automatically.
var jsContext = JSON.parse({{jsContext}}); // jsContext would output "{\"myProductName\": \"Sample Product\"}" which can feed directly into your JavaScript

console.log(jsContext.myProductName); // Will output: Sample Product
</script>
```

You can compose your JSON object across multiple pages to create a different set of client-side data depending on the currently loaded template context.

The stencil theme makes the jsContext available on both the active page scoped and global PageManager objects as `this.context`.

## Polyfilling via Feature Detection
Cornerstone implements [this strategy](https://philipwalton.com/articles/loading-polyfills-only-when-needed/) for polyfilling.

In `templates/components/common/polyfill-script.html` there is a simple feature detection script which can be extended to detect any recent JS features you intend to use in your theme code.

If any one of the conditions is not met, an additional blocking JS bundle configured in `assets/js/polyfills.js` will be loaded to polyfill modern JS features before the main bundle executes. 

This intentionally prioritizes the experience of the 90%+ of shoppers who are on modern browsers in terms of performance, while maintaining compatibility (at the expense of additional JS download+parse for the polyfills) for users on legacy browsers.

## Static assets
Some static assets in the Stencil theme are handled with Grunt if required. This
means you have some dependencies on grunt and npm. To get started:

First make sure you have Grunt installed globally on your machine:

```
npm install -g grunt-cli
```

and run:

```
npm install
```

Note: package-lock.json file was generated by Node version 24 and npm version 11. The app supports Node 24 as well as multiple versions of npm, but we should always use those versions when updating package-lock.json, unless it is decided to upgrade those, and in this case the readme should be updated as well. If using a different version for node OR npm, please delete the package-lock.json file prior to installing node packages and also prior to pushing to github.

If updating or adding a dependency, please double check that you are working on Node version 24 and npm version 11 and run ```npm update <package_name>```  or ```npm install <package_name>``` (avoid running npm install for updating a package). After updating the package, please make sure that the changes in the package-lock.json reflect only the updated/new package prior to pushing the changes to github.


### Icons
Icons are delivered via a single SVG sprite, which is embedded on the page in
`templates/layout/base.html`. It is generated via a grunt task `grunt svgstore`.

The task takes individual SVG files for each icon in `assets/icons` and bundles
them together, to be inlined on the top of the theme, via an ajax call managed
by svg-injector. Each icon can then be called in a similar way to an inline image via:

```
<svg><use xlink:href="#icon-svgFileName" /></svg>
```

The ID of the SVG icon you are calling is based on the filename of the icon you want,
with `icon-` prepended. e.g. `xlink:href="#icon-facebook"`.

Simply add your new icon SVG file to the icons folder, and run `grunt svgstore`,
or just `grunt`.

#### License

(The MIT License)
Copyright (C) 2015-present BigCommerce Inc.
All rights reserved.

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense,and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
