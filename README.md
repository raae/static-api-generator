<img src="https://raw.githubusercontent.com/eduardoboucas/static-api-generator/master/.github/logo.png" alt="Pluma logo" height="120"/>

[![npm (scoped)](https://img.shields.io/npm/v/pluma.svg?maxAge=10800&style=flat-square)](https://www.npmjs.com/package/pluma)
[![JavaScript Style Guide](https://img.shields.io/badge/code%20style-standard-brightgreen.svg?style=flat-square)](http://standardjs.com/)
<!--[![coverage](https://img.shields.io/badge/coverage-88%25-yellow.svg?style=flat?style=flat-square)](https://github.com/eduardoboucas/static-api-generator)
[![Build Status](https://travis-ci.org/eduardoboucas/static-api-generator.svg?branch=master)](https://travis-ci.org/eduardoboucas/static-api-generator)-->

Static API generator is a Node.js application that creates a basic JSON API from a tree of directories and files. Think of a static site generator, like Jekyll or Hugo, but for APIs.

It takes your existing data files, which you may already be using to feed a static site generator or similar, and creates an API layer with whatever structure you want, leaving the original files untouched. Static API generator helps you deliver your data to client-side applications or third-party syndication services.

Couple it with services like [GitHub Pages](https://pages.github.com/) or [Netlify](https://www.netlify.com/) and you can serve your API right from the repository too. 🦄

---

- [Installation](#installation)
- [Usage](#usage)
- [API](#api)
- [Q&A](#qa)

---

## 1. Installation

- Install via npm

    ```shell
    npm install static-api-generator --save
    ```

- Require the module and create an API

    ```js
    const API = require('static-api-generator')
    const api = new API(constructorOptions)
    ```

## 2. Usage

Imagine the following repository holding data about movies, organised by language, genre and year. Information about each movie will be in its own YAML file named after the movie.

```
input/
├── english
│   ├── action
│   │   ├── 2016
│   │   │   ├── deadpool.yaml
│   │   │   └── the-great-wall.yaml
│   │   └── 2017
│   │       ├── logan.yaml
│   │       └── the-fate-of-the-furious.yaml
│   └── horror
│       └── 2017
│           ├── alien-covenant.yaml
│           └── get-out.yaml
└── portuguese
    └── action
        └── 2016
            └── tropa-de-elite.yaml
```

### 2.1. Initialisation

Create an API by specifying its blueprint, so that the static API generator can understand how the data is structured, and the directory where the generated files should be saved.

```js
const moviesApi = new API({
  blueprint: 'source/:language/:genre/:year/:movie',
  targetDirectory: 'output'
})
```

### 2.2 Generating endpoints

The following will generate and endpoint for each movie (e.g. `/english/action/2016/deadpool.json`).

```js
moviesApi.generate({
  endpoints: ['movie']
})
```

Endpoints can be created for any data level. The following creates additional endpoints at the genre level (e.g. `/english/action.json`).

```js
moviesApi.generate({
  endpoints: ['genre', 'movie']
})
```

It's also possible to manipulate the hierarchy of the data by choosing a different root level. For example, one could generate endpoints for each genre without a separation enforced by language, its original parent level. This means being able to create endpoints like `/action.json` (as opposed to `/english/action.json`), where all action movies are listed regardless of their language.

```js
moviesApi.generate({
  endpoints: ['genre', 'movie'],
  root: 'genre'
})
```

## 3. API

### 3.1. Constructor

```js
const API = require('static-api-generator')
const api = new API({
  addIdToFiles: Boolean,
  blueprint: String,
  pluralise: Boolean,
  targetDirectory: String
})
```

The constructor method takes an object with the following properties.

---

- #### `addIdToFiles`

    Whether to add an `id` field to uniquely identify each data file. IDs are generated by computing an MD5 hash of the full path of the file.

    *Default:*

    `false`

    *Example result:*
    
    `"review_id": "96a9b996439528ecb9050774c3e79ff2"`


- #### `blueprint`

    **Required**. A path describing the hierarchy and nomenclature of the data. It should start with a static path to the directory where all the files are located, followed by the name of each data level (starting with a colon).

    For the [pluralise](#pluralise) option to work well, the names of the data levels should be singular (e.g. `languages` vs. `language`)

    *Example:*
    
    `'input/:language/:genre/:year/:movie'`

- #### `pluralise`

    The name of each data level (e.g. `"genre"`) is used in the singular form when identifying a single entity (e.g. `{"genre_id": "12345"}`) and in the plural form when identifying a list of entities (e.g. `{"genres": [...]}`). That behaviour can be disabled by setting this property to `false`.

    *Default:*

    `true`    
    
---

- #### `targetDirectory`

    **Required**. The path to the directory where endpoint files should be created.

    *Example:*
    
    `'output'`
    
---

### 3.2. Method: `generate`

```js
api.generate({
  endpoints: Array,
  entriesPerPage: Number,
  index: String,
  levels: Array,
  root: String,
  sort: Object
})
```

The `generate` method creates one or more endpoints. It takes an object with the following properties.

---

- #### `endpoints`

    The names of the data levels to create individual endpoints for, which means generating a JSON file for each file or directory that corresponds to a given level.

    *Default:*

    `[]`

    *Example:*
    
    `['genre', 'movie']`
    
---

- #### `entriesPerPage`

    The maximum number of entries (data files) to include in an endpoint file. If the number of entries exceeds this number, additional endpoints are created (e.g. `/action.json`, `/action-2.json`, etc.).

    *Default:*

    `10`    

    *Example:*
    
    `3`

---

- #### `index`

    The name of the main/index endpoint file.

    *Default:*
    
    The pluralised name of the root level (e.g. `languages`).

    *Example:*
    
    `languages` (generates `languages.json`)
    
---

- #### `levels`

    An array containing the names of the levels to include in the endpoints. By default, an endpoint for a level will include all its child levels (e.g. an endpoint for a *genre* will have a list of *years*, which will have a list of *movies*). If a level is omitted (e.g. *year*), then all its entries are grouped together (i.e. an endpoint for a *genre* will have a list of all its *movies*, without any separation by *year*).

    *Default:*
    
    All levels

    *Example:*
    
    `['language', 'genre', 'movie']`
    
---

- #### `root`

    The name of the root level. When this doesn't correspond to the first level in the blueprint, the data tree is manipulated so that all entries become grouped by the new root level.

    *Default:*

    The name of the first level of the blueprint (e.g. `language`)

    *Example:*

    `genre`
    
---

## 4. Q&A

- **Why did you build this?**

    GitHub has been the centrepiece of my daily workflow as a developer for many years. I love the idea of using a repository not only for version control, but also as the single source of truth for a data set. As a result, I created [several](https://staticman.net) [projects](https://speedtracker.org) that explore the usage of GitHub repositories as data stores, and I've used that approach in several professional and personal projects, including [my own site/blog](https://eduardoboucas.com).

- **Couldn't Jekyll, Hugo or XYZ do the same thing?**

    Possibly. Most static site generators are capable of generating JSON, but usually using awkward/brittle methods. Most of those applications were built to generate HTML pages and that's where they excel on. I tried to create a minimalistic and easy-as-it-gets way of generating something very specific: a bunch of JSON files that, when bundled together, form a very basic API layer.

- **Where can I host this?**

    [GitHub Pages](https://pages.github.com/) is a very appealing option, since you could serve the API right from the repository. It has CORS enabled, so you could easily consume it from a client-side application using React, Angular, Vue or whatever you prefer. You could even use a CI service like [Travis](https://travis-ci.org/) to listen for commits on a given branch (say `master`) and automatically run Static API Generator and push the generated output to a `gh-pages` branch, making the process of generating the API when data changes fully automated.

    [Netlify](https://www.netlify.com/) is also very cool and definitely worth trying.

- **Would it be possible to add feature X, Y or Z?**

    Probably. File an [issue](https://github.com/eduardoboucas/static-api-generator/issues) or, even better, a [pull request](https://github.com/eduardoboucas/static-api-generator/pulls) and I'm happy to help. Bare in mind that this is a side project (one of too many) which I'm able to dedicate a very limited amount of time to, so please be patient and try to understand if I tell you I don't have the capacity to build what you're looking for.

- **Who designed the logo?**

    The logo was created by [Arthur Shlain](https://thenounproject.com/ArtZ91/) from The Noun Project and it's licensed under a [Creative Commons Attribution](https://creativecommons.org/licenses/by/3.0/us/) license.
