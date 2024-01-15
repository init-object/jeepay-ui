pipeline {
    parameters {
        string(name: 'app_version', defaultValue: '2.2.2', description: '版本')
        gitParameter branchFilter: 'origin/(.*)', defaultValue: 'develop', name: 'BRANCH', type: 'PT_BRANCH'
        booleanParam(name: 'build_jeepay_payment', defaultValue: true, description: '是否构建jeepay-payment-ui')
        booleanParam(name: 'build_jeepay_manager', defaultValue: true, description: '是否构建jeepay-manager-ui')
        booleanParam(name: 'build_jeepay_merchant', defaultValue: true, description: '是否构建jeepay-merchant-ui')
        booleanParam(name: 'is_deploy', defaultValue: false, description: '是否deploy')
        string(name: 'notify_user_id', defaultValue: 'ggggfjhhqepwq1314', description: '通知群组或用户id')
        text(name: 'more_info', defaultValue: '', description: '通知信息')
    }
    environment {
        BRANCH = 'master'
        PAYMENT_APP_NAME = 'jeepay-payment-ui'
        MANAGER_APP_NAME = 'jeepay-manager-ui'
        MERCHANT_APP_NAME = 'jeepay-merchant-ui'
        GITHUB_URL = 'git@github.com:init-object/jeepay-ui.git'
        GITHUB_CREDENTIAL_ID = 'github'
        DOCKER_REGISTRY = 'registry.cn-hangzhou.aliyuncs.com'
        DOCKER_NAMESPACE = 'wang-kun'
        DOCKER_CREDENTIAL_ID = 'aliyun_docker_user'
    }
    agent {
        kubernetes {
            inheritFrom 'maven-docker'
            yaml '''
'''
        }
    }
    stages {
        stage('git checkout') {
            steps {
                git branch: "${params.BRANCH}", credentialsId: "${GITHUB_CREDENTIAL_ID}", url: "${GITHUB_URL}"
            }
        }

        stage('docker buildx ') {
            steps {
                container('docker') {
                    sh 'docker version'
                    sh 'docker pull tonistiigi/binfmt:latest'
                    sh 'docker run --rm --privileged tonistiigi/binfmt:latest --install all'
                    sh 'docker buildx version'
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIAL_ID}", passwordVariable: 'password', usernameVariable: 'username')]) {
                        sh "docker login -u $username ${DOCKER_REGISTRY} -p $password"
                    }
                    sh "docker buildx create --name builder-x --driver docker-container --buildkitd-flags '--allow-insecure-entitlement security.insecure --allow-insecure-entitlement network.host' --use"
                    sh 'docker buildx use builder-x'
                }
            }
        }
        stage('docker build build_jeepay_payment  ') {
            when { expression { return params.build_jeepay_payment } }
            steps {
                container('docker') {
                    sh "docker buildx build  . -f Dockerfile-cashier --build-arg PLATFORM=cashier --platform linux/amd64,linux/arm64 --tag ${DOCKER_REGISTRY}/${DOCKER_NAMESPACE}/${PAYMENT_APP_NAME}:v${params.app_version} --push  "
                }
            }
        }
        stage('docker build build_jeepay_manager  ') {
            when { expression { return params.build_jeepay_manager } }
            steps {
                container('docker') {
                    sh "docker buildx build  . -f Dockerfile --build-arg PLATFORM=manager --platform linux/amd64,linux/arm64 --tag ${DOCKER_REGISTRY}/${DOCKER_NAMESPACE}/${MANAGER_APP_NAME}:v${params.app_version} --push  "
                }
            }
        }
        stage('docker build build_jeepay_merchant  ') {
            when { expression { return params.build_jeepay_merchant } }
            steps {
                container('docker') {
                    sh "docker buildx build  . -f Dockerfile --build-arg PLATFORM=merchant --platform linux/amd64,linux/arm64 --tag ${DOCKER_REGISTRY}/${DOCKER_NAMESPACE}/${MERCHANT_APP_NAME}:v${params.app_version} --push  "
                }
            }
        }

        stage('logout docker') {
            steps {
                container('docker') {
                    sh "docker logout ${DOCKER_REGISTRY}"
                }
            }

        }
        stage('auto deploy') {
            when { expression { return params.is_deploy } }
            steps {
                build wait: false, job: 'jeepay-deploy', parameters: [string(name: 'app_version', value: "${params.app_version}"), string(name: 'notify_user_id', value: "${params.notify_user_id}"), text(name: 'more_info', value: "${params.more_info}"), booleanParam(name: 'issue_cert', value: false), booleanParam(name: 'deploy_jeecg_system_boot', value: "${params.build_jeecg_system_boot}"), booleanParam(name: 'deploy_jeecg_web', value: false), booleanParam(name: 'deploy_linktech_app_boot', value: "${params.build_linktech_app_boot}")]
            }
        }

    }
}
