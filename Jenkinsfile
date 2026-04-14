pipeline {
    agent any

    tools {
        maven 'Maven3'
        jdk 'JDK17' 
    }

    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Branche Git à builder')
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Environnement de déploiement cible')
        booleanParam(name: 'SKIP_TESTS', defaultValue: false, description: 'Ignorer les tests')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "Branch  : ${env.GIT_BRANCH}"
                echo "Commit  : ${env.GIT_COMMIT}"
            }
        }

        stage('Build') {
            steps {
                // Correction ici : ajout du dossier parent
                dir('projet_jenkins/ICDE848') {
                    bat 'mvn clean compile -B'
                }
            }
        }

        stage('Tests unitaires') {
            when { not { expression { params.SKIP_TESTS } } }
            steps {
                dir('projet_jenkins/ICDE848') {
                    bat 'mvn test -B'
                }
            }
            post {
                always {
                    junit 'projet_jenkins/ICDE848/**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Tests intégration') {
            when { not { expression { params.SKIP_TESTS } } }
            steps {
                dir('projet_jenkins/ICDE848') {
                    bat 'mvn verify -Dsurefire.skip=true -B'
                }
            }
            post {
                always {
                    junit 'projet_jenkins/ICDE848/**/target/failsafe-reports/*.xml'
                }
            }
        }

        stage('Couverture JaCoCo') {
            steps {
                dir('projet_jenkins/ICDE848') {
                    bat 'mvn jacoco:report -B'
                }
            }
            post {
                always {
                    jacoco(
                        execPattern: 'projet_jenkins/ICDE848/**/target/jacoco.exec',
                        classPattern: 'projet_jenkins/ICDE848/**/target/classes',
                        sourcePattern: 'projet_jenkins/ICDE848/**/src/main/java',
                        minimumLineCoverage: '70'
                    )
                }
            }
        }

        stage('Qualité') {
            steps {
                dir('projet_jenkins/ICDE848') {
                    bat 'mvn checkstyle:checkstyle pmd:pmd pmd:cpd spotbugs:spotbugs -B'
                }
            }
            post {
                always {
                    recordIssues(
                        enabledForFailure: true,
                        tools: [
                            checkStyle(pattern: 'projet_jenkins/ICDE848/**/checkstyle-result.xml'),
                            pmdParser(pattern: 'projet_jenkins/ICDE848/**/pmd.xml'),
                            cpd(pattern: 'projet_jenkins/ICDE848/**/cpd.xml'),
                            spotBugs(pattern: 'projet_jenkins/ICDE848/**/spotbugsXml.xml')
                        ],
                        qualityGates: [[threshold: 10, type: 'TOTAL', unstable: true]]
                    )
                }
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts(
                    artifacts: 'projet_jenkins/ICDE848/**/target/*.jar',
                    fingerprint: true,
                    allowEmptyArchive: false
                )
            }
        }
    }

    post {
        always {
            echo "Pipeline terminée — statut : ${currentBuild.currentResult}"
        }
    }
}