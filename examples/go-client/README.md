## Teleport Auth Go Client

### Introduction

This program demonstrates how to...

1. Authenticate the client using credential loaders.
2. Authorize API calls using an independent user and role.
3. Create a new client and make API calls.

### Authentication

Auth API clients must perform two-way authentication using x509 certificates to:

1. Validate the auth server x509 certificate to make sure the API endpoint can be trusted.
2. Offer their x509 certificate, which has been previously issued by the auth sever.

Credential loaders from the `github.com/gravitataional/teleport/api/client` package can be used to find and load certificates generated by `tctl` and `tsh` commands.

### Authorization

The server will authorize requests for the user associated with the certificates used to authenticate the client. Therefore, to use the API client, you need to create a user and any roles it may need for your use case. 

The client will act on behalf of that user and have access as defined by the user's role(s). It is recommended to create an independent user and role to manage API access control.

### Demo

This demo can be used to quickly get the API client up and running.

##### Create resources

Create the `access-admin` user and role using the following commands:

```bash
$ tctl create -f access-admin.yaml
$ tctl users add access-admin --roles=access-admin
```

##### Generate Credentials

This demo uses the Profile credential loader. Login with `tsh` and the Client will use the credentials from the profile directory (`~/.tsh`).

```bash
# login and automatically generate keys
$ tsh login --user=access-admin
```

NOTE: You can pass the `InsecureAddressDiscovery` in `client.Config` field to skip verification of the TLS certificate of the proxy. This is not recommended for production clients.

##### Run

```bash
$ go run main.go
```

To see more information on the Go Client and how to use it, visit our [API Documentation](https://goteleport.com/teleport/docs/api-reference/).
