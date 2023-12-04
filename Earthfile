VERSION 0.7

generate-git-hooks:
    FROM alpine:3.14

    RUN mkdir hooks

    # check message hook linux
    RUN echo "#! /bin/sh" > hooks/commit-msg
    RUN echo "earthly +lintCommit" >> hooks/commit-msg
    RUN chmod +x hooks/commit-msg
    # check message hook windows 
    RUN echo "earthly +lintCommit" > hooks/commit-msg.bat


    # Create the pre-commit hook linux
    RUN echo "#! /bin/sh" > hooks/pre-commit
    RUN echo "earthly +lint" >> hooks/pre-commit
    RUN chmod +x hooks/pre-commit

    # Create the pre-commit hook windows
    RUN echo "earthly +lint" > hooks/pre-commit.bat


    # Create the pre-push hook linux
    RUN echo "#! /bin/sh" > hooks/pre-push
    RUN echo "earthly +test" > hooks/pre-push
    RUN echo "earthly +init-repo" >> hooks/pre-push
    RUN chmod +x hooks/pre-push

    # Create the pre-push hook windows
    RUN echo "earthly +test" > hooks/pre-push.bat
    RUN echo "earthly +init-repo" >> hooks/pre-push.bat

    # Save the hooks as build artifacts
    SAVE ARTIFACT --keep-own hooks/commit-msg AS LOCAL ./.git/hooks/commit-msg
    SAVE ARTIFACT --keep-own hooks/commit-msg.bat AS LOCAL ./.git/hooks/commit-msg.bat
    SAVE ARTIFACT --keep-own hooks/pre-commit AS LOCAL ./.git/hooks/pre-commit
    SAVE ARTIFACT --keep-own hooks/pre-commit.bat AS LOCAL ./.git/hooks/pre-commit.bat
    SAVE ARTIFACT --keep-own hooks/pre-push AS LOCAL ./.git/hooks/pre-push
    SAVE ARTIFACT --keep-own hooks/pre-push.bat AS LOCAL ./.git/hooks/pre-push.bat

sync:  
    BUILD +generate-git-hooks

lintCommit: 
    DO +LINT_COMMIT
lint: 
    DO +LINT
test:
    DO +TEST 

LINT_COMMIT:
    COMMAND
    FROM registry.hub.docker.com/commitlint/commitlint:latest
    COPY .git/ .git/
    RUN echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
    RUN cat .git/COMMIT_EDITMSG
    RUN commitlint --edit ${1}

LINT: 
    COMMAND
    FROM mcr.microsoft.com/dotnet/sdk:7.0
    ARG csproj_root=./src
    COPY --if-exists $csproj_root ./src

    IF find ./src -maxdepth 1 \( -name "*.csproj" -o -name "*.fsproj" -o -name "*.vbproj" \)
      RUN dotnet format --no-restore --verify-no-changes ./src
    ELSE 
      RUN echo "No .NET project found, skipping format" 
    END

TEST: 
    COMMAND
    FROM mcr.microsoft.com/dotnet/sdk:7.0
    ARG csproj_root=./src
    COPY --if-exists $csproj_root ./src

    IF find ./src -maxdepth 1 \( -name "*.csproj" -o -name "*.fsproj" -o -name "*.vbproj" \)
      DO +RESTOR_DOTNET --csproj_file="$csproj_root/*.csproj"
      RUN dotnet test --no-restore ./src
    ELSE 
      RUN echo "No .NET project found, skipping test" 
    END


RESTOR_DOTNET:
    COMMAND
    ARG csproj_file
    COPY $csproj_file src/
    RUN dotnet restore src/

BUILD_DOTNET:
    COMMAND
    ARG csproj_root
    DO +RESTOR_DOTNET --csproj_file="$csproj_root/*.csproj"
    COPY $csproj_root src

    RUN dotnet restore src/
    # make sure you have /bin and /obj in .earthlyignore, as their content from context might cause problems
    RUN dotnet publish --no-restore src/ -o publish
    SAVE ARTIFACT publish AS LOCAL publish


CREACT_GITVERSION_CONF:
    COMMAND
    RUN mkdir /repo
    RUN echo "
mode: Mainline
branches: {
  main: {
    regex: '^master$|^main$',
    tag: useBranchName
  },
  develop: {
    increment: Patch,
    tag: useBranchName,
    is-mainline: true
  },
  feature: {
    regex: '^SCCC?[/-]',
    increment: Patch,
    tag: useBranchName,
    is-mainline: true
  }
}
ignore:
  sha: []
merge-message-formats: {}
    " > /repo/GitVersion.yml

GITVERSION:
    COMMAND
    FROM gittools/gitversion 
    RUN apt update && apt install jq -y
    ARG git_root

    DO +CREACT_GITVERSION_CONF
    COPY "$git_root/.git" /repo/.git
    ENTRYPOINT ["/tools/dotnet-gitversion"]

    RUN /tools/dotnet-gitversion /repo # print output to stdout
    RUN /tools/dotnet-gitversion /repo | jq -r "[.Major, .Minor, .Patch, .PreReleaseLabel | tostring ] | join(\" \")" > gitversion.json
    SAVE ARTIFACT gitversion.json 
    SAVE ARTIFACT gitversion.json AS LOCAL publish/gitversion.json

SAVE_IMAGS_WITH_GITVERSION_TAGS:
    COMMAND
    ARG CI_REGISTRY_IMAGE
    

    # Set ARG variables using RUN
    RUN read MAJOR MINOR PATCH PRE_RELEASE < gitversion.json && \
        IMAGE=${IMAGE_NAME:+/$IMAGE_NAME} && \
        APPEND_RELEASE=${PRE_RELEASE:+-$PRE_RELEASE} && \
        TAG_LATEST="${CI_REGISTRY_IMAGE}${IMAGE}:${PRE_RELEASE:-latest}" && \
        TAG_MAJOR="${CI_REGISTRY_IMAGE}${IMAGE}:${MAJOR}${APPEND_RELEASE}" && \
        TAG_MINOR="${CI_REGISTRY_IMAGE}${IMAGE}:${MAJOR}.${MINOR}${APPEND_RELEASE}" && \
        TAG_PATCH="${CI_REGISTRY_IMAGE}${IMAGE}:${MAJOR}.${MINOR}.${PATCH}${APPEND_RELEASE}" && \
        echo "$TAG_LATEST" > /tmp/tags && \
        echo "$TAG_MAJOR" >> /tmp/tags && \
        echo "$TAG_MINOR" >> /tmp/tags && \
        echo "$TAG_PATCH" >> /tmp/tags

    # Save image with different tags
    FOR tag IN $(cat /tmp/tags)
        SAVE IMAGE --push $tag
    END
     

BUILD_DOCKER_IMAGE:
    COMMAND
    ARG executable 
    ARG CI_REGISTRY_IMAGE=gitlab/example/registry
    ENTRYPOINT ./$executable  
    DO +SAVE_IMAGS_WITH_GITVERSION_TAGS --CI_REGISTRY_IMAGE="$CI_REGISTRY_IMAGE"


