# Git post-receive hook for Slack

This is a bash script that posts a message into your [Slack](https://slack.com) channel when changes are pushed.

Hook this script into `post-receive` for your git repositories.

## How to Install

Note: some git repositories may be "bare". You'll know if your repo is bare or not by checking for a `.git` folder where your repo lives.

Download [git-slack-hook](https://raw.githubusercontent.com/chriseldredge/git-slack-hook/master/git-slack-hook) onto the server which hosts your git repo.

For bare repos, copy/rename it as `/path/to/your/repo/hooks/post-receive`.

For normal/non-bare repos, copy/rename it as `/path/to/your/repo/.git/hooks/post-receive`.

Finally, `chmod +x post-receive` to allow the script to be executed.

## Configuration

Add an Incoming WebHooks integration in your Slack by going to:

    https://my.slack.com/services/new/incoming-webhook

Configure the webhook URL

    git config hooks.slack.webhook-url 'https://hooks.slack.com/services/...'

    Note: _Notifications are disabled if this key is set to an empty value. The hook raises an error if this key is not set._

## Optional
Specify a channel to post in Slack instead of the default:

    git config hooks.slack.channel '#general'

        '#channelname' - post to channel
        '@username' - direct message to user
        'groupname' - post to group

Specify a username to post as. If not specified, the default name `incoming-webhook` will be used:

    git config hooks.slack.username 'git'

Specify an icon to display in Slack instead of the default:

    git config hooks.slack.icon-url 'https://example.com/icon.png'

Specify an emoji icon to display in Slack instead of the default:

    git config hooks.slack.icon-emoji ':twisted_rightwards_arrows:'

Specify a repository nice name that will be shown in messages:

    git config hooks.slack.repo-nice-name 'My Awesome Repository'

Specify whether you want to show only the last commit (or all) when pushing multiple commits:

    git config hooks.slack.show-only-last-commit true

Specify whether you want to show the body of the commit message as well as the title:

    git config hooks.slack.show-full-commit true

Specify if you want to send only certain branches:

    git config hooks.slack.branch-regexp regexp

## Command line arguments

Defaults suitable for multiple repositories can be configured on the command line with one or more `-c KEY VALUE` options. If you use central management for your repositories, you may wish to provide default global values by linking to a common hook script. 

    #!/bin/bash
    
    path/to/git-slack-hook \
      -c hooks.slack.webhook-url 'https://hooks.slack.com/same-hook-for-everyone' \
      -c hooks.slack.channel '#general' \
      -c some.other.key ''

Configuration keys in repositories take precedence over those defined on the command line.

Other command line flags of interest:

    --debug  (prints, but doesn't send the notification. a dry run)
    --trace  (allows debugging the script)

## Linking to Changesets

When the following parameters are set, revision hashes will be turned into links to a web view of your repository.

    git config hooks.slack.repos-root '/path/to/repos'
    git config hooks.slack.changeset-url-pattern 'http://yourserver/%repo_path%/changeset/%rev_hash%'

For example, if your repository is in `/usr/local/repos/myrepo`, set repos_root to `/usr/local/repos/` and set `changeset_url_pattern` to `http://yourserver/%repo_path%/changeset/%rev_hash%` or whatever.

Links can also be created that summarize a list of commits:

    git config hooks.slack.compare-url-pattern 'http://yourserver/%repo_path%/compare/%old_rev_hash%..%new_rev_hash%'
