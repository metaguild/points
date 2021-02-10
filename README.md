# NEAR Experimental SourceCred Instance

This repository contains a template for running a SourceCred instance.

New users of SourceCred are encouraged to fork this repo to start their own
instance.

This repo comes with a GitHub action configured that will run SourceCred automatically
every 6 hours, as well as any time you change the configuration.

# About SourceCred Instances

SourceCred is organized around "instances". Every instance must have a
`sourcecred.json` file at root, which specifies which plugins are active in the
instance. Config and permanent data (e.g. the Ledger) are stored in the master branch.
All output or site data gets stored in the `gh-pages` branch by the Github Action.

Configuration files:
- `config/grain.json` specifies how much Grain to distribute every week. The `maxSimultaneousDistributions` parameter 
determines how many weeks of "back-distributions" to generate if the command hasn't been run in a while (or ever).
- `config/currencyDetails.json` By default, SourceCred uses "Grain" as the name for the currency it distributes to participants.
If you would like to change it to something custom for your community, you can do so by editing this file.
- `config/plugins/$PLUGIN_OWNER/$PLUGIN_NAME` stores plugin-specific data. Each
  plugin has its own directory. See the plugin section below to learn how to configure them.

Permanent Data:
- `data/ledger.json` keeps a history of all grain distributions and transfers as well as identities added / merged.

Generated Data: 

- `cache/` stores intermediate produced by the plugins. This directory should
  not be checked into Git at all.
- `output/` stores output data generated by SourceCred, including the Cred
  Graph and Cred Scores. This directory should be checked into Git; when
  needed, it may be removed and re-generated by SourceCred.
- `site/` which stores the compiled SourceCred frontend, which can display data
  stored in the instance.


# Setup and Usage

Using this instance as a starting point, you can update the config to include
the plugins you want, pointing at the data you care about. We recommend setting up
your instance locally first and make sure its working before pushing your changes
to master and using the Github Action.

1. Get [Yarn] and then run `yarn` to install SourceCred and dependencies.

2. Enable the plugins you want to use by updating the `sourcecred.json` file. e.g. 
to enable all the plugins:
```json
{
  "bundledPlugins": ["sourcecred/discourse", "sourcecred/discord", "sourcecred/github"]
}
```

3. If you are using the GitHub or Discord plugin, copy the `.env.example` file to a `.env` file:
```shell script
cp .env.example .env
```

4. Follow the steps in the [plugin guides below](#supported-plugins) to setup the config files and generate access tokens
for each plugin and then paste them into the `.env` file after the `=` sign.


5. Use the following commands to run your instance locally:

**Load Data**

- `yarn load` loads the data from each plugin into the cache. Run this anytime you want to re-load the data from 
your plugins.

**Run SourceCred**
- `yarn start` creates the cred graph, computes cred scores and runs the front end interface which you can access at `localhost:6006`
in your browser.

NOTE: this command will not load any new data from Discord / GitHub / Discourse, etc. If you want to re-load
all the latest user activity, run `yarn load` again.

**Clear Cache**


- `yarn clean` will clear any cached data that was loaded by the plugins. You can run this if any plugins fail to load. Run `yarn load` after this to re-load the data.

If you want to restart from a clean slate and remove all the generated graphs, you can do so via:
- `yarn clean-all` 

Run `yarn clean-all` if the `yarn start` command fails due to a change in the config or breaking changes in a new version of SourceCred.

### Distributing Grain
- `yarn grain` distributes Grain according to the current Cred scores, and the config in `config/grain.json`. 

This repo also contains a GitHub action for automatically distributing grain. It will run every Sunday and create a Pull Request
with the ledger updated with the new grain balances based on the users Cred scores. The amount of grain to get distributed
every week can be defined in the `config/grain.json` file. 


There are three different policies that can be used to control
how the grain gets distributed: `immediatePerWeek`, `balancedPerWeek`, and `recentPerWeek`. For info on what each policy does, how to choose the right policy for your community, and how Grain operates in general, see [How Grain Works](https://sourcecred.io/docs/beta/grain). 

Below is an example `grain.json` file for a configuration that uses a combination of all three policies. Here we tell SourceCred to distribute 1,000 grain every week, with 25% (250 grain) distributed according to `immediatePerWeek`, 25% (250 grain) distributed according to `balancedPerWeek`, and 75% (750 grain) distributed according to `recentPerWeek`. 

```
{
 "immediatePerWeek": 250,
 "balancedPerWeek": 250,
 "recentPerWeek": 750,
 "recentWeeklyDecayRate": 0.5,
 "maxSimultaneousDistributions": 100
}
```



### Low-level CLI
If you want to go deeper, you can access lower-level commands in the sourcecred CLI in the form of: `yarn sourcecred <command>`. 
For a list of what's available, and what each command does, run `yarn sourcecred help`.

### Publishing on GitHub pages

Once you've got the instance configured to your satisfaction (see instructions on plugins below),
commit and push your changes to master (or make a pull request). The Github Action will then generate the frontend
and deploy it to GitHub Pages. To enable GitHub Pages for your instance, check out [this guide](https://docs.github.com/en/github/working-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site).
Make sure you select `gh-pages` as the branch to publish from.

# Supported Plugins

## GitHub

The GitHub plugin loads GitHub repositories.

You can specify the repositories to load in
`config/plugins/sourcecred/github/config.json`.

The Github Action automatically has its own GITHUB_TOKEN, but if you need to load data from the 
GitHub plugin locally, you must have a GitHub API key in your `.env` file as
`SOURCECRED_GITHUB_TOKEN=<token>` (copy the `.env.example` file for reference). The key should be read-only without any special
scopes or permissions (unless you are loading a private GitHub repository, in which case
the key needs access to your private repositories).

You can generate a GitHub API key [here](https://github.com/settings/tokens).

## Discourse

The Discourse plugin loads Discourse forums; currently, only one forum can be loaded in any single instance. This does not require any special API
keys or permissions. You just need to set the server url in `config/plugins/sourcecred/discourse/config.json`.

## Discord

The Discord plugin loads Discord servers, and mints Cred on Discord reactions. In order for SourceCred to
access your Discord server, you need to generate a "bot token" and paste it in the `.env` file as
`SOURCECRED_DISCORD_TOKEN=<token>` (copy the `.env.example` file for reference). You will also need to add it
to your GitHub Action secrets. 

The full instructions for setting up the Discord plugin can be found in the [Discord plugin page](https://sourcecred.io/docs/beta/plugins/discord/#configuration)
 in the SourceCred documentation. 

# Removing plugins

To deactivate a plugin, just remove it from the `bundledPlugins` array in the `sourcecred.json` file.
You can also remove its `config/plugins/OWNER/NAME` directory for good measure.



[Yarn]: https://classic.yarnpkg.com/
