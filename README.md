# Steam Workshop Deploy

Github Action to upload items to the Steam Workshop. 
Supports an alternative Steam Guard 2FA approach that does not require the TOTP seed.
Instead, you can login through SteamCMD on your local computer once, and then add the session data to a GitHub secret.

Based on [Steam Deploy](https://github.com/game-ci/steam-deploy) by Webber Takken

## Setup

In order to configure this action, add a step that looks like the following:

_(The parameters are explained below)_

Option A. Using SteamCMD MFA files

```yaml
jobs:
  workshopUpload:
    runs-on: ubuntu-latest
    steps:
      - uses: m00nl1ght-dev/steam-workshop-deploy@v1
        with:
          username: ${{ secrets.STEAM_USERNAME }}          
          configVdf: ${{ secrets.STEAM_CONFIG_VDF }}
          path: build
          appId: 1234560
          publishedFileId: 1234560
```

Option B. Using TOTP

```yaml
jobs:
  workshopUpload:
    runs-on: ubuntu-latest
    steps:
      - uses: CyberAndrii/steam-totp@v1
        name: Generate TOTP
        id: steam-totp
        with:
          shared_secret: ${{ secrets.STEAM_SHARED_SECRET }}
      - uses: m00nl1ght-dev/steam-workshop-deploy@v1
        with:
          username: ${{ secrets.STEAM_USERNAME }}
          password: ${{ secrets.STEAM_PASSWORD }}
          totp: ${{ steps.steam-totp.outputs.code }}
          path: build
          appId: 1234560
          publishedFileId: 1234560
```

## Configuration

#### username

The username of the Steam Account that owns the workshop item.

#### configVdf

If Multi-Factor Authentication (MFA) through Steam Guard is enabled for the account, a valid TOTP code is required for login.
This means that simply using username and password isn't enough to authenticate with Steam. 
However, it is possible to go through the MFA process only once by setting up GitHub Secrets for `configVdf` with these steps:
1. Install [Valve's offical steamcmd](https://partner.steamgames.com/doc/sdk/uploading#1) on your local machine. All following steps will be done on your local machine.
1. Try to login with `steamcmd +login <username> <password> +quit`, which may prompt for the MFA code. If so, type in the MFA code that was emailed to your builder account's email address.
1. Validate that the MFA process is complete by running `steamcmd +login <username> +quit` again. It should not ask for the MFA code again.
1. The folder from which you run `steamcmd` will now contain an updated `config/config.vdf` file. Use `cat config/config.vdf | base64 > config_base64.txt` to encode the file. Copy the contents of `config_base64.txt` to a GitHub Secret `STEAM_CONFIG_VDF`.
1. `If:` when running the action you receive another MFA code via email, run `steamcmd +set_steam_guard_code <code>` on your local machine and repeat the `config.vdf` encoding and replace secret `STEAM_CONFIG_VDF` with its contents.

#### password

The password of the Steam Account that owns the workshop item. Only required if `configVdf` is not set.

#### totp

A valid Steam Guard TOTP code. Only required if `configVdf` is not set, and Steam Guard is enabled for the account.

#### appId

The app identifier of the workshop to upload the item to.

#### publishedFileId

The id of the workshop item to be uploaded.

#### path

The path of the directory to be uploaded, relative to the repository root.

#### changeNote

An optional changenote to describe the update.
