# JUGA - Just Use Git Already
JUGA is a git-powered origin server to be used for feeding CDNs.  It is based on docker.  

This server exposes the data contained in git branches to the internet.  It is designed to be used by a project using [git flow](https://nvie.com/posts/a-successful-git-branching-model/) to version control the assets that it is serving to its clients, but practically it can be used by any git workflow that involves numbered sequential tags to track deployments.

Some caveats about this server:
* this server is not designed to be exposed to the internet; it is designed to serve a CDN.
* this server pulls down a bunch of git files when it is run; it does not start up immediately
* If you can use *AWS S3*, *Azure Blob Storage*, or some other similar source for publishing static assets easily, you should not use this origin server -- this server is designed for a particular set of use cases.
* This server is restricted to run against github; it is recommended that you use a dedicated user for this purpose, similar to the one that you would use for running a CI/CD server.


## Configuration
Most configuration is by passing environment variables to the docker  container when it is run.  There are some build time options in the Dockerfile that you might want to look at changing depending on how your docker container is mounting filesystems.

`JFUG_GITHUB_URL`

This is the repository for the git repo to be used.  JUGA uses the token method of accessing git repos, which is specific to github. 

`JFUG_GITHUB_TOKEN`

This is the token that your dedicated CI/CD user uses for accessing Github.  You can generate a token for this [here](https://github.com/settings/tokens), but remember that you should use a deployment user for this and _have a specific token for this server_.  

`JFUG_RELEASES_PREFIX`

This prefix is designed to specify what tags should be pulled down into the server.  The default value of this tag is `v`., because the server is normally designed to take semantically versioned releases of the form `v1.0.0` .  It also works with sequential release of the form `v1`.  Another common value for this is `release`.

`JFUG_RELEASES_COUNT`

This is the number of releases to be available on the server, counting backwards from most recent.  The values are ordered according to semantic versioning, so `v1` is before `v2`, but `v2` is after `v1.1` irrespective of the dates that those versions were committed.

The default value is `*`, which is to pull down all releases.  If the value given is `0`, no releases will be pulled down.  This is independent of the releases given in`JFUG_RELEASES_LIST` — specifying a release there does not affect the releases pulled down by this command.

`JFUG_RELEASES_LIST`

This is a comma-separated list of releases to pull down.  This allows for any named tag to be pulled down, it also has the special value `HEAD` that will pull down the head of the repository.  Other common helpful values here are `master` and `develop`.

## Running

This is designed to be run through Docker.   It has several environment variables, so you may wish to load  those through an environment file for sanity and security if you're not already using docker-compose or some other package/deployment system.

### Build Issues

This Dockerfile has a hack to install both version 2.0.2 and the latest version of the bundler gem.  It is to be hoped that one of these is the version of Bundler that you used to bundle this gemfile -- `Gemfile.lock` is built with 2.0.2.  If you get an error like the following, this is a sign that there is a mismatch between the version of Bundler in the container and the version referenced in `Gemfile.lock`.

`/usr/local/lib/ruby/2.6.0/rubygems.rb:283:in `find_spec_for_exe': Could not find 'bundler' (2.0.2) required by your /app/Gemfile.lock. (Gem::GemNotFoundException)
To update to the latest version installed on your system, run `bundle update --bundler`.`
