# gh-version-stream
Script em python para versionar releases no Github e em containeres para aplicações em Golang 

## Examples

### Exemplo de Dockerfile

```Dockerfile
FROM alpine:latest

ARG GH_TOKEN
ARG GH_RELEASE_VERSION
ARG TAG_VERSION
ARG GH_REPO_NAME

ENV TIMEZONE America/Sao_Paulo
ENV GH_URL=""
ENV GH_RELEASE_VERSION=${GH_RELEASE_VERSION}
ENV TAG_VERSION=${TAG_VERSION}
ENV GH_REPO_NAME=${GH_REPO_NAME}

# Set the working directory
WORKDIR /app

RUN apk add curl jq

RUN echo "Creating release for ${GH_REPO_NAME} with version ${GH_RELEASE_VERSION} with tag ${TAG_VERSION}"

RUN curl -s --header 'Accept: application/octet-stream' --header 'Accept: application/vnd.github+json' --header "Authorization: Bearer ${GH_TOKEN}" https://api.github.com/repos/josiasnonato/${GH_REPO_NAME}/releases | jq ".[] | select(.name == \"${GH_RELEASE_VERSION}\") | .assets[] | select(.name == \"${GH_REPO_NAME}-${TAG_VERSION}\") | .id" >> /app/gh_asset_id.txt

RUN echo "Github Asset URL: $(cat /app/gh_asset_id.txt)"

RUN curl -s --header 'Accept: application/octet-stream' --header "X-GitHub-Api-Version: 2022-11-28" --header "Authorization: Bearer ${GH_TOKEN}" https://api.github.com/repos/josiasnonato/${GH_REPO_NAME}/releases/assets/$(cat /app/gh_asset_id.txt) -L -o "/app/${GH_REPO_NAME}-${TAG_VERSION}"

RUN rm /app/gh_asset_id.txt

RUN echo "Downloaded asset to /app/${GH_REPO_NAME}-${TAG_VERSION}"

RUN chmod 755 "/app/${GH_REPO_NAME}-${TAG_VERSION}"

# Command to run the application
CMD /app/${GH_REPO_NAME}-${TAG_VERSION}
```

### Comandos

```sh
./gh-releases.py  --create-image ./Dockerfile-Release --go-build --podman --gh-token "github_pat_XXXXXXXXXXXXXXXX..."
```
