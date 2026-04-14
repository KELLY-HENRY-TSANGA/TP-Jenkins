pipeline {
    agent any

    tools {
        maven 'Maven3'
        jdk 'JDK17'   // décommente seulement si configuré dans Jenkins
    }

    parameters {
        string(
            name: 'BRANCH',
            defaultValue: 'main',
            description: 'Branche Git à builder'
        )
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'staging', 'prod'],
            description: 'Environnement de déploiement cible'
        )
        booleanParam(
            name: 'SKIP_TESTS',
            defaultValue: false,
            description: 'Ignorer les tests (urgence uniquement !)'
        )
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
                dir('ICDE848') {
                    bat 'mvn clean compile -B'
                }
            }
        }

        stage('Tests unitaires') {
            when {
                not { expression { params.SKIP_TESTS } }
            }
            steps {
                dir('ICDE848') {
                    bat 'mvn test -B'
                }
            }
            post {
                always {
                    junit 'ICDE848/**/target/surefire-reports/*.xml'
                }
                failure {
                    echo 'Tests unitaires en ECHEC — vérifier les logs ci-dessus'
                }
            }
        }

        stage('Tests intégration') {
            when {
                not { expression { params.SKIP_TESTS } }
            }
            steps {
                dir('ICDE848') {
                    bat 'mvn verify -Dsurefire.skip=true -B'
                }
            }
            post {
                always {
                    junit 'ICDE848/**/target/failsafe-reports/*.xml'
                }
            }
        }

        stage('Couverture JaCoCo') {
            steps {
                dir('ICDE848') {
                    bat 'mvn jacoco:report -B'
                }
            }
            post {
                always {
                    jacoco(
                        execPattern: 'ICDE848/**/target/jacoco.exec',
                        classPattern: 'ICDE848/**/target/classes',
                        sourcePattern: 'ICDE848/**/src/main/java',
                        minimumLineCoverage: '70'
                    )
                }
            }
        }

        stage('Qualité') {
            steps {
                dir('ICDE848') {
                    bat 'mvn checkstyle:checkstyle pmd:pmd pmd:cpd spotbugs:spotbugs -B'
                }
            }
            post {
                always {
                    recordIssues(
                        enabledForFailure: true,
                        tools: [
                            checkStyle(pattern: 'ICDE848/**/checkstyle-result.xml'),
                            pmdParser(pattern: 'ICDE848/**/pmd.xml'),
                            cpd(pattern: 'ICDE848/**/cpd.xml'),
                            spotBugs(pattern: 'ICDE848/**/spotbugsXml.xml')
                        ],
                        qualityGates: [[
                            threshold: 10,
                            type: 'TOTAL',
                            unstable: true
                        ]]
                    )
                }
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts(
                    artifacts: 'ICDE848/**/target/*.jar',
                    fingerprint: true,
                    allowEmptyArchive: false
                )
                echo 'Artefact archivé avec succès'
            }
        }
    }

    post {
        always {
            echo "Pipeline terminée — statut : ${currentBuild.currentResult}"
        }

        failure {
            echo 'Envoi email ignoré ou à configurer plus tard'
        }

        fixed {
            echo 'Build de nouveau stable'
        }
    }
}
