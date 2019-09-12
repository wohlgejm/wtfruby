---
layout: post
comments: true
title: Python gspread token refresh
date: 2016-11-01 20:20:23
categories: python
short_description: Handling token refreshing with `ServiceAccountCredentials`
---

I've been using the Python [gspread](https://github.com/burnash/gspread) library to update a spreadsheet with
information from the Twitter API. After the script was running for a while,
I was getting an error, `401 Token invalid - Invalid token: Crypto credential expired.` To authenticate, I'm
using the [python-oauth2](https://github.com/joestump/python-oauth2) library's `ServiceAccountCredentails`.

To get around this, you can do something like:

```python
import os

import gspread
from oauth2client.service_account import ServiceAccountCredentials

GOOGLE_CREDENTIALS = os.environ['GOOGLE_CREDENTIALS']


class SheetWriter(object):
    SHEET_NAME = 'my_sheet'

    def __init__(self):
        self._gc = None
        self._credentials = None

    @property
    def credentials(self):
        if self._credentials is None:
            scope = ['https://spreadsheets.google.com/feeds']
            creds = ServiceAccountCredentials.from_json_keyfile_name(
                GOOGLE_CREDENTIALS, scope
            )
            self._credentials = creds
        return self._credentials

    @property
    def gc(self):
        if self._gc is None:
            self._gc = gspread.authorize(self.credentials)
        return self._gc

    @property
    def sheet(self):
        if self.credentials.access_token_expired:
            self.gc.login()
        return self.gc.open(self.SHEET_NAME).sheet1
```
