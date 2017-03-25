# api documentation for  [nopt (v4.0.1)](https://github.com/npm/nopt#readme)  [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-nopt.svg)](https://travis-ci.org/npmdoc/node-npmdoc-nopt)
#### Option parsing for Node, supporting types, shorthands, etc. Used by npm.

[![NPM](https://nodei.co/npm/nopt.png?downloads=true)](https://www.npmjs.com/package/nopt)

[![apidoc](https://npmdoc.github.io/node-npmdoc-nopt/build/screen-capture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-nopt_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-nopt/build..beta..travis-ci.org/apidoc.html)

![package-listing](https://npmdoc.github.io/node-npmdoc-nopt/build/screen-capture.npmPackageListing.svg)



# package.json

```json

{
    "author": {
        "name": "Isaac Z. Schlueter",
        "email": "i@izs.me",
        "url": "http://blog.izs.me/"
    },
    "bin": {
        "nopt": "./bin/nopt.js"
    },
    "bugs": {
        "url": "https://github.com/npm/nopt/issues"
    },
    "dependencies": {
        "abbrev": "1",
        "osenv": "^0.1.4"
    },
    "description": "Option parsing for Node, supporting types, shorthands, etc. Used by npm.",
    "devDependencies": {
        "tap": "^8.0.1"
    },
    "directories": {},
    "dist": {
        "shasum": "d0d4685afd5415193c8c7505602d0d17cd64474d",
        "tarball": "https://registry.npmjs.org/nopt/-/nopt-4.0.1.tgz"
    },
    "gitHead": "24921187dc52825d628042c9708bbd8e8734698c",
    "homepage": "https://github.com/npm/nopt#readme",
    "license": "ISC",
    "main": "lib/nopt.js",
    "maintainers": [
        {
            "name": "isaacs",
            "email": "i@izs.me"
        },
        {
            "name": "othiym23",
            "email": "ogd@aoaioxxysz.net"
        },
        {
            "name": "zkat",
            "email": "kat@sykosomatic.org"
        }
    ],
    "name": "nopt",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git+https://github.com/npm/nopt.git"
    },
    "scripts": {
        "test": "tap test/*.js"
    },
    "version": "4.0.1"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module nopt](#apidoc.module.nopt)
1.  [function <span class="apidocSignatureSpan">nopt.</span>clean (data, types, typeDefs)](#apidoc.element.nopt.clean)
1.  object <span class="apidocSignatureSpan">nopt.</span>typeDefs



# <a name="apidoc.module.nopt"></a>[module nopt](#apidoc.module.nopt)

#### <a name="apidoc.element.nopt.clean"></a>[function <span class="apidocSignatureSpan">nopt.</span>clean (data, types, typeDefs)](#apidoc.element.nopt.clean)
- description and source-code
```javascript
function clean(data, types, typeDefs) {
  typeDefs = typeDefs || exports.typeDefs
  var remove = {}
    , typeDefault = [false, true, null, String, Array]

  Object.keys(data).forEach(function (k) {
    if (k === "argv") return
    var val = data[k]
      , isArray = Array.isArray(val)
      , type = types[k]
    if (!isArray) val = [val]
    if (!type) type = typeDefault
    if (type === Array) type = typeDefault.concat(Array)
    if (!Array.isArray(type)) type = [type]

    debug("val=%j", val)
    debug("types=", type)
    val = val.map(function (val) {
      // if it's an unknown value, then parse false/true/null/numbers/dates
      if (typeof val === "string") {
        debug("string %j", val)
        val = val.trim()
        if ((val === "null" && ~type.indexOf(null))
            || (val === "true" &&
               (~type.indexOf(true) || ~type.indexOf(Boolean)))
            || (val === "false" &&
               (~type.indexOf(false) || ~type.indexOf(Boolean)))) {
          val = JSON.parse(val)
          debug("jsonable %j", val)
        } else if (~type.indexOf(Number) && !isNaN(val)) {
          debug("convert to number", val)
          val = +val
        } else if (~type.indexOf(Date) && !isNaN(Date.parse(val))) {
          debug("convert to date", val)
          val = new Date(val)
        }
      }

      if (!types.hasOwnProperty(k)) {
        return val
      }

      // allow '--no-blah' to set 'blah' to null if null is allowed
      if (val === false && ~type.indexOf(null) &&
          !(~type.indexOf(false) || ~type.indexOf(Boolean))) {
        val = null
      }

      var d = {}
      d[k] = val
      debug("prevalidated val", d, val, types[k])
      if (!validate(d, k, val, types[k], typeDefs)) {
        if (exports.invalidHandler) {
          exports.invalidHandler(k, val, types[k], data)
        } else if (exports.invalidHandler !== false) {
          debug("invalid: "+k+"="+val, types[k])
        }
        return remove
      }
      debug("validated val", d, val, types[k])
      return d[k]
    }).filter(function (val) { return val !== remove })

    if (!val.length) delete data[k]
    else if (isArray) {
      debug(isArray, data[k], val)
      data[k] = val
    } else data[k] = val[0]

    debug("k=%s val=%j", k, val, data[k])
  })
}
```
- example usage
```shell
...
hash is an object with a 'type' member and a 'validate' method.  The
'type' member is an object that matches what goes in the type list.  The
'validate' method is a function that gets called with 'validate(data,
key, val)'.  Validate methods should assign 'data[key]' to the valid
value of 'val' if it can be handled properly, or return boolean
'false' if it cannot.

You can also call 'nopt.clean(data, types, typeDefs)' to clean up a
config object and remove its invalid properties.

## Error Handling

By default, nopt outputs a warning to standard error when invalid values for
known options are found.  You can change this behavior by assigning a method
to 'nopt.invalidHandler'.  This method will be called with
...
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
