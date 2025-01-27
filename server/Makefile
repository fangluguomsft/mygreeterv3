
REGISTRY_NAME=servicehubregistry
IMG_NAME=service_hub_environment
IMG_DATE_TAG=20240329
MAIN_DIR := $(shell dirname -- $(shell dirname -- $(shell pwd)))
DIR_NAME=mygreeterv3
#The config file used to populate resourceName and subscriptionId, default assumes there is a config file
#in the main directory of project.
ENVCONFIG?="env-config.yaml"

all: tidy test build

template-files:
	@if [ -f ../../$(ENVCONFIG) ]; then \
		docker run --mount src=$(MAIN_DIR),target=/app/binded-data,type=bind $(REGISTRY_NAME).azurecr.io/$(IMG_NAME):$(IMG_DATE_TAG) \
		./maintain_service executeTemplatesInFolder --templatePrefix "template-" --templatePath binded-data/$(DIR_NAME) --envConfig binded-data/$(ENVCONFIG) --leftDelim "{{" --rightDelim "}}"; \
		echo "Templating server files was successful"; \
	else \
		echo "../../$(ENVCONFIG) does not exist. Please make sure either the path exists or rerun the command with the flag --ENVCONFIG 'pathtoconfigfile'"; \
		exit 1; \
	fi

# (Prerequisite) To deploy the shared resources needed, go to the directory of the shared resources and run the Makefile.
# Creates all the service specific resources and generates markdown files
# TODO: Investigate how to express dependency across different directories rather than write imperative code.
deploy-resources: template-files
	set -e; \
	cd ../api/v1; make service; cd ../../; \
	cd server/resource_provisioning; make all; \
	docker run --mount src=$(MAIN_DIR),target=/app/binded-data,type=bind $(REGISTRY_NAME).azurecr.io/$(IMG_NAME):$(IMG_DATE_TAG) ./maintain_service generateResourceFiles --svcDirName=binded-data/$(DIR_NAME); \

tidy: template-files
	go mod tidy

test:
	go test ./...

build: template-files
	go build -o bin/ dev.azure.com/service-hub-flg/service_hub/_git/service_hub.git/testing/canonical-output/mygreeterv3/server/cmd/client
	go build -o bin/ dev.azure.com/service-hub-flg/service_hub/_git/service_hub.git/testing/canonical-output/mygreeterv3/server/cmd/demoserver
	go build -o bin/ dev.azure.com/service-hub-flg/service_hub/_git/service_hub.git/testing/canonical-output/mygreeterv3/server/cmd/server
	
build-image: template-files
	cd generated; make build-image

push-image: template-files
	cd generated; make push-image


AZURESDK_PATH := artifacts/.azuresdk_properties_outputs.json	

# The install and uninstall information (k8s secret) is stored in the current namespace.
install: template-files
	cd generated; make connect-cluster
	if [ -f $(AZURESDK_PATH) ]; then \
		docker run --mount src=$(MAIN_DIR),target=/app/binded-data,type=bind $(REGISTRY_NAME).azurecr.io/$(IMG_NAME):$(IMG_DATE_TAG) /bin/bash -c \
		"cd binded-data/$(DIR_NAME)/server; export KUBECONFIG=\$$(pwd)/.kube/config; \
		cd deployments; helm install servicehub-mygreeterv3-client . -f values.yaml -f values-client.yaml -f ../$(AZURESDK_PATH); \
		helm install servicehub-mygreeterv3-server . -f values.yaml -f ../$(AZURESDK_PATH); \
		helm install servicehub-mygreeterv3-demoserver . -f values.yaml -f values-demoserver.yaml -f ../$(AZURESDK_PATH);"; \
	else \
		echo ".azuresdk_properties_outputs.json file does not exist. Run the make deploy-resources cmd"; \
		docker run --mount src=$(MAIN_DIR),target=/app/binded-data,type=bind $(REGISTRY_NAME).azurecr.io/$(IMG_NAME):$(IMG_DATE_TAG) /bin/bash -c \
		"cd binded-data/$(DIR_NAME)/server; export KUBECONFIG=\$$(pwd)/.kube/config; \
		cd deployments; helm install servicehub-mygreeterv3-client . -f values.yaml -f values-client.yaml; \
		helm install servicehub-mygreeterv3-server . -f values.yaml; \
		helm install servicehub-mygreeterv3-demoserver . -f values.yaml -f values-demoserver.yaml;"; \
	fi

upgrade: template-files
	if [ -f $(AZURESDK_PATH) ]; then \
		docker run --mount src=$(MAIN_DIR),target=/app/binded-data,type=bind $(REGISTRY_NAME).azurecr.io/$(IMG_NAME):$(IMG_DATE_TAG) /bin/bash -c \
		"cd binded-data/$(DIR_NAME)/server; export KUBECONFIG=\$$(pwd)/.kube/config; \
		cd deployments; helm upgrade servicehub-mygreeterv3-demoserver . -f values.yaml -f values-demoserver.yaml -f ../$(AZURESDK_PATH); \
		helm upgrade servicehub-mygreeterv3-server . -f values.yaml -f ../$(AZURESDK_PATH); \
		helm upgrade servicehub-mygreeterv3-client . -f values.yaml -f values-client.yaml -f ../$(AZURESDK_PATH);"; \	
	else \
		echo ".azuresdk_properties_outputs.json file does not exist. Run the make deploy-resources cmd"; \
		docker run --mount src=$(MAIN_DIR),target=/app/binded-data,type=bind $(REGISTRY_NAME).azurecr.io/$(IMG_NAME):$(IMG_DATE_TAG) /bin/bash -c \
		"cd binded-data/$(DIR_NAME)/server; export KUBECONFIG=\$$(pwd)/.kube/config; \
		cd deployments; helm upgrade servicehub-mygreeterv3-demoserver . -f values.yaml -f values-demoserver.yaml; \
		helm upgrade servicehub-mygreeterv3-server . -f values.yaml; \
		helm upgrade servicehub-mygreeterv3-client . -f values.yaml -f values-client.yaml;"; \
	fi

uninstall: template-files
	docker run --mount src=$(MAIN_DIR),target=/app/binded-data,type=bind $(REGISTRY_NAME).azurecr.io/$(IMG_NAME):$(IMG_DATE_TAG) /bin/bash -c \
	"cd binded-data/$(DIR_NAME)/server; export KUBECONFIG=\$$(pwd)/.kube/config; \
	helm uninstall servicehub-mygreeterv3-client; \
	helm uninstall servicehub-mygreeterv3-demoserver; \
	helm uninstall servicehub-mygreeterv3-server;";
