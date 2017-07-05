<div align="center">
<h1>babel-plugin-preval</h1>

Pre-evaluate code at build-time
</div>

<hr />

[![Build Status][build-badge]][build]
[![Code Coverage][coverage-badge]][coverage]
[![version][version-badge]][package]
[![downloads][downloads-badge]][npm-stat]
[![MIT License][license-badge]][LICENSE]

[![All Contributors](https://img.shields.io/badge/all_contributors-1-orange.svg?style=flat-square)](#contributors)
[![PRs Welcome][prs-badge]][prs]
[![Donate][donate-badge]][donate]
[![Code of Conduct][coc-badge]][coc]
[![Roadmap][roadmap-badge]][roadmap]
[![Examples][examples-badge]][examples]

[![Watch on GitHub][github-watch-badge]][github-watch]
[![Star on GitHub][github-star-badge]][github-star]
[![Tweet][twitter-badge]][twitter]

## The problem

You need to do some dynamic stuff, but don't want to do it at runtime. Or maybe
you want to do stuff like read the filesystem to get a list of files and you
can't do that in the browser.

## This solution

This allows you to specify some code that runs in Node and whatever you
`module.exports` in there will be swapped. For example:

```js
const x = preval`module.exports = 1`;

//      ↓ ↓ ↓ ↓ ↓ ↓

const x = 1;
```

Or, more interestingly:

```javascript
const x = preval`
  const fs = require('fs')
  const val = fs.readFileSync(__dirname + '/fixture1.md', 'utf8')
  module.exports = {
    val,
    getSplit: function(splitDelimiter) {
      return x.val.split(splitDelimiter)
    }
  }
`

//      ↓ ↓ ↓ ↓ ↓ ↓

const x = {
  val: '# fixture\n\nThis is some file thing...\n',
  getSplit: function getSplit(splitDelimiter) {
    return x.val.split(splitDelimiter)
  },
}
```

There's also `preval.require('./something')` and
`import x from /* preval */ './something'` (which can both take some arguments).

See more below.

## Installation

This module is distributed via [npm][npm] which is bundled with [node][node] and
should be installed as one of your project's `devDependencies`:

```
npm install --save-dev babel-plugin-preval
```

## Usage

Important notes:

1. All code run by `preval` is _not_ run in a sandboxed environment
2. All code _must_ run synchronously.
3. All code will be transpiled via `babel-core` directly or `babel-register`
   and should follow all of the normal rules for `.babelrc` resolution (the
   closest `.babelrc` to the file being run is the one that's used). This means
   you can rely on any babel plugins/transforms that you're used to using
   elsewhere in your codebase.

### Template Tag

**Before**:

```javascript
const greeting = preval`
  const fs = require('fs')
  module.exports = fs.readFileSync(require.resolve('./greeting.txt'), 'utf8')
`
```

**After** (assuming `greeting.txt` contains the text: `"Hello world!"`):

```javascript
const greeting = "Hello world!"
```

`preval` can also handle _some_ simple dynamic values as well:

**Before**:

```javascript
const name = 'Bob Hope'
const person = preval`
  const [first, last] = require('./name-splitter')(${name})
  module.exports = {first, last}
`
```

**After** (assuming `./name-splitter` is a function that splits a name into first/last):

```javascript
const name = 'Bob Hope';
const person = { "first": "Bob", "last": "Hope" };
```

### import comment

**Before**:

```javascript
import fileList from /* preval */ './get-list-of-files'
```

**After** (depending on what `./get-list-of-files does`, it might be something like):

```javascript
const fileList = ['file1.md', 'file2.md', 'file3.md', 'file4.md']
```

You can also provide arguments which themselves are prevaled!

**Before**:

```javascript
import fileList from /* preval(3) */ './get-list-of-files'
```

**After** (assuming `./get-list-of-files` accepts an argument limiting how many files are retrieved:

```javascript
const fileList = ['file1.md', 'file2.md', 'file3.md']
```

### preval.require

**Before**:

```javascript
const fileLastModifiedDate = preval.require('./get-last-modified-date')
```

**After**:

```javascript
const fileLastModifiedDate = '2017-07-05'
```

And you can provide _some_ simple dynamic arguments as well:

**Before**:

```javascript
const fileLastModifiedDate = preval.require('./get-last-modified-date', '../../some-other-file.js')
```

**After**:

```javascript
const fileLastModifiedDate = '2017-07-04'
```

## Configure with Babel

### Via `.babelrc` (Recommended)

**.babelrc**

```json
{
  "plugins": ["preval"]
}
```

### Via CLI

```sh
babel --plugins preval script.js
```

### Via Node API

```javascript
require('babel-core').transform('code', {
  plugins: ['preval'],
})
```

## FAQ

### How is this different from [prepack][prepack]?

`prepack` is intended to be run on your final bundle after you've run your
webpack/etc magic on it. It does a TON of stuff, but the idea is that your
code should work with or without prepack.

`babel-plugin-preval` is intended to let you write code that would _not_
work otherwise. Doing things like reading something from the file system
are not possible in the browser (or with prepack), but `preval` enables you
to do this.

## Inspiration

I needed something like this for the
[glamorous website](https://github.com/kentcdodds/glamorous-website).
I live-streamed developing the whole thing. If you're interested you can find
the recording on [my twitch](https://www.twitch.tv/kentcdodds).

## Other Solutions

I'm not aware of any, if you are please [make a pull request][prs] and add it
here!

## Contributors

Thanks goes to these people ([emoji key][emojis]):

<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->
| [<img src="https://avatars.githubusercontent.com/u/1500684?v=3" width="100px;"/><br /><sub>Kent C. Dodds</sub>](https://kentcdodds.com)<br />[💻](https://github.com/kentcdodds/babel-plugin-preval/commits?author=kentcdodds "Code") [📖](https://github.com/kentcdodds/babel-plugin-preval/commits?author=kentcdodds "Documentation") [🚇](#infra-kentcdodds "Infrastructure (Hosting, Build-Tools, etc)") [⚠️](https://github.com/kentcdodds/babel-plugin-preval/commits?author=kentcdodds "Tests") |
| :---: |
<!-- ALL-CONTRIBUTORS-LIST:END -->

This project follows the [all-contributors][all-contributors] specification.
Contributions of any kind welcome!

## LICENSE

MIT

[npm]: https://www.npmjs.com/
[node]: https://nodejs.org
[build-badge]: https://img.shields.io/travis/kentcdodds/babel-plugin-preval.svg?style=flat-square
[build]: https://travis-ci.org/kentcdodds/babel-plugin-preval
[coverage-badge]: https://img.shields.io/codecov/c/github/kentcdodds/babel-plugin-preval.svg?style=flat-square
[coverage]: https://codecov.io/github/kentcdodds/babel-plugin-preval
[version-badge]: https://img.shields.io/npm/v/babel-plugin-preval.svg?style=flat-square
[package]: https://www.npmjs.com/package/babel-plugin-preval
[downloads-badge]: https://img.shields.io/npm/dm/babel-plugin-preval.svg?style=flat-square
[npm-stat]: http://npm-stat.com/charts.html?package=babel-plugin-preval&from=2016-04-01
[license-badge]: https://img.shields.io/npm/l/babel-plugin-preval.svg?style=flat-square
[license]: https://github.com/kentcdodds/babel-plugin-preval/blob/master/LICENSE
[prs-badge]: https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square
[prs]: http://makeapullrequest.com
[donate-badge]: https://img.shields.io/badge/$-support-green.svg?style=flat-square
[donate]: http://kcd.im/donate
[coc-badge]: https://img.shields.io/badge/code%20of-conduct-ff69b4.svg?style=flat-square
[coc]: https://github.com/kentcdodds/babel-plugin-preval/blob/master/other/CODE_OF_CONDUCT.md
[roadmap-badge]: https://img.shields.io/badge/%F0%9F%93%94-roadmap-CD9523.svg?style=flat-square
[roadmap]: https://github.com/kentcdodds/babel-plugin-preval/blob/master/other/ROADMAP.md
[examples-badge]: https://img.shields.io/badge/%F0%9F%92%A1-examples-8C8E93.svg?style=flat-square
[examples]: https://github.com/kentcdodds/babel-plugin-preval/blob/master/other/EXAMPLES.md
[github-watch-badge]: https://img.shields.io/github/watchers/kentcdodds/babel-plugin-preval.svg?style=social
[github-watch]: https://github.com/kentcdodds/babel-plugin-preval/watchers
[github-star-badge]: https://img.shields.io/github/stars/kentcdodds/babel-plugin-preval.svg?style=social
[github-star]: https://github.com/kentcdodds/babel-plugin-preval/stargazers
[twitter]: https://twitter.com/intent/tweet?text=Check%20out%20babel-plugin-preval!%20https://github.com/kentcdodds/babel-plugin-preval%20%F0%9F%91%8D
[twitter-badge]: https://img.shields.io/twitter/url/https/github.com/kentcdodds/babel-plugin-preval.svg?style=social
[emojis]: https://github.com/kentcdodds/all-contributors#emoji-key
[all-contributors]: https://github.com/kentcdodds/all-contributors
[prepack]: https://github.com/facebook/prepack

