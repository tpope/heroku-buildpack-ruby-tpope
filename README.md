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

## Custom compilation tasks

If the `COMPILE_TASKS` config variable is set, it will be passed verbatim to a
`rake` invocation.  The `user-env-compile` Heroku labs feature is required in
order for this to work.

    heroku labs:enable user-env-compile

You can use this for all sorts of things.  My favorite is `db:migrate`.

### Database migration during compilation

Let's take a look at the standard best practice for deploying Rails apps to
Heroku:

1.  `heroku maintenance:on`.
2.  `git push heroku master`.  This restarts the application when complete.  If
    you have any schema additions, your app is now broken (hence the need for
    maintenance mode).
3.  `heroku run rake db:migrate`.
4.  `heroku restart`.  This is necessary so the app picks up on the schema
    changes.
5.  `heroku maintenance:off`.

That's five different commands, none of them instantaneous, and two restarts.
The most common response to this mess is to wrap deployment up in a Rake task,
but now you have two problems: a suboptimal deployment procedure, and
application code concerned with deployment.

Now let's take a look at a typical deploy when `COMPILE_TASKS` includes
`db:migrate`:

1.  `git push heroku master`.
    * First the standard stuff happens.  Bundling, asset precompilation, that
      sort of thing.
    * `rake db:migrate` fires.  The app continues working unless the
      migrations drop something from the schema.
    * The app restarts.  Everything is wonderful.

We've reduced it to a single step, limiting our need for maintenance mode to
destructive migrations.  Even in that case, it's not always strictly
necessary, since the window for breakage is frequently only a few seconds.  Or
with a bit of planning, you can [avoid this situation entirely][no downtime].

[Twelve-factor][] snobs (of which I am one) would generally argue that
[admin processes][] belong in the run stage, not the build stage.  I agree in
theory, but it in practice, boy does this make things a whole lot simpler.

[no downtime]: http://pedro.herokuapp.com/past/2011/7/13/rails_migrations_with_no_downtime/
[Twelve-factor]: http://www.12factor.net/
[Admin processes]: http://www.12factor.net/admin-processes

## Commit recording

This takes the upcoming and previously deployed commit SHAs and makes them
available as `$REVISION` and `$ORIGINAL_REVISION` for the duration of the
compile.  They are also written to `HEAD` and `ORIG_HEAD` in the root of the
application for easy access after the deploy is complete.

These can be used from `COMPILE_TASKS` to make a poor man's post-deploy hook.
