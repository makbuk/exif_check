# EXIF check

A Drupal 7 module that checks whether uploaded image(s) contain GPS coordinates
in their EXIF metadata and adds a new marker to a Leaflet map based on those
coordinates.

## Overview

When a node form for the `addproblem` content type is submitted, this module
validates the uploaded photos in the `field_photo` field. For each uploaded
image it reads the EXIF data and extracts the GPS coordinates. If at least one
photo contains GPS coordinates, the geographic center of all coordinates is
calculated and passed to a Leaflet map as a marker. If none of the uploaded
photos contain GPS coordinates, an error message is shown asking the user to
upload another photo.

## Requirements

- Drupal 7.x
- PHP with the [EXIF extension](https://www.php.net/manual/en/book.exif.php)
  enabled (`exif_read_data`)
- A node type named `addproblem` with an image field named `field_photo`
- A Leaflet map and a companion `exif_check.js` file that consumes the
  `Drupal.settings.exif_check.marker` setting to place the marker

## How it works

1. `exif_check_form_addproblem_node_form_alter()` attaches a custom element
   validator (`exif_check_validate_function`) to the `field_photo` field.
2. `exif_check_validate_function()` iterates over every uploaded file:
   - Resolves the file URL via `exif_check_get_file_url()` (`file_load` +
     `file_create_url`).
   - Checks for GPS data with `check_gps()`, which inspects the
     `GPSLatitude` / `GPSLongitude` EXIF tags.
   - Extracts decimal latitude/longitude with `get_latlon()` and the helper
     functions `getGps()` / `gps2Num()`.
3. If coordinates were collected, `GetCenterFromDegrees()` computes the central
   point of all coordinate pairs.
4. `exif_check_add_marker()` loads `exif_check.js` and adds the marker as a
   Drupal JavaScript setting so the Leaflet map can render it.

## Installation

1. Copy this module's directory into your site's modules folder
   (e.g. `sites/all/modules/exif_check`).
2. Enable the module on the **Modules** administration page or with Drush:

   ```sh
   drush en exif_check
   ```

3. Provide an `exif_check.js` file in the module directory that reads
   `Drupal.settings.exif_check.marker` and places the marker on your Leaflet
   map.

## Notes / Credits

- GPS EXIF extraction is based on
  [this Stack Overflow answer](http://stackoverflow.com/questions/2526304/php-extract-gps-exif-data).
- Center-point calculation is based on
  [this Stack Overflow answer](http://stackoverflow.com/questions/6671183/calculate-the-center-point-of-multiple-latitude-longitude-coordinate-pairs).
