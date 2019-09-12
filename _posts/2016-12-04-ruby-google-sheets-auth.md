---
layout: post
comments: true
title: Using ENVs for Google Drive API auth
date: 2016-12-04 15:20:30
categories: ruby
short_description: How to use ENV vars for Google API authorization.
---

The [docs](https://developers.google.com/sheets/quickstart/ruby) for the Google Ruby API could be much clearer.
Most of the complexity is around authorization. If you're using a service account (not tied to any particular user)
for authorization, Google provides you with a JSON file that contains your credentials.

Since you don't want to check these credentials into your VCS, you're only option is to upload them to s3
or another cloud store and pull them when needed.

There is another way, which was documented and merged in [here](https://github.com/google/google-api-ruby-client/issues/370),
but took me a while to find and get to work.

You can set up environment three environment variables:

```
GOOGLE_ACCOUNT_TYPE="service"
GOOGLE_CLIENT_EMAIL="wohlgejm@gmail.com"
GOOGLE_PRIVATE_KEY="BEGIN.."
```

And use these for sheets authorization like so:

```ruby
auth = Google::Auth::ServiceAccountCredentials.make_creds
auth.scope = Google::Apis::SheetsV4::AUTH_SPREADSHEETS_READONLY
auth.fetch_access_token
sheets = Google::Apis::SheetsV4::SheetsService.new
sheets.authorization = auth
sheets.get_spreadsheet(SHEET_ID)
```
