# Custom Gigalixir buildpack

Custom buildpack for [Gigalixir](https://gigalixir.com/) apps inspired by [HashNuke's buildpack for Elixir](https://github.com/HashNuke/heroku-buildpack-elixir).

Features:

- Easy to customize, single file `bin/compile` script,
- Sets up Erlang, Elixir and more,
- Runs your custom build script so that you're in control of building your app.

Gigalixir uses [Heroku's buildpacks](https://devcenter.heroku.com/articles/buildpack-api).
Buildpack is a set of scripts that power app builds. It consists of the following:

- `bin/detect`: Determines whether to apply this buildpack to an app (always true for this buildpack;
it assumes it's the only one that should be used).
- `bin/compile`: Builds the app.
- `bin/release`: Defines the application entrypoint.

`bin/compile` is run with paths to "build dir" and "cache dir". The "cache dir" is a directory for
build artifacts that are kept between builds to speed up the process. The "build dir" should contain
the final built application.

Note that currently the buildpack clears the cache on each build. To force cache clean, use
`git -c http.extraheader="GIGALIXIR-CLEAN: true" push gigalixir master`

The provided `bin/compile` script:
- Sets up the variables: `${build_dir}`, `${cache_dir}`, `${source_dir}`.
- Installs Erlang and Elixir in the specified versions. The binaries are available in the `PATH`.
- Runs custom build script from the app sources.

To use the buildpack:
1. Add a `.buildpacks` file to the root dir of your app.
1. Select the buildpack (this one or your own fork):
    ```
    https://github.com/stefanchrobot/gigalixir-buildpack.git
    ```
1. Add `buildpack/config` with the required variables, for example:
    ```
    erlang_version=22.2.3
    elixir_version=1.10.3
    entrypoint='bin/app start'
    ```
or if you need to [migrate the database before running the app](https://hexdocs.pm/phoenix/releases.html#ecto-migrations-and-custom-commands):
    ```
    erlang_version=22.2.3
    elixir_version=1.10.3
    entrypoint='bin/app eval "MyApp.Release.migrate" && bin/app start'
    ```
1. Add `buildpack/build`, for example:
    ```
    export MIX_ENV=prod

    mix deps.get --only prod
    mix compile
    npm install --prefix ./assets
    npm run deploy --prefix ./assets
    mix phx.digest
    mix release --path ${build_dir}
    ```

That's it! You're ready to deploy your app.
