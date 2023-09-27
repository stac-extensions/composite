# Composite Extension Specification

- **Title:** Composite
- **Identifier:** <https://stac-extensions.github.io/composite/v1.0.0/schema.json>
- **Field Name Prefix:** composite
- **Scope:** Item, Collection
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @emmanuelmathot

This document explains the Composite Extension to the [SpatioTemporal Asset Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification.
This meta-extension does not define any new fields but the usage of muliple extensions to compose
new assets as ([virtual-assets](https://github.com/stac-extensions/virtual-assets)).

The main purpose is to provide users with the processing information required to create a virtual asset using item or collection assets. The virtual asset can be defined on the collection level if the virtual asset could be produced for a collection-level asset and / or all child item assets.

For example, a virtual asset could be an RGB combination. The virtual asset definition would be a composite of the `processing:expression` and `raster:bands` information. This information would be used to create on the fly visualisation with dynamic tilers.

- Examples:
  - [Landsat-8 example](examples/item-landsat8.json): Shows the basic usage of the extension in a landsat-8 STAC Item
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Raster Composition using `virtual:assets`

This use case describes how to specify possible raster bands composition. 
This requires the usage of the [virtual-assets](https://github.com/stac-extensions/virtual-assets) extensions that allows to specify assets
composition and repositioning of one or more [raster bands](https://github.com/stac-extensions/raster) in assets.
At least one virtual asset is required to make a raster composite.
It cross references in the desired order the assets containing the bands for the composition.

### Simple RGB raster composition

A very simple case would be the composition of a RGB natural color image of a
[Sentinel-2 item](https://github.com/stac-extensions/raster/blob/main/examples/item-sentinel2.json).

```json
"virtual:assets":{
  "overview":
  {
    "title": "Sentinel-2 Natural Color",
    "href": [ 
      "https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-sentinel2.json#B04",
      "https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-sentinel2.json#B03",
      "https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-sentinel2.json#B02"
    ]
  }
}
```

### Single raster band index processing

The [processing:expression](https://github.com/stac-extensions/processing/pull/2) field can be used in `virtual:assets` item
to specify a processing formula to be applied to the pixel values in the bands.
The following example describes a virtual asset with the Normalized Difference Vegetation Index (NDVI) computed from 2 other bands.
The [`raster:band`](https://github.com/stac-extensions/raster) field is also used to specify the range of value produced.

```json
"virtual:assets":{
  "NDVI": 
  {
    "href": [ "#B04", "#B05" ],
    "title": "Normalized Difference Vegetation Index",
    "processing:expression": {
      "format": "rio-calc",
      "expression": "(B05–B04)/(B05+B04)"
    },
    "raster:bands": [
      { 
        "statistics": {
          "minimum": -1,
          "maximum": 1
        }
      }
    ]
  }
}
```

## Dynamic tile servers integration

Dynamic tile servers could exploit the information in the `composite` extension to automatically produce RGB tiles
from raster bands or composition using their parameters.

### Titiler

[titiler](https://github.com/developmentseed/titiler) offers a native
[STAC reader](https://github.com/developmentseed/titiler/blob/master/docs/endpoints/stac.md).
Some query parameters could be set with the information from raster extension.

#### Shortwave Infra-red visual thermal signature example

From the [Sentinel-2 item](https://github.com/stac-extensions/raster/blob/main/examples/item-sentinel2.json):

```json
"virtual:assets":{
  "SIR":
  {
    "title": "Shortwave Infra-red",
    "href": [ 
      "https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-sentinel2.json#B12",
      "https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-sentinel2.json#B8A",
      "https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-sentinel2.json#B04"
    ],
    "raster:bands": [
      {
        "statistics": {
            "minimum": 0,
            "maximum": 5000
        }
      },
      {
        "statistics": {
            "minimum": 0,
            "maximum": 7000
        }
      },
      {
        "statistics": {
            "minimum": 0,
            "maximum": 9000
        }
      }
    ]
  }
}
```

| Query key | value                                                                        | Example value                                                                                |
| --------- | ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| url       | STAC Item URL                                                                | `https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-sentinel2.json` |
| assets    | Assets keys defined in the `bands` objects with field `asset_key`            | `B12,B8A,B04`                                                                                |  |
| rescale   | Delimited Min,Max bounds defined in `statistics` object of the `raster:band` | `0,5000,0,7000,0,9000`                                                                                    |

URL: `https://api.cogeo.xyz/stac/crop/14.869,37.682,15.113,37.862/256x256.png?url=https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-sentinel2.json&assets=B12,B8A,B04&resampling_method=average&rescale=0,5000,0,7000,0,9000&return_mask=true`

**Result**: Lava thermal signature of Mount Etna eruption (February 2021)

![etna](images/etna.png)

#### Normalized Difference Vegetation Index (NDVI) example

From the [Landsat-8 example](examples/item-landsat8.json) \[[article](https://www.usgs.gov/core-science-systems/nli/landsat/landsat-normalized-difference-vegetation-index?qt-science_support_page_related_con=0#qt-science_support_page_related_con)]:

```json
"virtual:assets": {
  "NDVI": 
  {
    "href": [ "#B4", "#B5" ],
    "title": "Normalized Difference Vegetation Index",
    "processing:expression": {
      "format": "rio-calc",
      "expression": "(B05–B04)/(B05+B04)"
    },
    "raster:bands": [
      { 
        "statistics": {
          "minimum": -1,
          "maximum": 1
        }
      }
    ]
  }
}
```

| Query key  | value                                                     | Example value                                                                               |
| ---------- | --------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| url        | STAC Item URL                                             | `https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-landsat8.json` |  |
| rescale    | Delimited Min,Max bounds defined in `statistics` object of the `raster:band`        | `-1,1`                                                                                      |
| expression | Band math formula as defined in field `processing:expression` | `(B5–B4)/(B5+B4)`                                                                           |

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
