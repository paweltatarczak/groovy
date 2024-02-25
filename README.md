## FAQ

### Q: How to use prettyPrint to print json

```Groovy
import static groovy.json.JsonOutput.*

json = '{"uploader":{"name":"Surname, Name", "email":"Surname.Name@mail.com"}}'
println prettyPrint(json)
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

### Q: How to find unsuccessful builds from the last 24h and sort results by first column (string)

```Groovy
def sortJobs(jobs) {
    return jobs.sort { a,b -> "$a[0]".toLowerCase()<=>"$b[0]".toLowerCase() }
}


def date = new Date()
def today = date.getTime()
def yesterday = date.getTime()-24*60*60*1000

def failedJobs = []
Jenkins.instance.getAllItems(Job.class).each { job ->
    job.getBuildsByTimestamp(yesterday, today).each { build ->
        if (build.getResult().toString() == 'FAILURE' || build.getResult().toString() == 'ABORTED') {
            failedJobs << [build, build.getUrl(), build.getResult().toString(), build.number]
        }
    }
}
println sortJobs(failedJobs)
```

### Q: How to read basic information about worker nodes

```Groovy
Jenkins.instance.computers.each { c ->
    if (c.isOnline()) {
        def diskMon = c.getMonitorData()['hudson.node_monitors.DiskSpaceMonitor'] =~ /.+ (.+)GB left on (.+)\./
        println "${c.getDisplayName()} has ${diskMon[0][1]}GB left on ${diskMon[0][2]}"
    } else {
        println "Node ${c.getDisplayName()} is offline. Data is not available"
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

### Q: How to use simple template engine

```Groovy
String template = '''
<h1>Template</h1>
<h2>Title: ${status}</h2>
<% if (showTable) { %>
<table><tr><td>Column 1</td><td>Colum 2</td></table>
<% } %>
My favourite animals:<ul>
<% animals.each { animal -> %>
<li>${animal}
<% } %></ul>
'''

def values = ['status' : 'SUCCESS', 'showTable' : true, animals: ['cat', 'dog', 'frog']]
def engine = new groovy.text.SimpleTemplateEngine()
def createdTemplate = engine.createTemplate(template).make(values)
print createdTemplate.toString()
```