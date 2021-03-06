# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.4] - 2021-04-08

### Fixes

  - Inline QRious component otherwise Vite/SvelteKit fail with import error when component is installed from npm.

### Changes

  - Update license to GPL as QRious component is under GPL.

## [1.0.3] - 2021-04-08

### Fixes

  - Move svelte, etc., back to `devDependencies` in an attempt to get the component to work properly in SvelteKit when installed from npm.

## [1.0.2] - 2021-04-08

### Fixes

  - Mark dependencies as `dependencies` instead of `devDependencies` in package file so component can be used in SvelteKit projects.

## [1.0.1] - 2021-04-08

### Fixes

  - Regression: the ids of form elements was accidentally removed. They’re now back (accessibility).
  - Closing fieldset tag.

### General improvements

  - Interface no longer jumps between initial state and ready state (when exchange rates have loaded and the QRCode appears).
  - Add animated spinner to mark the initial exchange rate load.
  - QRCode and payment link now fade in gracefully.
  - Handles exchange rate load errors gracefully.
  - Select box now displays similarly across browsers.
  - Shows in-page error message if wallet address is missing (development-time initialisation error).
  - Improved visual hierarchy, spacing, and grouping.

### Accessibility improvements

  - Does not show loading spinner animation if _prefers reduced motion_ setting is in effect.

## [1.0.0] - 2021-04-06

Initial release.
