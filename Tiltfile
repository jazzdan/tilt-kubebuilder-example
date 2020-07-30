load('ext://restart_process', 'docker_build_with_restart')

IMG = 'controller:latest'
#docker_build(IMG, '.')
# TODO(dmiller): domain, group, version, kind variables
# error if they are not filled in
# TODO(dmiller): embed dockerfile

# Three commands to run for this to work
# TODO(dmiller): check if `kubebuilder init` has been run
# 1. kubebuilder init (I did this)
#   run manifests
#   run generate
# TODO(run these commands)
# 2. kubebuilder create api # to create resource
#   THEN kustomize build | kubectl apply
# 3. kubebuilder create controller # to create controller
#   THEN we can recompile controller

def yaml():
    return local('cd config/manager; kustomize edit set image controller=' + IMG + '; cd ../..; kustomize build config/default')

def manifests():
    return 'controller-gen crd:trivialVersions=true rbac:roleName=manager-role webhook paths="./..." output:crd:artifacts:config=config/crd/bases;'

def generate():
    return 'controller-gen object:headerFile="hack/boilerplate.go.txt" paths="./...";'

def vetfmt():
    return 'go vet ./...; go fmt ./...'

def binary():
    return 'CGO_ENABLED=0 GOOS=linux GOARCH=amd64 GO111MODULE=on go build -o bin/manager main.go'

local(manifests() + generate())

local_resource('crd', manifests() + 'kustomize build config/crd | kubectl apply -f -', deps=["api"])

#local_resource('un-crd', 'kustomize build config/crd | kubectl delete -f -', auto_init=False, trigger_mode=TRIGGER_MODE_MANUAL)

k8s_yaml(yaml())

# TODO(dmiller): also monitor for go files inside the api directory
local_resource('recompile', generate() + binary(), deps=['controllers', 'main.go'])

docker_build_with_restart(IMG, '.', 
 dockerfile='tilt.docker',
 entrypoint='/manager',
 only=['./bin/manager'],
 live_update=[
       sync('./bin/manager', '/manager'),
   ]
)
