# TrustFactor

Sadly there are a bunch of bad actors running around trying to give mod teams and other players a bad time by botting, spamming or using illegal image material for ingame sprays. While there is no good way to completely prevent this from happening, there is hop to in making it so tedious for these people that they (hopefully) simply don't bother any further.

This plugins aim to automate some legitimacy checks usually performed on players for SourceMod plugins. The provided API can be used by other plugins to limit features unless certain criteria are met. For example using chat or sprays could be limited until a certain amount of time was spent on the server.
Server moderators can also check a players trust flags with `/checktrust player`.

## TrustFlags and TrustLevel

The following trust flags are possible:
* Server Playtime (`t`)  playtime on server(network)
* Premium (`f `) is not using a free2play account (only tf2 and csgo rn)
* Donor (`d`)  player is donor / has spent money on the server
* Profile Public (`p`)  is community profile public
* Profile Setup (`s`)  is community profile set up
* Profile Level (`l`)  community profile level
* Profile Gametime (`g`)  total playtime for the game
* Profile Age (`o`)  it is not a new account
* Profile PoCBadge (`b`)  level for pillar of community badge

The TrustLevel is simply the sum of all trust flags a player has (currently max 9)

Other plugins might offer you to customize the required trust flags similar to admin flag strings. These consist of the letters above and optionally a number. The number is the minimum amount of flags that have to check out of those specified. For example the value `tf1` would mean that the player would either have to be premium or donor. `tfd2` would require at least two of the three flags. No number requires all flags.

Please keep in mind that depending on the setup, the TrustLevel might never exceed 2 (`td`)!

## Commands and Config

The only command is `/checktrust player` that will reply with a report of known values for the specified player. This command requires the generic admin flag.

The config is automatically generated in cfg/sourcemod/plugin.trustfactor.cfg

Only one of `sm_trustfactor_donorgroup` and `sm_trustfactor_donorflag` is required for the donator flag to be set. The former would be the name of the admin group to check, the latter would be a regular admin flag.

`sm_trustfactor_mingametime` requires the specified minimum amount of playtime account wide for the game in hours. If you set this to 0 it's ignored and the flag will always pass.

`sm_trustfactor_minservertime` requires the specified minimum amount of playtime on the server or server network. If you set this to 0 it's ignored and the flag will always pass.

The playtime value is store as client preference (same as !settings). Tipp: You can synchronize the client preferences (and playtime) between servers by changing the `clientprefs` entry in `database.cfg`

`sm_trustfactor_minpocprogress` requires the Pillar of Community Badge to have at least the specified level before it counts as trusted. If you set this to 0 it's ignored and the flag will always pass. If you did not enable `sm_trustfactor_checksteamlvl` it's recommended to set this to 0.

`sm_trustfactor_minsteamlevel` requires the players profile to have at least the specified steam level for this flag to be set. If you set this to 0 it's ignored and the flag will always pass. If you did not enable `sm_trustfactor_checksteamlvl` it's recommended to set this to 0.

## Installing

In order for the plugin to check the Profile flags, you need to set up the PHP script for Steams WebAPI. More information on how to get an API Key can be found here (a User Key is enough) https://partner.steamgames.com/doc/webapi_overview/auth

After you put the tfcache.php onto your web server (you can rename it if you want), you have to edit it using a text editor.
At the top, put in your API-Key and database configuration, then safe it.

```php
$config = [
	'apikey' => 'steam web api key',
	'database' => 'database name',
	'username' => 'db login user',
	'password' => 'db login password',
	'host'     => 'db host address'
];
```

After that is done, you have to go into the plugins configuration and put in the URL for the PHP script and enable the data groups you want to be checked.

```ini
sm_trustfactor_playercacheurl "https://myserver.net/pathto/tfcache.php"
sm_trustfactor_checkgametime "1" // -> will fetch account's game playtime
sm_trustfactor_checkprofile "1" // -> will fetch profile public, profile set up, fresh account
sm_trustfactor_checksteamlvl "1" // -> will fetch community level, badge level
```

The PHP script will cache player data for 12 hours, so if a player for example barely does not have enough game time, they will have to wait another 12 hours before that value is checked again. If certain checks are not enabled, the associated trust factors will appear as failed to the plugin. This is done also, so you're not exhausting your 100000 request/day limit any time soon.

## Dependencies

* Requires SteamWorks for networking (https://github.com/KyleSanderson/SteamWorks)
* MultiColor is required for compile only (https://github.com/Bara/Multi-Colors)

## Plugin Devs

You can check players trust levels actively or by callbacks. The trust level is only available for players (not bots) and only after `OnClientTrustFactorLoaded`. You can also check that with `IsClientTrustFactorLoaded`. If a players TrustFactor changes throughout them playing, you will get `OnClientTrustFactorChanged`.

It is recommended that you check trust levels with `CheckClientTrust`, but there are a few other options.
