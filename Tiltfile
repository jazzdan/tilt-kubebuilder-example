load('ext://restart_process', 'docker_build_with_restart')

IMG = 'controller:latest'

### FILL OUT THESE FIELDS
NAME = '' # name of Go module
DOMAIN = '' # domain for CRD
GROUP = '' # group for CRD
VERSION = '' # version for CRD
KIND = '' # kind for CRD
### END FIELDS THAT YOU NEED TO FILL OUT

DOCKERFILE = '''FROM golang:alpine
WORKDIR /
COPY ./bin/manager /
CMD ["/manager"]
'''

def ensure_set(name, var):
    if var == '':
        fail("%s is not set, take a look at lines 3-7 and set these variables" % name)

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

ensure_set("NAME", NAME)
ensure_set("DOMAIN", DOMAIN)
ensure_set("GROUP", GROUP)
ensure_set("VERSION", VERSION)
ensure_set("KIND", KIND)

initting = False
if os.path.exists('go.mod') == False:
    initting = True
    local("go mod init %s" % NAME)

if os.path.exists('PROJECT') == False:
    initting = True
    local("kubebuilder init --domain %s" % DOMAIN)

if os.path.exists('api') == False:
    initting = True
    local("kubebuilder create api --resource --controller --group %s --version %s --kind %s" % (GROUP, VERSION, KIND))

local(manifests() + generate())

local_resource('crd', manifests() + 'kustomize build config/crd | kubectl apply -f -', deps=["api"])

#local_resource('un-crd', 'kustomize build config/crd | kubectl delete -f -', auto_init=False, trigger_mode=TRIGGER_MODE_MANUAL)

k8s_yaml(yaml())

deps = ['controllers', 'main.go']
if initting == False:
    deps.append('api')
local_resource('recompile', generate() + binary(), deps=deps)

docker_build_with_restart(IMG, '.', 
 dockerfile_contents=DOCKERFILE,
 entrypoint='/manager',
 only=['./bin/manager'],
 live_update=[
       sync('./bin/manager', '/manager'),
   ]
)
