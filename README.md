# tap-salesforce

[![PyPI version](https://badge.fury.io/py/tap-mysql.svg)](https://badge.fury.io/py/tap-salesforce)
[![CircleCI Build Status](https://circleci.com/gh/singer-io/tap-salesforce.png)](https://circleci.com/gh/singer-io/tap-salesforce.png)


[Singer](https://www.singer.io/) tap that extracts data from a [Salesforce](https://www.salesforce.com/) Account and produces JSON-formatted data following the [Singer spec](https://github.com/singer-io/getting-started/blob/master/SPEC.md).

This is a forked version of [tap-salesforce (v1.4.24)](https://github.com/singer-io/tap-salesforce) that maintained by the Meltano team.

Main differences from the original version:

- Support for `username/password/security_token` authentication
- Support for JWT authentication using connected apps
- Support for concurrent execution (8 threads by default) when accessing different API endpoints to speed up the extraction process

# Quickstart

## Install the tap

This version of `tap-salesforce` is not available on PyPi, so you have to fetch it directly from the Meltano maintained project:

```bash
python3 -m venv venv
source venv/bin/activate
pip install git+https://gitlab.com/meltano/tap-salesforce.git
```

## Create a Config file

**Required**
```
{
  "api_type": "BULK",
  "select_fields_by_default": true,
}
```

**Required for OAuth based authentication**
```
{
  "client_id": "secret_client_id",
  "client_secret": "secret_client_secret",
  "refresh_token": "abc123",
}
```

**Required for username/password based authentication**
```
{
  "username": "Account Email",
  "password": "Account Password",
  "security_token": "Security Token",
}
```

**Required for JWT based authentication**
```
{
  "username": "Account Email",
  "consumer_key": "Connected App Consumer Key",
  "privatekey": "Private Key (PEM format)",
}
```

**Optional**
```
{
  "start_date": "2017-11-02T00:00:00Z",
  "state_message_threshold": 1000,
  "max_workers": 8
}
```

The `client_id` and `client_secret` keys are your OAuth Salesforce App secrets. The `refresh_token` is a secret created during the OAuth flow. For more info on the Salesforce OAuth flow, visit the [Salesforce documentation](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/intro_understanding_web_server_oauth_flow.htm).

For JWT authentication, the `consumer_key` is the Consumer Key from your Salesforce Connected App, and the `privatekey` should be the private key in PEM format corresponding to the certificate uploaded to your Connected App. JWT authentication is ideal for server-to-server integrations as it doesn't require user interaction.

**Private Key Format Options:**
The `privatekey` field accepts several formats:
- **PEM format** (direct): Include the key with proper newlines (works in AWS Parameter Store)
- **Base64 encoded**: Encode the entire PEM key as base64 (recommended for JSON config files)
- **Escaped newlines**: Replace actual newlines with `\n` (alternative for JSON config files)

Example for JSON config (base64 encoded):
```json
{
  "privatekey": "LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0t..."
}
```

For more details on JWT authentication with Salesforce, see the [Salesforce JWT documentation](https://help.salesforce.com/s/articleView?id=sf.remoteaccess_oauth_jwt_flow.htm).

The `start_date` is used by the tap as a bound on SOQL queries when searching for records.  This should be an [RFC3339](https://www.ietf.org/rfc/rfc3339.txt) formatted date-time, like "2018-01-08T00:00:00Z". For more details, see the [Singer best practices for dates](https://github.com/singer-io/getting-started/blob/master/BEST_PRACTICES.md#dates).

The `api_type` is used to switch the behavior of the tap between using Salesforce's "REST" and "BULK" APIs. When new fields are discovered in Salesforce objects, the `select_fields_by_default` key describes whether or not the tap will select those fields by default.

The `state_message_threshold` is used to throttle how often STATE messages are generated when the tap is using the "REST" API. This is a balance between not slowing down execution due to too many STATE messages produced and how many records must be fetched again if a tap fails unexpectedly. Defaults to 1000 (generate a STATE message every 1000 records).

The `max_workers` value is used to set the maximum number of threads used in order to concurrently extract data for streams. Defaults to 8 (extract data for 8 streams in paralel).

## Run Discovery

To run discovery mode, execute the tap with the config file.

```
tap-salesforce --config config.json --discover > properties.json
```

## Sync Data

To sync data, select fields in the `properties.json` output and run the tap.

```
tap-salesforce --config config.json --properties properties.json [--state state.json]
```

Copyright &copy; 2017 Stitch
