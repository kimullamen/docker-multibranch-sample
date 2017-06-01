#!/usr/bin/env groovy
pipeline {
    agent any

    stages {

        stage('setup') {
            steps {
                // https://github.com/moby/moby/issues/2259
                // マウントするディレクトリがないとrootで作られるため、ディレクトリを作っておく
                sh 'mkdir -p /tmp/docker/cache/.node_modules || true'
                sh 'mkdir -p /tmp/docker/cache/.m2 || true'
            }
        }
        stage('build') {
            steps {
                sh 'make ID=${BUILD_ID} -f Makefile.docker build'
                archiveArtifacts 'server/target/*jar'
            }
        }
        stage('unit') {
            steps {
                sh 'make ID=${BUILD_ID} -f Makefile.docker unit'
                junit 'server/target/surefire-reports/*.xml'
                jacoco()
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: true, reportDir: 'client/test/report/coverage/lcov-report/', reportFiles: 'index.html', reportName: 'カバレッジ（クライアント）', reportTitles: ''])
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: true, reportDir: 'client/test/report/karma/', reportFiles: 'index.html', reportName: 'Karma', reportTitles: ''])
            }
        }
        stage('lint') {
            steps {
                sh 'make ID=${BUILD_ID} -f Makefile.docker lint'
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: true, reportDir: 'client/test/report/eslint/', reportFiles: 'index.html', reportName: 'ESLint', reportTitles: ''])
                findbugs canComputeNew: false, defaultEncoding: '', excludePattern: '', healthy: '', includePattern: '', pattern: 'server/target/findbugsXml.xml', unHealthy: ''
                checkstyle canComputeNew: false, defaultEncoding: '', healthy: '', pattern: 'server/target/checkstyle-result.xml', unHealthy: ''
            }
        }

        stage('clean') {
            deleteDir()
        }
    }
}
