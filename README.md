## InspIRCd Packages

## About

This repository contains scripts used for generating packages for InspIRCd. It
is nowhere near finished so you should probably not use it just yet. Poke me
(SaberUK) in #inspircd.dev on irc.inspircd.org if you have any questions though.

## Requirements

- Git
- Rake
- Ruby 2.0+

If you are using an old distribution which does not ship Ruby 2.0+ then you can
install it with [RVM](http://rvm.io/rvm/install).

## Usage

Run `rake` with no target to view a list of package targets which can be built.

### Environment Variables

You can set various environment variables to configure the build. These are:

#### INSPIRCD_GIT

This variable sets the Git repository that InspIRCd is installed from. Defaults
to the main repository at `git://github.com/inspircd/inspircd.git`.

#### INSPIRCD_GROUP

The group which you plan to run InspIRCd as. Defaults to the `irc` group if it
exists and the primary group of the current user if it does not.

#### INSPIRCD_MODULES

A space delimited list of extra modules to install at build time. Module names
should not include the `m_` prefix or the `.cpp` file extension. Defaults to
`regex_posix` which is available on all UNIX systems.

#### INSPIRCD_REVISION

The Git revision to check out after cloning. This can be a branch, a tag, or a
commit hash. Defaults to the `master` branch.

#### INSPIRCD_USER

The user which you plan to run InspIRCd as. Defaults to the `irc` user if it
exists and the user you are running as if it does not.

## License

These scripts are licensed under the same license as InspIRCd (GPLv2).
