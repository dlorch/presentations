# Self-Documented Makefile https://marmelab.com/blog/2016/02/29/auto-documented-makefile.html

.DEFAULT_GOAL := help

install-tools: ## install dependencies
	brew install helm@2

helm-init: ## intialize helm on cluster (installs cluster-sided component tiller)
	helm init
	# tiller, see https://github.com/helm/helm/issues/5100#issuecomment-533787541
	kubectl create serviceaccount -n kube-system tiller
	kubectl create clusterrolebinding tiller-cluster-admin --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
	kubectl --namespace kube-system patch deploy tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
	# install
	helm install stable/prometheus-operator --name prometheus-operator --namespace monitoring

prometheus-port-forward: ## forwards prometheus to http://localhost:9090
	kubectl port-forward -n monitoring prometheus-prometheus-operator-prometheus-0 9090

grafana-port-forward: ## forward grafana to http://localhost:3000 (admin/prom-operator)
	kubectl port-forward $$(kubectl get pods --selector=app.kubernetes.io/name=grafana -n monitoring --output=jsonpath="{.items..metadata.name}") -n monitoring 3000

alertmanager-port-forward: ## forward alertmanager to http://localhost:9093
	kubectl port-forward -n monitoring alertmanager-prometheus-operator-alertmanager-0 9093

artifactory-install: ## install artifactory OSS
	helm repo add jfrog https://charts.jfrog.io
	helm install jfrog/artifactory --version 9.0.23 --namespace artifactory

artifactory-service: ## artifactory service IP
	export SERVICE_IP=$$(kubectl get svc --namespace artifactory nomadic-seal-artifactory-nginx -o jsonpath='{.status.loadBalancer.ingress[0].ip}') && echo http://$$SERVICE_IP/

argo-cd-install: ## install Argo CD
	helm repo add argo https://argoproj.github.io/argo-helm
	helm install argo/argo-cd --version 1.8.7 --namespace argo-cd

argo-cd-port-forward: ## forward argo-cd to http://localhost:8080 (admin / <name of server pod>)
	kubectl port-forward service/eyewitness-mite-argocd-server -n argo-cd 8080:443

argo-cd-serverpod: ## get name of argo-cd server pod
	kubectl get pods -n argo-cd -l app.kubernetes.io/name=argocd-server -o name | cut -d'/' -f 2

help:
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'
