# Media Magic

Mono repo for the Media Magic Core project.


## Project structure

- infrastructure/new - contains the new Pulumi infrastructure code. Any service specific infrastructure will be moved from this directory into the appropriate service directory over. The wider platyform infra will be moved from the infrastructure/old (Terraform), into Pulumi as well.
- libs - internal shared libraries. For example, Abzu, which is our internal framework for creating Go based webservices within Media Magic. This contains common tooling such as creating network requests, DI containers, cache and database connections, config management and more.
- services - contains all of the webservices that make up Media Magic. Most of the metadata is managed by Hasura, however there is quite a lot of additional business logic, handled by custom web services. Some of these respond to Hasura events, some are standalone GraphQL services, which Hasura federates into its own schema.
- scripts - contains end to end scripts
- webapps - contains any user interfaces, such as the test UI, which was created to test the API and assist UI teams, serving as an example as to how to communicate, authenticate and use Media Magic. It also contains a quick way of generating user and organisation tokens.
- deploy.sh - is a script that listens for changes within the microservices directory, and deploys any that have changed. (currently not in use)
- docker-compose.yml - for running the entire platform locally (needs updating currently).
- bins - is where compile binaries for each service are stored, these are not kept in the Gitlab repository, and are in the gitignore. To generate these, run `$ make build-all-binaries`.
- test-assets - is where test files such as images and videos are kept, as well as JSON payloads for testing against the REST api itself.


## Known issues / to be improved

- More tests / integration tests where possible.

### User Journey

1. A user will sign-up via Auth0, and the Auth0 UI library. The user service exposes the endpoints needed for Auth0 to complete the authentication process. 
2. A user token will be issued, this token is attached to a single user. This token is an OIDC compliant JWT token. 
3. A user token has access to create deployments, upload media and list jobs/deployments/models, essentially the core functionality.
4. A user can then create an organisation, which can encapsulate multiple users around the same resources. This will require invites, which we will be adding at a later date.
5. Once a user has created an organisation, they can create an organisation token. Resources created/fetched using an organisation token will be stored against that organisation id, rather than the individual user. If you use a user token, you will not be able to use these resources.
6. A user, or an organisation can upload media. Each media asset has an individual ID. 
7. A user, or an organisation can then create a deployment, which will need media ID's and a model ID, which can be found using the list models endpoint. Models are references to models deployed into our cluster manager and on premise hardware.
8. A deployment will create a job, which represents the state of a deployment against the cluster manager. 
9. A user, or an organisation can then list jobs to see the output and any generated urls for any results created.

## Running Locally 

### Per Service 

Each service has a Makefile, and each service has a `$ make run` command which runs the service locally. 

You will need to use something like grpcurl to test gRPC endpoints locally. Each service also has a `$ make start.local.dependencies`, which runs a Postgres and Redis docker image.

### Entire Stack (docker)

Another option is to run the entire stack using the provided `docker-compose file`: `$ docker compose build && docker compose up -d`.

### Entire Stack (non-docker)

Alternatively, you can also run `$ make build-all-binaries` and `$ make run-all-binaries` to run all the services together, then run `$ make run-gateway` to start running the API Gateway, also.

Each service uses 'auto migrate' from the gorm library, which means any changes to database models are updated in the underlying tables automatically.

### Connecting to the production database

1. Download CloudSQL Proxy from: https://cloud.google.com/sql/docs/mysql/connect-admin-proxy
2. Run it:
  ```bash
     ./cloud_sql_proxy -instances=mediamagic-core-ewanv:us-east4:mediamagic-dev-config-41b0c5b2=tcp:0.0.0.0:5432
  ```
3. Ask me for the database password.
4. The username is `mediamagic-dev-user`
5. The database is specific to the service you want to connect to. Each microservice has its own database. The format is: `<service>_service`. For example `deployment_service`.
6. The schema is the name of the service, without `_service`. For example, the schema for the deployment service is `deployment`.
7. The host is: `localhost:5432` as you're running the proxy locally.
8. A connection string example: `postgresql://localhost:5432/deployment_service`

### A note on Authentication

Media Magic utilises Auth0 for authentication, and storing personal information about users. This is for security and compliance reasons, as well as a time saver in not implementing authentication ourselves.

To create a front-end or app that integrates into our Auth0, you will need the public token, which can be provided on request.

### Tokens

#### User Token

There are two separate tokens in use with the Media Magic API's. The first is a short-lived user token, which Auth0 returns. Which expires after three days. This token should be used in mobile apps or web apps. This token should be used for basic actions needed by user interfaces.

The user token uses the following format: `Authorization: Bearer <JWT>`.

#### API Keys

The second is a long-lived token, which is stateful and can be invalidated with immediate effect. This token should be used for programatic use-cases. For example, interacting with the rest API to perform automated tasks. For projects and integrations, you will probably need this token. It's advised to use this token where not for use in web apps, apps, or places where a user logs in via the Auth0 login prompt.

API Keys use the following format: `x-mediamagic-key: <UUID>`.

#### API Keys

The second is a long-lived token, which is stateful and can be invalidated with immediate effect. This token should be used for programatic use-cases. For example, interacting with the rest API to perform automated tasks. For projects and integrations, you will probably need this token. It's advised to use this token where not for use in web apps, apps, or places where a user logs in via the Auth0 login prompt.

API Keys use the following format: `x-mediamagic-key: <UUID>`.

## Docs

You can find the full API documentation here: https://mediamagic.dev/docs

Below are some examples, but we would highly recommend using the docs ^

## Examples

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

## Upload a Media asset

This endpoint is currently outside of the REST API as it deals with file uploads, so the headers etc are slightly different.

```bash
$ curl -X POST https://mediamagic.dev/uploads/upload -F 'media=@sample-image.jpg' --header "Authorization: Bearer $ORG_TOKEN"
// {}
```

## Create an API Key

API keys are to be seen as long-lived API keys, to be used in other projects for example. 

```bash 
$ curl -X POST https://mediamagic.dev/api/v1/tokens \
    -H 'Authorization: Bearer $USER_TOKEN' \
    -H 'content-type: application/mediamagic.rest.v1+json'
    -d '{ "name": "my-token" }'
```

### Create a token for a team

```bash 
$ curl -X POST https://mediamagic.dev/api/v1/tokens \
    -H 'Authorization: Bearer $USER_TOKEN' \
    -H 'content-type: application/mediamagic.rest.v1+json'
    -d '{ "name": "my-token", "team_id": "<TEAM_ID>" }'
```

## Create a team

```bash 
$ curl -X POST https://mediamagic.dev/api/v1/tokens \
    -H 'Authorization: Bearer $USER_TOKEN'
    -H 'content-type: application/mediamagic.rest.v1_json'
    -d '{ "name": "my-team" }'
```

## Media

### Create media

```
 curl --location --request POST 'https://mediamagic.dev/upload' \
 --header 'Authorization: Bearer $TOKEN' \
 --form 'media=@"/home/repo-l5/Downloads/download.jpeg"' \
 --form 'tags="hair","face"'

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
 curl --location --request GET 'https://mediamagic.dev/api/v1/media' \
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
                "hair", "face"
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
 curl --location --request GET 'https://mediamagic.dev/api/v1/media?tags=face' \
 --header 'Authorization: Bearer $TOKEN'

 // Output example
 {
    "media": [
        {
            "id": "f5e14b6f-0ccb-11ed-b4c1-98fa9bf84765",
            "image_name": "images.jpeg",
            "media_key": "https://cdn.mediamagic.dev/media/f5e14b6f-0ccb-11ed-b4c1-98fa9bf84765.jpeg",
            "status": "live",
            "tags": [
                "face",
                "tags2",
                "hair"
            ],
            "type": "source_asset",
            "upload_date": "2022-07-26 15:46:07.102843 +0530 IST"
        }
    ],
    "media_key": "https://cdn.mediamagic.dev"
}
```

### Filter media files by exclude tags

```
curl --location --request GET 'https://mediamagic.dev/api/v1/media?excludeTags=face' \
--header 'Authorization: Bearer $TOKEN'

// Output example
{
    "media": [
        {
            "id": "3e822ca6-0ccc-11ed-b4c1-98fa9bf84765",
            "image_name": "images.jpeg",
            "media_key": "https://cdn.mediamagic.dev/media/3e822ca6-0ccc-11ed-b4c1-98fa9bf84765.jpeg",
            "status": "live",
            "tags": [
                "hair"
            ],
            "type": "source_asset",
            "upload_date": "2022-07-26 15:48:08.702076 +0530 IST"
        }
    ],
    "media_key": "https://cdn.mediamagic.dev"
}
```

### Filter media files by tags and exclude tags

```
curl --location --request GET ' https://mediamagic.dev/api/v1/media?tags=hair&excludeTags=test' \
--header 'Authorization: Bearer $TOKEN'

// Output example
{
    "media": [
        {
            "id": "3e822ca6-0ccc-11ed-b4c1-98fa9bf84765",
            "image_name": "images.jpeg",
            "media_key": "https://cdn.mediamagic.dev/media/3e822ca6-0ccc-11ed-b4c1-98fa9bf84765.jpeg",
            "status": "live",
            "tags": [
                "hair"
            ],
            "type": "source_asset",
            "upload_date": "2022-07-26 15:48:08.702076 +0530 IST"
        }
    ],
    "media_key": "https://cdn.mediamagic.dev"
}
```

### Update media files

```
curl --location --request PUT 'https://mediamagic.dev/api/v1/media/e8cc3bdb-0ca7-11ed-a94e-98fa9bf84765' \
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
curl --location --request DELETE 'https://mediamagic.dev/api/v1/media/4e06a91b-0cb7-11ed-b06e-98fa9bf84765' \
--header 'Authorization: Bearer $TOKEN'

// Output example
Status - 204
```

## Models

### Create model with id

Creating a model requires a model name, a container name (which references a container on the cluster manager), and args.

Args are a list of arguments by name, the value for each name is metadata about that argument. There's a `type` field, which is used to know what to replace the values with. For example, a file type fetches the file and replaces the placeholder with the full file route, assuming the value field is an image ID.

It's important to note that, currently, the args stored against each model, is used to describe what args need to be provided to the deployment. These are mostly for documentation purposes. There's no checks currently, eventually, the arguments included within a deployment, will be validated against the args defined on the model.

```
curl --location --request POST 'https://mediamagic.dev/api/v1/models' \
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
      "format": "--form image=@:value"
    },
    "source": {
      "type": "string",
      "format": "--form source=@:value"
    },
    "target": {
      "type": "string",
      "format": "--form target=@:value"
    },
    "alpha": {
      "type": "number",
      "format": "--form alpha=@:value",
      "min": 0,
      "max": 100
    },
    "beta": {
      "type": "number",
      "format": "--form beta=@:value",
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
  "command": "127.0.0.1:8013/styleclip {args} --form output=/tmp/clust_mgr/",
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

### Create model without id or random id

```
curl --location --request POST 'https://mediamagic.dev/api/v1/models' \
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
      "format": "--form image=@:value"
    },
    "source": {
      "type": "string",
      "format": "--form source=@:value"
    },
    "target": {
      "type": "string",
      "format": "--form target=@:value"
    },
    "alpha": {
      "type": "number",
      "format": "--form alpha=@:value",
      "min": 0,
      "max": 100
    },
    "beta": {
      "type": "number",
      "format": "--form beta=@:value",
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
  "command": "127.0.0.1:8013/styleclip {args} --form output=/tmp/clust_mgr/",
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
curl --location --request GET 'https://mediamagic.dev/api/v1/models/c691b065-2a8e-4db5-a8bb-3d33eaccf0d5' \
--header 'Authorization: Bearer $TOKEN'

// Output example
{
    "args": {
        "alpha": {
            "example": "",
            "format": "--form alpha=@:value",
            "max": 100,
            "min": 0,
            "optional": false,
            "type": "number"
        },
        "beta": {
            "example": "",
            "format": "--form beta=@:value",
            "max": 100,
            "min": 0,
            "optional": false,
            "type": "number"
        },
        "image": {
            "example": "",
            "format": "--form image=@:value",
            "optional": true,
            "type": "file"
        },
        "source": {
            "example": "",
            "format": "--form source=@:value",
            "optional": false,
            "type": "string"
        },
        "target": {
            "example": "",
            "format": "--form target=@:value",
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
curl --location --request GET 'https://mediamagic.dev/api/v1/models' \
--header 'Authorization: Bearer $TOKEN'

// Output example
[
    {
        "args": {
            "alpha": {
                "example": "",
                "format": "--form alpha=@:value",
                "max": 100,
                "min": 0,
                "optional": false,
                "type": "number"
            },
            "beta": {
                "example": "",
                "format": "--form beta=@:value",
                "max": 100,
                "min": 0,
                "optional": false,
                "type": "number"
            },
            "image": {
                "example": "",
                "format": "--form image=@:value",
                "optional": true,
                "type": "file"
            },
            "source": {
                "example": "",
                "format": "--form source=@:value",
                "optional": false,
                "type": "string"
            },
            "target": {
                "example": "",
                "format": "--form target=@:value",
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
                "format": "--form alpha=@:value",
                "max": 100,
                "min": 0,
                "optional": false,
                "type": "number"
            },
            "beta": {
                "example": "",
                "format": "--form beta=@:value",
                "max": 100,
                "min": 0,
                "optional": false,
                "type": "number"
            },
            "image": {
                "example": "",
                "format": "--form image=@:value",
                "optional": true,
                "type": "file"
            },
            "source": {
                "example": "",
                "format": "--form source=@:value",
                "optional": false,
                "type": "string"
            },
            "target": {
                "example": "",
                "format": "--form target=@:value",
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
curl --location --request GET 'https://mediamagic.dev/api/v1/models?tags=head' \
--header 'Authorization: Bearer $TOKEN'

[
    {
        "args": {
            "alpha": {
                "example": "",
                "format": "--form alpha=@:value",
                "max": 100,
                "min": 0,
                "optional": false,
                "type": "number"
            },
            "beta": {
                "example": "",
                "format": "--form beta=@:value",
                "max": 100,
                "min": 0,
                "optional": false,
                "type": "number"
            },
            "image": {
                "example": "",
                "format": "--form image=@:value",
                "optional": true,
                "type": "file"
            },
            "source": {
                "example": "",
                "format": "--form source=@:value",
                "optional": false,
                "type": "string"
            },
            "target": {
                "example": "",
                "format": "--form target=@:value",
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
curl --location --request PUT 'https://mediamagic.dev/api/v1/models/1c3c27d8-5560-41d6-bb56-8c9c3be889c5' \
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
      "format": "--form image=@:value"
    },
    "source": {
      "type": "string",
      "format": "--form source=@:value"
    },
    "target": {
      "type": "string",
      "format": "--form target=@:value"
    },
    "alpha": {
      "type": "number",
      "format": "--form alpha=@:value",
      "min": 0,
      "max": 100
    },
    "beta": {
      "type": "number",
      "format": "--form beta=@:value",
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
  "command": "127.0.0.1:8013/styleclip {args} --form output=/tmp/clust_mgr/",
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
            "format": "--form alpha=@:value",
            "max": 100,
            "min": 0,
            "optional": false,
            "type": "number"
        },
        "beta": {
            "example": "",
            "format": "--form beta=@:value",
            "max": 100,
            "min": 0,
            "optional": false,
            "type": "number"
        },
        "image": {
            "example": "",
            "format": "--form image=@:value",
            "optional": true,
            "type": "file"
        },
        "source": {
            "example": "",
            "format": "--form source=@:value",
            "optional": false,
            "type": "string"
        },
        "target": {
            "example": "",
            "format": "--form target=@:value",
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
curl --location --request DELETE 'https://mediamagic.dev/api/v1/models/56c7c6bf-4987-4e36-9cc9-9281e96a7d74' \
--header 'Authorization: Bearer $TOKEN'

// Output example
 Status - 204
```

## Create a Deployment

Valid types for args: int, float, string, file (file must be the UUID of an uploaded media asset).

```bash
$ curl -X POST --url https://mediamagic.dev/api/v1/deployments \
          -H "Authorization: Bearer $TOKEN" \
          -d '{ "name": "test", "model_id": "id", "args": { "source": "$(source)", "target": "$(target)" , "strength": 1 } }' \
          -H 'content-type: application/mediamagic.rest.v1+json'

// Output example
// {"id":"c63dac60-451d-4697-a335-fcd7e6c676ea","model_id":"18bac45c-f799-4202-8517-297c6e5e4574","source_asset_id":"b8c662c8-95bd-44c0-aab3-77f33f6442dc","target_asset_id":"54c7064d-8020-4d6e-adff-2b04cc849d65"}
```

## List Deployments

```bash
$ curl -X GET --url "https://mediamagic.dev/api/v1/deployments" \
	-H "Authorization: Bearer $TOKEN" \
	-H 'content-type: application/mediamagic.rest.v1+json'
```

## List Jobs

```bash
$ curl -X GET --url "https://mediamagic.dev/api/v1/jobs" \
	-H "Authorization: Bearer $TOKEN" \
	-H 'content-type: application/mediamagic.rest.v1+json'
```

## Get Job by ID 

```bash 
export ID=<id>
curl -X "GET" https://mediamagic.dev/api/v1/jobs/$ID \
		-H "Authorization: Bearer $TOKEN" \
		-H 'content-type: application/mediamagic.rest.v1+json'
```

## Get Deployment by ID

```bash 
export ID=<id>
curl -X "GET" https://mediamagic.dev/api/v1/deployments/$ID \
		-H "Authorization: Bearer $TOKEN" \
		-H 'content-type: application/mediamagic.rest.v1+json'
```

## Get current billing usage 

```bash 
$ curl -X GET https://mediamagic.dev/api/v1/billing/usage \
		-H "Authorization: Bearer $TOKEN" \
		-H 'content-type: application/mediamagic.rest.v1+json'
```

## Monitoring

Expose Grafana

```
export POD_NAME=(kubectl get pods --namespace default -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=loki-grafana" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace default port-forward $POD_NAME 3000
```

## Back-ups

### Database

The Postgres database is backed up every day between 3-7am, 7 back-ups are kept, and point in time recovery is enabled also.


## Disaster recovery - SQL

For full back-ups, see the Cloud SQL instance in Google Cloud.


### Full database restore from back-up

Find the Cloud SQL instance in our Google Cloud account, this is named `mediamagic-dev-db-41b0c5b2` currently. 

Click into that database instance in the Google Cloud UI, then click on 'Export'. This will provide several options, including a 'Destination' section.

Select SQL, then select `mediamagic-dev-db` from the 'Data to export' section.

A bucket has been created for database back-ups already, you will need to select that in the destination section, this is called `backups-mm`.

This will export the entire database to that Cloud Storage bucket. You can download the file directly, or you can import it into another instance within Google Cloud.


## Automated recovery

The database instance has point-in-time recovery enabled, with a 7 day tail.

You can find these under the 'Backups' menu item in the left-hand nav, when you navigate to the database instance in Google Cloud.
