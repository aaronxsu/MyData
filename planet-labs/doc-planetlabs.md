# Imagery Quickstart

Planet's imagery comes in different 'Item Types' and 'Asset Types'. An item is a single picture taken by a satellite at a certain time. Items have multiple asset types including the image in different formats, and supporting metadata files.

1. If performing scientific analysis and use the NIR band:
    * Select the `analytic` asset, which is Scaled Top of Atmosphere Radiance calibrated, per-sensor calibrated, and 16-bit.
    * Choose Multi-spectral Item Types only (Planet Scope Scene 4-Band - `PSScene4Band`, Planet Scope OrthoTile - `PSOrthoTile`, Rapideye OrthoTile - `REOrthoTile`, Landsat8 Scenes - `Landsat8L1G`, Copernicus Sentinel-2 Scenes - `Sentinel2L1C`). Then use only the    `analytic`, or `basic_analytic` asset types.

2. If doing your own orthorectification:
    * Planet uses the term `basic` to denote unorthorectified. The equivalent asset in the Data API is `basic_analytic`. Apart from orthorectification, `basic` assets are the same as the `analytic` asset.

4. Looking for imagery before Mid-2016:
    * Only the Planet Scope 3 Band (`PSScene3Band`, but it does not have NIR band) and RapidEye OrthoTile (`REOrthoTile`) Item Types contain imagery before mid-2016.

5. Want Top of Atmosphere reflectance for your analysis:
    * You can find the Reflectance coefficients in the XML metadata asset, named `radiometricScaleFactor`, and `reflectanceCoefficient`. Multiply pixel values in the GeoTIFF by these coefficients.

6. Band order:
    * For any multispectral asset, the order is BGRN. For any 3 band asset, it is RGB. More specifications:
        * `PSScene4Band`: [Link1](https://www.planet.com/docs/reference/data-api/items-assets/psscene4band/), [Link2](https://www.planet.com/products/satellite-imagery/planetscope-analytic-ortho-scene/)
        * `PSOrthoTile`: [Link1](https://www.planet.com/docs/reference/data-api/items-assets/psorthotile/), [Link2](https://www.planet.com/products/satellite-imagery/planetscope-analytic-ortho-tile/)
        * `REOrthoTile`: [Link1](https://www.planet.com/docs/reference/data-api/items-assets/reorthotile/), [Link2](https://www.planet.com/products/satellite-imagery/rapid-eye-visual-ortho-tile/)
        * [`Landsat8L1G`](https://www.planet.com/docs/reference/data-api/items-assets/landsat8l1g/)
        * [`Sentinel2L1C`](https://www.planet.com/docs/reference/data-api/items-assets/sentinel2l1c/)

7. Other asset types:
    * `analytic_dn`: non-radiometrically calibrated imagery suitable for analytic applications
    * `udm`: image mask file which provides information on areas of unusable data within an image (e.g. cloud and non-imaged areas)

# JavaScript Client
[API Documentation](http://planetlabs.github.io/client/api/)

# Data API

## Filter Items

### API End Point
`https://api.planet.com/data/v1/quick-search`

### Header
`Content-Type: application/json`

`Authorization: Base <your api key>`

### Request Body
Use a `POST` request with the following as request body:

```JavaScript
{
    //-----------------------
    // Datasource Filter
    //-----------------------
    'item_types': ['PSScene4Band'],     // an array of item type strings, this filters imagery sources
    'filter': {
        'type': 'AndFilter',     // this can also be 'OrFilter' or 'NotFilter'
        'config': [     // an array of filter objects
            //-----------------------
            // Date Range Filter
            //-----------------------
            {
                'type': 'DateRangeFilter',     // match dates that fall within a range.
                'field_name': 'acquired',     // three date fields: 'acquired',  'published', and 'updated'
                'config': {
                    'gte': '2016-09-01T00:00:00Z',
                    'lte': '2016-10-01T00:00:00Z'     //can also be 'gt' or 'lt'
                }
            },
            //-----------------------
            // Bounding Box Filter
            //-----------------------
            {
                'type': 'GeometryFilter',     // match using GeoJSON geometry.
                'field_name': 'geometry',
                'config': {     // the GeoJSON geometry
                    'type': 'Polygon',
                    'coordinates': [
                        [
	                           // an array or coordinates arrays
	                    ]
                    ]
                }
            },
            //-----------------------
            // Cloud Cover Filter
            //-----------------------
            {
                'type': 'RangeFilter',
                'field_name': 'cloud_cover',
                'config': {
                    'gte': 0,
                    'lt': 15     // can also be 'gt' or 'lt'
                }
            },
            //-----------------------
            // Sun Azimuth Filter
            //-----------------------
            {
                'type': 'RangeFilter',
                'field_name': 'sun_azimuth',
                'config': {
                    'gte': 0,
                    'lt': 180     // can also be 'gt' or 'lt'
                }
            },
            //-----------------------
            // Sun Elevation Filter
            //-----------------------
            {
                'type': 'RangeFilter',
                'field_name': 'sun_elevation',
                'config': {
                    'gte': 0,
                    'lt': 80     // can also be 'gt' or 'lt'
                }
            },
            //-----------------------
            // Permission Filter
            //-----------------------
            {   // allows for filtering data according to
                // the active user's permissions to that data.
                'type': 'PermissionFilter',
                'config': [
                    'assets:download',     // One or more assets types may be activated and downloaded.
                    'assets.visual:download',     // Visual assets may be activated and downloaded.
                    'assets.analytic:download']     // Analytic assets may be activated and downloaded.
            }
            //-----------------------
            // Some other filter types:
            //     'NumberInFilter'
            //     'StringInFilter'
            // Both config should be
            // an array of numbers or strings   
            //-----------------------
        ]
    }
}
```

### Response
```JavaScript
{
    '_links': {
       '_first': , // link for GET request for responses on the first page
       '_next': ,  // link for GET request for responses on the next page
       '_self':    // link for GET request for responses on the current page
   },
   'features': [
       {
            '_links': {
                '_self': 'https://api.planet.com/data/v1/item-types/PSScene4Band/items/20160930_180021_0e2f',
                // link for GET request to activate and download the asset
                'assets': 'https://api.planet.com/data/v1/item-types/PSScene4Band/items/20160930_180021_0e2f/assets/',
                // link for GET request to acquire the thumbnail of this imagery
                'thumbnail': 'https://api.planet.com/data/v1/item-types/PSScene4Band/items/20160930_180021_0e2f/thumb'
            },
            '_permissions': [
                'assets.analytic:download',
                'assets.analytic_xml:download',
                'assets.udm:download'
            ],
            'geometry': {
                'coordinates': [
                    [
                        // an array of coordinates
                    ]
                ],
                'type': 'Polygon'
            },
            'id': '20160930_180021_0e2f',
            'properties': {
                'acquired': '2016-09-30T18:00:21.252691Z',
                'anomalous_pixels': 0,
                'cloud_cover': 0.003,
                'columns': 8584,
                'epsg_code': 32610,
                'gsd': 4.054994209566383,
                'instrument': 'PS2',
                'item_type': 'PSScene4Band',
                'origin_x': 661518,
                'origin_y': 4016463,
                'pixel_resolution': 3,
                'provider': 'planetscope',
                'published': '2016-12-24T04:01:26Z',
                'quality_category': 'standard',
                'rows': 2110,
                'satellite_id': '0e2f',
                'strip_id': '256046',
                'sun_azimuth': 139.7,
                'sun_elevation': 42.6,
                'updated': '2017-03-17T06:40:16Z',
                'usable_data': 0,
                'view_angle': 0.9
            },
            'type': 'Feature'
        },
        ...
        // a lot of other features
}
```
## Get Thumbnail

### API End Point
Use a `GET` request to the value of `_links` object's `thumbnail`, which belongs to one of the features in the response:

e.g.
```JavaScript
https://api.planet.com/data/v1/item-types/PSScene4Band/items/20160930_180021_0e2f/thumb
```

### Header
`Authorization: Base <your api key>`

### Response
The result can be included in an `<img>` tag.

## Get Tiles
Planet Tile Services allow users to easily add Planet imagery mosaics to web mapping platforms that provide XYZ support, without having to download and maintain GeoTiff files. These services offer a low-friction way for web developers and GIS analysts to interact with and derive value from Planet imagery.

XYZ (OpenStreetMap) tile services provide a pyramid of tiles at 16 zoom levels so that it can be easily displayed in web browsers. As with other XYZ tiling services, the Planet XYZ Tiles have these attributes:

* Tiles are 256×256 pixels
* Tiles use the Web Mercator coordinate reference system (EPSG:3857)
* Tiles are available between zoom levels 0 and 15
* Tiles are rendered in PNG format with an alpha channel
* Grid is a rectangle with 2n rows and 2n columns, where n is the zoom level
* Grid uses 0,0 as the top, left corner in the grid
* Tiles are found at the path z/x/y.png, where z is the zoom level, and x and y are the positions in the tile grid

The API key may be passed as a parameter to the tile server: `api_key={your_api_key}`. For the Data API Tile Service, the user must also have permission to access the specific item-type in the request.

### API End Point
To access specific item-type tiles through the Data API Tile Service, use the following request structure:
```
https://tiles.planet.com/data/v1/{item_type}/{item_id}/{z}/{x}/{y}.png?api_key={your_api_key}
```
The following variables will be replaced by the client making the request:

| Parameter     | Type          | Value  						|
| ------------- |:-------------:| ----------------------------------------------------- |
| item_type     | string 	| Items in the Data API supported by the tiling service |
| item_id       | string      	| Unique id for the data tiles requested 		|
| z             | int      	| Zoom level 						|
| x             | int      	| Column in the grid 					|
| y             | int      	| Column in the grid 					|

## Activate the Asset/Scene/Imagery
### API End Point
Use a `GET` request to the value of `_links` object's `assets`, which belongs to one of the features in the response:

e.g.
```JavaScript
https://api.planet.com/data/v1/item-types/PSScene4Band/items/20160930_180021_0e2f/assets/
```

### Header
`Authorization: Base <your api key>`

### Response
```JavaScript
{
    'analytic': {
        '_links': {
            '_self': 'https://api.planet.com/data/v1/assets/eyJpIjogIjIwMTYwOTA1XzE5MjQ1Ml8wYzQxIiwgImMiOiAiUFNTY2VuZTRCYW5kIiwgInQiOiAiYW5hbHl0aWMiLCAiY3QiOiAiaXRlbS10eXBlIn0',
            'activate': 'https://api.planet.com/data/v1/assets/eyJpIjogIjIwMTYwOTA1XzE5MjQ1Ml8wYzQxIiwgImMiOiAiUFNTY2VuZTRCYW5kIiwgInQiOiAiYW5hbHl0aWMiLCAiY3QiOiAiaXRlbS10eXBlIn0/activate',
            'type': 'https://api.planet.com/data/v1/asset-types/analytic'
        },
        '_permissions': [
            'download'
        ],
        'md5_digest': null,
        'status': 'inactive',
        'type': 'analytic'
    },
    'analytic_xml': {
        // similar object to the above with different values
    },
    'udm': {
        // similar object to the above with different values
    }
}
```
### API End Point
Use a `GET` request to the value of `_links` object's `activate` (of analytic type in this example):

e.g.
```JavaScript
https://api.planet.com/data/v1/assets/eyJpIjogIjIwMTYwOTA1XzE5MjQ1Ml8wYzQxIiwgImMiOiAiUFNTY2VuZTRCYW5kIiwgInQiOiAiYW5hbHl0aWMiLCAiY3QiOiAiaXRlbS10eXBlIn0/activate
```

### Header
`Authorization: Base <your api key>`

### Response
There is no response body for this request but it has a status code `202` at first. This means that the scene/imagery/asset you requested is processing. After this request, sending `GET` requests to the same API end point at the beginning of this section (Activate the Asset/Scene/Imagery) will enable you to observe if the asset you request is ready yet. When it is ready, the `status` of it will become `active`, and there will be a new key `location` identifying the location of this asset with a `expires_at` key as well. Like the following (notice that this is the response body of the call to the first end point of this section:

```JavaScript
{
    "analytic": {
        "_links": {
            "_self": "https://api.planet.com/data/v1/assets/eyJpIjogIjIwMTYwOTA1XzE5MjQ1Ml8wYzQxIiwgImMiOiAiUFNTY2VuZTRCYW5kIiwgInQiOiAiYW5hbHl0aWMiLCAiY3QiOiAiaXRlbS10eXBlIn0",
            "activate": "https://api.planet.com/data/v1/assets/eyJpIjogIjIwMTYwOTA1XzE5MjQ1Ml8wYzQxIiwgImMiOiAiUFNTY2VuZTRCYW5kIiwgInQiOiAiYW5hbHl0aWMiLCAiY3QiOiAiaXRlbS10eXBlIn0/activate",
            "type": "https://api.planet.com/data/v1/asset-types/analytic"
        },
        "_permissions": [
            "download"
        ],
        "expires_at": "2017-06-30T22:18:11.368043",
        "location": "https://api.planet.com/data/v1/download?token=eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJUcndmNVJLSUdSMmpRL3ByWlN6N2krakovWGs2VmhmY01UWHEwL0lEMmVDdiswVzROQTkxaGcrVk91Z2QzMEl1T0czcHZmWXpmS2lJRnNLQ1NySTBudz09IiwiaXRlbV90eXBlX2lkIjoiUFNTY2VuZTRCYW5kIiwidG9rZW5fdHlwZSI6InR5cGVkLWl0ZW0iLCJleHAiOjE0OTg4NjEwOTEsIml0ZW1faWQiOiIyMDE2MDkwNV8xOTI0NTJfMGM0MSIsImFzc2V0X3R5cGUiOiJhbmFseXRpYyJ9.RO8xCegN9FLuxZmZ-JzbJU1wXdhKCU5PwRHKZ867m92WrH_oAICNeUBZ2b4wkOm2WzUrXY6S_O5e6FmYER8lwg",
        "md5_digest": "1a3f7cc98552795f3bdc62f587ec8d0b",
        "status": "active",
        "type": "analytic"
    },
    "analytic_xml": {
        // similar object to the above with different values
    },
    "udm": {
        // similar object to the above with different values
    }
}
```

## Download the Asset/Scene/Imagery
