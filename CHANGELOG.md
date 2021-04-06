# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.1] - Work in progress

### Fixed

  - Regression: the ids of form elements was accidentally removed. They’re now back (accessibility).
  - Closing fieldset tag.

### Todo

  - [] #1 Handle CoinGecko network error gracefully.
  - [] #2 Add progress indication for CoinGecko API request. WIP: Needs proper positioning. Depends on #3
  - [] #3 Stop interface from jumping between initial state and when exchange rates load and QRCode appears.

### Improved

  - Select box now displays similarly across browsers.
  - Shows in-page error message if wallet address is missing (development-time initialisation error).
  - Visual hierarchy and spacing/grouping.

## [1.0.0] - 2021-04-06

Initial release.
