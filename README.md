# OpenShift GitOps Automation v1
Enterprise-grade GitOps pipeline using Ansible, ArgoCD, and OpenShift.

## Stack
- Platform: OpenShift CRC
- GitOps: ArgoCD
- CI: GitHub Actions
- Infrastructure: Ansible

## Setup
ansible-playbook -i ansible/inventory.ini ansible/site.yml