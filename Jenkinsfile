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
    git branch: env.BRANCH_NAME, url: 'https://github.com/bsieraduml/week9.git'
    container('centos') {
      stage('rolling update calculator') {
        sh '''
        curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
        chmod +x ./kubectl
        
        echo 'week 9 exercise 1 part 2 a - test sum and div with first image'
        ./kubectl apply -f calculator.yaml -n staging
        ./kubectl apply -f hazelcast.yaml -n staging
        echo 'after deployment sleep 30 seconds'
        test $(curl calculator-service.staging.svc.cluster.local:8080/sum?a=10\\&b=2) -eq 12 && echo 'sum passes' || echo 'sum fails'
        test $(curl calculator-service.staging.svc.cluster.local:8080/div?a=10\\&b=2) -eq 5 && echo 'div passes' || echo 'div fails' 
                
        echo 'week 9 exercise 1 part 2 b - test sum and div with second image'
        ./kubectl apply -f calculator2.yaml -n staging
        echo 'after deployment sleep 30 seconds'        
        test $(curl calculator-service.staging.svc.cluster.local:8080/sum?a=10\\&b=2) -eq 12 && echo 'sum passes' || echo 'sum fails'
        test $(curl calculator-service.staging.svc.cluster.local:8080/div?a=10\\&b=2) -eq 5 && echo 'div passes' || echo 'div fails'        
          '''
        }
      }
    }       

  } // NODE pod label
} //root