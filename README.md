# Deploy Rancher on the EKS using Helm

Assumption:
You are having running EKS cluster with helm and git install.

Steps:
=======
1. Add the Helm chart repository
2. Create a namespace for Rancher
3. Choose your SSL configuration
   - Install cert-manager (unless you are bringing your own certificates, or TLS will be terminated on a load balancer)
4. Install Rancher with Helm and your chosen certificate option
5. Verify that the Rancher server is successfully deployed


1. Add the Helm chart repository

helm repo add rancher-latest https://releases.rancher.com/server-charts/latest

    # helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
    "rancher-latest" has been added to your repositories

2. Create a namespace for Rancher

kubectl create namespace cattle-system

    [root@ip-172-31-83-184 opt]# kubectl create namespace cattle-system
    namespace/cattle-system created
    
3. Choose your SSL configuration

The Rancher management server is designed to be secure by default and requires SSL/TLS configuration.

There are three recommended options for the source of the certificate used for TLS termination at the Rancher server:

    Rancher-generated TLS certificate: In this case, you will need to install cert-manager into the cluster. Rancher utilizes cert-manager to issue and maintain its certificates. Rancher will generate a CA certificate of its own, and sign a cert using that CA. cert-manager is then responsible for managing that certificate.
    Let’s Encrypt: The Let’s Encrypt option also uses cert-manager. However, in this case, cert-manager is combined with a special Issuer for Let’s Encrypt that performs all actions (including request and validation) necessary for getting a Let’s Encrypt issued cert. This configuration uses HTTP validation (HTTP-01), so the load balancer must have a public DNS record and be accessible from the internet.
    Bring your own certificate: This option allows you to bring your own public- or private-CA signed certificate. Rancher will use that certificate to secure websocket and HTTPS traffic. In this case, you must upload this certificate (and associated key) as PEM-encoded files with the name tls.crt and tls.key. If you are using a private CA, you must also upload that certificate. This is due to the fact that this private CA may not be trusted by your nodes. Rancher will take that CA certificate, and generate a checksum from it, which the various Rancher components will use to validate their connection to Rancher.

Install cert-manager
====================

# If you have installed the CRDs manually instead of with the `--set installCRDs=true` option added to your Helm install command, you should upgrade your CRD resources before upgrading the Helm chart:
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.5.1/cert-manager.crds.yaml

# Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io

# Update your local Helm chart repository cache
helm repo update

# Install the cert-manager Helm chart
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.5.1
 

Once you’ve installed cert-manager, you can verify it is deployed correctly by checking the cert-manager namespace for running pods:
 
# kubectl get pods --namespace cert-manager
NAME                                      READY   STATUS    RESTARTS   AGE
cert-manager-56b686b465-hlt8s             1/1     Running   0          25s
cert-manager-cainjector-75c94654d-rc4g8   1/1     Running   0          25s
cert-manager-webhook-69bd5c9d75-9nvvs     1/1     Running   0          25s

4. Install Rancher with Helm and Your Chosen Certificate Option

helm install rancher rancher-latest/rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org \
  --set replicas=3
  
Wait for Rancher to be rolled out:

kubectl -n cattle-system rollout status deploy/rancher

kubectl -n cattle-system get deploy rancher
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
rancher   3         3         3            3           3m
