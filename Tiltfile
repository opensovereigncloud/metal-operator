update_settings(k8s_upsert_timeout_secs=60)  # on first tilt up, often can take longer than 30 seconds

settings = {
    "allowed_contexts": [
        "kind-metal"
    ],
    "kubectl": "./bin/kubectl",
    "cert_manager_version": "v1.15.3",
}

if "allowed_contexts" in settings:
    allow_k8s_contexts(settings.get("allowed_contexts"))

def deploy_cert_manager():
    kubectl = settings.get("kubectl")
    version = settings.get("cert_manager_version")
    print("Installing cert-manager")
    local("{} apply -f https://github.com/cert-manager/cert-manager/releases/download/{}/cert-manager.yaml".format(kubectl, version), quiet=True, echo_off=True)

    print("Waiting for cert-manager to start")
    local("{} wait --for=condition=Available --timeout=300s -n cert-manager deployment/cert-manager".format(kubectl), quiet=True, echo_off=True)
    local("{} wait --for=condition=Available --timeout=300s -n cert-manager deployment/cert-manager-cainjector".format(kubectl), quiet=True, echo_off=True)
    local("{} wait --for=condition=Available --timeout=300s -n cert-manager deployment/cert-manager-webhook".format(kubectl), quiet=True, echo_off=True)

def waitforsystem():
    kubectl = settings.get("kubectl")
    print("Waiting for metal-operator to start")
    local("{} wait --for=condition=ready --timeout=300s -n metal-operator-system pod --all".format(kubectl), quiet=False, echo_off=True)

##############################
# Actual work happens here
##############################

deploy_cert_manager()

docker_build('controller', '.', target = 'manager')

yaml = kustomize('./config/dev')

k8s_yaml(yaml)

k8s_yaml('./config/samples/metal_v1alpha1_endpoint.yaml')
k8s_resource(
    objects=['endpoint-sample:endpoint'],
    new_name='test-endpoint',
    trigger_mode=TRIGGER_MODE_AUTO,
    auto_init=False
)
