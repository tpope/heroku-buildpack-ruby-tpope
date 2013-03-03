# Heroku buildpack: Ruby: tpope edition

This is my fork of the [official Heroku Ruby
buildpack](https://github.com/heroku/heroku-buildpack-ruby).  Use it when
creating a new app:

    heroku create myapp --buildpack \
      https://github.com/tpope/heroku-buildpack-ruby-tpope

Or add it to an existing app:

    heroku config:add \
      BUILDPACK_URL=https://github.com/tpope/heroku-buildpack-ruby-tpope

Or just cherry-pick the parts you like into your own fork.

Contained within are a few tiny but significant differences from the official
version, distilled from project-specific buildpacks I've created in the past.

## Strict asset precompilation

If asset precompilation fails, in lieu of enabling runtime asset
compilation, the deploy will be aborted.

## Commit recording

This takes the upcoming and previously deployed commit SHAs and makes them
available as `$REVISION` and `$ORIGINAL_REVISION` for the duration of the
compile.  They are also written to `HEAD` and `ORIG_HEAD` in the root of the
application for easy access after the deploy is complete.
