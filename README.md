# node-osrm

Routing engine for OpenStreetMap data implementing high-performance algorithms for shortest paths in road networks.

Provides bindings to the [Open Source Routing Machine - OSRM](https://github.com/DennisOSRM/Project-OSRM).

[![Build Status](https://secure.travis-ci.org/DennisOSRM/node-osrm.png)](https://travis-ci.org/DennisOSRM/node-osrm)

# Depends

 - Node.js v0.10.x or v0.8.x

# Installing

By default, binaries are provided for:

 - 64 bit OS X and 64 bit Linux
 - Node v0.8.x and v0.10.x

On those platforms no external dependencies are needed.

Just do:

    npm install osrm

However other platforms will fall back to a source compile: see [Source Build](#source-build) for details.

# Usage

The `node-osrm` module consumes data processed by OSRM core.

This repository contains a Makefile that does this automatically:

- Downloads an OSM extract
- Runs `osrm-extract` and `osrm-prepare`
- Has a OSRM config (ini) file that references the prepared data

Just run:

    make berlin-latest.osrm.hsgr

Once that is done then you can calculate routes in Javascript like:

```js
// Note: to require osrm locally do:
// require('./lib/osrm.js')
var osrm = require('osrm')
var opts = new osrm.Options("./test/data/berlin.ini");
var engine = new osrm.Engine(opts);
var query = new osrm.Query({coordinates: [[52.519930,13.438640], [52.513191,13.415852]]});
var sync_result = engine.run(query);
JSON.parse(engine.run(query));
{ status: 0,
  status_message: 'Found route between points',
  route_geometry: '{~pdcBmjfsXsBrD{KhS}DvHyApCcf@l}@kg@z|@_MbX|GjHdXh^fm@dr@~\\l_@pFhF|GjCfeAbTdh@fFqRp}DoEn\\cHzR{FjLgCnFuBlG{AlHaAjJa@hLXtGnCnKtCnFxCfCvEl@lHBzA}@vIoFzCs@|CcAnEQ~NhHnf@zUpm@rc@d]zVrTnTr^~]xbAnaAhSnPgJd^kExPgOzk@maAx_Ek@~BuKvd@cJz`@oAzFiAtHvKzAlBXzNvB|b@hGl@Dha@zFbGf@fBAjQ_AxEbA`HxBtPpFpa@rO_Cv_B_ZlD}LlBGB',
  route_instructions: 
   [ ... ],
  route_summary: 
   { total_distance: 2814,
     total_time: 211,
     start_point: 'Friedenstraße',
     end_point: 'Am Köllnischen Park' },
  alternative_geometries: [],
  alternative_instructions: [],
  alternative_summaries: [],
  route_name: 
   [ 'Lichtenberger Straße',
     'Holzmarktstraße' ],
  alternative_names: [ [ '', '' ] ],
  via_points: 
   [ [ 52.519934, 13.438647 ],
     [ 52.513162, 13.415509 ] ],
  via_indices: [ 0, 69 ],
  alternative_indices: [],
  hint_data: 
   { checksum: 222545162,
     locations: 
      [ '9XkCAJgBAAAtAAAA____f7idcBkPGuw__mMhA7cOzQA',
        'TgcEAFwFAAAAAAAAVAAAANIeb5DqBHs_ikkhA1W0zAA' ] } }
```

# Source Build

To build from source you will need:

 - OSRM `develop` branch, cloned from github.
 - OSRM build with `-DWITH_TOOLS=1` so that `libOSRM` is created
 - Lua, luabind, and stxxl headers

To build with OS X Mavericks you need to ensure the bindings link to `libc++`. An easy way to do this is to set:

    export CXXFLAGS=-mmacosx-version-min=10.9

before building `node-osrm`.

### Building

To build the bindings you need to first build and **install** the `develop` branch of `Project-OSRM`:

    # grab develop branch
    git clone -b develop https://github.com/DennisOSRM/Project-OSRM.git
    cd Project-OSRM
    mkdir build;
    cd build;
    cmake ../ -DWITH_TOOLS=1
    make
    sudo make install

NOTE: If you hit problems building Project-OSRM see [the wiki](https://github.com/DennisOSRM/Project-OSRM/wiki/Building%20OSRM) for details.

Then build `node-osrm` against `Project-OSRM` installed in `/usr/local`:

    git clone https://github.com/DennisOSRM/node-osrm.git
    cd node-osrm
    npm install


# Developing

Developers of `node-osrm` should set up a [Source Build](#source-build) and after changes to the code run:

    make

Under the hood this uses [node-gyp](https://github.com/TooTallNate/node-gyp) to compile the source code.

# Testing

Run the tests like:

    make test

# Releasing

Releasing a new version of `node-osrm` is mostly automated using travis.ci.

### Caveats

- If you create and push a new git tag Travis.ci will automatically publish both binaries and the package to the npm registry.

- Before tagging you can test publishing of just binaries by including the keyword `[publish binary]` in your commit message. But until [this feature](https://github.com/springmeyer/node-pre-gyp/issues/28) is implementedand be very careful that the `version` in package.json has been incremented since the last tag. Otherwise existing binaries will get overwritten, which you likely don't want.

### Steps to release

**1)** Confirm the desired OSRM branch and commit.

This is configurable via the `OSRM_BRANCH` and `OSRM_COMMIT` variables in travis.ci.

See [Issue 36](https://github.com/DennisOSRM/node-osrm/issues/36) for further ideas on streamlining this.

**2)** Bump node-osrm version

Update the `CHANGELOG.md` and the `package.json` version:

 - Add `-alpha` if you are testing experimental features or binary packaging.
 - Remove `-alpha` if you are preparting for a stable release and increment if needed.

**3)** Check Travis.ci

Ensure Travis.ci [builds are passing](https://travis-ci.org/DennisOSRM/node-osrm) after your last commit. This is important because upstream OSRM is being pulled in and may have changed.

**4)** Tag

Tag a new release:

    git add CHANGELOG.md package.json
    git commit -m "Tagging v0.3.0"
    git tag v0.3.0

**5)** Push the tag to github:

    git push origin master v0.3.0

This will trigger travis.ci to build Ubuntu binaries and publish the entire package to the npm registry upon success. The publishing will use the s3 and npm auth credentials of @springmeyer currently - this needs to be made more configurable in the future.

**6)** Merge into the `osx` branch

    git checkout osx
    git merge v0.3.0 -m "[publish binary]"
    git push origin osx

This will build and publish OS X binaries on travis.ci. Be prepared to watch the travis run and re-start builds that fail due to timeouts (the OS X machines are underpowered).

**7)** You are done

If the travis builds succeeded then you can rest assured the binaries are working since they not only publish but also test installing from what they published.

Now go ahead and use the new tag in your applications `package.json` as a dependency and you will get binaries rather than needing a source compile.

