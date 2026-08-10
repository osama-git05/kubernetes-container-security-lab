# Kubernetes Container Security and Runtime Detection Lab

A hands-on Kubernetes security project using Minikube, Nginx, Falco, and container hardening techniques.

## Status

Project in progress.

## Day 1 - Local Kubernetes Environment

### Goal

Set up the local Kubernetes lab environment and confirm that the Minikube cluster is working correctly.

### Environment

- Ubuntu 24.04 LTS Virtual Machine
- VirtualBox
- Docker
- kubectl
- Minikube
- Helm
- Git

### Implementation

The following tools were installed and configured:

- Git for project and repository management
- Docker as the local container platform
- kubectl for interacting with Kubernetes
- Minikube for creating a local Kubernetes cluster
- Helm for installing Kubernetes applications later in the project

The Minikube cluster was started using the Docker driver and containerd runtime.

```bash
minikube start --driver=docker --container-runtime=containerd --cpus=4 --memory=6144
