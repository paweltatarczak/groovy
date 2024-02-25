## FAQ

### Q: How to use prettyPrint to print json

```Groovy
import static groovy.json.JsonOutput.*

json = '{"uploader":{"name":"Surname, Name", "email":"Surname.Name@mail.com"}}'
println prettyPrint(jsonik)
```

### Q: How to remove Jenkins builds with 'FAILURE' status

```Groovy
builds = Jenkins.instance.getItemByFullName("TestDir/TestJob")._getRuns()
builds.each { build ->
    result = build.getResult().toString()
    if (result == 'FAILURE') {
        println "${build} is being deleted"
        build.delete()
    }
}
```

### Q: How to check whether particular Stage was executed in Jenkins builds

```Groovy
builds = Jenkins.instance.getItemByFullName("TestDir/TestJob")._getRuns()
builds.each { build ->
    url = build.getAbsoluteUrl()
    withCredentials([usernamePassword(credentialsId: 'jenkinsCredentials', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
        String returnStdout = sh(script: """
            set +x
            curl -s --user $USERNAME:$PASSWORD -X GET ${url}wfapi/describe | jq -r '.stages[] | select(.name=="EXEMPLARY_STAGE_NAME") | .status' || echo 'unknown'
        """, returnStdout: true).trim()
        if (returnStdout == 'NOT_EXECUTED') {
            println "${build} didn't execute 'EXEMPLARY_STAGE_NAME' Stage"
        } else {
            println "${build} executed 'EXEMPLARY_STAGE_NAME' Stage"
        }
    }
}
```

### Q: How to read logs from triggered Jenkins build and parse line by line

```Groovy
def triggerredJob = build job: 'JobNameToBeTriggered',
    wait: true,
    propagate: false,
    parameters: [string(name: 'Exemplary_string_param', value: 'Exemplary_value'),
        booleanParam(name: 'Exemplary_boolean_param', value: true)]

logEntries = []
log = triggerredJob.getRawBuild().getLog(50)
log.each { line ->
    if (line =~ /.*FLAG:(.*)/) {
        print "DEBUG: $line"
        logEntries << line.split('FLAG: ')[1]
    }
}
print logEntries
```

### Q: How to use when statement

```Groovy
stage('when with env') {
    when {
        expression {
            env.EXEMPLARY_VARIABLE.toBoolean()
        }
    }
}
stage('When with param') {
    when {
        expression {
            params.EXEMPLARY_PARAM.toBoolean()
        }
    }
}
stage('When with Date') {
    when {
        expression {
            new Date().format('u') == '2'
        }
    }
    when {
        expression {
            Calendar.DAY_OF_WEEK == 7
        }
    }
}
```

### Q: How to use switch statement

```Groovy
int code = 105
switch (code) {
    case 0:
        echo 'Successful data analysis'
        break
    case 100:
        echo 'FAILED: Change is empty!'
        error('Change is empty')
        break
    case 101:
        echo 'FAILED: Change eleates to more than one database element (directory in database)!'
        error('Change eleates to more than one database element (directory in database)!')
        break
    default:
        error('Unkown error')
        break
}
```