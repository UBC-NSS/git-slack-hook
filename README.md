# Git post-receive hook for Slack

This is a bash script that posts a message into your
[Slack](https://slack.com) channel when changes are pushed. It
understands creation/deletion/update, for branches, tags
(annotated or not), and tracking branches.

![alt tag](images/screenshot.png?raw=true "screenshot")

## How to Install

Download the latest release of [git-slack-hook](../../releases/latest) onto the server which hosts your git repo.

_Note: some git repositories may be "bare". You'll know if your repo is bare or not by checking for a `.git` folder where your repo lives._

For bare repos, copy/rename it as `/path/to/your/repo/hooks/post-receive`.

For normal/non-bare repos, copy/rename it as `/path/to/your/repo/.git/hooks/post-receive`.

Finally, `chmod +x post-receive` to allow the script to be executed.

## Configuration

1. Add an Incoming WebHooks integration in your Slack by going to:

       https://my.slack.com/services/new/incoming-webhook

1. Configure the webhook URL

   Once you have the link, you need to configure the git repo that will _receive_ the commits (generally that is on a remote server).

       git config hooks.slack.webhook-url 'https://hooks.slack.com/services/...'

   Note: _Notifications are disabled if this key is set to an empty value. The hook raises an error if this key is not set._

1. Configure the notification settings of this program.

   This program has many options. They are either chosen by `git config` parameters (listed below), or command line arguments.

### With Gitolite

If you use gitolite to manage multiple repositories, you may set
common values in `conf/gitolite.conf` in your gitolite-admin repo, and then rely on
per-repo configurations to tweak certain settings.

    repo    @all
        # enable globally
        config hooks.slack.webhook-url = "https://hooks.slack.com/services/..."
        config hooks.slack.repos.root = /var/git/repositories/
        ...

    repo personal/budget
        # setting the key to '' disables notifications for the repo
        config hooks.slack.webhook-url = ""

 > _Note: To allow the `hooks.slack.*` keys in the config, they need to
   be declared "safe", by adding the prefix to the set of allowed keys in
   `~/.gitolite.rc`_
 >
 > `$GL_GITCONFIG_KEYS = "cgit\\..* hooks\\.mailinglist ... hooks\\.slack\\..*"`

Another option, to keep the gitolite config free of extra config, is
to set some of the common options on command-line arguments to the
post-receive hook (which will be shared across repos). _(See [Command Line Args](#cli))_

## Other (optional) Settings

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

Truncate the notification message after a maximum number of commits:

    git config hooks.slack.max-commits-per-changeset 20

 > _Note: set to 0 to show just a summary (e.g. "user x pushed 3 commits"), set to 1 to show the summary + the most recent commit. If the number of commits in the changeset is higher than this value, output will be truncated, and there will be a note to that effect._

Specify whether you want to show the body of the commit message as well as the title:

    git config hooks.slack.show-full-commit false # default false

Specify if you want to notify only about branches matching the following (in POSIX extended syntax):

    git config hooks.slack.branch-regexp regexp  # e.g. '^master$|^my-tag$'

 > _Note: the regexp is matched against the short name of the ref/ (e.g. `master`)_

Provide a sender.cfg file to map system users (or gitolite usernames) to full name and addresses:

    git config hooks.slack.sender-cfg /path/to/sender.cfg

 > _Note: this is the same file you would provide to the post-receive email hook. This username may be different than the author of the commit. On gitolite systems, the username is extracted from the `$GL_USER` environment var._

## <a name="cli"></a> Command line arguments

Defaults suitable for multiple repositories can be configured on the command line with one or more `-c KEY VALUE` options. If you use central management for your repositories, you may provide sensible default global values by linking to a common hook script. This way you won't clog your git configs with keys.

    #!/bin/bash
    
    path/to/git-slack-hook \
      -c hooks.slack.webhook-url 'https://hooks.slack.com/same-hook-for-everyone' \
      -c hooks.slack.channel '#general' \
      -c some.other.key ''

 > _Note: Configuration keys in repositories take precedence over those defined on the command line._

Other command line flags of interest:

 >   `--debug`  (prints, but doesn't send out the notification. a dry run)
 >
 >   `--trace`  (allows debugging the script)

## Linking to Changesets

When the following parameters are set, revision hashes will be turned into links to a web view of your repository.

    git config hooks.slack.repos-root '/path/to/repos/'
    git config hooks.slack.changeset-url-pattern 'http://yourserver/%repo_path%/changeset/%rev_hash%'

 > For example, if your repository is in `/var/repos/myrepo`, set repos_root to `/var/repos/` and set `changeset_url_pattern` to `http://yourserver/%repo_path%/changeset/%rev_hash%` or whatever. Make sure repos-root ends with a trailing slash.

Links can also be created that summarize a list of commits:

    git config hooks.slack.compare-url-pattern 'http://yourserver/%repo_path%/compare/%old_rev_hash%..%new_rev_hash%'

## Templating

The notification message headers can be changed via config, subject to the templating rules below:

    git config hooks.slack.msg-create  \
      '*[%repo_name%]* %refname_type%/%short_refname% created -- %user_full%'
    git config hooks.slack.msg-delete \
      ... deleted -- %user_full%'
    git config hooks.slack.msg-update-many \
      ... updated (<comparelink>%num% new commits</comparelink>) -- %user_full%'
    git config hooks.slack.msg-update-one  \
      ... updated (<comparelink>1 new commit</comparelink>) -- %user_full%'

The following common variables are available to all templates:

| Variable | Substitution |
| :--- | --- |
|`%num%`          | The number of commits pushed in the changeset (this is 0 for non-update notifications)  |
|`%new_rev_hash%` | The revision after a branch update (i.e. the latest) |
|`%old_rev_hash%` | The revision before a branch update |
|`%short_refname%`| The shortname for a branch/tag/etc. e.g. `master`, or `feature/foo` |
|`%repo_name%`    | The display name of the repo. (either obtained via config, `$GL_REPO`, or working path) |
|`%user%`         | The username triggering the hook. On gitolite systems this is equivalent to `$GL_USER` |
|`%user_full%`    | The fullname+email of whoever's making the update.<br>This is the same as `%user%` if the username cannot be found in sender.cfg. |

URL templates can use the common variables, plus all the the following:

| Variable | Substitution |
| :--- | --- |
| `%repo_path%` | The path of the repository, relative to the repository root.<br>Derived from the directory and the value of `repos-root`. |
| `%rev_hash%` | In the changeset URL (for each commit), this will be the revision of the commit. |

Message `msg-*` config templates can use `<comparelink>LABEL</compare_link>` to configure the label of the link on Slack. Template variables inside `LABEL` will be evaluated.
