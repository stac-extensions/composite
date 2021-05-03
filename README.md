# Composite Extension Specification

- **Title:** Composite
- **Identifier:** <https://stac-extensions.github.io/composite/v1.0.0/schema.json>
- **Field Name Prefix:** composite
- **Scope:** Item, Collection
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @emmanuelmathot

This document explains the Template Extension to the [SpatioTemporal Asset Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification.
It is helping the user to have some rendering hints of the item using one or
more raster assets (RGB combination, simple band value processing) and to create on the fly visualisation with dynamic tilers.

- Examples:
  - [Item example](examples/item.json): Shows the basic usage of the extension in a STAC Item
  - [Collection example](examples/collection.json): Shows the basic usage of the extension in a STAC Collection
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Raster Composition using `virtual:assets`

This extension describes how to specify possible raster bands composition. This requires the usage of the [virtual-assets](https://github.com/stac-extensions/virtual-assets) extensions that allows to specify assets composition and repositioning and some fields of the [processing](https://github.com/stac-extensions/processing) extension).

At least one virtual asset is required to make a raster composite.

### Virtual Asset fields

Raster composites defines the following fields in `virtual:assets` items.

| Field Name               | Type      | Description                                                            |
| ------------------------ | --------- | ---------------------------------------------------------------------- |
| raster:range             | \[number] | range of valid pixels values in the composition                        |
| raster:resampling_method | string    | Resampling method, one of `nearest`, `average`, `bilinear` or `cubic`. |
| processing:expression    | string    | [https://github.com/stac-extensions/processing/pull/2]                 |

## Dynamic tile servers integration

Dynamic tile servers could exploit the information in the raster extension to automatically produce RGB
from raster bands or composition using their parameters.

### Titiler

[titiler](https://github.com/developmentseed/titiler) offers a native
[STAC reader](https://github.com/developmentseed/titiler/blob/master/docs/endpoints/stac.md).
Some query parameters could be set with the information from raster extension.

#### Shortwave Infra-red visual thermal signature example

From the [Sentinel-2 example](examples/item-sentinel2.json):

```json
"virtual:assets":{
  "SIR":
  {
    "title": "Shortwave Infra-red",
    "raster:range": [0, 10000],
    "href": [ "#B12", "#B8A", "#B04"]
  }
}
```

| Query key | value                                                             | Example value                                                                                |
| --------- | ----------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| url       | STAC Item URL                                                     | `https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-sentinel2.json` |
| assets    | Assets keys defined in the `bands` objects with field `asset_key` | `B12,B8A,B04`                                                                                |  |
| rescale   | Delimited Min,Max bounds defined in field `range`                 | `0,10000`                                                                                    |

URL: `https://api.cogeo.xyz/stac/crop/14.869,37.682,15.113,37.862/256x256.png?url=https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-sentinel2.json&assets=B12,B8A,B04&resampling_method=average&rescale=0,10000&return_mask=true`

**Result**: Lava thermal signature of Mount Etna eruption (February 2021)

![etna](images/etna.png)

#### Normalized Difference Vegetation Index (NDVI) example

From the [Landsat-8 example](examples/item-landsat8.json) \[[article](https://www.usgs.gov/core-science-systems/nli/landsat/landsat-normalized-difference-vegetation-index?qt-science_support_page_related_con=0#qt-science_support_page_related_con)]:

```json
"virtual:assets":{
  "NDVI": 
  {
    "href": [ "#B04", "#B05" ],
    "title": "Normalized Difference Vegetation Index",
    "raster:range": [-1, 1],
    "processing:expression": "(B05–B04)/(B05+B04)",
  }
}
```

| Query key  | value                                                     | Example value                                                                               |
| ---------- | --------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| url        | STAC Item URL                                             | `https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-landsat8.json` |  |
| rescale    | Delimited Min,Max bounds defined in field `range`         | `-1,1`                                                                                      |
| expression | Band math formula as defined in field `band_math_formula` | `(B5–B4)/(B5+B4)`                                                                           |
| color_map  | Color map defined in field `color_map`                    | `ylgn`                                                                                      |

URL:

`https://api.cogeo.xyz/stac/preview.png?url=https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-landsat8.json&expression=(B5–B4)/(B5+B4)&max_size=512&width=512&resampling_method=average&rescale=-1,1&color_map=ylgn&return_mask=true`

Result:  Landsat Surface Reflectance Normalized Difference Vegetation Index (NDVI) path 44 row 33.

![sacramento](https://api.cogeo.xyz/stac/preview.png?url=https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-landsat8.json&expression=(B5–B4)/(B5+B4)&max_size=512&width=512&resampling_method=average&rescale=-1,1&color_map=ylgn&return_mask=true)

## Contributing

All contributions are subject to the
[STAC Specification Code of Conduct](https://github.com/radiantearth/stac-spec/blob/master/CODE_OF_CONDUCT.md).
For contributions, please follow the
[STAC specification contributing guide](https://github.com/radiantearth/stac-spec/blob/master/CONTRIBUTING.md) Instructions
for running tests are copied here for convenience.

### Running tests

The same checks that run as checks on PR's are part of the repository and can be run locally to verify that changes are valid. 
To run tests locally, you'll need `npm`, which is a standard part of any [node.js installation](https://nodejs.org/en/download/).

First you'll need to install everything with npm once. Just navigate to the root of this repository and on 
your command line run:
```bash
npm install
```

Then to check markdown formatting and test the examples against the JSON schema, you can run:
```bash
npm test
```

This will spit out the same texts that you see online, and you can then go and fix your markdown or examples.

If the tests reveal formatting problems with the examples, you can fix them with:
```bash
npm run format-examples
```

## Relation types

The following types should be used as applicable `rel` types in the
[Link Object](https://github.com/radiantearth/stac-spec/tree/master/item-spec/item-spec.md#link-object).

| Type                | Description |
| ------------------- | ----------- |
| fancy-rel-type      | This link points to a fancy resource. |

## Contributing

All contributions are subject to the
[STAC Specification Code of Conduct](https://github.com/radiantearth/stac-spec/blob/master/CODE_OF_CONDUCT.md).
For contributions, please follow the
[STAC specification contributing guide](https://github.com/radiantearth/stac-spec/blob/master/CONTRIBUTING.md) Instructions
for running tests are copied here for convenience.

### Running tests

The same checks that run as checks on PR's are part of the repository and can be run locally to verify that changes are valid. 
To run tests locally, you'll need `npm`, which is a standard part of any [node.js installation](https://nodejs.org/en/download/).

First you'll need to install everything with npm once. Just navigate to the root of this repository and on 
your command line run:
```bash
npm install
```

Then to check markdown formatting and test the examples against the JSON schema, you can run:
```bash
npm test
```

This will spit out the same texts that you see online, and you can then go and fix your markdown or examples.

If the tests reveal formatting problems with the examples, you can fix them with:
```bash
npm run format-examples
```
