# Install Microk8s

Microk8s is deployed via Snaps. Snaps are containerised software packages that are simple to create and install. They auto-update and are safe to run. Moreover, because they bundle their dependencies, they work on all major Linux systems without modification.

The microk8s snap is frequently updated to match each release of Kubernetes. It can be installed using the command below:


```bash
snap install microk8s --classic --beta
```


To avoid colliding with a kubectl already installed and to avoid overwriting any existing Kubernetes configuration file, microk8s adds a microk8s.kubectl command.

To view the newly deployed node, run 

```bash
microk8s.kubectl get node
```

If you receive an error it means that microk8s is still starting in the background. Wait a couple of moments and try again.

As with kubectl, it's possible to use the existing commands, such as describing the details of the node.

```bash
microk8s.kubectl describe node host01
```

If you only are using microk8s, consider adding an alias using the following command:

```bash
snap alias microk8s.kubectl kubectl
```