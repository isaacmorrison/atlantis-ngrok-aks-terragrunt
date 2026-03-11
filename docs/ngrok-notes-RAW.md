export NGROK_API_KEY=ngrok_api_key_placeholder
export NGROK_AUTHTOKEN=ngrok_authtoken_placeholder
helm install ngrok-ingress-controller ngrok/kubernetes-ingress-controller \
  --namespace ngrok-ingress-controller \
  --create-namespace \
  --set credentials.apiKey=$NGROK_API_KEY \
  --set credentials.authtoken=$NGROK_AUTHTOKEN

  export NGROK_API_KEY=ngrok_api_key_placeholder
  export AUTHTOKEN=ngrok_authtoken_placeholder
  helm install ngrok-operator ngrok/ngrok-operator \
    --namespace ngrok-operator \
    --set credentials.apiKey=$NGROK_API_KEY \
    --set credentials.authtoken=$NGROK_AUTHTOKEN \
  --set useExperimentalGatewayApi=true


  ngrok config add-authtoken ngrok_authtoken_placeholder

  # API key/token placeholders above must be replaced with your own ngrok credentials

  kubectl get crds | grep ngrok.com | awk '{print $1}' | xargs kubectl delete crd

  kubectl get clusterrole | grep ngrok-ingress | awk '{print $1}' | xargs kubectl delete clusterrole

  kubectl edit crd command and need to update finalize block "spec": { "finalizers": [] } 


  kubectl apply -n ngrok-operator -f https://raw.githubusercontent.com/ngrok/ngrok-operator/main/manifest-bundle.yaml -n ngrok-operator

  kubectl patch namespace atlantis --patch '{"metadata": {"finalizers": null}}'
  kubectl delete namespace atlantis  --grace-period=0 --force --wait=false

  kubectl patch ingress atlantis-ingress --patch '{"metadata": {"finalizers": null}}'
  kubectl edit crd domains.ingress.k8s.ngrok.com