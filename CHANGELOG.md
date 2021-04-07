# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.1] - Work in progress

### Fixes

  - Regression: the ids of form elements was accidentally removed. They’re now back (accessibility).
  - Closing fieldset tag.

### General improvements

  - Interface no longer jumps between initial state and ready state (when exchange rates have loaded and the QRCode appears).
  - Add animated spinner to mark the initial exchange rate load.
  - Handles exchange rate load errors gracefully.
  - Select box now displays similarly across browsers.
  - Shows in-page error message if wallet address is missing (development-time initialisation error).
  - Improved visual hierarchy, spacing, and grouping.

### Accessibility improvements

  - Does not show loading spinner animation if _prefers reduced motion_ setting is in effect.

## [1.0.0] - 2021-04-06

Initial release.
