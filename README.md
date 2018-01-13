### What is this

See this [ClojureVerse thread](https://clojureverse.org/t/creating-a-central-documentation-repository-website-codox-complications/1287/)
about a central documentation hub for Clojure similar to Elixir's https://hexdocs.pm/.

If you want to know more, open an issue or contact me elsewhere.

## Design (wip)

- build an ecosystem-encompassing Grimoire store
- build various tools around that store, ideas:
  - static html API docs + guides
  - single page app to browse documentation
  - API
  - machine readable doc-bundles
  - [Docsets](https://kapeli.com/docsets) for documentation browsing apps
- build tools to raise overall quality of documentation (Github bots, Doc templates, etc)

## Progress

<!-- I'm using parts of Boot for the first prototypes of this,  -->
<!-- it's not set in stone that it uses Boot in the end. -->

#### HOSTING

- [x] setup s3 bucket / static website using [Confetti](https://github.com/confetti-clj/confetti)
- [x] sync files generated by `build-docs` to S3

#### SERVICE + ISOLATION

- [ ] create Docker setup to run `build-docs`
  - starting point `sudo docker run -it adzerk/boot-clj repl`
- [ ] create an API server that runs the Docker setup
- [ ] copy created files to host, upload to S3 (so that AWS keys are not exposed in Docker env)
- [ ] How to get notified when new stuff is released on Clojars?

#### API DOCS

- [ ] derive source-uri
  - probably needs parsing of project.clj or build.boot or perhaps we can derive source locations by overlaying jar contents
  - :face_with_head_bandage: from quick testing it seems that getting this to work with codox could be hard
  - Grimoire data contains source so perhaps source URI becomes less important
  - still would be nice to jump to the entire source file
- [x] give grimoire and related tools a go
- [x] Build Codox style index page based on grimoire info
  - :bulb: Generating pages straight from grimoire info results in pretty tight coupling
  - :bulb: We should probably generate an intermediate format and cache it in Transit
  - :bulb: This cache could be used to generate static and SPA pages alike
- [ ] think about how different platforms can be combined in API docs
- [ ] build static site generator or SPA that runs on top of grimoire data

#### GRIMOIRE

- :white_check_mark: Grimoire currently does not support `cljs` or `cljc`
    - [lib-grimoire #29](https://github.com/clojure-grimoire/lib-grimoire/issues/29) & [lib-grimoire #30](https://github.com/clojure-grimoire/lib-grimoire/issues/30)
    - :tada: Mixing Codox and Grimoire has proven productive
    - [x] codox [has support for clojurescript](https://github.com/weavejester/codox/blob/56066f4b86dd9d879845bcfc6a46ed3ae5151117/codox/src/codox/main.clj) - could this code be shared?
    - [x] *not necessary* maybe put this into a shared library that grimoire and codox can use
    - `lib-grimoire` doesn't actually analyse namespaces, it's done in [lein-grim](https://github.com/clojure-grimoire/lein-grim/blob/master/src/grimoire/doc.clj)
    - [x] look into replacing the code I copied from lein-grim with something similar to codox' implementation
      - [x] depend on codox, make sure it's reader implementation can work with grimoire
      - [x] *not necessary* fork codox, turn into codox-reader export api including [this stuff](https://github.com/weavejester/codox/blob/56066f4b86dd9d879845bcfc6a46ed3ae5151117/codox/src/codox/main.clj#L20-L42) in `codox.reader` namespace
- [ ] throw an error or something if grimoire does not find *anything*
- Building docs from jar vs src — what are the tradeoffs?
  - Not really a grimoire thing
- Are there any fundamental issues with grimoire that could become problematic later?
- How do you build docs for a [prj v] when another version is already on the classpath? (e.g. grimoire itself)
  - This will also apply to other libraries on the classpath for doc generation (codox, tools.namespace, etc)
  - One way could be to "rename" namespaces of the to be documented library and maintain the mapping, although that might show up in the docs in some way?
  - https://github.com/benedekfazekas/mranderson
  - :clock10: **While this will be needed eventually it's not necessary for a first version**

#### GITHUB + NON-API DOCS

- [x] read Github URL from pom.xml or Clojars
- [x] clone repo, copy `doc` directory, provide to codox
- [x] try to check out Git repo at specific tag for `doc/` stuff; warn if no tags
- [ ] figure out what other metadata should be imported
- [ ] Think about how stuff like https://shadow-cljs.github.io/docs/UsersGuide.html could be accomodated
- [ ] Figure out how to deal with changes between tagged releases
  - We probably don't want to pick up API changes since people commonly push development progress straight to `master`
  - Picking up changes to `doc/` would be nice and the above problem probably applies a little less
  - When updating `doc/` triggered by Github commits we need to derive the correct version
    - Take latest "RELEASE" version on Clojars?
    - Take version specified by most recent tag?

#### LONG SHOTS

- [ ] think about discovery of projects with same group-id
- [ ] think about how something like dynadoc (interactive docs) could be integrated
- [ ] think about how stack style REPL examples could be integrated

#### BOT

- [ ] notify people that there are api docs available for a jar they just published
- [ ] suggest to add some plain text documentation/guides + provide templates
