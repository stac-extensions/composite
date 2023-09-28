# Composite Extension Specification

- **Title:** Composite
- **Identifier:** <https://stac-extensions.github.io/composite/v1.0.0/schema.json>
- **Field Name Prefix:** composite
- **Scope:** Item
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @emmanuelmathot

This document explains the Composite Extension to the [SpatioTemporal Asset Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification.
This defines new fields with the usage of muliple extensions to compose
new assets from ([virtual-assets](https://github.com/stac-extensions/virtual-assets)) and their rendering.

The main purpose is to provide users with the information required to create a virtual asset using item or collection assets. The virtual asset can be defined on the collection level if the virtual asset could be produced for a collection-level asset and / or all child item assets.

For example, a virtual asset could be an RGB combination. The virtual asset definition would be a composite of the `processing:expression` and `raster:bands` information. This information would be used to create on the fly visualisation with dynamic tilers.

- Examples:
  - [Landsat-8 example](examples/item-landsat8.json): Shows the basic usage of the extension in a landsat-8 STAC Item
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Fields

The fields in the table below can be used in these parts of STAC documents:

- [ ] Catalogs
- [ ] Collections
- [ ] Item Properties (incl. Summaries in Collections)
- [x] Assets (for both Collections and Items, incl. Item Asset Definitions in Collections)
- [ ] Links

| Field Name            | Type         | Description                                                                                                                                  |
| --------------------- | ------------ | -------------------------------------------------------------------------------------------------------------------------------------------- |
| compose:rescale       | \[\[number]] | double array of delimited Min,Max range. 1 per band                                                                                          |
| compose:colormap_name | string       | Color map identifier that must be applied for a raster band                                                                                  |
| compose:colormap      | object       | [Color map JSON definition](https://developmentseed.org/titiler/advanced/rendering/#custom-colormaps) that must be applied for a raster band |
| compose:color_formula | string       | [Color formula](https://developmentseed.org/titiler/advanced/rendering/#color-formula) that must be applied for a raster band                |

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
"assets": {
  "overview": {
    "roles": [ "overview", "virtual" ],
    "title": "Sentinel-2 Natural Color",
    "type": "image/tiff; application=geotiff",
    "href": "https://api.cogeo.xyz/stac/preview.png?url=https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-sentinel2.json&assets=B04,B03,B02",
    "virtual:hrefs": [ 
      "https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-sentinel2.json#B04",
      "https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-sentinel2.json#B03",
      "https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-sentinel2.json#B02"
    ]
  }
}
```

### Single raster band index processing

The [processing:expression](https://github.com/stac-extensions/processing/pull/2) field can be used in combination with item
to specify a processing formula to be applied to the pixel values in the bands.
The following example describes a virtual asset with the Normalized Difference Vegetation Index (NDVI) computed from 2 other bands.
The [`raster:band`](https://github.com/stac-extensions/raster) field is also used to specify the range of value produced (e.g. for rescaling).

```json
"assets":{
  "NDVI": 
  {
    "virtual:hrefs": [ "#B04", "#B05" ],
    "title": "Normalized Difference Vegetation Index",
    "processing:expression": {
      "format": "rio-calc",
      "expression": "(B05–B04)/(B05+B04)"
    },
    "raster:bands": [[-1,1]]
  }
}
```

## Dynamic tile servers integration

Dynamic tile servers could exploit the information in the `composite` extension to automatically produce RGB tiles
from raster bands or composition using their parameters.

### Titiler

[titiler](https://github.com/developmentseed/titiler) offers a native
[STAC reader](https://github.com/developmentseed/titiler/blob/main/docs/src/endpoints/stac.md).
Some query parameters can be set with the information in the virtual asset from various extensions.

The following table describes the titiler query parameters that could be used and the corresponding extension fields.

Either the client building titiler url can use the information in the virtual asset to build the query parameters or the dynamic tile server could use the information in the virtual asset to build the query parameters by simply specifying the `url` and `assets` query parameters.

| Query key       | field                                  | Description                                                                                                                                                                                                                                                                             |
| --------------- | -------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `url`           | `href` in item self link               | STAC Item URL                                                                                                                                                                                                                                                                           |
| `assets`        | `virtual:hrefs` or asset key           | For a local composition (asset in the same item), the assets key can be retrieved from the virtual:hrefs and joined with comma (e.g. `B04,B03,B02`. Titiler may support to specify directly the virtual asset key and set internally all the parameters according to the current table. |
| `rescale`       | `compose:rescale`                      | Delimited Min,Max bounds defined in `compose:rescale` field                                                                                                                                                                                                                             |
| `expression`    | `processing:expression`                | Band math formula as defined in field `processing:expression` if format is `rio-calc`                                                                                                                                                                                                   |
| `nodata`        | `nodata` in `raster:bands`             | Nodata value defined in `nodata` field of the corresponding `raster:bands` item                                                                                                                                                                                                         |
| `unscale`       | `scale` and `offset` in `raster:bands` | Scale and Offset value defined in `scale` and `offset` fields of the corresponding `raster:bands` item                                                                                                                                                                                  |
| `colormap_name` | `compose:colormap_name`                | Color map name defined in `compose:colormap` field of the `asset`                                                                                                                                                                                                                       |
| `colormap`      | `compose:colormap`                     | Color map JSON definition as defined in `compose:colormap` object of the `asset` (overrides `colormap_name` if present )                                                                                                                                                                |
| `color_formula` | `compose:color_formula`                | Color formula as defined in `compose:color_formula` field of the `asset`                                                                                                                                                                                                                |
| `resampling`    | `compose:resampling`                   | Resampling method to use when reprojecting the raster.                                                                                                                                                                                                                                  |


#### Shortwave Infra-red visual thermal signature example

From the [Sentinel-2 item](https://github.com/stac-extensions/raster/blob/main/examples/item-sentinel2.json):

```json
"assets":{
  "SIR":
  {
    "roles": [ "overview", "virtual" ],
    "title": "Shortwave Infra-red",
    "virtual:hrefs": [ 
      "https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-sentinel2.json#B12",
      "https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-sentinel2.json#B8A",
      "https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-sentinel2.json#B04"
    ],
    "compose:rescale": [[0,5000],[0,7000],[0,9000]]
  }
}
```

| Query key | value                                                             | Example value                                                                                |
| --------- | ----------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| url       | STAC Item URL                                                     | `https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-sentinel2.json` |
| assets    | Assets keys defined in the `bands` objects with field `asset_key` | `B12,B8A,B04`                                                                                |
| rescale   | Delimited Min,Max bounds defined in `compose:rescale` field       | `0,5000,0,7000,0,9000`                                                                       |

URL: `https://api.cogeo.xyz/stac/crop/14.869,37.682,15.113,37.862/256x256.png?url=https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-sentinel2.json&assets=B12,B8A,B04&resampling_method=average&rescale=0,5000,0,7000,0,9000&return_mask=true`

**Result**: Lava thermal signature of Mount Etna eruption (February 2021)

![etna](images/etna.png)

#### Normalized Difference Vegetation Index (NDVI) example

From the [Landsat-8 example](examples/item-landsat8.json) \[[article](https://www.usgs.gov/core-science-systems/nli/landsat/landsat-normalized-difference-vegetation-index?qt-science_support_page_related_con=0#qt-science_support_page_related_con)]:

```json
"**assets**": {
  "NDVI": 
  {
    "href": [ "#B4", "#B5" ],
    "title": "Normalized Difference Vegetation Index",
    "processing:expression": {
      "format": "rio-calc",
      "expression": "(B05–B04)/(B05+B04)"
    },
    "compose:color_map" : "ylgn",
    "raster:bands": [[-1,1]]
  }
}
```

| Query key  | value                                                         | Example value                                                                               |
| ---------- | ------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| url        | STAC Item URL                                                 | `https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-landsat8.json` |
| rescale    | Delimited Min,Max bounds defined in `compose:rescale` field   | `-1,1`                                                                                      |
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
