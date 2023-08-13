# Open Cybersecurity Schema Framework (OCSF) Server Documentation

The Open Cybersecurity Schema Framework (OCSF) Server provides an HTTP-based platform for browsing and utilizing the OCSF schema. This documentation outlines how to set up and use the OCSF Schema Server, both locally and through Docker-based development. The OCSF Schema Server aids in improving security operations by allowing easy access to the OCSF schema, which has wide-ranging applications in cybersecurity.

## Accessing the OCSF Schema Server

The latest released version of the OCSF schema can be accessed through the OCSF Schema Server at [schema.ocsf.io](https://schema.ocsf.io).

## Running the OCSF Schema Server Locally

To run the OCSF Schema Server locally, follow these steps:

1. Clone the OCSF schema repository:

```shell
git clone https://github.com/ocsf/ocsf-schema.git
```

2. Clone the OCSF schema server repository:

```shell
git clone https://github.com/ocsf/ocsf-server.git
```

3. Build a Docker image for the server:

```shell
cd ocsf-server
docker build -t ocsf-server .
```

4. Run the server Docker image:

```shell
docker run -it --rm --volume /path/to/ocsf-schema:/app/schema -p 8080:8080 -p 8443:8443 ocsf-server
```

Replace `/path/to/ocsf-schema` with the absolute path to your local OCSF schema directory. The `-p 8443:8443` parameter enables HTTPS with a self-signed SSL certificate.

5. Access the schema server by opening `http://localhost:8080` or `https://localhost:8443` in your web browser.

## Running the OCSF Schema Server with a Local Schema Extension

You can extend the OCSF schema and test it using the following steps:

1. Run the server Docker image with a local schema extension:

```shell
docker run -it --rm --volume /path/to/ocsf-schema:/app/schema --volume /path/to/ocsf-schema:/app/extension -e SCHEMA_EXTENSION="/app/extension" -p 8080:8080 -p 8443:8443 ocsf-server
```

Replace `/path/to/ocsf-schema` with the absolute path to your local OCSF schema directory and the same path for the extension folder.

## Development with Docker-Compose

Docker-Compose provides an environment for development without additional dependencies:

1. Run the OCSF server and build the development container:

```shell
docker-compose up
```

2. Access the schema server by browsing to [http://localhost:8080](http://localhost:8080).

3. Test the schema with Docker-Compose:

```shell
docker-compose run ocsf-elixir mix test
```

## Local Usage and Development

For local development, the following steps are involved:

1. Install the required build tools for Elixir (Phoenix Web framework).

2. Fetch and compile the dependencies:

```shell
cd ocsf-server
mix do deps.get, deps.compile
```

3. Compile the source code:

```shell
mix compile
```

4. Test local schema changes:

```shell
SCHEMA_DIR=../ocsf-schema SCHEMA_EXTENSION=extensions mix test
```

5. Run the schema server:

```shell
SCHEMA_DIR=../ocsf-schema SCHEMA_EXTENSION=extensions iex -S mix phx.server
```

6. Access the schema server at `http://localhost:8080` or `https://localhost:8443`.

## Reloading the Schema

In the interactive shell (iex) of the schema server, you can force reload the schema with various extensions:

```elixir
# Reload the core schema without extensions
Schema.reload()

# Reload the schema only with the linux extension (folder relative to SCHEMA_DIR)
Schema.reload(["extensions/linux"])

# Reload the schema with all extensions defined in the extensions folder
Schema.reload(["extensions"])

# Reload the schema with extensions defined outside the SCHEMA_DIR folder (use an absolute or relative path)
Schema.reload(["/home/schema/cloud", "../dev-ext"])
```

## Runtime Configuration

The OCSF Schema Server utilizes environment variables for configuration:

- `HTTP_PORT`: The server HTTP port number (default: 8080).
- `HTTPS_PORT`: The server HTTPS port number (default: 8443).
- `SCHEMA_DIR`: The directory containing the schema (default: ../ocsf-schema).
- `SCHEMA_EXTENSION`: The directory containing the schema extensions, relative to SCHEMA_DIR or an absolute path.
- `RELEASE_NODE`: The Erlang node name (set it if running multiple servers on the same computer).

Example usage:

```shell
SCHEMA_DIR=../ocsf-schema SCHEMA_EXTENSION=extensions iex -S mix phx.server
```

By following these instructions, you can set up and run the OCSF Schema Server locally, test schema changes, and leverage its capabilities for cybersecurity purposes.
