---
title: Performance Optimizations for SPAs
date: 2019-05-01 11:04:28
tags:
 - performance
---

Single Page Application (SPA) architecture is a popular way to build modern web applications, generally utilizing client-side JavaScript code to handle much of the app's logic. In traditional non-SPA architecture a lot of this code, such as routing, templating, and other logic, would run only on the server. Consequently, optimization of client side code is of greater concern with a SPA. The most common SPA hosting setup I've seen is a React app hosted with AWS S3 and served via CloudFront, so I'll focus on that configuration, but these optimizations are generally applicable to any web app that serves static resources.

## Improve static asset delivery

Many SPAs — like those created with [Create React App](https://facebook.github.io/create-react-app/) and [Vue CLI](https://cli.vuejs.org/) — use [webpack](https://webpack.js.org/) to generate a directory of static assets required by the app. These files include JavaScript, CSS, images, etc. — anything that can be served from your CDN, since it doesn't need to be processed server side. Serving these static files to client web browsers will typically be the source of the overwhelming majority of the server-client data transfer for an app. (The rest is usually dynamically generated API responses, which are likely to be relatively small.)

### Compress files

Compressing your static files before delivering them can massively reduce the amount of data you transfer to your users. On a recent React app I worked on, enabling compression reduced the size of a randomly selected static JS file from 797 KB to 222 KB — a reduction of 72%, which is not at all atypical.

If you use CloudFront to distribute your static assets, enabling Gzip compression is easy (and not enabled by default). [This AWS News Blog post](https://aws.amazon.com/blogs/aws/new-gzip-compression-support-for-amazon-cloudfront/) does a good job of showing how to do it. If you use another CDN or server software to serve static files, you should be able to find similar documentation. For example, I use Caddy to host my websites, which utilizes the [http.gzip](https://caddyserver.com/docs/gzip) directive to accomplish the same thing.

To see how much Gzip is helping reduce filesize, you can use the Network tab in Chrome DevTools. In this example, the resource's uncompressed size was 797 KB, but only 222 KB needed to be transferred due to compression:

{% asset_img resource-detail.png "Resource detail" %}

By looking at the response headers, we can see the file was served with Gzip compression (`Content-Encoding: gzip`):

{% asset_img response-headers.png "Response headers" %}

Note: you can compress API responses, but this will be likely be done by an application server, not your CDN. For example, a Node application using Express can utilize [middleware to compress API responses](https://expressjs.com/en/advanced/best-practice-performance.html#use-gzip-compression). This is typically less useful than compressing static files, since API responses are usually a lot smaller so compression saves fewer bytes over the wire.

### Add cache control headers

Webpack has the ability to tag output file names with hashes based on their content (they'll look something like `app.bl4hbl4h1337.js`). Both Create React App and Vue CLI include these content hashes in their output by default. Since a filename will always refer to a specific version of a file, you don't have to worry about request for a file returning an old version of it; when the contents of a file update, its name will change as well. This property allows you to take advantage of long term caching for your static files.

If you serve your files with S3 and CloudFront, you can add caching headers by adding metadata to your files in S3. This is typically accomplished during your deployment routine, which might look something like this:

```bash
# (run webpack build, do other config stuff)

aws s3 cp $DEPLOY_DIR $S3_URI \
  --recursive \
  --exclude "*" \
  --include "*.js" \
  --include "*.css" \
  --include "*.map" \
  --include "*.html" \
  --exclude "index.html" \
  --cache-control public,max-age=31536000 # cache static files for a year

aws s3 cp $TEMP_INDEX $S3_URI/index.html --cache-control public,max-age=60 # cache index.html for a minute
```

An important thing to note here is that the entry point (index.html) should have a very short `max-age`. In this example I chose 60 seconds, but I have also seen it just set to 0. Since this value is the maximum amount of time for which index.html can be cached, it represents time users can potentially can be served outdated links to assets. The advantage of making it non-zero is to allow for some caching of index.html on websites that serve a lot of traffic.

## Split JavaScript code into multiple files

The general idea of code splitting is to split your big application bundle into multiple pieces that are loaded on demand (aka "lazy loaded"). This can have a lot of advantages: infrequently used or large sections of your codebase don't need to be loaded unnecessarily, infrequently updated chunks can benefit from caching (see above), and your app can parallelize the download and script evaluation stages by downloading a new chunk while evaluating the previous one.

In one React app I worked on, an authentication function checked if the user was logged in and should be able to see the dashboard; if not, the user would be redirected to a login page. Without code splitting, the entire app bundle — around 4 MB — needed to be downloaded and evaluated by the browser only to immediately redirect the user away from the app! With code splitting, the minimal code required to auth could instead be executed and the rest of the app code only loaded if the user were authorized.

Most apps see their developer-written code updated frequently, but vendor code such as in `node_modules` changes less often. For this reason, Create React App and Vue split vendor code into their own bundles by default. You can go a step farther by splitting out sections of the app that are rarely touched. If none of the code in a specific chunk changes, the hash in its filename won't change and your users will just use the version cached by their browser, completely eliminating download time.

The finer details of code splitting are well beyond this post, but you can get started easily by using webpack's [dynamic imports](https://webpack.js.org/guides/code-splitting/#dynamic-imports) or [React.lazy](https://reactjs.org/docs/code-splitting.html#reactlazy) (this solution is React-specific, if you couldn't guess.) You can confirm that your formerly monolithic app bundle is now being split into multiple pieces by looking at the output of your webpack build, or monitoring the Network tab in DevTools while browsing your app to see the different bundles load on-demand.

## Test your results

I like to use [https://www.webpagetest.org/](https://www.webpagetest.org/) to test that I've properly implemented static asset compression and caching headers, and verify that the app bundle is reasonably small or split apart such that time to load is reasonable.  I'll typically change the default number of runs to 9 (under Advanced Settings) to get a larger sample, and experiment with different connection speeds to see how the impact of various optimizations changes.

There are a lot of ways to improve performance in a web app. I've focused here on some common low-hanging fruit that are particularly relevant for the common use cases of Vue and React apps built using webpack. In a future post I'll discuss some framework-specific things to watch out for.