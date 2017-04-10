# Dumping pile

Here a lot of stuff will land that might be noteworthy or possibly landed into a blog post in the future.

## Only publish nuget-packages from the develop-branch

    IF "%teamcity.build.branch%"=="develop" (
        echo ##teamcity[message text='Executing NuGet Publish for branch %teamcity.build.branch%']
        .nuget\nuget.exe push build\SagaClient.%env.sagaClientVersion%.nupkg 6c25efdf-cb0d-4881-bd92-b2d04ec14262 -Source https://www.myget.org/F/openlibrarysolutions/api/v2/package 
        .nuget\nuget.exe push build\SagaClient.%env.sagaClientVersion%.symbols.nupkg 6c25efdf-cb0d-4881-bd92-b2d04ec14262 -Source http://nuget.gw.SymbolSource.org/MyGet/openlibrarysolutions
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
