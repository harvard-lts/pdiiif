# Harvard PDIIIF fork

This fork of PDIIIF is intended to be used for developing a customized coverpage API endpoint for use with PDIIIF. For all other uses of PDIIIF you should refer to [the main PDIIIF repository](https://github.com/jbaiter/pdiiif).

## Branching strategy

1. `main` is **locked** (read-only), but _can_ and **should** be used to synchronize with `jbaiter/pdiiif:main`. This will avoid incurring conflicts in directly in this shared branch.
2. `harvard-main` is the default branch, and is intended to keep commits dealing with Harvard specific changes out of `main`. Merge in `main` in order to pull in upstream changes and resolve any conflicts.
3. Create a feature branch for new tickets using the Jira ticket ID. Once complete open a PR against `harvard-main`

## The role of Docker in this project

Seems like it's primarily to provide the back-end service locally _however_, the web app is also accessible via the same port (`http://localhost:8080`).

It's uncertain if it's best to work inside Docker or outside of Docker for local development. You'll find an outline of both scenarios below.

## The role of pnpm

As a [monorepo](https://monorepo.tools/) there are multiple packages / apps managed in this one repository. Pnpm is a drop in replacement for npm/yarn, and is a popular choice for monorepos because it's faster, more efficient and can work well with the monorepo architecture.

You should be aware of it's existence, but most important that the `pnpm link` command was used to create symlinks to the `pdiiif-lib` directory. This means that although `pdiiif-web` appears to import from the `pdiiif` package in `node_modules`, it's actually linked to the compiled assets in the `pdiiif-lib` within this workspace.


## Choosing to use docker for development (Easier to get started, harder developer experience)

You should be able to follow the instructions in the [Quickstart](#quickstart) section to get things up and running.

Docker offers a more consistent development experience by its nature, but there's a few hoops you may need to jump through with the current setup:

1. There are no volumes defined out of the box. If you want to update a file on the host and see the change reflected in the container you'll need to run `docker cp <path to host file> pdiiif:<path to container file>`.
2. In order to see changes reflected in the packages, you may need to do one or more of the following:
    - If it's a code change in a source file you can shell into the container with `docker exec -it pdiiif bash` and use navigate to the appropriate package and use `pnpm build` compile the changes. You can also use the `-r` flag, e.g. `pnpm -r build` to build all the packages
    - You might need to restart the container (I've found this to be necessary for updating `coverpage.hbs`) using `docker restart pdiiif`

## Choosing not to use docker for development (Harder to get started, better developer experience)

You'll want to make sure that your local node version matches the one in the Dockerfile. I recommend [Node Version Manager](https://github.com/nvm-sh/nvm).

I recommend using pnpm and installing it globally.

Recursively install all the package dependencies:

`pnpm -r install`

You'll need to build all three of the packages **in a specific order** (web and api depend on the dist assets of lib), paste this code:

```
pnpm -C pdiiif-lib run build && \
pnpm -C pdiiif-api run build && \
pnpm -C pdiiif-web run build
```

It should then be possible to update files in `pdiiif-lib`, `pdiiif-api`, and `pdiiif-web` on the host machine while using their respective npm scripts to compile the code.

To view the results, you can start the local server and coverpage service by running:

`pnpm -C pdiiif-api run start` 

Which should start a local dev server running on port `31337`.

If you have have problems with this initially as I did, see below.


## Issues I've encountered

### To run the `start` / `dev` scripts in `pdiiif-web` (this _might_ just be a linux thing)

Update vite-plugin-mkcert to 1.13.3 . You may also need to create/delete/modify the permissions on a directory in your machines home directory. At first I had an error because it wasn't there, then one which suggested incorrect permissions, then finally deleting it again made it work. Go figure.

### To have unobfuscated code in your browser inspector:

You may need to add the following lines to the `tsconfig.cjs.json` in `pdiiif-lib`:

```
    "sourceMap": true,
    "declaration": true,
    "declarationMap": true
```

### To run the jest tests

Tests currently fail to run both inside and outside of the container. We think this is a known issue...

---

[![pdiiif logo](pdiiif-web/assets/logo.svg)](https://pdiiif.jbaiter.de)

[**Demo**](https://pdiiif.jbaiter.de)

[**Sample PDF generated with the library**](https://pdiiif.jbaiter.de/wunder.pdf)

[**Library API Documentation**](https://jbaiter.github.io/pdiiif)

**pdiiif** is a JavaScript library to **create PDFs from IIIF Manifests.**
For the most part, it runs both in browsers and as a Node.js server-side
application. When generating a PDF in the browser, almost all communication happens
directly between the user's browser and the IIIF APIs referenced from the Manifest.
The only exception is for generating the cover page, which by default needs to be
generated on the server. (see [this section](#cover-page-endpoints) for more details)

It comes with a small **sample web application** that demonstrates
how to use the library in the browser, you can check out a public instance
of it at https://pdiiif.jbaiter.de, the source code is located in the
[`pdiiif-web` subdirectory](https://github.com/jbaiter/pdiiif/tree/main/pdiiif-web).

A main goal of the library is to be as _memory-efficient_ as possible, by
never holding more than a few pages in memory and streaming directly to
the user's disk (precise method depends on the environment).

It is also well-suited for embedding in other applications due to
its relatively small footprint, for example, the example web application comes
in at **~120KiB gzipped** with all dependencies.

In addition to the images on the IIIF Canvases referenced in the manifest,
the library can create a **hidden text layer** from OCR associated with
each canvas (ALTO or hOCR referenced from a canvas' `seeAlso` property).

In order to not sever the connection between the PDF and the original IIIF
resources on the Web, every PDF generated by `pdiiif` includes the IIIF Manifest
as a PDF attachment, as well as every OCR file referenced in `seeAlso`.
Additionally, `pdiiif` can generate the PDFs in a way that also makes them
valid ZIP files that contain the manifest and all of the images and OCR files,
with almost no storage overhead. (thanks to [Ange Albertini](https://github.com/corkami)
and his work on [Poc||GTFO](https://pocorgtfo.hacke.rs/) for the inspiration!)

## Features

- [x] PDF Page for every single-image Canvas in a Manifest
- [x] Rendering Canvases with multiple images
- [x] PDF Table of Contents from IIIF Ranges
- [x] Cover page with metadata, attribution and licensing information
- [x] Hidden text layer from ALTO or hOCR OCR
- [x] Render IIIF layers as PDF "optional content groups" that can be toggled
- [x] Rendering of IIIF Annotations as PDF annotations
- [x] Include IIIF Manifest and referenced OCR files as PDF attachments
- [x] Generate polyglot PDFs that are also ZIP files of all resources

## Quickstart

Besides using the public instance at https://pdiiif.jbaiter.de, you can also run the app yourself.
The easiest way to do this is with Docker:

```
$ docker build . -t pdiiif
# SYS_ADMIN capabilities are required (for Puppeteer's headless Chrome instance to generate cover page PDFs)
$ docker run -p 8080:8080 --cap-add=SYS_ADMIN --name pdiiif pdiiif
```

## Cookbook Matrix

The [IIIF Cookbook](https://iiif.io/api/cookbook/) has a matrix of "recipes" with viewer support, here's an overview
of the recipe support in pdiiif:

<details>
<summary><strong>Basic Recipes</strong> (4 of 6 supported)</summary>

- [x] [Simplest Manifest - Single Image File](https://iiif.io/api/cookbook/recipe/0001-mvm-image/): Partial, only for JPEG images, **Cookbook example doesn't work** due to use of PNG
- [ ] [Simplest Manifest - Audio](https://iiif.io/api/cookbook/recipe/0002-mvm-audio/): NO, PDF has support for audio, but support in pdiiif unlikely, unless there is substantial demand for it
- [ ] [Simplest Manifest - Video](https://iiif.io/api/cookbook/recipe/0003-mvm-video/): NO, PDF has support for video, but support in pdiiif unlikely, unless there is substantial demand for it
- [x] [Support Deep Viewing with Basic Use of a IIIF Image Service](https://iiif.io/api/cookbook/recipe/0005-image-service/): YES, Deep Viewing isn't useful in PDF, but IIIF Image Services are fully supported
- [x] [Internationalization and Multi-language Values (label, summary, metadata, requiredStatement)](https://iiif.io/api/cookbook/recipe/0006-text-language/): YES
- [x] [Simple Manifest - Book](https://iiif.io/api/cookbook/recipe/0009-book-1/): YES
</details>

<details>
<summary><strong>IIIF Properties</strong> (8 of 15 supported)</summary>

- [x] [Embedding HTML in descriptive properties (label, summary, metadata, requiredStatement)](https://iiif.io/api/cookbook/recipe/0007-string-formats/): Partially, only for server-generated cover page
- [x] [Rights statement (rights, requiredStatement)](https://iiif.io/api/cookbook/recipe/0008-rights/): YES
- [x] [Viewing direction and Its Effect on Navigation (viewingDirection)](https://iiif.io/api/cookbook/recipe/0010-book-2-viewing-direction/): Partially, `right-to-left` and `left-to-right` only, viewer support very spotty
- [ ] [Book 'behavior' Variations (continuous, individuals) (behaviorimage)](https://iiif.io/api/cookbook/recipe/0011-book-3-behavior/): NO, general support unlikely since paging preference is global in PDF, but if behavior is global for the manifest, it should be doable
- [ ] [Load a Preview Image Before the Main Content (placeholderCanvas)](https://iiif.io/api/cookbook/recipe/0013-placeholderCanvas/): NO, not applicable
- [ ] [Audio Presentation with Accompanying Image (accompanyingCanvas)](https://iiif.io/api/cookbook/recipe/0014-accompanyingcanvas/): NO, no support for audio
- [ ] [Begin playback at a specific point - Time-based media (start)](https://iiif.io/api/cookbook/recipe/0015-start/): NO, no support for time-based media
- [x] [Metadata on any Resource (metadata)](https://iiif.io/api/cookbook/recipe/0029-metadata-anywhere/): Partial, only Manifest metadata
- [ ] [Providing Alternative Representations (rendering)](https://iiif.io/api/cookbook/recipe/0046-rendering/): NO, utility in PDF questionable
- [ ] [Linking to Structured Metadata (seeAlso)](https://iiif.io/api/cookbook/recipe/0053-seeAlso/): NO, could be placed on the cover page
- [x] [Image Thumbnail for Manifest (thumbnail)](https://iiif.io/api/cookbook/recipe/0117-add-image-thumbnail/): YES
- [x] [Displaying Multiple Values with Language Maps (label, summary, metadata, requiredStatement)](https://iiif.io/api/cookbook/recipe/0118_multivalue/): YES
- [x] [Load Manifest Beginning with a Specific Canvas (start)](https://iiif.io/api/cookbook/recipe/0202-start-canvas/): NO, but support possible
- [ ] [Navigation by Chronology (navDate)](https://iiif.io/api/cookbook/recipe/0230-navdate/): NO
- [x] [Acknowledge Content Contributors (provider)](https://iiif.io/api/cookbook/recipe/0234-provider/): YES
</details>

<details>
<summary><strong>Structuring Resources</strong> (3 of 6 supported)</summary>

- [x] [Table of Contents for Book Chapters (structures)](https://iiif.io/api/cookbook/recipe/0024-book-4-toc/): YES
- [ ] [Table of Contents for A/V Content](https://iiif.io/api/cookbook/recipe/0026-toc-opera/): NO
- [ ] [Multi-volume Work with Individually-bound Volumes](https://iiif.io/api/cookbook/recipe/0030-multi-volume/): NO
- [x] [Multiple Choice of Images in a Single View (Canvas)](https://iiif.io/api/cookbook/recipe/0033-choice/): YES
- [ ] [Foldouts, Flaps, and Maps (behavior)](https://iiif.io/api/cookbook/recipe/0035-foldouts/): NO, support unlikely due to global paging preference in PDF
- [x] [Composition from Multiple Images](https://iiif.io/api/cookbook/recipe/0036-composition-from-multiple-images/): Partial, as long as all images have a JPEG representation
</details>

<details>
<summary><strong>Image Recipes</strong> (6 of 6 supported)</summary>

- [x] [Simplest Manifest - Single Image File](https://iiif.io/api/cookbook/recipe/0001-mvm-image/): Partial, only for JPEG images, **Cookbook example doesn't work** due to use of PNG
- [x] [Image and Canvas with Differing Dimensions](https://iiif.io/api/cookbook/recipe/0004-canvas-size/): YES
- [x] [Support Deep Viewing with Basic Use of a IIIF Image Service](https://iiif.io/api/cookbook/recipe/0005-image-service/): YES, Deep Viewing isn't useful in PDF, but IIIF Image Services are fully supported
- [x] [Simple Manifest - Book](https://iiif.io/api/cookbook/recipe/0009-book-1/): YES
- [x] [Viewing direction and Its Effect on Navigation (viewingDirection)](https://iiif.io/api/cookbook/recipe/0010-book-2-viewing-direction/): Partially, `right-to-left` and `left-to-right` only, viewer support very spotty
- [x] [Load Manifest Beginning with a Specific Canvas (start)](https://iiif.io/api/cookbook/recipe/0202-start-canvas/): YES
</details>

<details>
<summary><strong>Annotation Recipes</strong> (4 of 5 supported)</summary>

- [x] Simple Annotation — Tagging: YES
- [ ] Tagging with an External Resource: NO
- [x] Annotation with a Non-Rectangular Polygon: YES
- [x] Simplest Annotation: YES
- [x] Embedded or referenced Annotations: YES
</details>

## Structure of the repository

- [`./pdiiif-lib`](https://github.com/jbaiter/pdiiif/tree/main/pdiiif-lib): Contains the library source code
- [`./pdiiif-api`](https://github.com/jbaiter/pdiiif/tree/main/pdiiif-api): Small node.js server application that
  is responsible for generating the cover pages and that can be used as a fallback for browsers that don't support
  the Native Filesystem API or service workers.
- [`./pdiiif-web`](https://github.com/jbaiter/pdiiif/tree/main/pdiiif-web): Sample web application (using Svelte)
  to demonstrate using pdiiif in the browser

## Cover Page Endpoints

pdiiif tries to includes a cover page with a thumbnail, descriptive metadata and rights and attribution information.
Since typesetting these pages is beyond the scope of what our bespoke PDF generator can provide (most notably, TTF/OTF
font retrieval for arbitrary languages/scripts and font subsetting), this cover page currently needs to be generated
elsewhere. By default, the library is using a public endpoint at https://pdiiif.jbaiter.de/api/coverpage, which generates
a PDF with the default template. The endpoint can be changed with the `coverPageEndpoint` configuration parameter in the
options passed to the `convertManifest` function.

If you want to customize the template that is being used, you can either host the API provided in this repository yourself
(see [Quickstart](quickstart)) and override the template by mounting your own custom [Handlebars](https://handlebarsjs.com/)
template into the image at `/opt/pdiiif/pdiiif-api/dist/asses/coverpage.hbs`. For a list of available helpers that you can
use, refer to [`handlebars-helpers`](https://github.com/helpers/handlebars-helpers#helpers). Also available are these two
custom helpers:

- `qrcode`, takes a value and an optional `{ width, height, padding, color, background, ecl }` options object and returns
  the value encoded as a SVG QR code image
- `sanitize-html`, takes an arbitrary HTML string and sanitizes it according to the [IIIF HTML rules](https://iiif.io/api/presentation/3.0/#45-html-markup-in-property-values)

If you want to provide your own implementation, make sure that your HTTP endpoint generates a valid PDF and accepts a JSON
POST body with the following shape (i.e. does not throw an error when encountering any of these fields):

```typescript
{
  title: string;
  manifestUrl: string;
  thumbnail?: {
    url: string;
    iiifImageService?: string;
  };
  provider?: {
    label: string;
    homepage?: string;
    logo?: string;
  };
  requiredStatement?: {
    label: string;
    value: string;
  };
  rights?: {
    text: string;
    url?: string;
    logo?: string;
  };
  // [key, value] pairs, with value either single- or multi-valued
  metadata?: Array<[string, string | Array<string>]>;
  pdiiifVersion: string;
}
```
