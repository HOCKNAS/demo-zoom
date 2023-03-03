

INSTALAR ROBUSTA:

    python3 --version # Python 3.10.8

    pip3 --version # pip 22.2.2

    pip3 install -U robusta-cli --no-cache

INSTALAR HELM:

    curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3

    chmod 700 get_helm.sh

    ./get_helm.sh

    o también

    brew install helm


CONFIGURAR ROBUSTA:

`robusta gen-config`

    Seleccionar la opción de Slack y luego ir a la url generada: https://api.robusta.dev/integrations/slack?id=537a7ae5-f6d1-4585-b0d4-2a1981659

    Click en el botón "Add to Slack"

    Seleccionamos el workspace y luego en Allow

    En la CLI ponemos el nombre del canal donde se enviarán las notificaciones: #demo-eks-cluster

    Aceptamos la opción para instalar Prometheus pre-configurado y configurar la Robusta UI

Al final debemos obtener un archivo `generated_values.yaml`, como este:

    globalConfig:
      signing_key: bbc7d8f7-4f21-4508-879b
      account_id: 09aecda1-d0f7-4269-9c97
    sinksConfig:
    - slack_sink:
        name: main_slack_sink
        slack_channel: demo-eks-cluster
        api_key: xoxb-4875974198807-4884251217414
    - robusta_sink:
        name: robusta_ui_sink
        token: <token>
    enablePrometheusStack: true
    runner:
      sendAdditionalTelemetry: false
    rsa:
      private: <some_value>
      public: <some_value>

Para poder integrar ChatGPT, debemos adicionar los siguientes valores:


    enablePlatformPlaybooks: true
    playbookRepos:
      chatgpt_robusta_actions:
        url: "https://github.com/robusta-dev/kubernetes-chatgpt-bot.git"
    customPlaybooks:
    - triggers:
      - on_prometheus_alert: {}
      actions:
      - chat_gpt_enricher: {}

De manera que el resultado final debe ser similar a esto:

    globalConfig:
      chat_gpt_token: <token_chatgpt>
      signing_key: bbc7d8f7-4f21-4508-879b
      account_id: 09aecda1-d0f7-4269-9c97
    sinksConfig:
    - slack_sink:
        name: main_slack_sink
        slack_channel: demo-eks-cluster
        api_key: xoxb-4875974198807-4884251217414
    - robusta_sink:
        name: robusta_ui_sink
        token: <token>
    enablePrometheusStack: true
    enablePlatformPlaybooks: true
    playbookRepos:
      chatgpt_robusta_actions:
        url: "https://github.com/robusta-dev/kubernetes-chatgpt-bot.git"
    customPlaybooks:
    - triggers:
      - on_prometheus_alert: {}
      actions:
      - chat_gpt_enricher: {}
    runner:
      sendAdditionalTelemetry: false
    rsa:
      private: <some_value>
      public: <some_value>


Luego ejecutar:

    helm repo add robusta https://robusta-charts.storage.googleapis.com && helm repo update

    helm install robusta robusta/robusta -f ./generated_values.yaml --set clusterName=demo-eks-cluster -n robusta --create-namespace

Y para desinstalar:

    helm uninstall robusta -n robusta
    
    kubectl delete namespace robusta
    

PROBAR LA INTEGRACION CON CHATGPT:

 
Luego de acceder al pod `robusta-runner` podemos ejecutar alguna de las siguientes peticiones para simular una alerta:


CrashLoopBackOff: Esta alerta se dispara cuando un contenedor en un pod se ha bloqueado o ha fallado repetidamente, lo que ha provocado que el pod se reinicie continuamente. 


    curl -X POST http://localhost:5000/api/trigger -H 'Content-Type: application/json' -d '{"action_name": "prometheus_alert", "action_params": {"name": "podinfo", "namespace": "default", "kind": "pod", "alert_name": "CrashLoopBackOff"}}'
    
    
KubePodCrashLooping: Esta alerta se dispara cuando un pod se reinicia constantemente debido a un error de aplicación o de configuración.

    curl -X POST http://localhost:5000/api/trigger -H 'Content-Type: application/json' -d '{"action_name": "prometheus_alert", "action_params": {"name": "podinfo", "namespace": "default", "kind": "pod", "alert_name": "KubePodCrashLooping"}}'

 
KubePodNotReady: Esta alerta se dispara cuando un pod no está disponible para el servicio debido a un fallo en la aplicación o en la configuración.

    curl -X POST http://localhost:5000/api/trigger -H 'Content-Type: application/json' -d '{"action_name": "prometheus_alert", "action_params": {"name": "podinfo", "namespace": "default", "kind": "pod", "alert_name": "KubePodNotReady"}}'

KubeNodeNotReady: Esta alerta se dispara cuando un nodo no está disponible debido a un fallo en el hardware, en el sistema operativo o en la red.

    curl -X POST http://localhost:5000/api/trigger -H 'Content-Type: application/json' -d '{"action_name": "prometheus_alert", "action_params": {"name": "podinfo", "namespace": "default", "kind": "pod", "alert_name": "KubeNodeNotReady"}}'

KubeDeploymentNotAvailable: Esta alerta se dispara cuando un despliegue no está disponible debido a un fallo en la aplicación o en la configuración.

    curl -X POST http://localhost:5000/api/trigger -H 'Content-Type: application/json' -d '{"action_name": "prometheus_alert", "action_params": {"name": "podinfo", "namespace": "default", "kind": "pod", "alert_name": "KubeDeploymentNotAvailable"}}'

KubeServiceNotAvailable: Esta alerta se dispara cuando un servicio no está disponible debido a un fallo en la aplicación o en la configuración.

    curl -X POST http://localhost:5000/api/trigger -H 'Content-Type: application/json' -d '{"action_name": "prometheus_alert", "action_params": {"name": "podinfo", "namespace": "default", "kind": "pod", "alert_name": "KubeServiceNotAvailable"}}'

KubeletTooManyPods: Esta alerta se dispara cuando un nodo está sobrecargado con demasiados pods y no puede manejar más debido a limitaciones de recursos.
     
     curl -X POST http://localhost:5000/api/trigger -H 'Content-Type: application/json' -d '{"action_name": "prometheus_alert", "action_params": {"name": "podinfo", "namespace": "default", "kind": "pod", "alert_name": "KubeletTooManyPods"}}'

KubeContainerCreating: Esta alerta se dispara cuando se intenta crear un contenedor, pero algo ha fallado y el contenedor no se ha creado correctamente.

    curl -X POST http://localhost:5000/api/trigger -H 'Content-Type: application/json' -d '{"action_name": "prometheus_alert", "action_params": {"name": "podinfo", "namespace": "default", "kind": "pod", "alert_name": "KubeContainerCreating"}}'

KubeContainerTerminating: Esta alerta se dispara cuando un contenedor se está terminando, pero algo ha fallado y el contenedor no se ha detenido correctamente.

    curl -X POST http://localhost:5000/api/trigger -H 'Content-Type: application/json' -d '{"action_name": "prometheus_alert", "action_params": {"name": "podinfo", "namespace": "default", "kind": "pod", "alert_name": "KubeContainerTerminating"}}'

KubePodScheduled: Esta alerta se dispara cuando un pod está programado para ejecutarse en un nodo, pero no se ha podido programar debido a limitaciones de recursos o fallas en la configuración del clúster.

    curl -X POST http://localhost:5000/api/trigger -H 'Content-Type: application/json' -d '{"action_name": "prometheus_alert", "action_params": {"name": "podinfo", "namespace": "default", "kind": "pod", "alert_name": "KubePodScheduled"}}'

KubeletOutOfDisk: Esta alerta se dispara cuando un nodo se está quedando sin espacio en disco y no puede programar más pods.

    curl -X POST http://localhost:5000/api/trigger -H 'Content-Type: application/json' -d '{"action_name": "prometheus_alert", "action_params": {"name": "podinfo", "namespace": "default", "kind": "pod", "alert_name": "KubeletOutOfDisk"}}'

KubeletHighMemoryUsage: Esta alerta se dispara cuando un nodo está utilizando demasiada memoria y no puede manejar más pods debido a limitaciones de recursos.
    
    curl -X POST http://localhost:5000/api/trigger -H 'Content-Type: application/json' -d '{"action_name": "prometheus_alert", "action_params": {"name": "podinfo", "namespace": "default", "kind": "pod", "alert_name": "KubeletHighMemoryUsage"}}'
