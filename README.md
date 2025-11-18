# tap-googleads

`tap-googleads` is a Singer tap for GoogleAds.

This fork of `tap-googleads` will sync your GoogleAds data for the specific accounts you authorize using a new, simplified config.

Built with the [Meltano Tap SDK](https://sdk.meltano.com) for Singer Taps.

## Installation

To install and use this tap with Meltano:

```bash
meltano add extractor tap-googleads
```

To use standalone, you can use the following:

```bash
pip install https://github.com/Matatika/tap-googleads.git
```

## Configuration

### Supported Config Options

A full list of supported settings and capabilities for this tap is available by running:

```bash
tap-googleads --about
```

### Authentication
How to get these settings can be found in the following Google Ads documentation:

- https://developers.google.com/adwords/api/docs/guides/authentication
- https://developers.google.com/google-ads/api/docs/first-call/dev-token

**Required for all configs:**
- `client_id`
- `client_secret`
- `refresh_token`
- `developer_token`

#### Account Selection

**You have two options to select which Google Ads accounts to sync:**

1. **Federated (All Accessible Accounts):**
   Provide only `login_customer_id` to sync *all* accounts you can access via your Google MCC/manager account.
   ```json
   {
     "developer_token": "<DEVELOPER_TOKEN>",
     "login_customer_id": "<LOGIN_CUSTOMER_ID>"
   }
   ```

2. **Explicit List (`locations`):**
   Provide a `locations` array, with each object specifying an `id` (the account/customer ID). `name` is allowed but ignored.
   - *Single* account:
     ```json
     {
       "developer_token": "<DEVELOPER_TOKEN>",
       "locations": [
         { "id": "<GOOGLE_CUSTOMER_ID_1>", "name": "<OPTIONAL_NAME>" }
       ]
     }
     ```
   - *Multiple* accounts:
     ```json
     {
       "developer_token": "<DEVELOPER_TOKEN>",
       "locations": [
         { "id": "<GOOGLE_CUSTOMER_ID_1>", "name": "<OPTIONAL_NAME1>" },
         { "id": "<GOOGLE_CUSTOMER_ID_2>", "name": "<OPTIONAL_NAME2>" }
       ]
     }
     ```
- The `"locations"` option takes a list of accounts; only the `"id"` property is used by the tap.
- If `"locations"` is not provided, specifying `"login_customer_id"` (on its own) will sync all accessible accounts via that MCC.

**Additional optional fields:**
- `refresh_proxy_url`, `refresh_proxy_url_auth` (for Proxy OAuth, see below)
- `start_date`/`end_date`: Date range for data extraction (default: last 90 days)
- `enable_click_view_report_stream`: Enables extra stream for click view reporting (default: false, may require Google Ads permissions)
- `user_agent`: Custom HTTP User-Agent

### Proxy OAuth Credentials

You may use a Proxy OAuth server for authentication instead of providing `client_id`/`client_secret` directly:

```json
{
  "developer_token": "YOUR_GOOGLE_DEVELOPER_TOKEN",
  "refresh_token": "YOUR_OAUTH_REFRESH_TOKEN",
  "refresh_proxy_url": "https://your-oauth-proxy.example.com/refresh",
  "refresh_proxy_url_auth": "Bearer YOUR_PROXY_TOKEN",
  "locations": [
    { "id": "<GOOGLE_CUSTOMER_ID_1>" }
  ]
}
```

If you use Proxy OAuth, do not include `client_id` and `client_secret`. The usage of `locations` and `login_customer_id` remains exactly as described above.

---

## Usage

You can easily run `tap-googleads` by itself or in a pipeline using [Meltano](https://meltano.com/).

### Executing the Tap Directly

```bash
tap-googleads --version
tap-googleads --help
tap-googleads --config CONFIG --discover > ./catalog.json
```

## Developer Resources

### Initialize your Development Environment

```bash
pipx install poetry
poetry install
```

### Create and Run Tests

Create tests within the `tap_googleads/tests` subfolder and then run:

```bash
poetry run pytest
```

You can also test the `tap-googleads` CLI interface directly using `poetry run`:

```bash
poetry run tap-googleads --help
```

### Testing with [Meltano](https://www.meltano.com)

_**Note:** This tap will work in any Singer environment and does not require Meltano.
Examples here are for convenience and to streamline end-to-end orchestration scenarios._

Your project comes with a custom `meltano.yml` project file already created. Open the `meltano.yml` and follow any _"TODO"_ items listed in the file.

Next, install Meltano (if you haven't already) and any needed plugins:

```bash
# Install meltano
pipx install meltano
# Initialize meltano within this directory
cd tap-googleads
meltano install
```

Now you can test and orchestrate using Meltano:

```bash
# Test invocation:
meltano invoke tap-googleads --version
# OR run a test `elt` pipeline:
meltano elt tap-googleads target-jsonl
```

### SDK Dev Guide

See the [dev guide](https://sdk.meltano.com/en/latest/dev_guide.html) for more instructions on how to use the SDK to develop your own taps and targets.
