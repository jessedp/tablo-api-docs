---
title: Unofficial Tablo API Docs & Info

language_tabsDISABLED: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - <a href='https://community.tablotv.com/c/tablo-apps/third-party-apps-plex/17' target="_new">Tablo 3rd Party Community Forum</a>

includesDISABLED:
  - errors

search: true

code_clipboard: true

meta:
  - name: description
    content: Documentation for the Tablo API
---

# Tablo API Introduction

This is **UNOFFICIAL** and not endorsed by Tablo or Nuvyyo.

This contains a hodge podge of information gathered while working with the API for [Tablo](https://www.tablotv.com/) devices by Nuvyyo. Technically an NDA can be signed with them for acccess to real documentation and maybe more.

Tablo offers both [**Network Connected**](https://www.tablotv.com/tablo-dvrs-how-they-work/) and [**TV Connected**](https://www.tablotv.com/tablo-tv-connected-dvrs-how-they-work/) devices.

**ONLY Network Connected** devices are supposed to work with the API.

# Authentication

There is no Authentication or Authorization required for the publicly availale API.

There _is_ a private API used by the Tablo Connect web app (and possible other official clients) that uses [Meteor](https://www.meteor.com/), but access to the channels, etc in use are not publicly accessible.

# Network Options

## Discovery

```shell
curl "https://api.tablotv.com/assocserver/getipinfo/"
```

> The above command returns JSON structured like this:

```json
{
  "success": true,
  "cpes": [
    {
      "serverid": "SID_5087B901A089",
      "host": "duo",
      "name": "DenTablo",
      "board": "duo",
      "server_version": "2.2.42rc2225809",
      "public_ip": "100.200.100.50",
      "private_ip": "192.168.1.242",
      "http": 21020,
      "ssl": 0,
      "slip": 21021,
      "roku": 0,
      "last_seen": "2023-01-02 15:53:41.941603+00:00",
      "modified": "2023-01-02 15:53:41.942854+00:00",
      "inserted": "2017-11-23 02:50:04.988414+00:00"
    }
  ]
}
```

The IP addresses for a Tablo devices on a local network can be discovered 2 ways:

1. An HTTP request to [https://api.tablotv.com/assocserver/getipinfo/](https://api.tablotv.com/assocserver/getipinfo/) . This is a service hosted by Tablo - see the example to the right.
2. A UDP broadcast request on port `8881`, listening for a response of port `8881`. Python [example #1](https://github.com/Nuvyyo/script.tablo/blob/master/lib/tablo/discovery.py#L107)[example #2](https://github.com/jessedp/tut/blob/master/tablo/discovery.py#L125), javascript/node [example #1](https://github.com/jessedp/tablo-api-js/blob/main/src/discovery.ts#L20)

You will only care about the **private** IP address.

## Connectivity

A Tablo network device will expose several ports on the discovered IP for varying uses:

| Port  | Can Be Used | Description                                                                                                                                           |
| ----- | ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| 80    | **Yes**     | HTTP server for streaming (via `/watch`)                                                                                                              |
| 8885  | **Yes**     | Standard HTTP RESTful server - this is used for most things                                                                                           |
| 8887  | _No_        | HTTP Server External Ports forwarded to                                                                                                               |
| 18080 | **Yes**     | HTTP server with Indexes on / Psudeo-FTP server - provides direct, read-only access to part of hard drive and interal server files (eg, `/pvr/` path) |

# Server

Methods to obatin basic information about the server

## Get General Server Info

```shell
curl "http://192.168.1.242:8885/server/info"
```

> The above command returns JSON structured like this:

```json
{
  "server_id": "SID_5087B901A089",
  "name": "DenTablo",
  "timezone": "",
  "version": "2.2.42",
  "local_address": "192.168.1.242",
  "setup_completed": true,
  "build_number": 2225809,
  "model": {
    "wifi": true,
    "tuners": 2,
    "type": "duo",
    "name": "Tablo DUAL 64GB"
  },
  "availability": "ready",
  "cache_key": "4d3b41b1-a912-5555-6666-9876543210",
  "product": "tablo"
}
```

### HTTP Request

`GET http://192.168.1.242:8885/server/info`

## Get Server Capabilities

```shell
curl "http://192.168.1.242:8885/server/capabilities"
```

> The above command returns JSON structured like this:

```json
{
  "capabilities": [
    "guide_recording_refs",
    "recordings_keep",
    "recording_options",
    "xSwup",
    "search",
    "subscription_services",
    "ac3",
    "manual_programs_edit",
    "airings_by_day",
    "movie_ratings",
    "live_quality",
    "genres",
    "conflicts",
    "cp",
    "lc",
    "rc",
    "rf",
    "snap_grid",
    "params",
    "scan_stop",
    "sap"
  ]
}
```

This returns information about all hard drives connectded to the Tablo.

### HTTP Request

`GET http://192.168.1.242:8885/server/capabilities`

## Get Overall Guide Status

```shell
curl "http://192.168.1.242:8885/server/guide/status"
```

> The above command returns JSON structured like this:

```json
{
  "guide_seeded": true,
  "last_update": "2023-01-02T09:27:02Z",
  "limit": "2023-01-15T08:32Z",
  "download_progress": null
}
```

### HTTP Request

`GET http://192.168.1.242:8885/server/guide/status`

## Get Status of the Server's update progress

```shell
curl "http://192.168.1.242:8885/server/update/info"
```

> The above command returns JSON structured like this:

```json
{
  "details": null,
  "available_update": null,
  "last_checked": "2023-01-01T19:15:40Z",
  "last_update": null,
  "sequence": ["downloading", "installing", "rebooting"],
  "current_step": null,
  "state": "none",
  "error": null
}
```

If in progress, retrieve information about the status on any ongoing server updates.

### HTTP Request

`GET http://192.168.1.242:8885/server/update/info`

## Get All Hard drives

```shell
curl "http://192.168.1.242:8885/server/harddrives"
```

> The above command returns JSON structured like this:

```json
[
  {
    "error": null,
    "connected": true,
    "format_state": "authorized",
    "name": "WD Elements 25A2 1019 (1000 GB)",
    "busy_state": "ready",
    "kind": "external",
    "size": 984340578304,
    "size_mib": 938740,
    "usage": 927380713472,
    "usage_mib": 884419,
    "free": 56959864832,
    "free_mib": 54321,
    "limit": 984340578304,
    "limit_mib": 938740
  }
]
```

This returns information about all hard drives connectded to the Tablo.

### HTTP Request

`GET http://192.168.1.242:8885/server/harddrives`

# Settings

Methods to gather information about some basic server setttings

## Get General Settings

```shell
curl "http://192.168.1.242:8885/settings/info"
```

> The above command returns JSON structured like this:

```json
{
  "led": "on",
  "recording_quality": "/settings/recording_qualities/10",
  "extend_live_recordings": true,
  "auto_delete_recordings": true,
  "exclude_duplicates": true,
  "fast_live_startup": true,
  "audio": "aac",
  "commercial_skip": "on",
  "livetv_quality": "/settings/recording_qualities/10",
  "data_collection": true,
  "preferred_audio_track": "default"
}
```

This returns the general settings for the device

### HTTP Request

`GET http://192.168.1.242:8885/settings/info`

## Get Recording Quality Options

```shell
curl "http://192.168.1.242:8885/settings/recording_qualities"
```

> The above command returns JSON structured like this:

```json
[
  "/settings/recording_qualities/10",
  "/settings/recording_qualities/8",
  "/settings/recording_qualities/5",
  "/settings/recording_qualities/3",
  "/settings/recording_qualities/2"
]
```

This returns the recording quality options supported by the device.

### HTTP Request

`GET http://192.168.1.242:8885/settings/recording_qualities`

## Get Specific Recording Quality info

```shell
curl "http://192.168.1.242:8885/settings/recording_qualities/10"
```

> The above command returns JSON structured like this:

```json
{
  "object_id": 10,
  "path": "/settings/recording_qualities/10",
  "name": "HD 1080 – 10 Mbps, 720@60fps",
  "recommended": false,
  "available": true,
  "hourly_mb": 4615
}
```

This returns details on a specific recording quality option for the device.

### HTTP Request

`GET http://192.168.1.242:8885/settings/recording_qualities/<NUM>`

The **number** at the end of each entry can be passed to [Get a Specific Recording Quality Setting](#get-specific-recording-quality-info). Also note that each entry here is also a **path** that can simply be appended to your `http://server:8885/` address.

### URL Parameters

| Parameter | Description                                                                                                                                                          |
| --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| NUM       | The number of the recording quality info to return. Known: **2**, **3**, **5**, **8**, **10**. See also [Recording Quality Settings](#get-recording-quality-options) |

# Guide

Methods to interact with the Guide. These would typically be used either if trying to replicate parts of the current Guide display or possibly during scheduling.

## Get Airings

```shell
curl "http://192.168.1.242:8885/guide/airings"
```

> The above command returns JSON structured like this:

```json
[
  "/guide/series/episodes/2271819",
  "/guide/movies/airings/2267295",
  "/guide/sports/events/2265867",
  "/guide/series/episodes/2267392",
  ...
]
```

This endpoint retrieves a `path` to every Airing avaiable - for additional information, load the `path` or a set may be **POST**ed to [/batch](#batch-submissions).

### HTTP Request

`GET http://192.168.1.242:8885/guide/airings`

## Get Shows

```shell
curl "http://192.168.1.242:8885/guide/shows"
```

> The above command returns JSON structured like this:

```json
[
  "/guide/series/2100779",
  "/guide/series/631734",
  "/guide/programs/974119",
  "/guide/movies/2266038",
  "/guide/sports/2274202",
  "/guide/series/2252673"
]
```

This endpoint retrieves a `path` to ANY shows matching the query parameters that the Guide currently has available to it. For more information, each `path` must be **POST**ed to [/batch](#batch-submissions).

### HTTP Request

`GET http://192.168.1.242:8885/guide/shows`

### URL Parameters

| Parameter | Description                                                               |
| --------- | ------------------------------------------------------------------------- |
| qualifier | A _qualifier_ to limit the items returned. Known: **premiering**, **new** |

The **qualifier** parameter only limits the list returned - there is no change in the output data shape.

<code>
curl "http://192.168.1.242:8885/guide/shows?qualifier=new"
</code> - 226+ airings? not sure what these are

<code>
curl "http://192.168.1.242:8885/guide/shows?qualifier=premiering"
</code> - seems to be 25 airings... about to be aired? be aired for the first time?

## Get All Channels

```shell
curl "http://192.168.1.242:8885/guide/channels"
```

> The above command returns JSON structured like this:

```json
[
  "/guide/channels/612675",
  "/guide/channels/982743",
  "/guide/channels/975971",
  "/guide/channels/612660",
...
]
```

This will return `paths` for all `channels` registerered on the device. The example is truncated on purpose.

### HTTP Request

`GET http://192.168.1.242:8885/guide/channels`

## Get Specific Channel

```shell
curl "http://192.168.1.242:8885/guide/channels/982743"
```

> The above command returns JSON structured like this:

```json
{
  "object_id": 982743,
  "path": "/guide/channels/982743",
  "channel": {
    "call_sign": "WAGA-HD",
    "call_sign_src": "WAGA-HD",
    "major": 5,
    "minor": 1,
    "network": "FOX",
    "resolution": "hd_720",
    "favourite": false,
    "tms_station_id": "20430",
    "tms_affiliate_id": "10212",
    "source": "ota"
  }
}
```

This will return all data for the channel `path` requested.

### HTTP Request

`GET http://192.168.1.242:8885/guide/channels/<object_id>`

### URL Parameters

| Parameter     | Description                      |
| ------------- | -------------------------------- |
| **object_id** | The **object_id** of the Channel |

## Get All Movies

```shell
curl "http://192.168.1.242:8885/guide/movies"
```

> The above command returns JSON structured like this:

```json
[
  "/guide/movies/2267291",
  "/guide/movies/2274033",
  "/guide/movies/2273923",
  "/guide/movies/2271908",
...
]
```

This will return `paths` for all `movies` registerered on the device. The example is truncated on purpose.

### HTTP Request

`GET http://192.168.1.242:8885/guide/movies`

## Get Specific Movie

```shell
curl "http://192.168.1.242:8885/guide/movies/2267291"
```

> The above command returns JSON structured like this:

```json
{
  "object_id": 2267291,
  "path": "/guide/movies/2267291",
  "movie": {
    "title": "Coach of the Year",
    "plot": "An ex-football-player Vietnam veteran (Robert Conrad) coaches delinquents at a juvenile correctional facility.",
    "original_runtime": 6000,
    "release_year": 1980,
    "film_rating": null,
    "quality_rating": 6,
    "cast": [
      "Robert Conrad",
      "Erin Gray",
      "Red West",
      "Daphne Maxwell",
      "David Raynr",
      "Ricky Paull Goldin",
      "Dean Hill",
      "Joneal Joplin"
    ],
    "directors": ["Don Medford"],
    "awards": [],
    "background_image": { "image_id": 2267294, "has_title": false },
    "cover_image": { "image_id": 2267293, "has_title": false },
    "thumbnail_image": { "image_id": 2267292, "has_title": true },
    "genres": ["Drama"]
  },
  "show_counts": {
    "airing_count": 1,
    "conflicted_count": 0,
    "scheduled_count": 0
  },
  "keep": { "rule": "none", "count": null },
  "recordings_path": null
}
```

This will return all data for the Movie `path` requested.

### HTTP Request

`GET http://192.168.1.242:8885/guide/movies/<object_id>`

## Get All Series

```shell
curl "http://192.168.1.242:8885/guide/series"
```

> The above command returns JSON structured like this:

```json
[
  "/guide/series/2100779",
  "/guide/series/631734",
  "/guide/series/631686",
  "/guide/series/2272366",
  ...
]
```

This will return `paths` for all `series` registerered on the device. The example is truncated on purpose.

### HTTP Request

`GET http://192.168.1.242:8885/guide/series`

## Get Specific Series

```shell
curl "http://192.168.1.242:8885/guide/series/2100779"
```

> The above command returns JSON structured like this:

```json
{
  "object_id": 2100779,
  "path": "/guide/series/2100779",
  "schedule": {
    "rule": "new",
    "channel_path": null,
    "offsets": { "start": 0, "end": 0, "source": "none" }
  },
  "schedule_rule": "new",
  "series": {
    "title": "Abbott Elementary",
    "genres": ["Sitcom"],
    "description": "A group of dedicated, passionate teachers — and a slightly tone-deaf principal — find themselves thrown together in a Philadelphia public school where, despite the odds stacked against them, they are determined to help their students succeed in life. Though these incredible public servants may be outnumbered and underfunded, they love what they do — even if they don't love the school district's less-than-stellar attitude toward educating children.",
    "orig_air_date": "2021-12-07",
    "episode_runtime": 1800,
    "series_rating": "tvpg",
    "cast": [
      "Quinta Brunson",
      "Tyler James Williams",
      "Janelle James",
      "Chris Perfetti",
      "Lisa Ann Walter",
      "Sheryl Lee Ralph",
      "William Stanford Davis"
    ],
    "awards": [
      {
        "won": true,
        "name": "Emmy (Primetime)",
        "category": "Outstanding Supporting Actress in a Comedy Series",
        "year": 2022,
        "nominee": "Sheryl Lee Ralph"
      },
      {
        "won": true,
        "name": "Emmy (Primetime)",
        "category": "Outstanding Writing for a Comedy Series",
        "year": 2022
      }
    ],
    "background_image": { "image_id": 2265261, "has_title": false },
    "cover_image": { "image_id": 2265260, "has_title": true },
    "thumbnail_image": { "image_id": 2265259, "has_title": true }
  },
  "show_counts": {
    "airing_count": 3,
    "conflicted_count": 0,
    "scheduled_count": 2
  },
  "keep": { "rule": "none", "count": null },
  "recordings_path": "/recordings/series/2183172"
}
```

This will return all data for the Series `path` requested.

### HTTP Request

`GET http://192.168.1.242:8885/guide/series/<object_id>`

### URL Parameters

| Parameter     | Description                     |
| ------------- | ------------------------------- |
| **object_id** | The **object_id** of the Series |

## Get Active Series Episode

```shell
curl "http://192.168.1.242:8885/guide/series/episodes/2272260"
```

> The above command returns JSON structured like this:

```json
{
  "path": "/guide/series/episodes/2272260",
  "object_id": 2272260,
  "series_path": "/guide/series/2272255",
  "season_path": "/guide/series/seasons/2272259",
  "episode": {
    "title": "Auditions 1",
    "description": "Winners, finalists, fan favorites and viral sensations from AGT and Got Talent franchises around the world audition; one act is given a Golden Buzzer and earns a spot in the finals; the AGT super fans vote on one additional act to move to the finals.",
    "number": 1,
    "season_number": 1,
    "orig_air_date": "2023-01-02"
  },
  "airing_details": {
    "datetime": "2023-01-03T01:00Z",
    "duration": 7200,
    "channel_path": "/guide/channels/975971",
    "channel": {
      "object_id": 975971,
      "path": "/guide/channels/975971",
      "channel": {
        "call_sign": "WXIA-TV",
        "call_sign_src": "WXIA-TV",
        "major": 11,
        "minor": 1,
        "network": "NBC",
        "resolution": "hd_1080",
        "favourite": false,
        "tms_station_id": "19587",
        "tms_affiliate_id": "10991",
        "source": "ota"
      }
    },
    "show_title": "America's Got Talent: All-Stars"
  },
  "qualifiers": ["new", "series_premiere", "cc", "primetime"],
  "schedule": {
    "state": "none",
    "qualifier": "none",
    "skip_reason": "none",
    "offsets": { "start": 0, "end": 0, "source": "none" }
  }
}
```

This will return all data for the Active Series Episode requested.

### HTTP Request

`GET http://192.168.1.242:8885/guide/series/episodes/<object_id>`

### URL Parameters

| Parameter     | Description                     |
| ------------- | ------------------------------- |
| **object_id** | The **object_id** of the Series |

## Get All Sports

```shell
curl "http://192.168.1.242:8885/guide/sports"
```

> The above command returns JSON structured like this:

```json
[
  "/guide/sports/2274202",
  "/guide/sports/2270947",
  "/guide/sports/2261429",
  "/guide/sports/2232007",
  ...
]
```

This will return `paths` for all `sports` registerered on the device. The example is truncated on purpose.

### HTTP Request

`GET http://192.168.1.242:8885/guide/sports`

## Get Specific Sport

```shell
curl "http://192.168.1.242:8885/guide/sports/2274202"
```

> The above command returns JSON structured like this:

```json
{
  "object_id": 2274202,
  "path": "/guide/sports/2274202",
  "schedule": {
    "rule": "none",
    "channel_path": null,
    "offsets": { "start": 0, "end": 0, "source": "none" }
  },
  "schedule_rule": "none",
  "sport": {
    "title": "AMA Supercross",
    "description": "Coverage of AMA Supercross racing action from various locations.",
    "genres": ["Motorcycle Racing"],
    "background_image": { "image_id": 2274205, "has_title": false },
    "cover_image": { "image_id": 2274204, "has_title": false },
    "thumbnail_image": { "image_id": 2274203, "has_title": true }
  },
  "show_counts": {
    "airing_count": 2,
    "conflicted_count": 0,
    "scheduled_count": 0
  },
  "keep": { "rule": "none", "count": null },
  "recordings_path": null
}
```

This will return all data for the Sports `path` requested.

### HTTP Request

`GET http://192.168.1.242:8885/guide/sports/<object_id>`

### URL Parameters

| Parameter     | Description                    |
| ------------- | ------------------------------ |
| **object_id** | The **object_id** of the Sport |

## Get Active Sport Event

```shell
curl "http://192.168.1.242:8885/guide/sports/events/2265867"
```

> The above command returns JSON structured like this:

```json
{
  "path": "/guide/sports/events/2265867",
  "object_id": 2265867,
  "sport_path": "/guide/sports/2264596",
  "event": {
    "title": "Lakeland Magic at College Park Skyhawks",
    "description": "From Gateway Center Arena at College Park in College Park, Ga.",
    "season": "2022-2023",
    "season_type": "regular",
    "venue": null,
    "teams": [
      { "name": "Lakeland Magic", "team_id": 15400 },
      { "name": "College Park Skyhawks", "team_id": 20034 }
    ],
    "home_team_id": 20034
  },
  "airing_details": {
    "datetime": "2023-01-03T00:00Z",
    "duration": 7800,
    "channel_path": "/guide/channels/612673",
    "channel": {
      "object_id": 612673,
      "path": "/guide/channels/612673",
      "channel": {
        "call_sign": "WPCH-TV",
        "call_sign_src": "WPCH-TV",
        "major": 17,
        "minor": 1,
        "network": null,
        "resolution": "hd_720",
        "favourite": false,
        "tms_station_id": "32795",
        "tms_affiliate_id": "Independent",
        "source": "ota"
      }
    },
    "show_title": "NBA G League Basketball"
  },
  "qualifiers": ["live", "primetime"],
  "schedule": {
    "state": "none",
    "qualifier": "none",
    "skip_reason": "none",
    "offsets": { "start": 0, "end": 0, "source": "none" }
  }
}
```

This will return the data for the specific Active Sports Event requested.

### HTTP Request

`GET http://192.168.1.242:8885/guide/sports/events/<object_id>`

### URL Parameters

| Parameter     | Description                                 |
| ------------- | ------------------------------------------- |
| **object_id** | The **object_id** of the Active Sport Event |

# Recordings

## Get Airings

```shell
curl "http://192.168.1.242:8885/recordings/airings"
```

> The above command returns JSON structured like this:

```json
[
  "/recordings/series/episodes/2267413",
  "/recordings/series/episodes/2265212",
  "/recordings/programs/airings/2261284",
  "/recordings/programs/airings/2260157",
  ...
]
```

This endpoint retrieves a `path` to every Recording avaiable - for additional information, load the `path` or a set may be **POST**ed to [/batch](#batch-submissions).

### HTTP Request

`GET http://192.168.1.242:8885/recordings/airings`

## Get Shows

```shell
curl "http://192.168.1.242:8885/recordings/shows"
```

> The above command returns JSON structured like this:

```json
[
  "/recordings/series/2183172",
  "/recordings/movies/974203",
  "/recordings/programs/906209",
  "/recordings/sports/946945"
  "/recordings/series/2170883",
  ...
]
```

This endpoint retrieves a `path` to all Shows matching existing Recordings. For more information, each `path` must be **POST**ed to [/batch](#batch-submissions).

### HTTP Request

`GET http://192.168.1.242:8885/recordings/shows`

## Get All Channels

```shell
curl "http://192.168.1.242:8885/recordings/channels"
```

> The above command returns JSON structured like this:

```json
[
  "/recordings/channels/568024",
  "/recordings/channels/577763",
  "/recordings/channels/634690",
  "/recordings/channels/974152",
  ...
]
```

This will return `paths` for all `Channels` used for existing `Recordings`. The example is truncated on purpose.

### HTTP Request

`GET http://192.168.1.242:8885/recordings/channels`

## Get Specific Channel

```shell
curl "http://192.168.1.242:8885/recordings/channels/634690"
```

> The above command returns JSON structured like this:

```json
{
  "object_id": 634690,
  "path": "/recordings/channels/634690",
  "channel": {
    "call_sign": "WSB-HD",
    "call_sign_src": "WSB-HD",
    "major": 2,
    "minor": 1,
    "network": "ABC",
    "resolution": "hd_720",
    "favourite": false,
    "tms_station_id": "19586",
    "tms_affiliate_id": "10003",
    "source": "ota"
  }
}
```

This will return all data for the channel `path` requested.

### HTTP Request

`GET http://192.168.1.242:8885/recordings/channels/<object_id>`

### URL Parameters

| Parameter     | Description                      |
| ------------- | -------------------------------- |
| **object_id** | The **object_id** of the Channel |

## Get All Movies

```shell
curl "http://192.168.1.242:8885/recordings/movies"
```

> The above command returns JSON structured like this:

```json
[
  "/recordings/movies/974203",
  "/recordings/movies/887235",
  "/recordings/movies/653522",
  "/recordings/movies/625353",
  ...
]
```

This will return `paths` for all recorded `movies` on the device. The example is truncated on purpose.

### HTTP Request

`GET http://192.168.1.242:8885/recordings/movies`

## Get Specific Movie

```shell
curl "http://192.168.1.242:8885/recordings/movies/974203"
```

> The above command returns JSON structured like this:

```json
{
  "object_id": 974203,
  "path": "/recordings/movies/974203",
  "movie": {
    "title": "Adventures in Silverado",
    "plot": "A stagecoach driver seeks to capture \"The Monk,\" a notorious highwayman, in order to clear himself.",
    "original_runtime": 4500,
    "release_year": 1948,
    "film_rating": null,
    "quality_rating": 6,
    "cast": ["William Bishop", "Forrest Tucker", "Gloria Henry"],
    "directors": ["Phil Karlson"],
    "awards": [],
    "background_image": { "image_id": 930054, "has_title": true },
    "cover_image": { "image_id": 930053, "has_title": true },
    "thumbnail_image": { "image_id": 930052, "has_title": true },
    "tms_id": "MV000204150000",
    "genres": ["Western"]
  },
  "show_counts": {
    "airing_count": 1,
    "unwatched_count": 1,
    "protected_count": 1,
    "watched_and_protected_count": 0,
    "failed_count": 0
  },
  "user_info": { "up_next": "/recordings/movies/airings/974202" },
  "keep": { "rule": "none", "count": null },
  "guide_path": null
}
```

This will return all data for the Movie `path` requested.

### HTTP Request

`GET http://192.168.1.242:8885/recordings/movies/<object_id>`

### URL Parameters

| Parameter     | Description                    |
| ------------- | ------------------------------ |
| **object_id** | The **object_id** of the Movie |

## Get All Series

```shell
curl "http://192.168.1.242:8885/recordings/series"
```

> The above command returns JSON structured like this:

```json
[
  "/recordings/series/2183172",
  "/recordings/series/2170891",
  "/recordings/series/2170883",
  "/recordings/series/597009",
  "/recordings/series/8647",
  ...
]
```

This will return `paths` for all `series` recorded on the device. The example is truncated on purpose.

### HTTP Request

`GET http://192.168.1.242:8885/recordings/series`

## Get Specific Series

```shell
curl "http://192.168.1.242:8885/recordings/series/2183172"
```

> The above command returns JSON structured like this:

```json
{
  "object_id": 2183172,
  "path": "/recordings/series/2183172",
  "series": {
    "title": "Abbott Elementary",
    "genres": ["Sitcom"],
    "description": "A group of dedicated, passionate teachers — and a slightly tone-deaf principal — find themselves thrown together in a Philadelphia public school where, despite the odds stacked against them, they are determined to help their students succeed in life. Though these incredible public servants may be outnumbered and underfunded, they love what they do — even if they don't love the school district's less-than-stellar attitude toward educating children.",
    "orig_air_date": "2021-12-07",
    "episode_runtime": 1860,
    "series_rating": "tvpg",
    "cast": [
      "Quinta Brunson",
      "Tyler James Williams",
      "Janelle James",
      "Chris Perfetti",
      "Lisa Ann Walter",
      "Sheryl Lee Ralph",
      "William Stanford Davis"
    ],
    "awards": [],
    "background_image": { "image_id": 2265261, "has_title": false },
    "cover_image": { "image_id": 2265260, "has_title": true },
    "thumbnail_image": { "image_id": 2265259, "has_title": true },
    "tms_series_id": "20001974",
    "tms_id": "SH038712170000"
  },
  "show_counts": {
    "airing_count": 6,
    "unwatched_count": 0,
    "protected_count": 0,
    "watched_and_protected_count": 0,
    "failed_count": 1
  },
  "user_info": { "up_next": "/recordings/series/episodes/2235655" },
  "keep": { "rule": "none", "count": null },
  "guide_path": "/guide/series/2100779"
}
```

This will return all data for the Series `path` requested.

### HTTP Request

`GET http://192.168.1.242:8885/recordings/series/<object_id>`

### URL Parameters

| Parameter     | Description                     |
| ------------- | ------------------------------- |
| **object_id** | The **object_id** of the Series |

## Get Specific Series Episode

```shell
curl "http://192.168.1.242:8885/recordings/series/episodes/2235655"
```

> The above command returns JSON structured like this:

```json
{
  "object_id": 2235655,
  "path": "/recordings/series/episodes/2235655",
  "series_path": "/recordings/series/2183172",
  "season_path": "/recordings/series/seasons/2183173",
  "snapshot_image": { "image_id": 2248104, "has_title": false },
  "airing_details": {
    "datetime": "2022-12-01T02:00Z",
    "duration": 1860,
    "channel_path": "/recordings/channels/634690",
    "channel": {
      "object_id": 634690,
      "path": "/recordings/channels/634690",
      "channel": {
        "call_sign": "WSB-HD",
        "call_sign_src": "WSB-HD",
        "major": 2,
        "minor": 1,
        "network": "ABC",
        "resolution": "hd_720",
        "favourite": false,
        "tms_station_id": "19586",
        "tms_affiliate_id": "10003",
        "source": "ota"
      }
    },
    "show_title": "Abbott Elementary"
  },
  "video_details": {
    "state": "finished",
    "clean": true,
    "cloud": false,
    "uploading": false,
    "audio": "aac",
    "size": 1369825280,
    "duration": 1918,
    "width": 1280,
    "height": 720,
    "comskip": { "state": "ready", "error": null },
    "has_snap_grid": true,
    "schedule_offsets": { "start": 2, "end": 65, "deprecated": true },
    "recorded_offsets": { "start": 2, "end": 65 },
    "airing_offsets": { "start": 0, "end": 0, "source": "none" },
    "seek": 0,
    "error": null,
    "warnings": []
  },
  "user_info": { "position": 0, "watched": true, "protected": false },
  "episode": {
    "title": "Sick Day",
    "description": "Janine is out sick for the day, so Ava grows desperate and must step in to help out; Barbara and Melissa revel in the quieter-than-normal teacher's lounge.",
    "number": 9,
    "season_number": 2,
    "orig_air_date": "2022-11-30",
    "tms_id": "EP038712170023"
  },
  "qualifiers": ["cc"]
}
```

This will return all data for the Active Series Episode requested.

### HTTP Request

`GET http://192.168.1.242:8885/recordings/series/episodes/<object_id>`

### URL Parameters

| Parameter     | Description                     |
| ------------- | ------------------------------- |
| **object_id** | The **object_id** of the Series |

## Get All Sports

```shell
curl "http://192.168.1.242:8885/recordings/sports"
```

> The above command returns JSON structured like this:

```json
[
  "/recordings/sports/946945",
  "/recordings/sports/889644",
  "/recordings/sports/873643",
  ....
]
```

This will return `paths` for all `sports` recordings on the device. The example is truncated on purpose.

### HTTP Request

`GET http://192.168.1.242:8885/recordings/sports`

## Get Specific Sport

```shell
curl "http://192.168.1.242:8885/recordings/sports/946945"
```

> The above command returns JSON structured like this:

```json
{
  "object_id": 946945,
  "path": "/recordings/sports/946945",
  "sport": {
    "title": "2016 NBA Finals",
    "description": "Replay of games from the 2016 series between the Cavaliers and Warriors.",
    "genres": ["Basketball"],
    "background_image": { "image_id": 941950, "has_title": false },
    "cover_image": { "image_id": 941949, "has_title": false },
    "thumbnail_image": { "image_id": 941948, "has_title": true },
    "tms_series_id": "18154447"
  },
  "show_counts": {
    "airing_count": 1,
    "unwatched_count": 1,
    "protected_count": 1,
    "watched_and_protected_count": 0,
    "failed_count": 0
  },
  "user_info": { "up_next": "/recordings/sports/events/946944" },
  "keep": { "rule": "none", "count": null },
  "guide_path": null
}
```

This will return all data for the Sports `path` requested.

### HTTP Request

`GET http://192.168.1.242:8885/recordings/sports/<object_id>`

### URL Parameters

| Parameter     | Description                    |
| ------------- | ------------------------------ |
| **object_id** | The **object_id** of the Sport |

## Get Specific Sport Event

```shell
curl "http://192.168.1.242:8885/guide/sports/events/946944"
```

> The above command returns JSON structured like this:

```json
{
  "object_id": 946944,
  "path": "/recordings/sports/events/946944",
  "sport_path": "/recordings/sports/946945",
  "snapshot_image": { "image_id": 946946, "has_title": false },
  "airing_details": {
    "datetime": "2020-05-10T00:00Z",
    "duration": 10800,
    "channel_path": "/recordings/channels/634690",
    "channel": {
      "object_id": 634690,
      "path": "/recordings/channels/634690",
      "channel": {
        "call_sign": "WSB-HD",
        "call_sign_src": "WSB-HD",
        "major": 2,
        "minor": 1,
        "network": "ABC",
        "resolution": "hd_720",
        "favourite": false,
        "tms_station_id": "19586",
        "tms_affiliate_id": "10003",
        "source": "ota"
      }
    },
    "show_title": "2016 NBA Finals"
  },
  "video_details": {
    "state": "finished",
    "clean": true,
    "cloud": false,
    "uploading": false,
    "audio": "ac3",
    "size": 13290188800,
    "duration": 10877,
    "width": 1280,
    "height": 720,
    "comskip": { "state": "ready", "error": null },
    "has_snap_grid": true,
    "schedule_offsets": { "start": -15, "end": 64, "deprecated": true },
    "recorded_offsets": { "start": -15, "end": 64 },
    "airing_offsets": { "start": 0, "end": 0, "source": "none" },
    "seek": 15,
    "error": null,
    "warnings": []
  },
  "user_info": { "position": 0, "watched": false, "protected": true },
  "event": {
    "title": "Game 7: Cleveland Cavaliers at Golden State Warriors",
    "description": "The Cleveland Cavaliers become the first team in NBA history to come back from a 3-1 deficit to win the NBA Finals. From June 19, 2016.",
    "season": null,
    "season_type": null,
    "venue": null,
    "teams": [],
    "home_team_id": null,
    "tms_id": "EP034792120002"
  },
  "qualifiers": []
}
```

This will return the data for the specific Active Sports Event requested.

### HTTP Request

`GET http://192.168.1.242:8885/recordings/sports/events/<object_id>`

### URL Parameters

| Parameter     | Description                                 |
| ------------- | ------------------------------------------- |
| **object_id** | The **object_id** of the Active Sport Event |

# Modifying Recordings

Recordings can be updated in 4 ways:

- Toggle the `user_info.watched` field
- Toggle the `user_info.protected` field
- Toggle the `schedule` related fields (unknown, not yet covered)
- Delete a recording

## Patch

```shell
#  curl -X PATCH "http://192.168.1.242:8885/recordings/series/episodes/2228935" -d '{"watched": false}'
```

> This sets **watched** to **false** - the full Airing record is returned witht the field updated (record truncated)

```json
{
  "object_id": 2228935,
  "path": "/recordings/series/episodes/2228935",
  "series_path": "/recordings/series/2170891",
  "season_path": "/recordings/series/seasons/2170892",
  "snapshot_image": { "image_id": 2241603, "has_title": false },
  "airing_details": {
    "datetime": "2022-11-24T02:00Z",
    "show_title": "The Amazing Race",
    ...
  },
  "video_details": {
    "state": "finished",
    "clean": true,
    "cloud": false,
    "uploading": false,
    ...
  },
  "user_info": { "position": 0, "watched": false, "protected": false },
}
```

### HTTP Request

`POST http://192.168.1.242:8885/<PATH>`

### URL Parameters

| Parameter | Description                                                                  |
| --------- | ---------------------------------------------------------------------------- |
| **PATH**  | Any **full path** to a specific Recording, Channel, or Live Show (probably?) |

For example, the `path` field returned from:

- [Recordings - Get Specfic Movie](#get-specific-movie-2)
- [Recordings - Get Specfic Series Episode](#get-specific-series-episode)
- [Recordings - Get Specfic Sport Event](#get-specific-sport-event)
- [Channels - Get Specific Channel](#get-specific-channel)

### POST DATA

A basic JSON object specfiying the field to change passed as PATCH data. Ie:

- `{"watched": false}` or `{"watched": true}`
- `{"protected": false}` or `{"protected": true}`
- `{"position": <NUM>}` - sets the position the recording has been watched until
- `{"schedule": <RULE>}` - no info yet on how to create/set scheduling rules

# Batch Submissions

Batch operations are **much** faster than a bunch of single operations. For example, instead of making **50** requests for the first **50** recordings returned by [Recordings - Get Airings](#get-airings-2), you can simply take those 50 paths and make **1** request to `/batch` to receive all of the same data.

## Batch

```shell
# curl -X POST  "http://192.168.1.242:8885/batch" -d '["/recordings/series/episodes/2267413","/recordings/series/episodes/2265212","/recordings/programs/airings/2261284",  "/recordings/programs/airings/2260157"]'
```

> This will return an Object with a record for each **path** submitted

```json
{
  "/recordings/series/episodes/2267413": {
    "object_id": 2267413,
    "path": "/recordings/series/episodes/2267413",
    "series_path": "/recordings/series/1625726",
    ...
  },
  "/recordings/series/episodes/2265212": null,
  "/recordings/programs/airings/2261284": {
    "path": "/recordings/programs/airings/2261284",
    "object_id": 2261284,
    "program_path": "/recordings/programs/974166",
    ...
  },
  "/recordings/programs/airings/2260157": {
    "path": "/recordings/programs/airings/2260157",
    "object_id": 2260157,
    "program_path": "/recordings/programs/906209",
    ...
  }
}
```

### HTTP Request

`POST http://192.168.1.242:8885/batch`

### POST DATA

Post Data should be a JSON **array** of **path** fields as returned by calls such as [Recordings - Get Airings](#get-airings-2),[Recordings - Get Channels](#get-all-channels-2), other similar calls, or just a list of paths you've collected.

# Watching

Watching essentially doubles as your method for exporting recorded videos/saving live streams or actually viewing streams.

## Watch

```shell
# curl -X POST "http://192.168.1.242/recordings/series/episodes/2267414/watch"
```

> The above command returns JSON structured like this:

```json
{
  "token": "f52ebc70-8867-44c8-8679-10b72a80f1e5",
  "expires": "2023-01-03T17:17:20Z",
  "keepalive": 120,
  "playlist_url": "http://192.168.1.242:80/stream/pl.m3u8?IFLV85YRwrkBP4R0GmYQKw",
  "bif_url_sd": "http://192.168.1.242:80/stream/bif?-50ihDaQWXv5XUfCgjZ3Fw",
  "bif_url_hd": "http://192.168.1.242:80/stream/bif?-50ihDaQWXv5XUfCgjZ3Fw&hd",
  "video_details": { "width": 0, "height": 0 },
  "cutlist": {
    "nc": 4,
    "cuts": [
      { "ts": 0.003, "te": 23.476 },
      { "ts": 1045.267, "te": 1322.972 },
      { "ts": 1889.86, "te": 2164.964 },
      { "ts": 2514.993, "te": 2800.916 }
    ]
  }
}
```

```shell
# ffmpeg -i "http://192.168.1.242:80/stream/pl.m3u8?IFLV85YRwrkBP4R0GmYQKw" show.mp4
```

> Tada! The show _should_ appear in `show.mp4` when that finishes

**Watching** consists of 2 parts:

1. A `POST` request to a specific **watch** URL
2. Grabbing the `playlist_url` returned and passing that to a media player, ffmpeg, or whatever else you want to process the video stream with

### HTTP Request

`POST http://192.168.1.242:8885/<PATH>/watch`

### URL Parameters

| Parameter | Description                                                                  |
| --------- | ---------------------------------------------------------------------------- |
| **PATH**  | Any **full path** to a specific Recording, Channel, or Live Show (probably?) |

For example, the `path` field returned from:

- [Recordings - Get Specfic Movie](#get-specific-movie-2)
- [Recordings - Get Specfic Series Episode](#get-specific-series-episode)
- [Recordings - Get Specfic Sport Event](#get-specific-sport-event)

# 3rd Party Tools

Some projects that use it are:

- [script.tablo](https://github.com/Nuvyyo/script.tablo) - A Kodi add-on (defuct) by Nuvyyo, so kind of a reference implementation
- [Official 3rd Party Tool Forum](https://community.tablotv.com/c/tablo-apps/third-party-apps-plex/17) - Tablo's official 3rd Party Tool forum covering much of what you'll find here

- [tablo-api-js](https://github.com/jessedp/tablo-api-js) - a small wrapper for connecting to a Tablo device via javascript
- [tut](https://github.com/jessedp/tut) - Python 3+ CLI for exporting, deleting recordings
- [Tablo Tools](https://github.com/jessedp/tablo-tools-electron) - An Electron UI project (uses tablo-api-js) for exporting, deleting recordings and more
- [Tablo Ripper](https://github.com/cyclej/TabloRipper/wiki) - Windows app for ripping recordings from your device(s) (no longer under development)
- [SurLaTablo](https://endlessnow.com/ten/SurLaTablo/) - Python 2.7 CLI script for extracting recordings from your device(s)
- [Tablo Exporter](https://jettsoft.com/products.html) - Java UI for exporting recordings from your device(s)
- [APL](http://remedylegacy.com/tablo/) - Java+JFX UI for exporting recordings from your device(s)
