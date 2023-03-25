podTemplate(yaml: '''
    apiVersion: v1
    kind: Pod
    spec:
      containers:
      - name: cloud-sdk
        image: google/cloud-sdk
        command:
        - sleep
        args:
        - 999999
        volumeMounts:
        - name: shared-storage
          mountPath: /mnt
        - name: google-cloud-key
          mountPath: /var/secrets/google
        env:
        - name: GOOGLE_APPLICATION_CREDENTIALS
          value: /var/secrets/google/umls23-381500-911c6262500f.json    
      - name: gradle
        image: gradle:8-jdk8
        command:
        - sleep
        args:
        - 99d
        volumeMounts:
        - name: shared-storage
          mountPath: /mnt        
      - name: centos
        image: centos
        command:
        - sleep
        args:
        - 99d
      restartPolicy: Never
      volumes:
      - name: shared-storage
        persistentVolumeClaim:
          claimName: jenkins-pv-claim
      - name: google-cloud-key
        secret:
            secretName: sdk-key          
      - name: kaniko-secret
        secret:
            secretName: dockercred
            items:
            - key: .dockerconfigjson
              path: config.json
''') {
  node(POD_LABEL) {

  stage('k8s') {
    git branch: env.BRANCH_NAME, url: 'https://github.com/bsieraduml/week9.git'

    stage('Build a gradle project') {
      git branch: 'main', url: 'https://github.com/bsieraduml/week8.git'
      container('gradle') {
        stage("Smoke Test") {
              try {
                sh '''
                chmod +x gradlew
                ./gradlew smokeTest
                  '''
              } catch (Exception E) {
                  echo 'Failure detected'
              }     

        } // stage smoke test 
      } // gradle
    } // Build a gradle project 

  stage('Deploying to prod') {
    container('cloud-sdk') {
      stage('Deploy to prod from cloud-sdk container') {
        sh '''
        echo 'namespaces in the staging environment'
        kubectl get ns
        gcloud auth login --cred-file=$GOOGLE_APPLICATION_CREDENTIALS
        gcloud projects list
        # gcloud config set project week9-lab2
        # gcloud container clusters get-credentials hello-cluster --region us-west1 --project week9-lab2
        # echo 'namespaces in the prod environment'
        # kubectl get ns
'''
        }
      }
    }

  }// k8s       

  } // NODE pod label
} //root