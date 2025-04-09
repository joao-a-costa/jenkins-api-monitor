pipeline {
    agent any

    environment {
        TIMEZONE = 'CET'
    }

    stages {
        stage('API Health Check with Retry Logic') {
            steps {
                script {
                    def now = new Date()
                    def hour = now.format('HH', TimeZone.getTimeZone(env.TIMEZONE)).toInteger()
                    def withinWorkHours = (hour >= 9 && hour < 18)

                    def workHoursOnlyAPIs = [
                        "http://intranet.example.com/"
                    ]

                    def apis = [
                        [name: "API One", url: "http://api1.example.com/health", key: "APIKEY1"],
                        [name: "API Two", url: "http://api2.example.com/health", key: "APIKEY2"],
                        [name: "API Three", url: "http://api3.example.com/health", key: "APIKEY3"],
                        [name: "API Four", url: "http://api4.example.com/health", key: "APIKEY4"],
                        [name: "API Five", url: "http://api5.example.com/health", key: "APIKEY5"],
                        [name: "API Six", url: "http://api6.example.com/health", key: "APIKEY6"],
                        [name: "API Demo", url: "http://demo.example.com/health", key: "DEMOKEY"],
                        [name: "http://intranet.example.com/", url: "http://intranet.example.com/health", key: "DEMOKEY"]
                    ]

                    def toNotify = []
                    def recovered = []

                    sh 'mkdir -p .failures'

                    apis.each { api ->
                        def cmd = """
                        curl -s -o /dev/null -w "%{http_code}" --max-time 10 -X GET --header 'Accept: application/json' --header 'x-api-key: ${api.key}' '${api.url}'
                        """
                        def response = ""
                        try {
                            response = sh(script: cmd, returnStdout: true).trim()
                        } catch (Exception e) {
                            echo "❌ ${api.name} failed to connect"
                            response = "FAILED"
                        }

                        def safeName = api.name.replaceAll(/[^a-zA-Z0-9]/, "_")
                        def failFile = ".failures/${safeName}.txt"
                        def statusFile = ".failures/${safeName}_status.txt"
                        def failureCount = fileExists(failFile) ? readFile(failFile).trim().toInteger() : 0
                        def lastStatus = fileExists(statusFile) ? readFile(statusFile).trim() : "OK"

                        if (response == "200") {
                            echo "✅ ${api.name} OK"
                            writeFile file: failFile, text: "0"
                            if (lastStatus == "FAIL") {
                                if (!workHoursOnlyAPIs.contains(api.name) || withinWorkHours) {
                                    recovered << "${api.name} - Now OK"
                                } else {
                                    echo "⏰ Skipping recovery notification for ${api.name} (outside working hours)"
                                }
                            }
                            if (fileExists(failFile)) { sh "rm ${failFile}" }
                            if (fileExists(statusFile)) { sh "rm ${statusFile}" }
                        } else {
                            failureCount++
                            writeFile file: failFile, text: failureCount.toString()
                            echo "❌ ${api.name} NOT OK — Status: ${response} — Fail count: ${failureCount}"

                            if (failureCount % 3 == 0) {
                                if (!workHoursOnlyAPIs.contains(api.name) || withinWorkHours) {
                                    toNotify << "${api.name} — Status: ${response} (Failed ${failureCount} times) — URL: ${api.url}"
                                } else {
                                    echo "⏰ Skipping notification for ${api.name} (outside working hours)"
                                }
                            }
                        }

                        writeFile file: statusFile, text: (response == "200") ? "OK" : "FAIL"
                    }

                    def message = ""
                    if (!toNotify.isEmpty()) {
                        message += "🚨 **API Check Failed for**:\n" + toNotify.collect { "- ${it}" }.join("\n") + "\n\n"
                    }
                    if (!recovered.isEmpty()) {
                        message += "🎉 **API Recovery Detected for**:\n" + recovered.collect { "- ${it}" }.join("\n") + "\n\n"
                    }

                    if (message) {
                        withCredentials([string(credentialsId: 'AlertsTeamsWebhookPlaceholder', variable: 'AlertsTeamsWebhook')]) {
                            def payload = """
                            {
                              "@type": "MessageCard",
                              "@context": "http://schema.org/extensions",
                              "summary": "API Health Check Alert",
                              "themeColor": "FF0000",
                              "title": "⚠️ API Health Check Alert",
                              "text": "${message}"
                            }
                            """
                            sh """
                            curl -H 'Content-Type: application/json' \\
                                 -d '${payload}' \\
                                 "\${AlertsTeamsWebhook}"
                            """
                        }
                    } else {
                        echo "✅ All good. No notifications to send."
                    }
                }
            }
        }
    }
}
