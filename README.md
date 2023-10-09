# Keycloak - GraphDB - nginx

This is a POC Docker orchestration using [Keycloak](https://www.keycloak.org/) as authentication service, [GraphDB](https://graphdb.ontotext.com/) as a resource server/client and [nginx](https://nginx.org/en/) as proxy server.

## Configuration Notes

Here are some important configuration notes for the whole orchestration as well as individual services.

### Keycloak

Use the exported realm configuration (configured in `docker-compose.yml`). It contains a client configured for GraphDB running at `http://172.17.0.1` (Docker public interface on my computer - update this value as necessary). The GraphDB client (`graphdb`) is set to use `public` access type, mainly because the GraphDB workbench UI is calling Keycloak, so there is no secret-based access. It also defines a couple of roles specific for GraphDB (they start with the `GDB_` prefix). GraphDB admin should be assigned the `GDB_ADMIN_ROLE` and there should be at least one so that the admin can create repositories and provide other users access to them. Anyone without a role will have the role `ROLE_USER` assigned by GraphDB itself (see below). Roles `GDB_READ_REPO_*` and `GDB_WRITE_REPO_*` authorize the user to read/write to all repositories. These are the roles that should be used for regular users. It is also possible to allow access only to specific repositories (see the [docs](https://graphdb.ontotext.com/documentation/10.3/access-control.html#what-s-in-this-document) for details).

There are two scopes required by the client. One is a role mapping scope that adds a claim containing user roles (`realm_access.roles`) to the access token. The other adds an `aud` claim to the access token.

### GraphDB

As for GraphDB, the following configuration is used (passed as `GDB_JAVA_OPTS`):

```
graphdb.auth.methods = openid
graphdb.auth.openid.issuer = ${URL}/auth/realms/termit
graphdb.auth.openid.client_id = graphdb
graphdb.auth.openid.username_claim = preferred_username
graphdb.auth.openid.auth_flow = code
graphdb.auth.openid.token_type = access
graphdb.auth.database = oauth
graphdb.auth.oauth.roles_claim = realm_access.roles
graphdb.auth.oauth.roles_prefix = GDB_
graphdb.auth.oauth.default_roles = ROLE_USER
```

The configuration is based on [GraphDB Access Control docs](https://graphdb.ontotext.com/documentation/10.3/access-control.html) and a helpful [Metaphacts blog post](https://blog.metaphacts.com/sso-and-identity-management-with-metaphactory).

Additional GraphDB notes:
- It is necessary to enable security on first startup in GraphDB workbench (it is disabled by default), otherwise GraphDB will be accessible to anyone.
- Since Keycloak startup is quite slow and GraphDB checks for availability of the authentication service on startup, it may fail to start. In that case it is necessary to restart the GraphDB service (`restart: always` does not help).


## Configuration Customization

To run the orchestration somewhere else than on localhost (or when the Docker network adapter assigns a different IP  than `172.17.0.1`), the following need to be changed:

- Set `URL` in `.env` to the public URL of the target server
- Update `graphdb` client `redirectUris` and `webOrigins` values in `keycloak/realm-export.json` to the public URL of the server
- If necessary, service URL paths can be changed from `/auth` for Keycloak and `/db-server` for GraphDB. In this case, all their occurrences need to be updated - in `keycloak/realm-export.json`, `nginx/nginx.conf`, `docker-compose.yml`


## License

Individual services are licensed under their respective licenses. This orchestration configuration is licensed under the MIT license.

