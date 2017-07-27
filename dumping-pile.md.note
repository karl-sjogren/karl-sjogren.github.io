# Dumping pile

Here a lot of stuff will land that might be noteworthy or possibly landed into a blog post in the future.

## Only publish nuget-packages from the develop-branch

    IF "%teamcity.build.branch%"=="develop" (
        echo ##teamcity[message text='Executing NuGet Publish for branch %teamcity.build.branch%']
        .nuget\nuget.exe push build\SagaClient.%env.sagaClientVersion%.nupkg 00000000-0000-0000-0000-000000000000 -Source https://www.myget.org/F/openlibrarysolutions/api/v2/package 
        .nuget\nuget.exe push build\SagaClient.%env.sagaClientVersion%.symbols.nupkg 00000000-0000-0000-0000-000000000000 -Source http://nuget.gw.SymbolSource.org/MyGet/openlibrarysolutions
    )
    
    IF NOT "%teamcity.build.branch%"=="develop" (
        echo ##teamcity[message text='Ignoring NuGet Publish for branch %teamcity.build.branch%']
    )

Branch specification

    +:refs/heads/(develop)
    +:refs/heads/feature/(*)
    +:refs/heads/hotfix/(*)
    +:refs/heads/(master)
    +:refs/heads/(release)
    +:refs/heads/support/(*)
