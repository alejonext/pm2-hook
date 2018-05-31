# pm2-hook

[![npm](https://img.shields.io/npm/v/pm2-hook.svg)](https://www.npmjs.com/package/pm2-hook)
[![npm](https://img.shields.io/npm/dm/pm2-hook.svg)](https://www.npmjs.com/package/pm2-hook)

[PM2](https://github.com/Unitech/pm2) module to process webhooks and update your project realtime. Supports multiple ports and pathes, pre-hook and post-hook features, comparing branches, and different types of updating.

This module is an advanced version of [pm2-webhook](https://github.com/oowl/pm2-webhook) created by [Anton Isaykin](https://github.com/oowl). (The most important conceptual features have been rewritten. A lot of features have been added.)

## Installation

You must have pm2 installed. Just add the module:

```sh
pm2 install pm2-hook
```

## Usage

### GitHub/Bitbucket webhook

Your repository page → Settings → Webhooks & services → Add webhook

| Field | Value |
|:---:|:---:|
| Payload URL | http://example.com:27777/webhook |
| Content Type | application/json |
| Secret | some secret phrase |

### PM2 config

Options:

| Option | Type | Example | Required | Default |
|:---:|:---:|:---:|:---:|:---:|
| port | `number` | 27777 | yes | |
| path | `string` | "/webhook" | no | `/` |
| secret | `string` | "some secret phrase" | no | |
| action | `string` | "pullAndReload" | no | `pullAndRestart` |
| pre_hook | `string` | "npm run stop" | no | |
| post_hook | `string` | "npm run generate_docs" | no | |
| request_branch | `string` | "data.request.data" | no | `push.changes[0].new.name` |


Some notes:

1. You can use all the actions described in the [PM2 docs](http://pm2.keymetrics.io/docs/usage/pm2-api/) that take process name as argument.
2. Webhook has a compare branches feature. It makes a pull request only if catches a request from VCS with the correct branch (if the current branch on your local git is the same as the remote branch contained in the VCS request).

Add environment variables in your [ecosystem.json](http://pm2.keymetrics.io/docs/usage/application-declaration/) file. Only the `port` variable is mandatory.

```sh
{
    "apps": [
        {
            "name": "app",
            ...
            "env_webhook": {
                "port": 23928,
                "path": "/webhook",
                "secret": "some secret phrase",
                "action": "pullAndReload",
                "pre_hook": "npm run stop",
                "post_hook": "npm run generate_docs"
            },
            ...
        },
        ...
    ]
}
```
If your process has been already started, first kill it using the command `pm2 delete ecosystem.json` (We need this, because PM2 has some problems with reloading process configuration and if you only restart your process nothing will work :cry:).
Start your processes with `pm2 start ecosystem.json`.

That's it. Each time you push to your repository, this module runs `pm2 <action> <app name>`.

## Example request


```
POST http://example.com:27777/webhook

{
  "data" : {
    "request" : {
        "data" : "BRANCH_SERVER"
    }
  }
}
```

## Copyright and license

Copyright 2016-2017 Yurii Kramarenko, Dmitry Poddubniy.

Licensed under the [MIT License](https://github.com/Dalas/pm2-webhook/blob/master/LICENSE).
