# Keycloak - GraphDB - nginx

This is a POC Docker orchestration using [Keycloak](https://www.keycloak.org/) as authentication service, [GraphDB](https://graphdb.ontotext.com/) as a resource server/client and [nginx](https://nginx.org/en/) as proxy server.

## Configuration Notes

Here are some important configuration notes for the whole orchestration as well as individual services.

### Keycloak

Use the exported realm configuration. It contains a client configured for GraphDB running at `http://172.17.0.0.1` (Docker public interface on my computer - update this value as necessary). The GraphDB client (`graphdb`) is set to use `public` access type, mainly because the GraphDB workbench UI is calling Keycloak, so there is no secret-based access. It also defines a couple of roles specific for GraphDB (they start with the `GDB_` prefix). GraphDB admin should be assigned the `GDB_ADMIN_ROLE`. Anyone without a role will have the role `ROLE_USER` assigned by GraphDB itself (see below). There are two scopes required by the client. One is a role mapping scope that adds a `roles` claim to the access token (GraphDB does not support nested role claims that are by default provided by Keycloak). The other adds a `aud` claim to the access token.

### GraphDB

As for GraphDB, the following configuration is used (passed as `GDB_JAVA_OPTS`):
```
graphdb.auth.methods = openid
graphdb.auth.openid.issuer = http://172.17.0.1/auth/realms/termit
graphdb.auth.openid.client_id = graphdb
graphdb.auth.openid.username_claim = preferred_username
graphdb.auth.openid.auth_flow = code
graphdb.auth.openid.token_type = access
graphdb.auth.database = oauth
graphdb.auth.oauth.roles_claim = roles
graphdb.auth.oauth.roles_prefix = GDB_
graphdb.auth.oauth.default_roles = ROLE_USER
```

The configuration is based on (GraphDB Access Control docs)(https://graphdb.ontotext.com/documentation/10.3/access-control.html) and a helpful (Metaphacts blog post)[https://blog.metaphacts.com/sso-and-identity-management-with-metaphactory].

It is also necessary to enable security on first startup in GraphDB workbench, otherwise GraphDB will be accessible to anyone.

## TODO
- Get the setup working with Nginx in Docker. Currently, only running Keycloak and GraphDB on bare metal (without Docker) works.



