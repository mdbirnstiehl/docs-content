---
mapped_pages:
  - https://www.elastic.co/guide/en/serverless/current/security-project-settings.html
applies_to:
  serverless: ga
navigation_title: Manage projects with API
---

# Manage serverless projects using the API [serverless-api]

On this page, you can find examples of how to create and manage serverless projects using the [Elastic Cloud Serverless API](https://www.elastic.co/docs/api/doc/elastic-cloud-serverless/). 

To learn about API principles, authentication, and how to use the OpenAPI specification, refer to the [Elastic Cloud Serverless API](https://www.elastic.co/docs/api/doc/elastic-cloud-serverless/) documentation.

The available APIs are grouped by project type:

- APIs for [Search projects](https://www.elastic.co/docs/api/doc/elastic-cloud-serverless/group/endpoint-elasticsearch-projects)
- APIs for [Observatibility projects](https://www.elastic.co/docs/api/doc/elastic-cloud-serverless/group/endpoint-observability-projects)
- APIs for [Security projects](https://www.elastic.co/docs/api/doc/elastic-cloud-serverless/group/endpoint-security-projects)

To try the examples in this section, [set up an API key](#general-manage-project-with-api-set-up-api-key) and [create an {{serverless-full}} project](#general-manage-project-with-api-create-a-serverless-elasticsearch-project).

## Set up an API key [general-manage-project-with-api-set-up-api-key]

1. [Create an API key](https://www.elastic.co/docs/deploy-manage/api-keys/elastic-cloud-api-keys).
2. Store the generated API key as an environment variable so that you don’t need to specify it again for each request:

```console
export API_KEY="YOUR_GENERATED_API_KEY"
```

## Create an {{serverless-full}} project [general-manage-project-with-api-create-a-serverless-elasticsearch-project]

```bash
curl -H "Authorization: ApiKey $API_KEY" \
     -H "Content-Type: application/json" \
     "https://api.elastic-cloud.com/api/v1/serverless/projects/elasticsearch" \
     -XPOST --data '{
        "name": "My project",  <1>
        "region_id": "aws-us-east-1"  <2>
     }'
```
1. Replace `My project` with a more descriptive name in this call.
2. You can obtain a [list of available regions](#general-manage-project-with-api-list-available-regions). 

The response from the create project request will include the created project details, such as the project ID, the credentials to access the project, and the endpoints to access different apps such as {{es}} and {{kib}}.

Example of `Create project` response:

```console-response
{
    "id": "cace8e65457043698ed3d99da2f053f6",
    "endpoints": {
        "elasticsearch": "https://sample-project-c990cb.es.us-east-1.aws.elastic.cloud",
        "kibana": "https://sample-project-c990cb-c990cb.kb.us-east-1.aws.elastic.cloud"
    },
    "credentials": {
        "username": "admin",
        "password": "abcd12345"
    }
    (...)
}
```

You can store the project ID as an environment variable for the next requests:

```console
export PROJECT_ID=cace8e65457043698ed3d99da2f053f6
```

## API examples

The following examples show how to interact with the APIs, covering common operations such as:

- [Creating a project](#general-manage-project-with-api-create-a-serverless-elasticsearch-project)
- [Retrieving project details](#general-manage-project-with-api-get-project)
- [Retrieving the project's status](#general-manage-project-with-api-get-project-status)
- [Resetting credentials](#general-manage-project-with-api-reset-credentials)
- [Deleting a project](#general-manage-project-with-api-delete-project)
- [Updating a project](#general-manage-project-with-api-update-project)
- [Listing regions where projects can be created](#general-manage-project-with-api-list-available-regions)

### Get project details [general-manage-project-with-api-get-project]

You can retrieve your project details through an API call:

```bash
curl -H "Authorization: ApiKey $API_KEY" \
    "https://api.elastic-cloud.com/api/v1/serverless/projects/elasticsearch/${PROJECT_ID}"
```

### Get the project status [general-manage-project-with-api-get-project-status]

The 'status' endpoint indicates whether the project is initialized and ready to be used. In the response, the project's `phase` will change from "initializing" to "initialized" when it is ready:

```bash
curl -H "Authorization: ApiKey $API_KEY" \
    "https://api.elastic-cloud.com/api/v1/serverless/projects/elasticsearch/${PROJECT_ID}/status"
```

Example response:

```console-response
{
    "phase":"initializing"
}
```

### Reset credentials [general-manage-project-with-api-reset-credentials]

If you lose the credentials provided at the time of the project creation, you can reset the credentials by using the following endpoint:

```bash
curl -H "Authorization: ApiKey $API_KEY" \
    -XPOST \
    "https://api.elastic-cloud.com/api/v1/serverless/projects/elasticsearch/${PROJECT_ID}/_reset-credentials"
```

### Delete a project [general-manage-project-with-api-delete-project]

You can delete your project via the API:

```bash
curl -XDELETE -H "Authorization: ApiKey $API_KEY" \
    "https://api.elastic-cloud.com/api/v1/serverless/projects/elasticsearch/${PROJECT_ID}"
```

### Update a project [general-manage-project-with-api-update-project]

You can update your project using a PATCH request. Only the fields included in the body of the request will be updated.

```bash
curl -H "Authorization: ApiKey $API_KEY" \
    -H "Content-Type: application/json" \
    "https://api.elastic-cloud.com/api/v1/serverless/projects/elasticsearch/${PROJECT_ID}" \
    -XPATCH --data '{
        "name": "new name",
        "alias": "new-project-alias"
     }'
```

### List available regions [general-manage-project-with-api-list-available-regions]

You can obtain the list of regions where projects can be created using the API:

```bash
curl -H "Authorization: ApiKey $API_KEY" \
    "https://api.elastic-cloud.com/api/v1/serverless/regions"
```
