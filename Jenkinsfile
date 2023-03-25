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

  boolean testPassed = true
  stage('k8s') {
    stage('Build a gradle project') {
      git branch: 'main', url: 'https://github.com/bsieraduml/week8.git'
      container('gradle') {
        stage("Smoke Test") {
              try {
                sh '''
                chmod +x gradlew
                ./gradlew smokeTest -Dcalculator.url=http://calculator-service.staging.svc.cluster.local:8080
                  '''
              } catch (Exception E) {
                  echo 'Failure detected'
                  testPassed = false
              }     

        } // stage smoke test 
      } // gradle
    } // Build a gradle project 

  stage('Deploying to prod') {
    if (testPassed)
    {
      container('cloud-sdk') {
        git branch: 'main', url: 'https://github.com/bsieraduml/week9.git'
        stage('Deploy to prod from cloud-sdk container') {
          sh '''
          echo 'test passed pushing to production'

          echo 'pwd'
          pwd
          echo 'ls -lart'
          ls -lart
          
          echo 'namespaces in staging'
          kubectl get ns
          gcloud auth login --cred-file=$GOOGLE_APPLICATION_CREDENTIALS
          gcloud container clusters get-credentials hello-cluster --region us-west1 --project umls23-381500
          
          echo 'namespaces in prod'
          kubectl get ns
          # kubectl apply -f hazelcast.yaml
          # kubectl apply -f calculator2.yaml
          '''
        }
      }
    } else {
      echo 'test failed not deploying to production'
    }
    } // deploying to prod

  }// k8s       

  } // NODE pod label
} //root