podTemplate(yaml: '''
    apiVersion: v1
    kind: Pod
    spec:
      containers:
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
      - name: kaniko-secret
        secret:
            secretName: dockercred
            items:
            - key: .dockerconfigjson
              path: config.json
''') {
  node(POD_LABEL) {

  stage('k8s') {
    git branch: 'main', url: 'https://github.com/bsieraduml/week9.git'
    container('centos') {
      stage('rolling update calculator') {
        sh '''
        curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
        chmod +x ./kubectl
        ./kubectl apply -f calculator.yaml -n staging
        ./kubectl apply -f hazelcast.yaml -n staging
          '''
        }
      }
    }

stage('Build a gradle project') {
      git branch: 'main', url: 'https://github.com/bsieraduml/week8.git'
      container('gradle') {

        stage("Acceptance Testing") {
            //exercise 8 part 1 test via curl

              try {
                sh '''
                chmod +x gradlew
                ./gradlew smokeTest
                  '''
              } catch (Exception E) {
                  echo 'Failure detected'
              }     

        } // stage code coverage  
      }
    }        

  } // NODE pod label
} //root