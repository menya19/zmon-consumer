@Library('retort-lib') _
def label = "jenkins-${UUID.randomUUID().toString()}"

podTemplate(label:label,
    containers: [
        containerTemplate(name: 'helm', image: 'lachlanevenson/k8s-helm', ttyEnabled: true, command: 'cat')
    ],
    volumes: [
        configMapVolume(mountPath: '/var/config/menya-test', configMapName: 'zmon-kube-config')
    ]) {

    node(label) {
        stage('SOURCE CHECKOUT') {
            def repo = checkout scm
        }

        withCredentials([usernamePassword(credentialsId: 'influx_admin', usernameVariable: 'ADMIN', passwordVariable: 'ADMINPASS')]) {
          stage('IFX_1 CONSUMER') {
              container('helm') {
                withEnv(['KUBECONFIG=/var/config/menya-test/kube-config-zmon.yml']) {
                  withFolderProperties{
                    yaml.update file: 'values.yaml', update: ['.config.inputs[0].kafka_consumer.brokers[0]':"${env.KAFKA_BROKER_1}", '.config.inputs[0].kafka_consumer.brokers[1]':"${env.KAFKA_BROKER_2}", '.config.inputs[0].kafka_consumer.brokers[2]':"${env.KAFKA_BROKER_3}", '.config.inputs[0].kafka_consumer.topics[0]':"telegraf__${TENANT}" , '.config.inputs[0].kafka_consumer.consumer_group':"consumer_${TENANT}_ifx1" , '.config.outputs[0].influxdb.urls[0]': "${IFX_1}",  '.config.outputs[0].influxdb.database': "zmon_${TENANT}", '.config.outputs[0].influxdb.username': "${ADMIN}", '.config.outputs[0].influxdb.password': "${ADMINPASS}"]
                  }
                  sh "cat values.yaml"
                  sh 'helm install --namespace ${TENANT} --name consumer-${TENANT}-ifx1 -f values.yaml .'
                }
              }
          }

          stage('IFX_2 CONSUMER') {
              container('helm') {
                withEnv(['KUBECONFIG=/var/config/menya-test/kube-config-zmon.yml']) {
                  withFolderProperties{
                    yaml.update file: 'values.yaml', update: ['.config.inputs[0].kafka_consumer.brokers[0]':"${env.KAFKA_BROKER_1}", '.config.inputs[0].kafka_consumer.brokers[1]':"${env.KAFKA_BROKER_2}", '.config.inputs[0].kafka_consumer.brokers[2]':"${env.KAFKA_BROKER_3}", '.config.inputs[0].kafka_consumer.topics[0]':"telegraf__${TENANT}" , '.config.inputs[0].kafka_consumer.consumer_group':"consumer_${TENANT}_ifx2" , '.config.outputs[0].influxdb.urls[0]': "${IFX_2}",  '.config.outputs[0].influxdb.database': "zmon_${TENANT}", '.config.outputs[0].influxdb.username': "${ADMIN}", '.config.outputs[0].influxdb.password': "${ADMINPASS}"]
                  }
                  sh "cat values.yaml"
                  sh 'helm install --namespace ${TENANT} --name consumer-${TENANT}-ifx2 -f values.yaml .'
                }
              }
          }
        }
    }
}
