podTemplate(
  containers: [
    containerTemplate(
        name: 'worker',
        image: 'node:lts-alpine',
        ttyEnabled: true, 
        command: 'cat',
        envVars: [
            containerEnvVar(key: 'THREAD', value: '4'),
            containerEnvVar(key: 'NUMBER_OF_RETRIES', value: '5'),
            containerEnvVar(key: 'OWNER', value: 'MaaAssistantArknights'),
            containerEnvVar(key: 'MINIO_BUCKET', value: 'maa-release'),
            containerEnvVar(key: 'MINIO_ENDPOINT_DOMAIN', value: 'minio.local'),
            containerEnvVar(key: 'MINIO_ENDPOINT_PORT', value: '9080'),
            containerEnvVar(key: 'MINIO_WAIT_TIME_AFTER_UPLOAD_MS', value: '1000'),
            containerEnvVar(key: 'RELEASE_TAG', value: params.release_tag)
        ]
    )
  ]
) {
  node(POD_LABEL) {
    stage('Checkout Repo') {
      container('worker') {
        sh 'apk --no-cache update'
        sh 'apk add git uuidgen parallel'
        sh 'git clone --depth 1 https://github.com/MaaAssistantArknights/MaaRelease.git'
      }
    }

    stage('Install the dependencies') {
      container('worker') {
        sh 'cd MaaRelease/scripts && npm run ciInCI'
      }
    }

    stage('Download files from GitHub Release and upload files to Minio') {
      withCredentials([
          string(credentialsId: 'maa-jenkins-robot-token', variable: 'GITHUB_PAT'),
          string(credentialsId: 'maa-minio-robot-access-key', variable: 'MINIO_ACCESS_KEY'),
          string(credentialsId: 'maa-minio-robot-secret-key', variable: 'MINIO_SECRET_KEY'),
          string(credentialsId: 'annangela-qqbot-token', variable: 'ANNANGELA_QQBOT_TOKEN')
      ]) {
          container('worker') {
            sh 'cd MaaRelease/scripts ; export TZ=Asia/Shanghai ; export OWNER=MaaAssistantArknights ; function MaaRelease_s3_sync() { REPO=MaaRelease node s3-sync/index.js; } ; function MaaAssistantArknights_s3_sync() { REPO=MaaAssistantArknights node s3-sync/index.js; } ; export -f MaaRelease_s3_sync ; export -f MaaAssistantArknights_s3_sync ; parallel ::: "MaaRelease_s3_sync" "MaaAssistantArknights_s3_sync" ; [ $? -ne 0 ] && s3-sync/errorReport.js'
          }
      }
      
    }
  }
}
