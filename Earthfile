VERSION 0.7

generate-git-hooks:
    FROM alpine:3.14

    # Create the pre-commit hook linux
    RUN mkdir hooks
    RUN echo "#! /bin/sh" > hooks/pre-commit
    RUN echo "earthly +lint" >> hooks/pre-commit
    RUN chmod +x hooks/pre-commit

    # Create the pre-commit hook windows
    RUN echo 'earthly +lint' >> hooks/pre-commit.bat

    RUN echo "#! /bin/sh" > hooks/commit-msg
    RUN echo "earthly +lintCommit" >> hooks/commit-msg
    RUN chmod +x hooks/commit-msg
    RUN echo "earthly +lintCommit" > hooks/commit-msg.bat

    # Create the pre-push hook
    RUN echo "#! /bin/sh" > hooks/pre-push
    RUN echo "earthly +test" > hooks/pre-push
    RUN chmod +x hooks/pre-push

    # Create the pre-push hook windows
    RUN echo 'earthly +lint' >> hooks/pre-push.bat

    # Save the hooks as build artifacts
    SAVE ARTIFACT --keep-own hooks/pre-commit AS LOCAL ./.git/hooks/pre-commit
    SAVE ARTIFACT --keep-own hooks/pre-commit.bat AS LOCAL ./.git/hooks/pre-commit.bat
    SAVE ARTIFACT --keep-own hooks/pre-push AS LOCAL ./.git/hooks/pre-push
    SAVE ARTIFACT --keep-own hooks/pre-push.bat AS LOCAL ./.git/hooks/pre-push.bat

lintCommit:
    FROM registry.hub.docker.com/commitlint/commitlint:latest
    RUN npx commitlint --from=HEAD~1

lint: 
    FROM mcr.microsoft.com/dotnet/sdk:7.0
    RUN dotnet format

test: 
    FROM mcr.microsoft.com/dotnet/sdk:7.0
    RUN dotnet test
