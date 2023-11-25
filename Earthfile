VERSION 0.7

generate-git-hooks:
    FROM alpine:3.14

    RUN mkdir hooks

    # check message hook linux
    RUN echo "#! /bin/sh" > hooks/commit-msg
    RUN echo "earthly github.com/marnyg/earthlytemplate+LINT_COMMIT" >> hooks/commit-msg
    RUN chmod +x hooks/commit-msg
    # check message hook windows 
    RUN echo "earthly github.com/marnyg/earthlytemplate+LINT_COMMIT" > hooks/commit-msg.bat


    # Create the pre-commit hook linux
    RUN echo "#! /bin/sh" > hooks/pre-commit
    RUN echo "earthly github.com/marnyg/earthlytemplate+LINT" >> hooks/pre-commit
    RUN chmod +x hooks/pre-commit

    # Create the pre-commit hook windows
    RUN echo "earthly github.com/marnyg/earthlytemplate+LINT" > hooks/pre-commit.bat


    # Create the pre-push hook linux
    RUN echo "#! /bin/sh" > hooks/pre-push
    RUN echo "earthly github.com/marnyg/earthlytemplate+TEST" > hooks/pre-push
    RUN chmod +x hooks/pre-push

    # Create the pre-push hook windows
    RUN echo "earthly github.com/marnyg/earthlytemplate+TEST" > hooks/pre-push.bat

    # Save the hooks as build artifacts
    SAVE ARTIFACT --keep-own hooks/commit-msg AS LOCAL ./.git/hooks/commit-msg
    SAVE ARTIFACT --keep-own hooks/commit-msg.bat AS LOCAL ./.git/hooks/commit-msg.bat
    SAVE ARTIFACT --keep-own hooks/pre-commit AS LOCAL ./.git/hooks/pre-commit
    SAVE ARTIFACT --keep-own hooks/pre-commit.bat AS LOCAL ./.git/hooks/pre-commit.bat
    SAVE ARTIFACT --keep-own hooks/pre-push AS LOCAL ./.git/hooks/pre-push
    SAVE ARTIFACT --keep-own hooks/pre-push.bat AS LOCAL ./.git/hooks/pre-push.bat

sync:  
    BUILD +generate-git-hooks

LINT_COMMIT:
    FROM registry.hub.docker.com/commitlint/commitlint:latest
    COPY .git/ .git/
    RUN echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
    RUN cat .git/COMMIT_EDITMSG
    RUN commitlint --edit ${1}

LINT: 
    FROM mcr.microsoft.com/dotnet/sdk:7.0
    COPY . .
    IF [ -f *.csproj ] -a [ -f *.fsproj ] -a [ -f *.vbproj ]
      RUN dotnet format
    ELSE 
      RUN echo "No .NET project found, skipping format" 
    END

TEST: 
    FROM mcr.microsoft.com/dotnet/sdk:7.0
    COPY . .
    IF [ -f *.csproj ] -a [ -f *.fsproj ] -a [ -f *.vbproj ]
      RUN dotnet test
    ELSE 
      RUN echo "No .NET project found, skipping test" 
    END

deps:
    COPY src/sendra-release-tools.csproj src/
    RUN dotnet restore src/

build:
    FROM +deps
    COPY src src

    RUN dotnet restore src/
    # make sure you have /bin and /obj in .earthlyignore, as their content from context might cause problems
    RUN dotnet publish --no-restore src/ -o publish

    SAVE ARTIFACT publish AS LOCAL publish

gitversion:
    FROM gittools/gitversion
    COPY . /repo
    ENTRYPOINT ["/tools/dotnet-gitversion"]
    RUN /tools/dotnet-gitversion /repo 
    RUN /tools/dotnet-gitversion /repo /output file /outputfile gitversion.json
    SAVE ARTIFACT gitversion.json 
    SAVE ARTIFACT gitversion.json AS LOCAL gitversion.json

docker-build:
    #EXAMPLE https://docs.earthly.dev/basics/part-4-args#passing-args-in-from-build-and-copy 
    FROM mcr.microsoft.com/dotnet/runtime:7.0
    COPY +build/publish .
    COPY +gitversion/gitversion.json .
    ##ENTRYPOINT ["dotnet", ".dll"]
    #TODO: look into the docker HEALTHCHECK command: https://scoutapm.com/blog/how-to-use-docker-healthcheck
    ENTRYPOINT ["ls"]
    SAVE IMAGE  earthly/examples:dotnet

