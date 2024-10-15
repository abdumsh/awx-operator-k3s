# Variables
K3S_OPTIONS := --write-kubeconfig-mode 644
AWX_HOST := awx.awx-operator.com
POSTGRES_DIR := /data/postgres-15
PROJECTS_DIR := /data/projects
NAMESPACE := awx

# Default target
.PHONY: all
all: update_upgrade install_k3s clone_awx_operator setup_directories generate_cert replace_ingress_host deploy_awx_operator deploy_awx show_logs

# Update and upgrade the server
.PHONY: update_upgrade
update_upgrade:
	sudo apt-get update && sudo apt-get upgrade -y || { echo 'Update failed'; exit 1; }

# Install K3s
.PHONY: install_k3s
install_k3s:
	curl -sfL https://get.k3s.io | sh -s - $(K3S_OPTIONS)

# Clone the AWX Operator repo
.PHONY: clone_awx_operator
clone_awx_operator:
	git clone https://github.com/abdumsh/awx-operator-k3s.git ~/awx-operator-k3s

# Replace Ingress Host in AWX YAML
.PHONY: replace_ingress_host
replace_ingress_host:
	@echo "Replacing AWX host in base/awx.yaml..."
	envsubst < ~/awx-operator-k3s/base/awx.yaml > ~/awx-operator-k3s/base/awx-deployed.yaml

# Deploy AWX Operator
.PHONY: deploy_awx_operator
deploy_awx_operator:
	kubectl apply -k ~/awx-operator-k3s/operator -n $(NAMESPACE)

# Generate SSL certificate for AWX Ingress
.PHONY: generate_cert
generate_cert:
	openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -out ~/awx-operator-k3s/base/tls.crt -keyout ~/awx-operator-k3s/base/tls.key -subj "/CN=$(AWX_HOST)/O=$(AWX_HOST)" -addext "subjectAltName=DNS:$(AWX_HOST)"

# Create Postgres and Projects directories with permissions
.PHONY: setup_directories
setup_directories:
	sudo mkdir -p $(POSTGRES_DIR) $(PROJECTS_DIR) && sudo chown 1000:0 $(PROJECTS_DIR)

# Deploy AWX
.PHONY: deploy_awx
deploy_awx:
	kubectl apply -f ~/awx-operator-k3s/base/awx-deployed.yaml -n $(NAMESPACE)

# Display AWX Operator Logs
.PHONY: show_logs
show_logs:
	kubectl -n $(NAMESPACE) logs -f deployments/awx-operator-controller-manager

# Display AWX Resources
.PHONY: show_resources
show_resources:
	kubectl -n $(NAMESPACE) get awx,all,ingress,secrets

# Help target
.PHONY: help
help:
	@echo "Usage:"
	@echo "  make all               - Run all steps"
	@echo "  make update_upgrade    - Update and upgrade the server"
	@echo "  make install_k3s      - Install K3s"
	@echo "  make clone_awx_operator- Clone AWX Operator repository"
	@echo "  make replace_ingress_host - Replace AWX host in the YAML file"
	@echo "  make deploy_awx_operator - Deploy the AWX Operator"
	@echo "  make generate_cert     - Generate SSL certificate for AWX Ingress"
	@echo "  make setup_directories  - Create Postgres and Projects directories"
	@echo "  make deploy_awx        - Deploy AWX"
	@echo "  make show_logs         - Show AWX Operator logs"
	@echo "  make show_resources    - Display AWX resources"

# Cleanup target
.PHONY: cleanup
cleanup:
	kubectl delete -k ~/awx-operator-k3s/operator -n $(NAMESPACE) || echo 'No resources to clean up'
	kubectl delete -f ~/awx-operator-k3s/base/awx-deployed.yaml -n $(NAMESPACE) || echo 'No resources to clean up'
