# MediaMagic-Readme

For each of these examples, we're using the user or JWT token. But you can also use the API key:

```bash
x-mediamagic-key:<API_KEY>
```

Rather than:

```bash 
Authorization:Bearer <JWT>
```

### Authenticate

First we grab a generated login URL from the API. Open the link returned, this will take you through the user authentication process.

Once you've authenticated, a callback url will be called with an access token. Which is then validated and swapped for a user token.

```bash
$ curl -X GET https://mediamagic.dev/api/v1/auth/login -H 'content-type: application/mediamagic.rest.v1+json'

// Example output
// {"access_token":"<token>","refresh_token":"<refresh-token>","id_token":"","token_type":"Bearer"}
```

Now set your auth token as an environment variable for later use.

```bash
export TOKEN=<token>
```

## Media

### Create a media asset

```
 curl --location --request POST 'http://127.0.0.1:9090/upload' \
 --header 'Authorization: Bearer $TOKEN' \
 --form 'media=@"/home/repo-l5/Downloads/download.jpeg"' \
 --form 'tags="hair,face"'

 // Output example
 {
    "media": {
        "id": "e8cc3bdb-0ca7-11ed-a94e-98fa9bf84765",
        "url": "https://cdn.mediamagic.dev/media/e8cc3bdb-0ca7-11ed-a94e-98fa9bf84765.jpeg"
    },
    "base_url": "https://cdn.mediamagic.dev"
 }
```


### List media files

```
 curl --location --request GET 'http://127.0.0.1:5001/api/v1/media' \
 --header 'Authorization: Bearer $TOKEN'

 // Output example
 {
    "media": [
        {
            "id": "e8cc3bdb-0ca7-11ed-a94e-98fa9bf84765",
            "image_name": "download.jpeg",
            "media_key": "https://cdn.mediamagic.dev/media/e8cc3bdb-0ca7-11ed-a94e-98fa9bf84765.jpeg",
            "status": "live",
            "tags": [
                "hair,face"
            ],
            "type": "source_asset",
            "upload_date": "2022-07-26 11:28:03.259276 +0530 IST"
        }
    ],
    "media_key": "https://cdn.mediamagic.dev"
 }
```

### Filter media files by tags

```
 curl --location --request GET 'http://127.0.0.1:5001/api/v1/media?tags=hair' \
 --header 'Authorization: Bearer $TOKEN'

 // Output example
 {
    "media": [
        {
            "id": "e8cc3bdb-0ca7-11ed-a94e-98fa9bf84765",
            "image_name": "download.jpeg",
            "media_key": "https://cdn.mediamagic.dev/media/e8cc3bdb-0ca7-11ed-a94e-98fa9bf84765.jpeg",
            "status": "live",
            "tags": [
                "hair,face"
            ],
            "type": "source_asset",
            "upload_date": "2022-07-26 11:28:03.259276 +0530 IST"
        }
    ],
    "media_key": "https://cdn.mediamagic.dev"
 }
```

### Filter media files by exclude tags

```
curl --location --request GET 'http://127.0.0.1:5001/api/v1/media?excludeTags=hair' \
--header 'Authorization: Bearer $TOKEN'

// Output example
{
    "media": {
        "id": "e8cc3bdb-0ca7-11ed-a94e-98fa9bf84765",
        "media_key": "media/e8cc3bdb-0ca7-11ed-a94e-98fa9bf84765.jpeg",
        "status": "live",
        "tags": [
            "face",
            "tags",
            "hair"
        ],
        "title": "media-upload"
    },
    "media_key": "https://cdn.mediamagic.dev"
}
```

### Filter media files by tags and exclude tags

```
curl --location --request GET 'http://127.0.0.1:5001/api/v1/media?tags=face&excludeTags=hair' \
--header 'Authorization: Bearer $TOKEN'

// Output example
{
    "media": {
        "id": "e8cc3bdb-0ca7-11ed-a94e-98fa9bf84765",
        "media_key": "media/e8cc3bdb-0ca7-11ed-a94e-98fa9bf84765.jpeg",
        "status": "live",
        "tags": [
            "face",
            "tags",
            "hair"
        ],
        "title": "media-upload"
    },
    "media_key": "https://cdn.mediamagic.dev"
}
```

### Update media files

```
curl --location --request PUT 'http://127.0.0.1:5001/api/v1/media/e8cc3bdb-0ca7-11ed-a94e-98fa9bf84765' \
--header 'Authorization: Bearer $TOKEN' \
--header 'Content-Type: application/json' \
--data-raw '{
    "tags": [
                "face",
                "tags",
                "hair"
            ]
}'

// Output example
{
    "media": {
        "id": "e8cc3bdb-0ca7-11ed-a94e-98fa9bf84765",
        "media_key": "media/e8cc3bdb-0ca7-11ed-a94e-98fa9bf84765.jpeg",
        "status": "live",
        "tags": [
            "face",
            "tags",
            "hair"
        ],
        "title": "media-upload"
    },
    "media_key": "https://cdn.mediamagic.dev"
}
```

### Delete media files

```
curl --location --request DELETE 'http://127.0.0.1:5001/api/v1/media/4e06a91b-0cb7-11ed-b06e-98fa9bf84765' \
--header 'Authorization: Bearer $TOKEN'

// Output example
Status - 404
```

## Models

### Create model with id

```
curl --location --request POST 'http://127.0.0.1:5001/api/v1/models' \
--header 'Authorization: Bearer $TOKEN' \
--header 'Content-Type: application/json' \
--data-raw '{
  "args": {
    "image": {
      "tags": [
        "mask"
      ],
      "optional": true,
      "type": "file",
      "format": "--form image=:value"
    },
    "source": {
      "type": "string",
      "format": "--form source=:value"
    },
    "target": {
      "type": "string",
      "format": "--form target=:value"
    },
    "alpha": {
      "type": "number",
      "format": "--form alpha=:value",
      "min": 0,
      "max": 100
    },
    "beta": {
      "type": "number",
      "format": "--form beta=:value",
      "min": 0,
      "max": 100
    }
  },
  "name": "styleClip",
  "container": "ubuntu",
  "machine_type": "container2",
  "machine_count": 2,
  "tags": [
    "face",
    "head",
    "body"
  ],
  "id": "c691b065-2a8e-4db5-a8bb-3d33eaccf0d5",
  "icon": "icon.jpg",
  "command": "127.0.0.1:8013/styleclip {image} {source} {target} {alpha} {beta} --form output=/tmp/clust_mgr/",
  "description": "it does manipulation based on text query from user",
  "example_outputs": [
    "https://studio213.us/gpu/ai/input/em.jpg"
  ]
}'

// Output example
{
    "classification": "fast",
    "example_outputs": [
        "https://studio213.us/gpu/ai/input/em.jpg"
    ],
    "id": "c691b065-2a8e-4db5-a8bb-3d33eaccf0d5",
    "name": "styleClip",
    "tags": [
        "face",
        "head",
        "body"
    ]
}
```

### Create model without id or aandom id

```
curl --location --request POST 'http://127.0.0.1:5001/api/v1/models' \
--header 'Authorization: Bearer $TOKEN' \
--header 'Content-Type: application/json' \
--data-raw '{
  "args": {
    "image": {
      "tags": [
        "mask"
      ],
      "optional": true,
      "type": "file",
      "format": "--form image=:value"
    },
    "source": {
      "type": "string",
      "format": "--form source=:value"
    },
    "target": {
      "type": "string",
      "format": "--form target=:value"
    },
    "alpha": {
      "type": "number",
      "format": "--form alpha=:value",
      "min": 0,
      "max": 100
    },
    "beta": {
      "type": "number",
      "format": "--form beta=:value",
      "min": 0,
      "max": 100
    }
  },
  "name": "styleClip",
  "container": "ubuntu",
  "machine_type": "container2",
  "machine_count": 2,
  "tags": [
    "face",
    "body"
  ],
  "icon": "icon.jpg",
  "command": "127.0.0.1:8013/styleclip {image} {source} {target} {alpha} {beta} --form output=/tmp/clust_mgr/",
  "description": "it does manipulation based on text query from user",
  "example_outputs": [
    "https://studio213.us/gpu/ai/input/em.jpg"
  ]
}'

// Output example
{
    "classification": "fast",
    "example_outputs": [
        "https://studio213.us/gpu/ai/input/em.jpg"
    ],
    "id": "1c3c27d8-5560-41d6-bb56-8c9c3be889c5",
    "name": "styleClip",
    "tags": [
        "face",
        "body"
    ]
}
```
### Get model by id

```
curl --location --request GET 'http://127.0.0.1:5001/api/v1/models/c691b065-2a8e-4db5-a8bb-3d33eaccf0d5' \
--header 'Authorization: Bearer $TOKEN'

// Output example
{
    "args": {
        "alpha": {
            "example": "",
            "format": "--form alpha=:value",
            "max": 100,
            "min": 0,
            "optional": false,
            "type": "number"
        },
        "beta": {
            "example": "",
            "format": "--form beta=:value",
            "max": 100,
            "min": 0,
            "optional": false,
            "type": "number"
        },
        "image": {
            "example": "",
            "format": "--form image=:value",
            "optional": true,
            "type": "file"
        },
        "source": {
            "example": "",
            "format": "--form source=:value",
            "optional": false,
            "type": "string"
        },
        "target": {
            "example": "",
            "format": "--form target=:value",
            "optional": false,
            "type": "string"
        }
    },
    "classification": "fast",
    "example_outputs": [
        "https://studio213.us/gpu/ai/input/em.jpg"
    ],
    "icon": "icon.jpg",
    "id": "c691b065-2a8e-4db5-a8bb-3d33eaccf0d5",
    "name": "styleClip",
    "tags": [
        "face",
        "head",
        "body"
    ],
    "updated_at": "2022-07-26 12:09:53.770645 +0530 IST"
}
```

### List models

```
curl --location --request GET 'http://127.0.0.1:5001/api/v1/models' \
--header 'Authorization: Bearer $TOKEN'

// Output example
[
    {
        "args": {
            "alpha": {
                "example": "",
                "format": "--form alpha=:value",
                "max": 100,
                "min": 0,
                "optional": false,
                "type": "number"
            },
            "beta": {
                "example": "",
                "format": "--form beta=:value",
                "max": 100,
                "min": 0,
                "optional": false,
                "type": "number"
            },
            "image": {
                "example": "",
                "format": "--form image=:value",
                "optional": true,
                "type": "file"
            },
            "source": {
                "example": "",
                "format": "--form source=:value",
                "optional": false,
                "type": "string"
            },
            "target": {
                "example": "",
                "format": "--form target=:value",
                "optional": false,
                "type": "string"
            }
        },
        "classification": "fast",
        "description": "it does manipulation based on text query from user",
        "example_outputs": [
            "https://studio213.us/gpu/ai/input/em.jpg"
        ],
        "icon": "icon.jpg",
        "id": "c691b065-2a8e-4db5-a8bb-3d33eaccf0d5",
        "name": "styleClip",
        "tags": [
            "face",
            "head",
            "body"
        ],
        "updated_at": "2022-07-26 12:09:53.770645 +0530 IST"
    },
    {
        "args": {
            "alpha": {
                "example": "",
                "format": "--form alpha=:value",
                "max": 100,
                "min": 0,
                "optional": false,
                "type": "number"
            },
            "beta": {
                "example": "",
                "format": "--form beta=:value",
                "max": 100,
                "min": 0,
                "optional": false,
                "type": "number"
            },
            "image": {
                "example": "",
                "format": "--form image=:value",
                "optional": true,
                "type": "file"
            },
            "source": {
                "example": "",
                "format": "--form source=:value",
                "optional": false,
                "type": "string"
            },
            "target": {
                "example": "",
                "format": "--form target=:value",
                "optional": false,
                "type": "string"
            }
        },
        "classification": "fast",
        "description": "it does manipulation based on text query from user",
        "example_outputs": [
            "https://studio213.us/gpu/ai/input/em.jpg"
        ],
        "icon": "icon.jpg",
        "id": "1c3c27d8-5560-41d6-bb56-8c9c3be889c5",
        "name": "styleClip",
        "tags": [
            "face",
            "body"
        ],
        "updated_at": "2022-07-26 12:12:24.577571 +0530 IST"
    }
]
```

### Filter models by tags

```
curl --location --request GET 'http://127.0.0.1:5001/api/v1/models?tags=head' \
--header 'Authorization: Bearer $TOKEN'

[
    {
        "args": {
            "alpha": {
                "example": "",
                "format": "--form alpha=:value",
                "max": 100,
                "min": 0,
                "optional": false,
                "type": "number"
            },
            "beta": {
                "example": "",
                "format": "--form beta=:value",
                "max": 100,
                "min": 0,
                "optional": false,
                "type": "number"
            },
            "image": {
                "example": "",
                "format": "--form image=:value",
                "optional": true,
                "type": "file"
            },
            "source": {
                "example": "",
                "format": "--form source=:value",
                "optional": false,
                "type": "string"
            },
            "target": {
                "example": "",
                "format": "--form target=:value",
                "optional": false,
                "type": "string"
            }
        },
        "classification": "fast",
        "description": "it does manipulation based on text query from user",
        "example_outputs": [
            "https://studio213.us/gpu/ai/input/em.jpg"
        ],
        "icon": "icon.jpg",
        "id": "c691b065-2a8e-4db5-a8bb-3d33eaccf0d5",
        "name": "styleClip",
        "tags": [
            "face",
            "head",
            "body"
        ],
        "updated_at": "2022-07-26 12:09:53.770645 +0530 IST"
    }
]
```

### Update model

```
curl --location --request PUT 'http://127.0.0.1:5001/api/v1/models/1c3c27d8-5560-41d6-bb56-8c9c3be889c5' \
--header 'Authorization: Bearer $TOKEN' \
--header 'Content-Type: application/json' \
--data-raw '{
  "args": {
    "image": {
      "tags": [
        "mask"
      ],
      "optional": true,
      "type": "file",
      "format": "--form image=:value"
    },
    "source": {
      "type": "string",
      "format": "--form source=:value"
    },
    "target": {
      "type": "string",
      "format": "--form target=:value"
    },
    "alpha": {
      "type": "number",
      "format": "--form alpha=:value",
      "min": 0,
      "max": 100
    },
    "beta": {
      "type": "number",
      "format": "--form beta=:value",
      "min": 0,
      "max": 100
    }
  },
  "name": "styleClip",
  "container": "ubuntu",
  "machine_type": "container2",
  "machine_count": 2,
  "tags": [
    "face",
    "body",
    "face2"
  ],
  "icon": "icon.jpg",
  "command": "127.0.0.1:8013/styleclip {image} {source} {target} {alpha} {beta} --form output=/tmp/clust_mgr/",
  "description": "it does manipulation based on text query from user",
  "example_outputs": [
    "https://studio213.us/gpu/ai/input/em.jpg"
  ]
}'

// Output example
 {
    "args": {
        "alpha": {
            "example": "",
            "format": "--form alpha=:value",
            "max": 100,
            "min": 0,
            "optional": false,
            "type": "number"
        },
        "beta": {
            "example": "",
            "format": "--form beta=:value",
            "max": 100,
            "min": 0,
            "optional": false,
            "type": "number"
        },
        "image": {
            "example": "",
            "format": "--form image=:value",
            "optional": true,
            "type": "file"
        },
        "source": {
            "example": "",
            "format": "--form source=:value",
            "optional": false,
            "type": "string"
        },
        "target": {
            "example": "",
            "format": "--form target=:value",
            "optional": false,
            "type": "string"
        }
    },
    "command": "127.0.0.1:8013/styleclip {image} {source} {target} {alpha} {beta} --form output=/tmp/clust_mgr/",
    "container": "ubuntu",
    "description": "it does manipulation based on text query from user",
    "example_outputs": [
        "https://studio213.us/gpu/ai/input/em.jpg"
    ],
    "hidden": "false",
    "icon": "icon.jpg",
    "machine_count": 2,
    "machine_type": "container2",
    "name": "styleClip",
    "tags": [
        "face",
        "body",
        "face2"
    ],
    "updated_at": "2022-07-26 12:43:04.030819 +0530 IST"
}
```


### Delete model

```
curl --location --request DELETE 'http://127.0.0.1:5001/api/v1/models/56c7c6bf-4987-4e36-9cc9-9281e96a7d74' \
--header 'Authorization: Bearer $TOKEN'

// Output example
 Status - 404
```
