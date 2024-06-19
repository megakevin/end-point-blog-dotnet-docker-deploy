# .NET Docker deploy demo

This repo contains the configuration for deploying the [Vehicle Quotes](https://github.com/megakevin/end-point-blog-dotnet-8-demo) system components. A .NET demo application.

It uses [Docker](https://www.docker.com/) and [Docker Compose](https://docs.docker.com/compose/). It is organized using [git submodules](https://www.atlassian.com/git/tutorials/git-submodule).

## Deployment summary

In order to deploy, do the following:

1. Checkout the desired tag or commit in `source`.
2. Check that the correct values are set in the `.env` file and in the files inside the `secrets` dir.
3. Rebuild and restart the system with `docker compose up -d --build`.
4. If there are database changes, run the migrations with:
   1. `docker compose exec maintenance bash` to connect to the maintenance container.
   2. `export ConnectionStrings__VehicleQuotesContext=$(cat /run/secrets/vehicle-quotes-db-connection-string)` to load the connection string into an env var.
   3. `dotnet ef migrations list --startup-project ./VehicleQuotes.AdminPortal --project ./VehicleQuotes.Core` to check if there are pending migrations.
   4. `dotnet ef database update --startup-project ./VehicleQuotes.AdminPortal --project ./VehicleQuotes.Core` to run the migrations.
5. Now the various system components should be running and accessible via `localhost`, each in a separate port. The default configuration (as given by the `.env` file) is this:
   1. The Admin Portal on port 8001.
   2. The Web Api on port 8002.
   3. The database on port 5432.

> If you want to shut down the system, you can stop the containers with `docker compose stop`.

> If you want to monitor the system while deploying, tail the logs with `docker compose logs -f`.

## Stop and tear down

You can stop all containers with `docker compose stop`. This will not delete the containers, it will only stop them.

Tear down with `docker compose down`. This will delete all containers.

These commands need to be run from within this repo's root directory.

## Logs

You can access the logs on each of the individual apps. This can be done with:

- For the Admin Portal: `docker compose logs -f admin-portal`.
- For the Web API: `docker compose logs -f web-api`.
- For the database: `docker compose logs -f db`.

You can see the logs from all the containers in a single stream with:

```sh
docker compose logs -f
```

These commands need to be run from within this repo's root directory.

## Database

The database process will run in a container as well. The data files will be stored in a [Volume](https://docs.docker.com/storage/volumes/) named `db-postgres-data`.

You can connect to the database using:

```sh
psql -h localhost -d vehicle_quotes -U vehicle_quotes -W
```

You can also connect to it from within the `maintenance` container. First connect to the container. From within this repo's root directory, this can be done like so:

```sh
docker compose exec maintenance bash
```

And then:

```sh
psql -h db -d vehicle_quotes -U vehicle_quotes -W
```

> The password can be found in `secrets/vehicle-quotes-db-password.txt`.

## Keys for .NET's Data Protection API

Our system uses cookies which are protected using [.NET's Data Protection API](https://learn.microsoft.com/en-us/aspnet/core/security/data-protection/introduction?view=aspnetcore-8.0). To make sure it works well across deployments, part of its configuration involves storing the keys in a non-volatile location. The `data-protection-keys` directory serves that purpose.

More info here:
- [The key was not found in the key ring. Unable to validate token](https://stackoverflow.com/a/63319132)
- [Configure ASP.NET Core Data Protection - PersistKeysToFileSystem](https://learn.microsoft.com/en-us/aspnet/core/security/data-protection/configuration/overview?view=aspnetcore-8.0#persistkeystofilesystem)
- [Configure ASP.NET Core Data Protection - Persisting keys when hosting in a Docker container](https://learn.microsoft.com/en-us/aspnet/core/security/data-protection/configuration/overview?view=aspnetcore-8.0#persisting-keys-when-hosting-in-a-docker-container)

## Dangling container images

During normal operations of building and rebuilding images and containers, the docker engine will accumulate old, untagged, unreferenced images on disk. These are called "dangling images".

To see all the existing images, use `docker image ls -a`. The ones which have `<none>` in their TAG and REPOSITORY are the dangling ones.

From time to time, it's good to clean these up. This can be done with `docker image prune`.

> https://docs.docker.com/reference/cli/docker/image/prune/
