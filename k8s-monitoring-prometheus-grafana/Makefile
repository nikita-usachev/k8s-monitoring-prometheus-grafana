NAMESPACE = monitoring

CONTEXT = aws eks update-kubeconfig --region us-east-1 --name k8s-cluster

MANIFESTS_DIR = /devops-tools-22c-redhat/kubernetes/k8s-monitoring/k8s

deploy:
	kubectl config use-context $(CONTEXT)
	kubectl apply -f $(MANIFESTS_DIR)/clusterRole.yaml -n $(NAMESPACE)
	kubectl apply -f $(MANIFESTS_DIR)/configmap.yaml -n $(NAMESPACE)
	kubectl apply -f $(MANIFESTS_DIR)/prometheus-deployment.yaml -n $(NAMESPACE)
	kubectl apply -f $(MANIFESTS_DIR)/prometheus-service.yaml -n $(NAMESPACE)
	kubectl apply -f $(MANIFESTS_DIR)./grafana-deployment.yaml -n $(NAMESPACE)
	kubectl apply -f $(MANIFESTS_DIR)./grafana-datasource-config.yaml -n $(NAMESPACE)
	kubectl apply -f $(MANIFESTS_DIR)./grafana-service.yaml -n $(NAMESPACE)
	kubectl apply -f $(MANIFESTS_DIR)./grafana-ingress.yaml -n $(NAMESPACE)


delete:
	kubectl config use-context $(CONTEXT)
	kubectl delete -f $(MANIFESTS_DIR)/clusterRole.yaml -n $(NAMESPACE)
	kubectl delete -f $(MANIFESTS_DIR)/configmap.yaml -n $(NAMESPACE)
	kubectl delete -f $(MANIFESTS_DIR)/prometheus-deployment.yaml -n $(NAMESPACE)
	kubectl delete -f $(MANIFESTS_DIR)/prometheus-service.yaml -n $(NAMESPACE)
	kubectl delete -f $(MANIFESTS_DIR)./grafana-deployment.yaml -n $(NAMESPACE)
	kubectl delete -f $(MANIFESTS_DIR)./grafana-datasource-config.yaml -n $(NAMESPACE)
	kubectl delete -f $(MANIFESTS_DIR)./grafana-service.yaml -n $(NAMESPACE)
	kubectl delete -f $(MANIFESTS_DIR)./grafana-ingress.yaml -n $(NAMESPACE)

delete-ns:
	kubectl delete all -n $(NAMESPACE)
	