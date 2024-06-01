pipeline {
    agent any
    environment{
        harborHost = '192.168.91.129:80'
        harborRepo = 'repository'
        harborUser = 'admin'
        harborPasswd = 'Harbor12345'
        HARBOR_PROJECT = 'repository/tasktest_pipeline'
    }


    // 存放所有任务的合集
    stages {

        stage('拉取Git代码') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '${tag}']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/iSmartSV/jenkinsTest.git']]])
            }
        }

        stage('构建代码') {
            steps {
                sh '/var/jenkins_home/apache-maven-3.9.6/bin/mvn clean package -DskipTests'
            }
        }

        stage('检测代码质量') {
            steps {
                sh '/var/jenkins_home/apache-maven-3.9.6/bin/mvn sonar:sonar -P sonar' 
            }
        }

        stage('制作自定义镜像并发布Harbor') {
            steps {
                sh '''cp ./target/*.jar ./
                docker build -t ${JOB_NAME}:${tag} ./'''

                sh '''docker login -u ${harborUser} -p ${harborPasswd} ${harborHost}
                docker tag ${JOB_NAME}:${tag} ${harborHost}/${harborRepo}/${JOB_NAME}:${tag}
                docker push ${harborHost}/${harborRepo}/${JOB_NAME}:${tag}'''
            }
        }
    }
}
def getHarborTags() {
    // 获取 Harbor 镜像标签
    def response = sh(script: "curl -u ${env.harborUser}:${env.harborPasswd} ${env.harborHost}/projects/${env.HARBOR_PROJECT}/repositories/${env.harborRepo}/artifacts", returnStdout: true).trim()
    def images = readJSON text: response
    def tags = images.collect { it.tags[0].name }
    return tags.join('\n')
}
